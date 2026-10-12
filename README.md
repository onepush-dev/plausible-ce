# Plausible Community Edition — lightweight, privacy-friendly Google Analytics alternative. Deploy on Railway with one click.

[![Deploy on Railway](https://railway.app/button.svg)](https://railway.app/new/template/M9GUKc)

## Services
| Service | Source | Notes |
|---|---|---|
| plausible | `ghcr.io/plausible/community-edition:v3.2.1` | port 8000, public domain, volume `/var/lib/plausible`, start: createdb → migrate → run |
| Postgres | Railway Postgres | users, sites, settings |
| ClickHouse | `clickhouse/clickhouse-server:24.12-alpine` | events; volume `/var/lib/clickhouse` |

## Variables
### plausible
| Variable | Template value | Secret |
|---|---|---|
| BASE_URL | `https://${{RAILWAY_PUBLIC_DOMAIN}}` | |
| SECRET_KEY_BASE | `${{secret(64)}}` | yes |
| TOTP_VAULT_KEY | `${{secret(43, "ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz0123456789+/")}}=` (base64, 32 bytes) | yes |
| HTTP_PORT / PORT | `8000` | |
| LISTEN_IP | `::` | |
| DATABASE_URL | `${{Postgres.DATABASE_URL}}` | reference |
| CLICKHOUSE_DATABASE_URL | `http://${{ClickHouse.CLICKHOUSE_USER}}:${{ClickHouse.CLICKHOUSE_PASSWORD}}@${{ClickHouse.RAILWAY_PRIVATE_DOMAIN}}:8123/${{ClickHouse.CLICKHOUSE_DB}}` | reference |
| ECTO_CH_IPV6 | `true` | |
| DISABLE_REGISTRATION | `invite_only` (first user can always register) | |
| ENABLE_EMAIL_VERIFICATION | `false` | |
| TMPDIR | `/var/lib/plausible/tmp` | |
| RAILWAY_RUN_UID | `0` (volume write permissions) | |

### ClickHouse
| Variable | Template value | Secret |
|---|---|---|
| CLICKHOUSE_USER | `plausible` | |
| CLICKHOUSE_PASSWORD | `${{secret(32)}}` | yes |
| CLICKHOUSE_DB | `plausible_events_db` | |
| CLICKHOUSE_DEFAULT_ACCESS_MANAGEMENT | `1` | |

## Notes
- Health: `/api/health` returns postgres/clickhouse/sessions status.
- Open the URL and register the first (admin) account at `/register`; later users need invites.
- Optional: add SMTP (`MAILER_EMAIL`, `SMTP_HOST_ADDR`, ...) and `MAXMIND_LICENSE_KEY` for geolocation.
