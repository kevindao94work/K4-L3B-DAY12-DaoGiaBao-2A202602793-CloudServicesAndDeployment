# Deployment Report — Checkpoint 5

## Thông tin học viên và repository

| Field | Value |
|-------|-------|
| Name | Dao Gia Bao |
| Mã học viên | 2A202602793 |
| Repository | https://github.com/kevindao94work/K4-L3B-DAY12-DaoGiaBao-2A202602793-CloudServicesAndDeployment |

## Service status

| Field | Value |
|-------|-------|
| Public URL | Not deployed; using the documented local fallback |
| Platform | Docker Compose local fallback; no Railway or Render service was created |
| Verification date | 2026-09-29 |

The local stack is available at `http://localhost:8000` while Docker Compose is
running. `/health` returned `{"status":"ok","service":"day12-agent","version":"1.0.0"}` and
`/ready` returned `{"status":"ready","redis":true}`. `docker compose ps` showed
the `agent` and `redis` services healthy. No cloud deployment was attempted:
Railway CLI and deployment credentials are not available in this environment.

## Environment variables

No cloud environment is configured. For the local Compose stack:

| Variable | Local configuration |
|----------|---------------------|
| `PORT` | Set to `8000` by Compose |
| `AGENT_API_KEY` | Read from the ignored local `.env` file |
| `REDIS_URL` | Compose service address `redis://redis:6379/0` |
| `RATE_LIMIT_PER_MINUTE` | Application default `10` |
| `MONTHLY_BUDGET_USD` | Application default `10.0` |
| `LOG_LEVEL` | Application default `INFO` |

## Local verification

```bash
curl -i http://localhost:8000/health
curl -i http://localhost:8000/ready
curl -i -X POST http://localhost:8000/ask \
  -H "Content-Type: application/json" \
  -d '{"question":"Hello"}'
```

The health endpoint returned HTTP 200, readiness returned HTTP 200 with Redis
available, and an unauthenticated `/ask` request is expected to return HTTP 401.
The actual browser capture of `/health` is [screenshots/health.png](screenshots/health.png).

`LOCAL_FALLBACK=true` is set in the ignored local `.env` file for the CP5
fallback checks. There is no cloud URL or platform dashboard screenshot because
no cloud service was deployed.
