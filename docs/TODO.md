# PyStrategy TODO

งานค้าง + schedule

## 🚨 Security — URGENT

- [ ] **Revoke ghp_ token** ที่หลุดใน .git/config
      - Token prefix: [REDACTED — see local .git/config] (17 Apr 2026)
      - Action: GitHub → Settings → Developer settings → Personal access tokens
      - Revoke old + create new with minimum scopes (repo only)
      - Store in Replit Secrets as GH_TOKEN
      - Update git remote: git remote set-url origin https://github.com/USER/REPO.git
      - Setup credential helper: git config --global credential.helper store
      - **Due: 18 Apr 2026**

- [ ] Audit .metaapi/ folder
      - Contains UUID in filenames
      - Add to .gitignore if not already
      - Check: git ls-files | grep metaapi

## 🔴 Urgent

- [ ] Review 3-slot system → **22 Apr 2026**
- [ ] Discovery Hall page implementation

## 🟡 Near-term

- [ ] PyStrategy UI/UX unification
- [ ] MQ5/EA code generation
- [ ] QA automation (qa-core-testing-hub)

## 🟢 Later

- [ ] Payment integration (Omise/PromptPay)
- [ ] AskMeShop Always On + Line OA
- [ ] Review MetaApi live data → **May 2026**

## 📅 Schedule Map

| เมื่อไหร่ | เรื่อง | ที่ไหน | Status |
|---|---|---|---|
| **18 Apr 2026** | 🚨 Revoke ghp_ token | manual | PENDING |
| ทุก 5 นาที | pyst-watcher heartbeat | local | auto |
| ทุก 10 นาที | Sync reminder check | local | auto |
| 22 Apr 2026 | Review 3-slot system | manual | pending |
| May 2026 | Review MetaApi decision | manual | pending |
