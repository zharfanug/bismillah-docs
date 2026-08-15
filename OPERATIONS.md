# Operations
## Wedding Announcement Website

**Status:** Live (Cloudflare Tunnel → `web` on :3000, prod build). `web-dev` (:3001, hot reload) runs permanently alongside for local iteration — never tunneled.
**Last updated:** 2026-08-15 (§6 rewritten, header status line updated — frontend split into two permanent containers, `web`/prod/:3000 and `web-dev`/dev/:3001, replacing the old single-container `APP_ENV` mode toggle; see TECH_STACK.md §5 for the full architecture change. Previous: 2026-08-11, §6: `go-prod.sh`/`run-dev.sh` — self-verifying wrappers around `stack.sh restart` that assert the actual process, not just HTTP status, added after a real incident — both scripts are now retired, see §6 below)

Day-to-day operational tasks — currently database management (guest comments) and deployment verification. Add more sections here as other operational needs come up, rather than starting a new doc per task.

**The site is live and receiving real guest comments right now** (20 as of this writing) — nothing below is a drill. Read a command fully before running it, especially anything in §4/§5 (destructive). When in doubt, back up first (§3).

---

## 1. Where the data lives

- Single table that matters: `wedding_comment` (guest name + message + timestamp) in the `wedding` MariaDB database — see `backend/wedding/models.py`.
- Stored in the `db_data` Docker named volume (`docker-compose.yml`), not bind-mounted to the repo — it survives `stack.sh restart`/container recreation, but **not** `docker compose down -v` or `docker volume rm` (§5.3).
- Credentials live in `.env` (`DB_NAME`/`DB_USER`/`DB_PASSWORD`/`DB_ROOT_PASSWORD`) — not committed to the repo.

