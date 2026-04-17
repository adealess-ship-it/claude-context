# PyStrategy — Claude Code Rules

## ก่อนแก้ไขทุกครั้ง
1. ตรวจด้วย shell 10-20 ข้อก่อนเสมอ เพื่อเข้าใจ state ปัจจุบัน
2. ห้าม assume ค่าใดๆ โดยไม่ verify จากข้อมูลจริง

## หลังแก้ไขทุกครั้ง
1. git push ทันที
2. รัน bash ~/workspace/poll_qa.sh รอผล GitHub Actions
3. ถ้า QA fail → วนแก้จนผ่านโดยไม่ต้องถาม
4. ถ้าครบ 2 ชม. ยังไม่เสร็จ → หยุดและแจ้งว่า "ครบ 2 ชม. ทำไม่เสร็จ"

## Permissions
- ทุกคำสั่งใน .claude/settings.local.json → allow หมด ไม่ต้องถาม
- ห้ามถาม confirmation ทุกกรณี รันเลย

## QA System
- Screenshots จาก GitHub Actions = source of truth
- E2E flow: login → chart → selftest → screenshot
- ถ้า screenshot แสดง Authentication required = token inject ไม่สำเร็จ แก้ทันที

## Stack
- FastAPI port 8000, Express port 80
- PostgreSQL via DATABASE_URL (internal Replit only)
- GITHUB_TOKEN ใน env สำหรับ poll_qa.sh

## Shell Script Safety Rules
- ห้าม start Express / uvicorn / vite ใหม่ใน script ใดๆ
- ห้าม listen() / app.listen() / server.listen() ใน script
- ห้ามใช้ nohup node ... PORT=8080 — Replit workflow จัดการเอง
- ทุก script ที่ต้องการ HTTP → ใช้ fetch/curl ไปที่ localhost:80 เท่านั้น
- ก่อนรัน script ที่ใช้ browser → ตรวจว่า server alive ก่อนเสมอ

## Screenshot / Browser Automation Rules
- ห้ามรัน screenshot-all.js โดยไม่ถามผู้ใช้ก่อน
- 22 หน้ารวด + dashboard auto-refresh = 500+ requests → Replit SIGTERM server
- ถ้าจะรัน: ทีละ 5 หน้า, เพิ่ม delay 3-5 วินาทีระหว่างหน้า
- ห้ามรันขณะผู้ใช้กำลังใช้เว็บอยู่

## Pre-commit Checks (บังคับทุกครั้งก่อน git commit)

### 1. TDZ Guard — chart.html
ก่อน commit chart.html ทุกครั้ง ให้รัน:
```bash
# ตรวจ line number ของ const chartEl และ const chart
grep -n "const chartEl\|const chart " artifacts/api-server/public/chart.html

# ตรวจว่าไม่มี code ที่ใช้ chartEl/chart ก่อน line ที่ declare
grep -n "chartEl\.\|chart\." artifacts/api-server/public/chart.html | \
  awk -F: '$1 < [LINE_OF_CONST_CHARTEL]'
```
ถ้าพบ code ใช้ variable ก่อน declare → แก้ก่อน commit ทุกครั้ง

### 1b. TDZ Guard — upload.html (Dashboard)
ก่อน commit upload.html ทุกครั้ง:
```bash
# ตรวจว่า let/const ทุกตัวใน IIFE declare ก่อนถูกใช้
# โดยเฉพาะ variable ที่ใช้ใน showDashboard/doRefresh/loadAll
grep -n "^  let \|^  const " artifacts/api-server/public/upload.html | head -20

# ตรวจว่าไม่มี .catch(() => {}) กลืน error เงียบ
grep -n "catch(() => {})" artifacts/api-server/public/upload.html
```
- ทุก let/const ต้องอยู่ **ก่อน** function ที่ใช้มัน (ระวัง async .then() เรียก function ก่อน declare)
- ห้ามใช้ `.catch(() => {})` → ใช้ `.catch(e => console.warn('context:', e.message))` แทน

### 2. Silent Fail Guard
ห้ามใช้ try/catch แบบ empty:
```javascript
// ❌ ห้ามใช้
try { something(); } catch(e) {}

// ✅ ใช้แบบนี้
try { something(); } catch(e) { console.warn('context:', e.message); }
```

