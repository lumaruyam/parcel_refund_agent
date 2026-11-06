# ParcelCheck — damaged-parcel refunds decided in a minute

**Course:** Data & AI Product: From Diagnosis to Delivery (MSA-DAT07-04) · Albert School · shared project with Agentic Workflows (MSA-DAT07-03)
**Cohort:** M2-INTER-2

## Team

| Name | Git handle | Also in the Agentic Workflows team? |
|---|---|---|
| Luli | lumaruyam | yes |
| Rania | raniamehria | yes |
| Wei-Ling | whung99 | yes |
| Chanelle | csanchez-cyber | yes |

## Our agent

- **Starting point:** P4 — Refund and cancellation agent for an e-commerce logistics SaaS, narrowed to damaged-parcel claims and unused-label refunds. The switch from P7 was approved by the professor.
- **The company:** a shipping-label and returns platform with 15,000 merchants, 25,000 tickets a month (30% refunds) and 40 support agents (guess).
- **The job, in one sentence:** When a merchant reports a parcel delivered damaged and uploads a photo, the agent describes and classifies the damage, applies the refund policy and writes a decision receipt, producing an executed refund ≤ €20 or a ready-to-approve decision for a person above it.
- **Trust level per step:**
  - L4 ≤ €20 when every check passes;
  - L2 €20–100, and every decline;
  - L1 > €100, unclear photos, and any incoherence (person only).
- **One thing it must never do:** pay an amount the policy does not give for the facts on the label.
- **Decision so far (`02_investment_memo.md`):**
  - NO-GO on the full build as sized;
  - GO first on rules-only unused-label refunds;
  - CONDITIONAL GO on the AI module: a damaged claim must cost ≥ €17.50 to handle by hand, and the agent must reach ≥ 40% L4 with zero wrong refunds.

## Repository structure

One numbered file per session. Every member authors commits in at least four of the numbered files.

| File | Session | Status |
|---|---|---|
| `00_pitch.md` | before S3 — only if you bring your own idea | not needed (P4) |
| `01_problem_statement.md` | S1–S2 — Assignment 1 | v1 for P4; personas invented, to check in the interviews |
| `governance.md` | created for Assignment 1, one line added every session | S1, S2, S3 sections |
| `02_sizing.xlsx` | S2 — Assignment 2 | v1 for P4 (open in Excel to recalculate) |
| `02_investment_memo.md` | S2 — Assignment 2 | v1 for P4 |
| `03_product_brief.md` | S3 | v1; v2 §9–10 after the users |
| `03_ai_lean_canvas.md` | S3 | v1 |
| `03_user_research.md` | S3 | kit ready; **interviews + time study to do** |
| `03_prototype_link.md` | S3 | Claude Design prompt ready; **link to add** |
| `04_architecture.md` | S4 | next |
| `05_metrics.md` | S5 | |
| `06_ab_protocol.md` | S6 | |
| `07_operations.md` | S7 | |
| `08_launch_memo.md` | S8 | |
| `spec.md`, `product_brief.md` | final | `spec.md` v1 (S3 skeleton, completed ahead) |
| `/prototype`, `/evals` | from S4 | `evals/eval_set_v0.csv`: 20 cases, **labels to check by hand** |

## Rules

- Commit with your own Git account: the individual question at the oral is about a commit you signed.
- Tag the final version `v1.0` before the first oral of your cohort. No commits after the first oral starts.
