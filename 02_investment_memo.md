# 02 — Investment memo (one page) · ParcelCheck, damaged-parcel refund agent (P4)

**Recommendation:**
- **NO-GO on the full damaged-parcel AI build as sized.**
- **GO first on the rules-only auto-refund of unused labels** (the hurdle).
- **CONDITIONAL GO on the damaged-parcel AI module, only if both of these hold:**
  1. a 30-claim time study shows that a damaged claim costs **≥ €17.50** to handle by hand (≈ 24 min all-in, about 2× an average ticket);
  2. the S4 prototype resolves **≥ 40%** of decidable claims at L4, with **zero** wrong automatic refunds on test A + B.

> Line one is the decision. Everything below is evidence for that line. Same base case as `02_sizing.xlsx`.

| €/year, year 1 | Pessimistic | Base | Optimistic | Expected value |
|---|---|---|---|---|
| Net annual value | −91k | +117k | +449k | **+121k** |
| Probability | 0.30 | 0.50 | 0.20 | — |
| Do-nothing (value of the current process improving on its own) | 92k | 108k | 111k | 108k (base) |
| Next-best project: rules-only auto-refund of unused / void labels (the hurdle) | 31k | 159k | 368k | 159k (base) |

## 1. How much?

**Company (P4 proposal, measured):** 25,000 tickets a month, 30% refunds, merchants wait 3 days. Cost of a human ticket: €9 all-in (analogy). This is 40 agents (a guess) × €42k ÷ 300,000 tickets = €5.6 of agent time, plus leads, QA and tools. Three categories:

- **Damaged-parcel claims** (the AI part):
  - 9% of tickets = 2,250 a month; 75% are decidable (photo, delivered status, label data);
  - **40% of those resolved at L4** gives 675 a month at steady state;
  - with the year-1 ramp (10 → 30 → 50 → 60%, average 37.5%), that is 3,037 claims ≈ **€27k** gross;
  - **+ €16k** of time saved on the claims a person still decides with a pre-filled decision (Other value).
- **Unused / void label refunds** (rules inside the agent):
  - 21% = 5,250 a month × 90% decidable × 92% auto ≈ **€176k** gross in year 1.
- **Delivery questions:** out of scope (Include = 0).

Total after error cost and tokens: **€214k**; minus fixed costs of **€97.5k** = **€117k**.

- **Measured:** volume, the refund share, the 3-day wait.
- **Analogies:** €9 per ticket, salary, GCP and Gemini prices.
- **The guesses that matter:** the damaged share of refunds (30%), the L4 rate (40%), and the hand-handling cost of a damaged claim.

## 2. How sure?

Expected value **€121k**: it beats do-nothing (€108k) but **not the hurdle (€159k)**.

**No deflection rate on damaged claims flips the decision.** The sensitivity table goes from −€48k vs the hurdle at 30% to −€20k at 75%. The reason: the rules-only project captures the label-refund value without the AI fixed costs, so the AI module must pay its own fixed costs (≈ €81k a year) from about 2,250 claims a month.

**What does flip it:**
- the **cost of one damaged claim ≥ €17.50** instead of the €9 average (our as-is guess is ≈ €12);
- or **damaged claims ≥ 19% of tickets** instead of 9%.

Year 2 (adoption flat at 60%) closes most of the gap: €246k vs €264k.

**The S4 prototype tests the L4 rate and the zero wrong refunds** (test A synthetic, test B real photos). The **time study** (3 interviews + 30 replayed claims) tests the cost per claim.

## 3. What if we're wrong?

**It is not the model that dominates the cost but the fixed lines**, €97.5k a year:
- integration €30k a year (amortised);
- monitoring incl. a 24/7 T4 endpoint €22k;
- evaluation and hand-labelling €20k;
- supervision (the 5% weekly sample) €13.5k;
- re-evaluation at model changes €12k.

**Error cost, priced:** 3% wrong L4 decisions × €35 (pessimistic: 6% × €60). The **tail** is a systematic over-refund (for example a premium €500 cap paid on standard cover). The guardrail recomputes every amount from the policy, and the rollback triggers at **€630 of leakage per 1,000 damaged claims** (2× the error cost priced here) or **> 2% wrong refusals** in any week.

**Governance line:**
- **Wrong refusals are a legal and reputational risk**, not just money: the receipt is a statement the company is held to (Moffatt v. Air Canada, 2024). **Owner: Head of Support.**
- Over-refunds are money. **Owner: Finance.**

## 4. Compared to what?

**Do-nothing:** the queue improves by ≈ 4% a year (≈ €108k) and merchants still wait 3 days.

**Next-best project (the same team):** rules-only auto-refund of unused labels, €159k base. It is cheap, fast and has no model risk. **The pessimistic case (−€91k) does not clear the pessimistic hurdle (€31k).**

Hence the sequence:
1. Ship the rules first.
2. Keep the AI module as a prototype until the two conditions are measured.

## 5. Build, buy or orchestrate?

**We orchestrate, and build only the part that earns it.** Gemini (Vertex AI, EU) reads the photo; our rules and guardrail decide the money. The fine-tuned Phi-3 decision model stays only if it beats the prompt-only base model by ≥ 30 points on clause-and-amount (the Agentic Workflows before/after).

Weighted scores (`Build or buy` tab): **Build 3.5 · Buy 2.65 · Orchestrate 3.5**. The tie is settled by that before/after test.

Buying is cheaper at year-1 volume: €21k a year of vendor fees at ≈ €0.92 per resolution, against €62k of build-specific lines. But no vendor agent recomputes the amount from our policy, or shows why it paid. **If one does, at ≤ €1 per resolution with exportable logs, we buy.**

*Sources:*
- P4 proposal (Albert MSA-DAT07-04) — volume, refund share, pain;
- S2 slides — cost per ticket and per conversation, Octopus Energy ramp;
- P4 blueprint — policy-only L4 ceiling of 46.7% on synthetic claims;
- Google Cloud Vertex AI pricing (Gemini 2.5 Flash, n1-standard-8 + T4) and Intercom Fin public pricing — recheck on the day;
- Moffatt v. Air Canada, 2024 BCCRT 149.

The calculations are in `02_sizing.xlsx` (Inputs, Upside, Decision, Build or buy).
