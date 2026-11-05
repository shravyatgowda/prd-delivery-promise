# PRD: Live Delivery-Time Promise & Delay Handling

**Author:** Shravya T  |  **Status:** Draft v1 (self-initiated case study)  |  **Domain:** 10-minute food delivery

> **Note:** This is a self-initiated product spec written to practise PM craft. It is not tied to any real company's data or roadmap. Every number marked *(assumed)* is an assumption to be replaced with real data before a real launch.

## 1. Problem statement
In rapid food delivery, the promise is the product. When an order's real delivery time drifts past the time shown at checkout, customers lose trust, contact support, or cancel, and the next order is less likely.

**Hypothesis:** customers tolerate a late order far better when the app tells them early and honestly than when the countdown silently slips.

## 2. Users and their pain
| User | Pain today (assumed) |
|---|---|
| Customer | Sees one ETA at checkout; no warning when the order is slipping; has to chat with support to learn why |
| Operations / support | Absorbs "where is my order" tickets that are really missing-information problems |
| Product | Cannot separate a late order from a surprised customer in the data |

## 3. Goals and non-goals
**Goals:** (1) reduce delay-related cancellations; (2) reduce "where is my order" contacts; (3) keep conversion at checkout unchanged.
**Non-goals (v1):** changing rider assignment logic, compensation or refund policy, delivery-time prediction models (owned by data science).

## 4. Requirements
| # | User story | Acceptance criteria |
|---|---|---|
| R1 | As a customer, I see a delivery-time promise at checkout | Promise shown before payment; shown as a range once predicted prep + ride time is available |
| R2 | As a customer, I am told as soon as my order is likely to be late | Proactive in-app message fires when predicted ETA exceeds the promise by more than a set threshold; message states the new time and reason category (kitchen / rider / traffic) |
| R3 | As a customer, I can see updated ETA without contacting support | Tracking screen updates ETA at a set refresh interval; no stale value older than the interval |
| R4 | As an ops lead, I can see which orders are breaching and why | Dashboard lists breaching orders with reason category; refreshes near real time |
| R5 | As a PM, I can measure the feature | Events logged: promise_shown, delay_notified, eta_updated, order_cancelled (with reason), support_contacted |

**Edge cases to specify:** ETA improves after a delay message; multiple delay updates on one order; kitchen-side delay vs rider-side delay; app offline during delay.

## 5. User flow (text)
Checkout → promise shown → order placed → tracking screen → (on predicted breach) delay message with new ETA + reason → updated tracking → delivered → post-order rating prompt records whether the promise held.

## 6. Success metrics
| Type | Metric | Direction |
|---|---|---|
| Primary | Delay-related cancellation rate | Down |
| Secondary | "Where is my order" contacts per 1,000 orders | Down |
| Secondary | Rating on delayed orders | Up |
| Guardrail | Checkout conversion rate | No significant drop |
| Guardrail | Promise-kept rate | Not worse than baseline |

## 7. Experiment plan
- **Design:** 50/50 randomised split of orders at checkout; control = current experience, variant = proactive delay messaging and live ETA.
- **Baseline (assumed):** delay-related cancellation rate 4%.
- **Sample size:** to detect a 1.0 percentage-point reduction (4% → 3%) at 5% significance and 80% power, about **5,275 orders per arm**; to detect 0.5 pp, about **22,639 per arm**. *(Computed with a two-proportion power calculation; see section 10.)*
- **Analysis:** two-proportion Z-test on cancellation; checkout conversion as guardrail; check consistency across zones and kitchens before trusting the topline (same approach as my [A/B test project](https://github.com/shravyatgowda/ab-testing-framework)).
- **Decision rule:** scale if primary metric improves significantly and no guardrail degrades; iterate if the direction is right but not significant; roll back if conversion drops.

## 8. Rollout plan
| Phase | Scope | Go / no-go to next phase |
|---|---|---|
| 0. Readiness | Spec signed off by Product, Tech, Ops, Support; events verified | Event QA passed |
| 1. UAT | Internal test orders covering kitchen-delay and rider-delay cases | No P0/P1 bugs open |
| 2. Pilot | One zone, small share of orders | Messaging accurate; no spike in contacts |
| 3. A/B | Planned test above | Sample size reached |
| 4. Scale | Remaining zones in waves | Guardrails stable per wave |

## 9. Risks and open questions
- **Over-notifying** could increase anxiety; threshold needs tuning (test two thresholds).
- **Wrong ETA** makes the message worse than silence; depends on prediction accuracy from data science.
- **Reason categories** may expose operational problems; align wording with Ops and Support.
- Open: should proactive compensation ever be offered? (out of scope v1, needs finance input)

## 10. Post-launch review checklist
- [ ] Primary and guardrail metrics read out with a scale / iterate / roll-back call
- [ ] Top 10 breaching orders reviewed for root cause
- [ ] Support tickets tagged and trended
- [ ] SOP and launch checklist updated with what we learned

### Sample-size calculation (reproducible)
```python
from statsmodels.stats.power import NormalIndPower
from statsmodels.stats.proportion import proportion_effectsize
es = proportion_effectsize(0.04, 0.03)
print(NormalIndPower().solve_power(es, alpha=0.05, power=0.8))  # ~5,275 per arm
```

## License
MIT, see [LICENSE](LICENSE).
