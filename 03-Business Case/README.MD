# Project 3 — Data-Driven Business Case: Should QuickKlinik invest in the reminder & self-service platform change?
**Client:** QuickKlinik (fictional) · **Role:** Junior BA · **Timebox:** Weeks 5–6 · **Status:** Complete
**Business question:** Should the company invest in fixing no-shows (auto-confirm + reminders + self-service), and what is the expected return?
**Decision to be made:** Approve/reject the build (Option A: full build now; Option B: phased — self-service first, reminders later; or Do Nothing).

## 1. Decision framing
**3 questions the data must answer:**
1. Do no-shows actually cost meaningful money? (size of problem)
2. What drives no-shows — booking lead time, reminder contact, prior reschedules? (where to act)
3. Is the expected recovery big enough to pay back the build? (worth it)

*Data: Olist public e-commerce dataset used as an **analogy dataset** (public, real transactions) to demonstrate SQL + cross-check discipline; QuickKlinik volumes are clearly labelled estimates. This mirrors how a junior BA validates method on public data when client data is unavailable.*

## 2. Analysis (SQL + Excel, queries commented — method shown)
```sql
-- Q1: How big is the late/no-show problem? (Olist analogy: late deliveries vs repeat purchase)
SELECT CASE WHEN order_delivered_customer_date > order_estimated_delivery_date
            THEN 'LATE' ELSE 'ON_TIME' END AS delivery_flag,
       COUNT(*) AS orders,
       AVG(CASE WHEN o2.customer_id IS NOT NULL THEN 1 ELSE 0 END) AS repeat_rate
FROM orders o LEFT JOIN orders o2 ON o.customer_id = o2.customer_id AND o2.order_id <> o.order_id
GROUP BY 1;
-- Finding (public data): late-delivered orders show markedly lower repeat-purchase behaviour
-- → service failures destroy future revenue. Analogy conclusion: no-shows are a revenue leak, not an admin nuisance.
```
**Cross-check:** SQL result reconciled to an Excel pivot on the same extract (difference 0.0%); both files retained.

**QuickKlinik estimate model (assumption-flagged):**
| Input | Value | Basis |
|---|---|---|
| Bookings per clinic per day | 100 | *Estimate from 14 clinics, 1,400 bookings/day total* |
| No-show rate today | 18% | Project 1 baseline |
| Realistic post-fix no-show rate | 11% | Base case; literature + Olist-analogy haircut (worst 14%, best 8%) |
| Value per attended slot | RM 60 | *Assume RM 120 consult, slot half-wasted* |
| Recovered slots kept as revenue | 50% | *Conservatism: some recovered patients would have rebooked anyway* |
| Front-desk hours saved | 1.7 h/day/clinic | Project 1 estimate |
| Loaded staff cost | RM 18/h | *Malaysian SME estimate* |

## 3. Three charts that each prove one point (titles = takeaways)
1. **"Late service kills repeat business"** — bar: repeat rate on-time vs late (Olist). *Point: failures have revenue tails.*
2. **"18% of slots evaporate to no-shows"** — waterfall: booked → cancelled → no-show → attended (QuickKlinik estimate). *Point: problem size ≈ RM 458k/yr gross slot value at full realisation.*
3. **"Two-thirds of the saving is staff time and slot recovery, not new demand"** — stacked bar of benefit components by year. *Point: benefits are defensible, not speculative growth.*

## 4. Options comparison
| Criterion | Do Nothing | A: Full build (6 wks) | B: Phased (self-service 4 wks → reminders 4 wks) |
|---|---|---|---|
| Cost (3-yr TCO) | RM 0 | ~RM 165,000 | ~RM 172,000 |
| Expected benefit (3-yr, base) | RM 0 | ~RM 496,000 | ~RM 480,000 |
| Payback | — | ~8 months | ~12 months |
| Risk | No-show stays 18%; churn of clinics | Big-bang delivery risk | Lower delivery risk; benefits later |
| Time to first value | Never | Month 3 | Month 2 (self-service) |

## 5. Financial model (see `ROI_Model.xlsx`)
- One-off costs: build RM 96k (2 eng × 6 wks loaded) + SMS gateway setup RM 4k = **RM 100k**.
- Running costs: gateway RM 3.5k/yr, cloud uplift RM 6k/yr, support RM 12k/yr = **RM 21.5k/yr**.
- Benefits/yr (base): slot recovery ~RM 98k + front-desk time ~RM 80k = **~RM 177k/yr gross** (~RM 156k net of running costs).
- **Base case: payback ~8 months, 3-yr ROI ≈ 200%, 3-yr net benefit ≈ RM 332k. Worst case: payback ~29 months, ROI ≈ 8% — still positive. Best case: payback ~3 months.** (Switch cell F15 in the model to see all three.) All assumptions in blue input cells; scenario switch cell toggles best/base/worst.

## 6. RAID log (extract — full in tracker)
| ID | Type | Description | I/L | Mitigation | Status |
|---|---|---|---|---|---|
| R-001 | Risk | Patients distrust/resist app changes | M/M | Beta with 2 clinics; keep phone fallback | Open |
| A-001 | Assumption | Clinic volumes ~100 bookings/day | — | Validate in week 1 of build via admin stats | Open |
| I-001 | Issue | SMS gateway pricing not yet contracted | H/M | Get 2 quotes before Sprint 2 | Open |
| D-001 | Dependency | PDPA review of data flows | M/L | Schedule legal review in Sprint 1 | Open |

## 7. Recommendation & next steps
**Recommend Option A (full build)** — payback ~8 months, worst case still positive; phased option saves little risk because the two features share one code path. *Next steps:* contract SMS gateway (owner: PO, week 1); PDPA review (owner: BA, week 1); beta with 2 clinics at Sprint 3; go/no-go gate after UAT (Project 4).

## 8. Five-slide pitch (3-minute script)
1. **Problem:** 18% of QuickKlinik's slots evaporate; staff spend 2h/day phoning. *(waterfall chart)*
2. **Evidence:** service failures kill repeat business (Olist analogy); failure cost ≈ RM 458k/yr gross. *(bar + waterfall)*
3. **Options:** Do nothing / Full build RM 148k / Phased RM 162k. *(comparison table)*
4. **Recommendation:** Full build — payback ~8 months, 3-yr ROI ~200%, worst case still positive. *(ROI curve)*
5. **Next steps:** gateway contract, PDPA review, 2-clinic beta, UAT go/no-go gate. *(timeline)*

**Limitations stated:** correlation ≠ causation in Olist analogy; QuickKlinik volumes are estimates; benefits haircut applied; model is deliberately conservative.

---
*Independent case study. All QuickKlinik figures are labelled estimates; Olist is public data used for method demonstration.*
