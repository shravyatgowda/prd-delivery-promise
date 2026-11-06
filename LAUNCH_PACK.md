# Launch Pack: Live Delivery-Time Promise

Companion to the [PRD](README.md). *Self-initiated case study; scenarios are illustrative, not from a real company.*

## 1. UAT test cases
| ID | Scenario | Steps | Expected result |
|---|---|---|---|
| UAT-1 | Normal order, no delay | Place order; watch tracking to delivery | Promise shown at checkout; no delay message; ETA stays within promise |
| UAT-2 | Kitchen-side delay | Hold order in kitchen past threshold | Delay message fires once with new ETA and reason "kitchen" |
| UAT-3 | Rider-side delay | Delay rider pickup past threshold | Delay message fires once with reason "rider"; ops dashboard lists the order |
| UAT-4 | ETA recovers | Delay, then speed up delivery | Customer sees updated, shorter ETA; no repeated alarming messages |
| UAT-5 | Multiple delays | Trigger two breaches on one order | At most one message per defined interval; latest ETA always shown |
| UAT-6 | App offline during delay | Disable network, then restore | Latest ETA and message appear on reconnect; no stale value |
| UAT-7 | Cancellation after delay message | Cancel from tracking screen | Cancel reason logged; `order_cancelled` event carries delay flag |
| UAT-8 | Event logging | Run UAT-1 to UAT-7 | `promise_shown`, `delay_notified`, `eta_updated`, `order_cancelled`, `support_contacted` all logged correctly |

**Bug handling:** log each bug with severity (P0 blocks launch, P1 fix before pilot, P2 backlog), owner, and status; follow up with Engineering until closed.

## 2. Launch readiness checklist (go / no-go)
- [ ] PRD signed off by Product, Tech, Ops, Support
- [ ] All UAT cases passed; no open P0/P1 bugs
- [ ] Event tracking verified end to end
- [ ] Ops dashboard live and tested with kitchen and rider leads
- [ ] Support briefed; macros and FAQ ready for delay questions
- [ ] Pilot zone and order share agreed
- [ ] Rollback plan owner and trigger defined (e.g. conversion guardrail breach)
- [ ] Post-launch review booked

## 3. Post-launch review template
| Item | Notes |
|---|---|
| Primary and guardrail metrics vs target | Scale / iterate / roll back |
| Top 10 breaching orders: root cause | Kitchen / rider / traffic / system |
| Support tickets: volume and themes | |
| What worked / what did not | |
| SOP and checklist updates | |
| Next actions, owners, dates | |
