# Thong Tin Deploy - Checkpoint 5

File nay ghi lai cach kiem tra ban deploy cua bai lab. Khong ghi gia tri that cua `AGENT_API_KEY` vao repo.

## Thong Tin Hoc Vien

| Muc | Noi dung |
|-----|----------|
| Ho va ten | Pham Thi Lien |
| Mã học viên | 01795 |
| Repo | https://github.com/lienpt812/Day12-01795-PhamThiLien |

## Service

| Muc | Noi dung |
|-----|----------|
| Public URL | https://day12-01795-phamthilien-production.up.railway.app |
| Platform | Railway |
| Ngay deploy | 2026-08-10 |

## Bien Moi Truong Da Set

Chi liet ke ten bien va nguon gia tri, khong ghi secret that.

| Bien | Da set | Ghi chu |
|------|--------|---------|
| `PORT` | yes | Railway tu gan |
| `AGENT_API_KEY` | yes | Dat trong Railway dashboard, khong nam trong repo |
| `REDIS_URL` | yes | Railway Redis service variable, khong dung localhost |
| `RATE_LIMIT_PER_MINUTE` | yes | 10 |
| `MONTHLY_BUDGET_USD` | yes | 10.0 |
| `LOG_LEVEL` | yes | INFO |

## Lenh Kiem Tra Local Fallback

Stack local da duoc chay bang:

```powershell
docker compose up -d --build
docker compose ps
```

Ket qua mong doi:

```text
agent: Up, healthy, port 8000
redis: Up, healthy, port 6379
```

Kiem tra API:

```powershell
Invoke-RestMethod -Uri http://localhost:8000/health
Invoke-RestMethod -Uri http://localhost:8000/ready
try {
  Invoke-WebRequest -Uri http://localhost:8000/ask -Method POST -ContentType 'application/json' -Body '{"question":"Hello"}'
} catch {
  $_.Exception.Response.StatusCode.value__
}
```

Ket qua da quan sat:

```text
/health -> 200, {"status":"ok","service":"day12-agent","version":"1.0.0"}
/ready  -> 200, {"status":"ready","redis":true}
/ask without API key -> 401
```

## Anh Chung Minh

Anh can dat trong thu muc `screenshots/`:

- `screenshots/dashboard.png` hoac `screenshots/docker-compose.png`: trang dashboard cloud hoac terminal `docker compose ps`
- `screenshots/health.png`: ket qua goi `/health` tu browser/curl/PowerShell

## Cau Hinh Cloud Can Lam De Dat CP5 Full

Neu deploy that len Railway hoac Render, thay dong Public URL ben tren bang URL HTTPS that cua service. Khong de URL mau trong file vi test se tu dong bat URL HTTPS dau tien.

Can set cac bien moi truong tren dashboard cloud:

```text
AGENT_API_KEY=<secret rieng cua ban>
REDIS_URL=<connection string Redis cua platform>
RATE_LIMIT_PER_MINUTE=10
MONTHLY_BUDGET_USD=10.0
LOG_LEVEL=INFO
```

Khong can set `PORT` neu platform tu cap bien nay. Neu platform khong tu cap, set `PORT=8000`.

Sau khi co Public URL, chay:

```powershell
.\.venv\Scripts\python.exe -m pytest tests\test_cp5.py -q
```

Neu muon test them `/ask` tren ban deploy that, set rieng trong `.env` local:

```text
DEPLOY_API_KEY=<cung gia tri voi AGENT_API_KEY tren cloud>
```