**Security note:** `.env`'s current `DB_PASSWORD`/`DB_ROOT_PASSWORD`/`DJANGO_SECRET_KEY` are still the scaffold's placeholder values (`wedding`/`root`/`dev-secret-key-change-me`), even though the site is publicly tunneled. Worth rotating to real random values before this matters (the database port itself isn't published to the host in `docker-compose.yml`, so it's not directly internet-reachable today — but the weak values are still worth fixing since they're the kind of thing that becomes a problem the moment anything changes).

## 2. Everyday task: moderating a guest comment

This is the one you'll actually use day-to-day — deleting a specific inappropriate comment that slipped past the denylist (CONTENT_DESIGN.md §6). **Django Admin, not SQL** — it's already built for this, no need to touch the database directly.

1. Log into `http://localhost:8000/admin/` (or your tunnel's backend URL, if you ever publish it — see PRD.md §3 on why it currently isn't published) with the Django superuser account.
2. **Wedding → Comments** — list view shows name/message/timestamp, searchable by name or message text.
3. Select the row(s), **Delete selected comments** from the action dropdown, confirm.

**`/admin/` returns a bare "Bad Request (400)" instead of the login page:** Django's `ALLOWED_HOSTS` rejected the `Host: localhost:8000` header — happens in `prod` mode (`DEBUG=False` hides the real error behind a generic 400). `.env`'s `DJANGO_ALLOWED_HOSTS` needs `localhost,127.0.0.1` alongside the tunnel domain and `backend` (already the default in this repo's `.env`/`.env.example`). After editing `.env`, recreate just the backend container to pick it up — no need to touch `web`/`db`:
```sh
docker compose up -d --force-recreate backend
```

No superuser yet, or forgot the password:
```sh
docker compose exec backend python manage.py createsuperuser
# or, to reset an existing one's password:
docker compose exec backend python manage.py changepassword <username>
```

Toggling the whole Guest Wishes section off (`content/wedding.ts`, `wishes.enabled = false`) hides the form/list from the site but does **not** delete the underlying data — see CONTENT_DESIGN.md §2.

## 3. Backup

Before anything in §4/§5, or just periodically:

```sh
# Dump the whole database to a timestamped SQL file, from the host.
docker compose exec -T db mariadb-dump -u root -p"$(grep DB_ROOT_PASSWORD .env | cut -d= -f2)" wedding \
  > "backup-$(date +%Y%m%d-%H%M%S).sql"
```

Restore from a dump (overwrites current data — read §4's "know what you're restoring into" note, same idea applies here):

```sh
docker compose exec -T db mariadb -u root -p"$(grep DB_ROOT_PASSWORD .env | cut -d= -f2)" wedding < backup-20260810-120000.sql
```

## 4. Direct database access (advanced — Admin/`manage.py` covers most needs)

Interactive SQL shell:
```sh
docker compose exec db mariadb -u root -p"$(grep DB_ROOT_PASSWORD .env | cut -d= -f2)" wedding
```

From there, e.g.:
```sql
SELECT id, name, LEFT(message, 50), created_at FROM wedding_comment ORDER BY created_at DESC;
DELETE FROM wedding_comment WHERE id = 17;         -- one specific row
DELETE FROM wedding_comment WHERE name = 'Spam Bot'; -- by name, if a bot got through
```

Or, without leaving the shell, via Django's ORM (same effect, arguably safer — validates through Django rather than raw SQL):
```sh
docker compose exec backend python manage.py shell -c "
from wedding.models import Comment
Comment.objects.filter(id=17).delete()
"
```

**Know what you're deleting before you run a `DELETE`/`.filter().delete()`** — `SELECT`/`.filter()` the same condition first and eyeball the result set. There's no undo outside of a backup restore (§3).

## 5. Wiping data (destructive — confirm this is actually what you want)

### 5.1 Delete all comments, keep the table/schema
```sh
docker compose exec backend python manage.py shell -c "from wedding.models import Comment; print(Comment.objects.all().delete())"
```
Prints how many rows were deleted. The table stays, ready for new comments — this is the right call for "clear the guestbook and start over," not "something's broken with the database itself."

### 5.2 Reset just this app's tables (re-run migrations from scratch)
```sh
docker compose exec backend python manage.py flush --noinput
```
Deletes all data from every Django-managed table (currently just `wedding_comment`) and resets auto-increment IDs — **also deletes any Django Admin superusers**, so you'll need to `createsuperuser` again afterward (§2).

### 5.3 Nuke the whole database (start completely fresh)
```sh
./stack.sh down
docker volume rm bismillah_db_data   # confirm the exact name first: docker volume ls | grep db_data
./stack.sh up
```
Deletes the `db_data` volume entirely — MariaDB reinitializes from empty on the next `up`, migrations reapply automatically (`backend/entrypoint.sh`), but **every comment and every admin user is gone, permanently, no undo without a backup** (§3). This is the "the database itself is corrupted/misconfigured and needs to start over" nuclear option, not a normal moderation task — §5.1 or Django Admin (§2) cover everything short of this.

## 6. Confirming a container is running the process you expect

**Historical context (why this section exists at all).** Until Phase 11 (2026-08-15), there was a single `web` container whose mode (`next dev` vs `next start`) was picked at build time by an `.env` flag (`APP_ENV`) — a **mutable** value, re-read on every restart. A real incident happened because of that: after switching `.env` back to `APP_ENV=prod` and running `./stack.sh restart`, the restart was interrupted partway through (the command ran across a session boundary and never finished). A follow-up health check that only asserted `curl localhost:3000/` returned `200` looked clean and passed — but the container underneath was actually still running the *previous* `npm run dev` process. Dev mode stayed live on the public tunnel for a while as a result, causing real, intermittent breakage for actual visitors (CONTENT_DESIGN.md §29, a "stuck loading" report that turned out to have nothing to do with application code). `go-prod.sh`/`run-dev.sh` existed specifically to catch this — they wrapped `stack.sh restart` with a `ps aux` assertion and exited non-zero if the wrong process was found.

**That ambiguity is now structurally impossible, not just guarded against.** `docker-compose.yml` gives `web` a fixed `target: prod` and `web-dev` a fixed `target: dev` — both **literal strings**, never interpolated from any env var. There is no code path, interrupted restart or not, by which the container named `web` ends up running `next dev`, or `web-dev` ends up running `next start` — the mode is baked into which service definition you're looking at, not a runtime toggle. `go-prod.sh` and `run-dev.sh` are retired; there's nothing left for them to verify that isn't already guaranteed by the compose file itself.

**Still worth a plain sanity check after any restart** — not to catch mode ambiguity (gone), just ordinary "did the container actually come up and start the process I expect" hygiene:
```sh
docker compose ps                                    # all 4 services Up?
docker exec bismillah-web-1 ps aux | head -5          # expect next-server / npm run start
docker exec bismillah-web-dev-1 ps aux | head -5      # expect next dev
docker exec bismillah-backend-1 ps aux | head -5      # expect gunicorn
```

**`./update-prod.sh`** rebuilds + recreates just `web` from current source (what's been tested live on `web-dev`) and includes this same kind of check — waits for `localhost:3000/` to respond, then asserts `npm run start` is the actual running process, exiting non-zero if not:
```sh
./update-prod.sh   # "promote" web-dev's tested changes to prod (web)
./stack.sh restart web-dev   # restart just the dev container, e.g. after a dependency change
```