### 3. API Version Check
ก่อนเรียก method ใหม่บน external library ให้ grep หา version ก่อน:
```bash
grep -n "lightweight-charts" artifacts/api-server/public/chart.html | head -3
```
- v3.x: ไม่มี setCrosshairPosition, clearCrosshairPosition
- v4+: มีครบ

---

## PROJECT CONTEXT (verified 2026-04-17)

### Architecture
- **Frontend**: Express Node.js port 80 — `artifacts/api-server/` (TypeScript, build → `dist/index.mjs`)
- **Backend**: FastAPI Python port 8000 — `artifacts/pytrigger-api/`
- **DB**: PostgreSQL via `DATABASE_URL` (Replit-internal)
- **No R2/S3/Cloudflare storage** — `.ex5` binaries เก็บใน Postgres BYTEA

### Real Express paths (24 routes — verified from src/app.ts)
- `/login` `/register` `/profile` `/forgot-password` `/reset-password` `/verify-email`
- `/login-qr` `/qr/verify/:token`
- `/chart` `/discovery-hall`
- `/test-pyai` ← **คือ Laboratory** (UI nav ใช้ชื่อนี้)
- `/account/backtest` `/account/backtest/history` `/account/leaderboard` `/account/marketplace`
- `/account/indicator-builder` `/account/indi-index`
- `/account/qr-test` `/account/wine` `/account/receivedfile` ← Dashboard files page
- ❌ ไม่มี `/laboratory` `/backtest` `/dashboard` (ใช้ path เต็มข้างบน)

### Real FastAPI paths (94 endpoints, prefix `/pytrigger`)
- Auth: `/auth/login` `/auth/register` `/auth/qr-*` `/auth/forgot-password` `/auth/reset-password` `/me`
- Strategies: `/strategies/my` (GET list) `/strategies/save` (POST create) `/strategies/{sid}` (GET/DELETE)
- EA Generation: `/strategy/{sid}/generate-mq5` `/strategy/{sid}/export-ea` (singular!)
- MQL5 Compile: `/mql5/compile` `/mql5/status/{job_id}` `/mql5/download/{job_id}`
- Backtest: `/backtest/run` `/backtest/optimize` `/backtest/status/{job_id}` `/backtest/runs` `/backtest/queue`
- Discovery: `/discovery-hall` (NOT `/discovery/hall`) `/discovery/user/{user_id}` `/discovery/nickname`
- Leaderboard: `/leaderboard/strategies` `/leaderboard/indicators` `/leaderboard/my-stats`
- Indicators: `/indi/list` `/indi/standard` `/indi/custom/*` `/indi/ai-assist` (POST)
- Candles: `/candles?symbol=XAU/USD&...` (slash format!) `/candles/symbols`
- Account: `/account/kt-code` `/account/kt-code/set-birthday`
- Admin: (admin routes — see private repo)
- Health: `/healthz` `/test-report`

### DB Tables (verified)
- `users` — auth + user profile (schema — see private repo)
- `saved_strategies` — strategies table (schema — see private repo)
- `leaderboard_strategies` — discovery items (schema — see private repo)
- `backtest_runs` + `backtest_results` — engine output (schema — see private repo)
- `price_candle` — large historical dataset (schema — see private repo)
- `mql5_compile_cache` — EA binary cache (schema — see private repo)
- `qr_sessions` `private_nicknames` (KT code system)

### Backtest engine quirks
- `_load_candles` ดึง `timeframe='1min'` เสมอ → aggregate ใน pandas เป็น TF เป้าหมาย (M5/H1/H4/D1)
- Symbol ต้องใช้ `XAU/USD` (มี slash) ไม่ใช่ `XAUUSD`
- Engine return keys ≠ DB schema:
  - engine: `win_rate`, `total_return` (%), `final_capital` ($)
  - DB: `win_rate_pct`, `total_profit_pct`, `total_profit_usd` (ส่วน persist แปลงเอง)

### Credentials (verified env)
- `DATABASE_URL` `GITHUB_TOKEN` `JWT_SECRET` `SESSION_SECRET`
- `QA_PASSWORD` (admin login สำหรับ QA scripts)
- `METAAPI_TOKEN` `TWELVEDATA_API_KEY` `RESEND_API_KEY`
- `AI_INTEGRATIONS_ANTHROPIC_API_KEY` (PyAI ภายใน app) + `GEMINI_API_KEY`

