# Future Work · Agent Assistance Layer

English / [简体中文](docs/future/FUTURE_AGENT_CN.md)

---
> Related documents: SRS v0.1 ([English](REQUIREMENTS.md) / [中文](docs/requirements/REQUIREMENTS_CN.md)) · [Future Work (Backlog)](FUTURE.md)
> Last updated: 2026-10-07
> Proposed by: Backend 1 (code E, M1 Identity and access control)
> Status: **candidate proposal, not adopted in any version**. The v0.1 MVP (M1–M7) proceeds as agreed in the meeting minutes; the group leader selects items from this file afterwards.

**Scope statement (read this first)**

- This file **does not change the v0.1 MVP scope**: M1–M7, the milestone schedule and the acceptance criteria all stay as they are;
- It is a **candidate list**, meant to be picked from item by item in a later version;
- Adopting an item follows the change discipline in [FUTURE.md](FUTURE.md): group-leader decision → move the item back into the requirements document → bump its version → record the reason → re-check the related FR / BR / AC (English and Chinese kept in sync);
- This file uses its own IDs (`FWA-xx`) to avoid colliding with `FW-01 … FW-14`. On adoption the group leader assigns the module number (`M8` is already the audit log, so `M9` is the next free slot).

**Why this is a separate document**

Agent capabilities are a **cross-cutting concern**: they reuse M1 authentication, depend on M3 candidate data and add entry points on M4 / M7, yet they belong to **none** of M1–M7. Keeping them in one file makes them selectable one by one instead of silently widening another member's column.

## 1. Positioning and red lines (binding for every item)

| # | Red line |
| --- | --- |
| R1 | **Read-only**: never writes ballot data (`ballots`, `ballot_choices`, `vote_participation`), and cannot see who voted for whom |
| R2 | **No recommendation**: never answers "who should I vote for / who is better / who will win", never ranks candidates |
| R3 | **Citations required**: every claim carries a source; without a source the answer states "not covered by the material" |
| R4 | **Admission**: requires login (M1) plus the election eligibility check (M2 / M4); the final authorization check stays in the backend |
| R5 | **Privacy**: a voter's question can reveal voting intent, so `agent_run` is anonymised by default and stores no personally linkable question text |
| R6 | **Degradation**: when the model is unavailable, return the source material or an explicit notice — never a 500 and never an invented answer |

## 2. Candidate items (the selection menu)

| ID | Item | Notes | Depends on | Value | Cost | Risk |
| --- | --- | --- | --- | --- | --- | --- |
| FWA-01 | Structured candidate material | Add "submitted material" (long text / documents) per candidate, captured in fixed slots: education / work experience / record / platform / commitments / disputed or unanswered points | M3 candidate management | High | Low (plain CRUD, no AI dependency) | Low |
| FWA-02 | Material retrieval layer (RAG ingestion) | Material → chunk → embedding → vector index. MySQL has no native vector type, so the MVP stores embeddings as BLOB with in-process cosine similarity behind a `VectorIndex` port | FWA-01 | Medium | Medium | Medium (selection and operations) |
| FWA-03 | Voter information assistant (Q&A) | Answers "what is this candidate's record / platform"; fixed three-part answer (supported by material / no response / not covered) with mandatory citations | FWA-01, FWA-02 | Highest | Medium | Medium (hallucination, fairness) |
| FWA-04 | Neutrality and refusal guardrails | Out-of-scope intent detection with fixed refusal wording; symmetric retrieval (equal top-k per candidate); ordering decided by the system, never by the model | FWA-03 | Highest | Low | Medium (drives the review impression) |
| FWA-05 | Candidate comparison view | Same slots shown side by side for several candidates — parallel presentation only, no verdict | FWA-01 | High | Low | Low |
| FWA-06 | Tally explanation assistant | Turns the deterministic tally output into plain language; **no arithmetic of any kind** | M6 tally output (read-only) | High | Low | Low |
| FWA-07 | Agent audit and feedback | `agent_run` (call / model / latency / cost / status) and `agent_feedback` (human adoption rate) | All items | Medium | Low | Low (complements M8 audit log) |
| FWA-08 | Evaluation and fairness metrics | Citation accuracy, hallucination rate, refusal rate, schema validity plus coverage balance and counterfactual consistency; a 30–50 item evaluation set | FWA-03 | Highest | Medium | Low |
| FWA-09 | Process FAQ assistant | Answers "how do I vote / when does it close / why am I not eligible" — process only, never who to vote for | M1, M2, M4 | Medium | Low | Low |
| FWA-10 | Offline debate research module (optional) | Multi-stance agent debate plus aggregation, compared with the real outcome as research | None | Low | Medium | Medium (must state it is not public opinion) |

> Value and cost are the proposer's own assessment and serve ordering only; the group leader decides what is adopted.

## 3. Dependencies and preconditions

1. **Data first**: FWA-01 is a precondition for FWA-02 / 03 / 05. With only the v0.1 short candidate profile (one paragraph per candidate), a retrieval layer is meaningless — material comes first, RAG second.
2. **Reused interfaces**: authentication from M1 (FR-05 / FR-06); admission from the M2 / M4 eligibility check (FR-07).
3. **Cost and secrets**: LLM / embedding keys may only live in backend environment variables, never in the repository or the frontend; decide who provides and pays for them, plus rate limiting.
4. **Privacy review**: R5 shares its root with AC-07 anonymity, so adoption requires backend 3 to review it jointly.
5. **Architecture decision**: adopting any item starts with an ADR under `docs/architecture/`.

## 4. Explicitly out of scope for this proposal

- Any write to ballot data, automatic invalidation of a ballot, or "the agent votes for the user";
- Any change to the MVP (M1–M7) scope, schedule or acceptance criteria;
- Using real voter data; recommending or ranking candidates.

## 5. Suggested adoption order, if ever approved

1. `FWA-01` structured candidate material (pure data, no AI dependency, safe to land first)
2. `FWA-08` evaluation framework (measure before building model capability)
3. `FWA-02 + FWA-03 + FWA-04` smallest useful Q&A (retrieval + generation + guardrails)
4. `FWA-07` audit and feedback
5. `FWA-05 / FWA-06 / FWA-09` experience extensions
6. `FWA-10` research bonus

## 6. Adoption process

Identical to [FUTURE.md](FUTURE.md): **group-leader decision → move the item back into the requirements document → bump its version → record the reason → re-check the related FR / BR / AC (both language versions)**; add an ADR for any Agent-related architecture choice. This file changes no existing v0.1 conclusion.
