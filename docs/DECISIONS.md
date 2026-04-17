# PyStrategy Decisions Log

บันทึกการตัดสินใจสำคัญ พร้อมเหตุผล + ทางเลือกที่พิจารณา

---

## 2026-04-17: 3-slot session system (not MemPalace yet)

**Context:** memory across Claude Code sessions
**Options:**
- MemPalace (ChromaDB, 200MB)
- 3-slot custom system (68KB)
- No system

**Decision:** 3-slot first, review 22 Apr 2026
**Criteria → MemPalace:** ถ้า pain > 50%

---

## Decision Log — Live Data Source (Apr 2026)

### ตัวเลือกที่พิจารณา

| Option | ราคา | Real-time | หมายเหตุ |
|---|---|---|---|
| Dukascopy datafeed (ปัจจุบัน) | $0 | ❌ delay 1 วัน | Historical only Jan 2024+ |
| MetaApi + CXMDirect Demo | ~$14/เดือน | ✅ | deploy ทดสอบแล้ว Apr 2026 |
| MetaApi + Dukascopy Broker | ~$14/เดือน | ✅ | ต้อง deposit $100 + KYC + video call |
| TwelveData Free | $0 | ⚠️ delay 15min | 2000+ symbols แต่ไม่ใช่ broker data |

### สถานะ Apr 2026

- ทดสอบ MetaApi + CXMDirect Demo แล้ว (account ID เก็บใน Replit Secrets: METAAPI_ACCOUNT_ID)
- MetaApi balance: $2.90 (free credit) — ยังไม่มีบัตรผูก
- ยังไม่ตัดสินใจจ่ายจริง

### TODO — รีวิวเดือนหน้า (พ.ค. 2026)

- [ ] live candle ทำงานได้จริงไหม?
- [ ] ค่าใช้จ่ายจริง $14/เดือน คุ้มไหมกับ user ที่มี?
- [ ] ถ้าไม่คุ้ม → เปลี่ยนเป็น TwelveData Free (delay 15min)
- [ ] ถ้าคุ้ม → ผูกบัตร + ตั้ง auto deposit
