# Project 4 — Agile Delivery & UAT: Deliver the reschedule/auto-confirm release and prove it works
**Client:** QuickKlinik (fictional) · **Role:** Junior BA (also acting SM in this mock team) · **Timebox:** Weeks 7–8 · **Status:** Complete
**Business question:** How do we deliver the Project 2 requirements in sprints and prove they work?

## 1. Product backlog (Trello/Jira board mirrored in `BA_Project_Tracker_Filled.xlsx`)
Epics: Booking · Reschedule · Waitlist · Clinic · Platform
| Story | Epic | Pts | Priority | Sprint |
|---|---|---|---|---|
| US-001 Auto-confirm | Booking | 5 | High | S1 |
| US-002 Confirmation details | Booking | 2 | High | S1 |
| US-007 Clinic notifications | Clinic | 5 | High | S1 |
| US-010 Audit logging | Platform | 3 | High | S1 |
| US-003 Self-service reschedule | Reschedule | 8 | High | S2 |
| US-004 Self-service cancel | Reschedule | 5 | High | S2 |
| US-011 Performance NFR | Platform | 5 | High | S2 |
| US-005 <2h warning flag | Reschedule | 3 | Med | S3 |
| US-006 Waitlist offers | Waitlist | 8 | Med | S3 |
| US-008 Exceptions dashboard | Clinic | 5 | Med | S3 |
| US-009 KPI report | Clinic | 5 | Med | S3 |
| US-012 Accessibility pass | Platform | 3 | Med | S3 |

**Sprint goals:** S1 — *"A patient booking a free slot gets instant confirmation, and clinics see it in real time."* S2 — *"Patients can fully self-serve reschedule and cancel."* S3 — *"Exceptions, waitlist and KPIs close the loop."*

## 2. Estimation & planning notes
Story points via team Fibonacci against US-001=5 anchor. Sprint 1 capacity 2 devs × 10 days × velocity 0.7 ≈ 14 pts → committed 15 (slight stretch, flagged). Velocity plan 13/15/17.

## 3. Definition of Ready / Done
**DoR:** story + AC reviewed by PO; design/wireframe linked; NFRs identified; testable.
**DoD:** code merged + reviewed; unit tests pass; AC verified by BA on staging; NFR checks run; analytics/audit events firing; PO sign-off; documentation updated.

## 4. Sample ceremony artefacts
- **Sprint planning (S2):** committed US-003/004/011 = 18 pts; risk: US-003 8-pt story may split (slot-hold race condition); action: spike half-day first.
- **Daily stand-up update (day 3, S2):** "Yesterday: slot-hold spike done — optimistic locking chosen. Today: implement US-003 happy path. Blocker: SMS gateway sandbox keys not received (Issue I-001) — PO chasing, fallback = email for beta."
- **Sprint review summary (S2):** demo to 2 clinic beta users; US-003/004 pass happy path; US-011 p95 = 1.4s at 25 bookings/min (better than NFR). Feedback: "make the cancel button less prominent" — parked as backlog item, not scope creep.
- **Retrospective:** *Went well:* early clinic feedback caught wording issues. *Didn't:* gateway dependency discovered late. *Actions:* vendor contract checklist added to DoR; spike-first rule for >5-pt stories.

## 5. RACI
| Activity | Patient rep | BA (me) | PO | Engineers | Clinic manager |
|---|---|---|---|---|---|
| Backlog priority | C | R | A | C | C |
| BRD / requirements | C | R/A | C | C | C |
| Build & unit test | I | C | C | R/A | I |
| UAT execution | C | R | A | C | C |
| Go/no-go decision | I | R | A | C | C |

## 6. UAT plan
- **Scope:** all Must stories (US-001, 002, 003, 004, 007, 008, 010, 011); negative & boundary cases included.
- **Entry criteria:** build deployed to staging; test data loaded; 3 beta users trained (30-min walkthrough).
- **Exit criteria:** 100% Must-linked tests executed; ≥95% pass; zero open Sev-1/2 defects; UAT sign-off by PO + clinic manager.
- **Roles:** BA = test coordinator; clinic manager + receptionist = business testers; PO = sign-off; engineer on-call for defects.
- **Environment:** staging clone of production data (anonymised), SMS in sandbox mode, Figma interactive prototype for pre-code walkthrough.
- **Defect severity:** Sev-1 = blocks core flow/data loss · Sev-2 = workaround exists, material impact · Sev-3 = cosmetic · Sev-4 = suggestion.

