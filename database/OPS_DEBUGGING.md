# Database container readiness investigation (PostgreSQL)

## Symptom
PostgreSQL reported as "not ready" on port **5001**.

## What was found
- `.env` contains `PORT=5001`
- `startup.sh` hardcodes `DB_PORT="5000"`
- `db_connection.txt` points to `localhost:5000`
- Host checks:
  - `pg_isready -p 5001` => no response
  - `pg_isready -p 5000` => accepting connections
- `ss -ltnp` shows Postgres listening on `127.0.0.1:5000`

## Root cause
**Port mismatch**: the system expects PostgreSQL on **5001**, but Postgres is configured/running on **5000**.

## Recommended next actions
1. Choose the correct target port:
   - If expected port is **5001**, update Postgres startup/config to use 5001 (and ensure it binds appropriately for other containers).
   - If expected port is **5000**, update platform/backend expectations and `DATABASE_URL` accordingly.
2. If backend is a separate container, ensure Postgres listens on an interface reachable from other containers (often `0.0.0.0`) and the backend uses the correct hostname (not `localhost`).

## Useful commands
- Check readiness:
  - `pg_isready -h localhost -p 5000`
  - `pg_isready -h localhost -p 5001`
- Check listeners:
  - `ss -ltnp | grep -E '5000|5001'`
