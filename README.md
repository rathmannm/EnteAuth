# EnteAuth — self-hosted Ente Auth (Auth-only)

Museum + Postgres, deployed via Dockhand as a Git stack. No secrets live in
this repo — they are set in Dockhand's Environment Variables UI (stored
AES-256-GCM encrypted) and override the placeholders in `museum.yaml` at
runtime (Museum reads env vars prefixed `ENTE_`, mapping `.`/`-` to `_`).

## Environment variables to set in Dockhand

| Variable              | Notes                                             |
| --------------------- | ------------------------------------------------- |
| `POSTGRES_USER`       | `pguser`                                          |
| `POSTGRES_DB`         | `ente_db`                                         |
| `POSTGRES_PASSWORD`   | must equal `ENTE_DB_PASSWORD`                     |
| `ENTE_DB_PASSWORD`    | same value as `POSTGRES_PASSWORD`                 |
| `ENTE_API_ORIGIN`     | `http://<host-ip>:8080` at first, then proxied URL|
| `ENTE_KEY_ENCRYPTION` | 32-byte base64 secret                             |
| `ENTE_KEY_HASH`       | 64-byte base64 secret                             |
| `ENTE_JWT_SECRET`     | 32-byte urlsafe-base64 secret                     |

`POSTGRES_PASSWORD` and `ENTE_DB_PASSWORD` are the same password fed to two
places (Postgres init, and Museum's DB client). Set both to the same value.

Secrets live only in Dockhand — never commit them here. Record them in
Vaultwarden + paper. Losing `ENTE_KEY_ENCRYPTION` makes the vault unrecoverable.

To regenerate secrets later, from `ente/server`:
`go run tools/gen-random-keys/main.go`

## First run
1. Deploy stack in Dockhand from this repo; add the env vars above.
2. Watch `ente-museum` logs come up clean (DB connected).
3. Create your account in the Ente Auth app (dev mode: tap screen 7x, enter URL).
4. Verification code is in the Museum logs — search `verif`.
5. Whitelist admin: `docker exec -it ente-postgres psql -U pguser -d ente_db`,
   `SELECT user_id, encrypted_email FROM users;`, then set `ENTE_INTERNAL_ADMINS`.
6. Reverse proxy / Tailscale in front of :8080; update `ENTE_API_ORIGIN`.

## Backups
Back up the `postgres-data` volume (encrypted seeds). The secrets live in
Dockhand, not here. Losing `ENTE_KEY_ENCRYPTION` makes the vault unrecoverable.