### GitHub Repo Secrets (6 ตัว)
`API_TOKEN` `DATABASE_URL` `GH_EMAIL` `GH_PASSWORD` `JWT_SECRET` `REPLIT_DEV_DOMAIN`

### EA / MQL5 System
- EA source files in (QA test suite)
- Generator: (internal module)
  - **Important**: ห้าม emit `// ... )` ใน `if(...)` (line comment กิน paren). ใช้ `/* ... */` แทน
  - Unimplemented indicators → `if(false /* X — not implemented */)` (fail-safe)
- Compile pipeline: `mql5_compile.yml` workflow, semaphore=2, cache by SHA256
- EA binaries cached in DB
- ⚠️ Race condition ใน `_find_run` — fix อยู่ใน mql5_compiler.py แล้ว, active หลัง server restart (ตอนนั้นค่อย add `run-name` กลับใน workflow yml)

### Status (2026-04-17)
- All Phase Test: **657/657 PASS**
- Discovery Hall: discovery items deployed
- EA Compiled: EA files compiled
- QA 100-route: เพิ่ง fix paths รัน batch ใหม่อยู่

### Known Pitfalls
- ❌ `XAUUSD` (no slash) → 404 จาก `/candles`
- ❌ `/pytrigger/strategies` (plain) → 404, ต้อง `/strategies/my`
- ❌ `/pytrigger/discovery/hall` → 404, ต้อง `/discovery-hall`
- ❌ `/pytrigger/optimisation/*` → 404 (ไม่มี namespace นี้, ใช้ `/backtest/optimize`)
- ❌ `/pytrigger/strategies/{sid}/mq5` → 404, ต้อง `/strategy/{sid}/generate-mq5` (singular!)
- ❌ `/pyai/ask` → 404, ต้อง `/indi/ai-assist`
- ❌ uvicorn ไม่มี `--reload` — Python file changes ไม่ active จน Replit restart workflow

### Session Start Checklist
1. `curl -s -o /dev/null -w "%{http_code}" http://localhost:8000/docs` ต้อง 200
2. `git log --oneline -5` ดู commit ล่าสุด
3. `bash qa_100.sh` (ถ้ามี QA_PASSWORD) เพื่อ baseline

---
## Multi-session awareness (3-slot system)

Added 2026-04-17 by pyst setup. Version 2.2 (with sync reminders).

### Active sessions
- Each Claude Code tab runs in slot 1/2/3 (max 3)
- Own slot: read $PYST_SLOT env var
- Check others: cat ~/workspace/.claude/session-{1,2,3}.md
- Before destructive changes: pyst-list (bash)

### Concurrency rules
- Max 3 concurrent sessions
- NEVER run DB migrations concurrently with other slots
- NEVER call PyAI concurrently (queue MAX_RUNNING=1)
- Check file conflicts before major edits

### Sync reminder system
- Watcher checks every 5 min
- If session has changes AND last pyst-export was > 10 min ago
  → appends "📎 SYNC REMINDER" section to session file
- pyst-list shows "⚠️ unsynced Xmin" next to slot
- When you see reminder: run pyst-export in another terminal

### Security — NEVER log to session/PROGRESS files
- Credentials, tokens, API keys (METAAPI_TOKEN, Replit Secrets)
- Customer PII
- .env contents
Redact before logging if seen in output.


---
## Where to run what (critical — inline)

### MUST GitHub Actions (NOT Replit)
- MQ5 compile — Wine blocked
- VDO / video — ffmpeg + CPU
- Playwright tests — 500MB
- Docker — Replit ไม่รองรับ

### MUST Replit (NOT GA)
- FastAPI / Express / Vite dev
- PostgreSQL queries
- pyst session work

### ถ้าไม่แน่ใจ → docs/WHERE-TO-RUN.md

---
## See Also (index)

- **Work history** → `.claude/PROGRESS.md`
- **TODOs + schedule** → `docs/TODO.md`
- **Decision records** → `docs/DECISIONS.md`
- **Detailed Replit vs GA** → `docs/WHERE-TO-RUN.md`
- **Permissions** → `.claude/settings.local.json`
- **Sync Claude.ai** → run `pyst-sync-context`
