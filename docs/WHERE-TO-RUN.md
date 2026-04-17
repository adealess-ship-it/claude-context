# Where to Run What — Replit vs GitHub Actions

## MUST GitHub Actions (NOT Replit)

| Task | Reason | Workflow |
|---|---|---|
| MQ5 → EX5 compile | Wine blocked | .github/workflows/mql5_compile.yml |
| VDO / video generation | ffmpeg + CPU | not created yet |
| Playwright QA tests | 500MB install | repo qa-core-testing-hub |
| Docker images | Replit ไม่รองรับ | ubuntu-latest |
| Heavy C-compile (TA-Lib) | Native libs | build-tools runner |

## MUST Replit (NOT GA)

- FastAPI / Express / Vite dev
- PostgreSQL queries (DB internal)
- pyst session work
- Interactive debugging

## ข้อห้ามสำคัญบน Replit

- ❌ start uvicorn/express/vite ใหม่ → Replit workflow จัดการ
- ❌ listen() / nohup PORT=8080 → SIGTERM
- ❌ 500+ browser requests/session → SIGTERM
- ❌ Wine / MetaEditor → seccomp blocked

## Threshold

- RAM > 2GB → GA
- Install > 200MB → พิจารณา GA
- ต้อง Docker/Wine → ต้อง GA
- Not sure → ถาม Bank
