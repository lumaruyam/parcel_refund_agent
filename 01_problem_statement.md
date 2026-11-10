# 01 — Problem statement

**Team:** Luli, Rania, Wei-Ling, Chanelle · **Agent:** P4 — ParcelCheck, damaged-parcel refund agent (e-commerce logistics) · **Last updated:** 06/11/2026 (switched from P7, with the professor's approval)

> Every number says where it comes from: **measured**, **analogy** (a comparable, with its source) or **guess** (and how you will test it). Same base case as `02_sizing.xlsx`.

## 1. The problem in one sentence

Support agents at a shipping-label and returns platform handle **7,500 refund tickets a month** by hand. About **2,250** of them are damaged-parcel claims that need a photo check. This costs **≈ €810k a year** in handling, and merchants wait **3 days**, while refunds under €20 cost more to process than they are worth. Today the agents read each claim, open the photo, look up the label, check the policy and reply by hand. Success looks like this: within 6 months, refunds ≤ €20 are executed in under a minute for **≥ 40% of the decidable damaged-parcel claims** and **≥ 90% of unused-label refunds**, with **zero** wrong automatic refunds.

| Number in the sentence | Value | Measured / analogy / guess | Source or test |
|---|---|---|---|
| Tickets per month | 25,000 | **Measured** | P4 proposal (company logs) |
| Share about refunds | 30% → 7,500/month | **Measured** | P4 proposal |
| Damaged-parcel claims | 30% of refunds → 2,250/month (9% of all tickets) | **Guess** | Tag 200 refund tickets from the export |
| Cost of one ticket handled by a human | €9 all-in | **Analogy** | S2 slides: €5–15 per human ticket; 40 agents × €42k ÷ 300,000 tickets = €5.6 of agent time + leads, QA, tools |
| Handling cost of refund tickets | 7,500 × 12 × €9 ≈ €810k/year | Derived | — |
| Merchant wait | 3 days | **Measured** | P4 proposal |
| "Refunds under €20 cost more than they are worth" | €9–12 to process a €4.90–€20 refund | **Measured** (pain) + analogy (cost) | P4 proposal; §6 time study |
| Success: 40% of decidable damaged claims at L4 | 40% | **Guess** → S4 test | Policy-only upper bound on synthetic claims: 46.7% (P4 blueprint) |

## 2. Problem tree — from symptom to cause

- **Symptom** (what people complain about): merchants wait 3 days for a refund of a few euros, and the support queue is full of refund tickets.
- **Problem** (what is actually going wrong): every refund, whatever its size, goes through the same manual check. A €4.90 label refund costs the same 12–17 minutes as a €400 premium claim.
- **Root cause** (5 Whys):
  1. Why do merchants wait 3 days? Refund tickets queue behind everything else, and each one takes 12–17 minutes.
  2. Why so long? The agent has to open the photo, find the label in another tool (cover, declared value, fragile option, dates) and work out the policy by hand.
  3. Why by hand? The policy has 6 clauses and caps, and the facts sit in three places (claim, photo, label system).
  4. Why does no rule do it? Rules handle the label facts, but nothing reads the **photo** or the merchant's free text ("le carton est écrasé mais le vase semble intact").
  5. Why is that the bottleneck? Because a refund decision needs both the facts and the damage evidence, and only a person can join them today.
- **What would fix the cause:**
  - For **unused / void labels**, a **rule** on the tracking events ("never scanned → refund the label fee") fixes it. No AI is needed: this is the hurdle project.
  - For **damaged parcels**, the fix is a **hybrid**: pretrained vision models read the photo, a small fine-tuned LLM maps the evidence to a clause and writes the receipt, and rules recompute the amount and route by money.

## 3. Is AI the right tool?

| Question | Our answer |
|---|---|
| Is there a rule that solves 80% of it? | **For unused labels, yes:** ship the rule (it is our hurdle). For damaged parcels, no: the rule needs to know *what the photo shows*, and that is the part only a model can read. The amount is still computed by a rule |
| Is it a prediction / generation problem, or an organisational decision? | The **policy is an organisational decision** (the caps, the 7-day window, the €20 threshold): we write it down and do not let a model invent it. AI only **classifies the evidence** and **writes the receipt** |
| Do we have the data — which, how much, who owns it? | No real claims: synthetic claims labelled by the policy (3,000), real public parcel photos (Roboflow, CC BY 4.0) hand-labelled by two of us (100), pretrained models. In production the platform owns the label, tracking and claim data; merchants own the photos (see `governance.md`) |
| What does one error cost — and who pays it? | **Over-refund:** up to the cap (€25 standard, €500 premium) plus the label fee, paid by the platform: silent money loss. **Wrong refusal:** a merchant who re-contacts (≈ 2 tickets) or leaves, paid in churn and reputation. Both are counter-metrics |

**Level chosen:** a **hybrid workflow** (a fixed path, not a free agent loop): rules + pretrained vision models + a small LLM (prompt-only first, fine-tuned if the eval asks for it), with the money decided by rules. **Why not the level below:** a classical classifier on label fields cannot see the damage. A prompt-only LLM ignores our JSON format and our clause logic, which the Agentic Workflows before/after test measures.

## 4. Why this agent — the four selection questions

| Criterion | Our answer, one line |
|---|---|
| High frequency | ≈ 75 damaged claims and 175 unused-label refunds every working day |
| Bounded decision space | 6 clauses, 3 decisions (refund / decline / more evidence), 1 amount, 3 trust levels: one slide |
| Measurable success | Per claim: right clause **and** right amount (to the cent), and the right trust level |
| Recoverable failure | Borderline (as P4 says). Money paid is hard to take back, so L4 only at ≤ €20, the amount is always recomputed by rules, every decline goes to a person, and there is a weekly sample and a kill switch |

## 5. Press release (Working Backwards)

**Paris, 1 June 2027 — Refunds for damaged parcels now take a minute, not three days.**
Merchants on our label platform can photograph a damaged parcel in their dashboard and get a decision straight away. For small refunds (up to €20) the money is back before they close the page. Above that, a support agent approves a decision that is already prepared. Every decision comes with a short receipt: the rule applied, what the photo showed, and the amount. The same text goes to the merchant and into our audit log. In the first quarter, 4 in 10 eligible damaged-parcel claims were refunded automatically, and not one automatic refund was wrong. *"I used to chase three emails for a €6 label. Now I know in a minute, and I can see exactly why,"* says the owner of a ceramics e-shop (illustrative quote).

## 6. What would have to be true?

| # | Hypothesis | Measured / analogy / guess | How we will test it | Session |
|---|---|---|---|---|
| 1 | ≥ 40% of decidable damaged claims can be resolved at L4 with zero wrong automatic refunds | Guess (46.7% is the policy-only ceiling on synthetic data) | Prototype on test A (300 synthetic) + test B (100 real photos) | S4 prototype |
| 2 | A damaged-parcel claim costs ≥ €17.50 to handle by hand (≈ 24 min all-in, about 2× an average ticket). **This is the assumption that flips the investment decision** (`02_investment_memo.md`) | Guess (our as-is estimate is ≈ 17 min ≈ €12) | Time study: shadow or replay 30 claims with a support agent, or 3 interviewees' last claims step by step | S3 interviews → S5 |
| 3 | Merchants accept an automatic decision with a receipt, and do not re-contact more than with a human decision | Guess | Shadow mode then A/B by merchant: re-contact within 7 days | S6 A/B |

## 7. Do-nothing baseline

In 12 months, without the agent, the queue still works:
- agents get faster with macros and a better refund form (≈ 4% a year, about **€108k** of value — sizing row 59);
- ticket volume grows with the merchant base;
- the 3-day wait stays.

The real competitor is the **rules-only auto-refund of unused labels**, which the same team could ship in weeks. It is worth ≈ €159k in year 1 (base case), and it is our hurdle.

## 8. Persona and journey maps

**Persona 1: the refund agent (who uses the reviewer queue).** *Invented: to check in the interviews.*
- **Who:** Inès, 29, tier-1 support at the label platform, refund queue.
- **Goal:** clear 30 tickets a day at QA ≥ 90.
- **Pain:** jumping between the ticket, the photo and the label system, and doing the cap arithmetic.
- **Fear:** being blamed for a wrong refusal, or for the bot's over-refund.
- **Tools:** helpdesk, label back-office, payments tool.

**Persona 2: the merchant (who files the claim).** *Invented: to check in the interviews.*
- **Who:** Thomas, owner of a 3-person ceramics e-shop, ships 400 parcels a month.
- **Goal:** get the money back and reship before the end customer complains.
- **Pain:** three emails and three days for a €6 label fee.
- **Fear:** a refusal with no explanation.

**Agent journey (as-is, damaged claim):**

| Step | Minutes (guess) | Where the pain sits |
|---|---|---|
| 1 Read the claim, identify the type | 2 | |
| 2 Open the photo, judge the damage | 3 | **pain 1: judgement, 10% disagreement (guess)** |
| 3 Look up the label: cover, declared value, fragile option, dates | 3 | **pain 2: another tool, copy-paste** |
| 4 Apply the policy, compute the amount | 2 | **pain 2: caps and the 7-day window, 5% errors (guess)** |
| 5 Team-lead approval if > €20 | 2 + waiting | |
| 6 Execute the refund, write the reply | 3 | |

Total ≈ 15 min agent time + 2 min lead time ≈ **€12**. The time study (hypothesis 2) checks this.

**Merchant journey (as-is):**
1. Sees a damaged parcel, or the end customer complains.
2. Looks for the claim form (often emails instead).
3. Attaches photos.
4. Waits.
5. Chases ("any news?" ≈ 0.5 extra tickets per claim, guess).
6. Gets a reply after about 3 days.
7. Reships or refunds the end customer, out of pocket meanwhile.

## 9. Stakeholder map

| Quadrant | Who | Why |
|---|---|---|
| **Manage closely** (high power, high interest) | Head of Support · CFO / Finance | Support owns the decisions and the queue; Finance owns the refund leakage and the rollback figure |
| **Keep satisfied** (high power, low interest) | Legal / DPO · Payments & fraud team | Quiet until the first incident: a wrong refusal quoted publicly, photos with personal data, refunds executed by a machine |
| **Keep informed** (low power, high interest) | Refund agents (L2/L1 reviewers) · Merchant success managers | They carry adoption: the reviewer queue is their tool; they hear merchant complaints first |
| **Monitor** | Merchants · Carriers | One screenshot of an unexplained refusal moves merchants to "manage closely"; carriers matter for later claim filing (v1) |