## 7. UAT test cases (19 executed against staging + Figma prototype — extract; full in tracker)
| Test ID | Req ID / Story | Scenario | Key steps | Expected | Result |
|---|---|---|---|---|---|
| UAT-001 | FR-001 / US-001 | Auto-confirm free slot | Book free slot | CONFIRMED <60s with details shown | Pass |
| UAT-002 | FR-001 / US-001 | Slot taken mid-booking | Concurrent booking of last slot | Second user told slot unavailable, offered alternatives | Pass |
| UAT-003 | FR-001 / US-001 | Network timeout on confirm | Kill connection at confirm | No duplicate booking; state recoverable; message shown | **Fail (Sev-2)** → duplicate pending record created; fixed, retest Pass |
| UAT-004 | FR-002 / US-003 | Reschedule >2h before | Pick new free slot | Old slot released, new CONFIRMED, details updated | Pass |
| UAT-005 | FR-002 / US-003 | Reschedule into past/fully-booked day | Attempt invalid slot | Not selectable; clear message | Pass |
| UAT-006 | FR-005 / US-005 | Reschedule <2h before | Attempt within 2h | Warning shown; action completes with flag to clinic | Pass |
| UAT-007 | FR-002 / US-004 | Cancel confirmed booking | Tap Cancel + confirm | Status CANCELLED; clinic notified <5s | Pass |
| UAT-008 | FR-002 / US-004 | Cancel twice (idempotency) | Double-tap cancel | Single CANCELLED event; no duplicate notification | **Fail (Sev-3)** → duplicate SMS queued; fixed |
| UAT-009 | FR-003 / US-007 | Clinic sees reschedule in real time | Reschedule as patient | Badge + event list update <5s | Pass |
| UAT-010 | FR-003 / US-008 | Exceptions list sorted | Generate late-cancel + failed offer | Both appear in today's exceptions, sorted by time, with phone no. | Pass |
| UAT-011 | FR-004 / US-006 | Waitlist offer within 15 min | Cancel slot with waitlisted patient | First patient gets SMS + 15-min hold link | Pass |
| UAT-012 | FR-004 / US-006 | Offer expires → next patient | Ignore SMS >15 min | Slot passes to next waitlist entry; first patient notified politely | **Fail (Sev-2)** → slot released to public instead of next waitlist; fixed, retest Pass |
| UAT-013 | NFR-001 / US-011 | p95 latency at 25 bookings/min | Load test on staging | <2s at p95 | Pass (1.4s) |
| UAT-014 | NFR-002 / US-010 | PDPA: data encrypted | Review config + sample record | TLS 1.2+, at-rest encryption; no phone numbers in logs | Pass |
| UAT-015 | NFR-004 / US-010 | Audit trail completeness | Perform book→reschedule→cancel | Audit entries for every event with actor + before/after | Pass |
| UAT-016 | US-002 | Confirmation shows prep notes | Clinic adds prep note pre-booking | Note visible on confirmation screen | Pass |
| UAT-017 | US-012 | Keyboard-only reschedule | Tab through flow with keyboard only | Full flow completable; focus visible; contrast AA | **Fail (Sev-3)** → focus ring missing on date strip; fixed |
| UAT-018 | US-009 | KPI report vs source events | Month-end report | No-show rate matches event log within 0.5% | Pass |

**Defect log (5 defects — 4 found above, plus 2 exploratory):** Sev-2 ×2 (duplicate booking state; waitlist expiry routing), Sev-3 ×3 (duplicate SMS, focus ring, wording on warning). All retested and closed (original cases updated to Pass after retest; see tracker). **Final pass rate: 19/19 = 100%; Sev-1/2 open: 0.**

## 8. Requirements Traceability Matrix (Req → Story → Test)
| Req | Story | UAT | Coverage |
|---|---|---|---|
| BR-001 | US-001, US-003, US-004, US-009 | UAT-016, UAT-018 | ✔ |
| BR-002 | US-001, US-007, US-008 | UAT-019 | ✔ |
| FR-001 | US-001, US-002 | UAT-001–003, UAT-016 | ✔ |
| FR-002 | US-003, US-004 | UAT-004–008 | ✔ |
| FR-003 | US-007, US-008 | UAT-009, UAT-010 | ✔ |
| FR-004 | US-006 | UAT-011, UAT-012 | ✔ |
| FR-005 | US-005 | UAT-006 | ✔ |
| NFR-001 | US-011 | UAT-013 | ✔ |
| NFR-002 | US-010 | UAT-014, UAT-015 | ✔ |
| NFR-003 | US-012 | UAT-017 | ✔ |
| NFR-004 | US-010 | UAT-015 | ✔ |
**Must-have coverage: 100%** (verified automatically in the tracker Summary sheet).

## 9. Go / No-Go statement
**GO.** All Must requirements covered by passing tests; zero Sev-1/2 defects open; performance exceeds NFR; PDPA review complete; beta clinics confirmed usability. *Conditions:* phone fallback retained for 60 days; KPI dashboard reviewed at day 30 against the ≤10% no-show trajectory.

---
*Independent case study. Fictional client, mock ceremonies executed against a Figma prototype and staging environment simulated for learning.*
