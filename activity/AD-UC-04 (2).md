# Activity Diagram: AD-UC-04 Record Blood Usage

| Field | Value |
|---|---|
| AD ID | `AD-UC-04` |
| Use case | `UC-04 Record Blood Usage` |
| Feature | `F-02 Inventory Monitoring & Usage` (W0 Table 2) |
| Primary actor | Hospital Staff |
| Lanes | Hospital Staff · NBSP (name taken from the workbook, confirm against the Context Diagram) · no external systems |
| Inputs (L3-L5) | Context Diagram v<n> · Use Case Diagram v<n> · Sequence: `SD-UC-04` · Architecture/ADR: not used · W1 rows: FR-02-01 (IF clause proposed, see Q1) |
| Trigger | Hospital Staff records blood type, unit quantity, used or discarded status and reason (FR-02-01). |
| Preconditions | Hospital Staff is signed in (assumed; UC-06 / FR-04-01). The hospital holds an inventory of that blood type. |
| Success end state | Units are deducted from the hospital inventory and the transaction is in the usage history. |
| Failure end state(s) | None in this flow: a record that is invalid or exceeds the inventory is rejected with a reason and the flow returns to the check until the record is accepted. Abandoning the record is not modeled (no FR covers it). |
| FRs covered | `FR-02-01` |
| Business rules applied | proposed: deduct units with the earliest expiry date first (BR-xx to be assigned, Q3); the rule defining a valid record is still missing (Q2) |
| Version / status | v0.2 · Draft |
| Author | <owner of F-02, W0 Table 2> |

## 1. Diagram

```mermaid
swimlane-beta TB
  subgraph HS["Hospital Staff"]
    start@{ shape: sm-circ }
    a1("Record blood type, quantity, used or discarded status and reason")
    a2("Correct the record and resubmit")
  end
  subgraph SYS["NBSP"]
    m1{" "}
    d1{"Record valid and enough units in inventory?"}
    s3("Reject record and show the reason (FR-02-01)")
    s1("Deduct units from hospital inventory (FR-02-01)")
    s2("Append transaction to usage history (FR-02-01)")
    stop1@{ shape: fr-circ }
  end
  start --> a1 --> m1 --> d1
  d1 -->|"[accepted]"| s1 --> s2 --> stop1
  d1 -->|"[invalid or not enough units]"| s3 --> a2 --> m1
```

Reading notes:

- **The rejection branch is a proposal, not an approved requirement.** FR-02-01 has no IF clause. The decision `d1` exists because a record can be incomplete or ask for more units than the hospital holds. The loop is drawn as option B of Q1. The row FR-02-01 in W1 needs this IF clause (EARS): *IF the record is incomplete or the quantity exceeds the hospital inventory, the system SHALL change nothing, SHALL show the reason, and Hospital Staff SHALL be able to correct and resubmit.*
- `m1` is a merge. The first submission and every resubmission enter the check through it.
- **Deduction order is a proposed business rule (Q3):** units with the earliest expiry date are deducted first. FR-02-01 does not say this today, so BR-xx must be added to the business rules and linked to the row FR-02-01. Suggested wording: *The system SHALL deduct units in order of earliest expiry date.*
- Used and discarded follow the same flow: both deduct units and append history; only the recorded status differs.
- What counts as a valid record is not defined yet (Q2).

## 2. Node trace

| Node | Kind | Text | Traces to |
|---|---|---|---|
| m1 | Merge | (first submission or resubmission) | none (control node) |
| d1 | Decision | Record valid and enough units in inventory? | proposed IF clause (Q1); BR to be added (Q2) |
| s3 | Action | Reject record and show the reason | FR-02-01, proposed IF clause (Q1) |
| s1 | Action | Deduct units from hospital inventory, earliest expiry first | FR-02-01; BR-xx proposed (Q3) |
| s2 | Action | Append transaction to usage history | FR-02-01 |

## 3. Branches and acceptance tests

Values in these rows are examples. Replace them when Q2 is answered.

| Branch | Given | When | Then |
|---|---|---|---|
| d1 [accepted] | Hospital O- inventory is 6: 2 units expiring 5 Nov 2026 and 4 units expiring 20 Nov 2026 | Staff records O-, quantity 3, status Used, reason Surgery | O- inventory is 3, all of it from the 20 Nov lot (both 5 Nov units and 1 of the 20 Nov units were deducted), and the usage history has a new entry for O- x 3, Used, Surgery |
| d1 [invalid or not enough units] | Hospital O- inventory is 1 | Staff records O-, quantity 2, status Discarded, reason Expired, then corrects quantity to 1 and resubmits | After the first submission nothing changes and the reason is shown (proposed, Q1). After the resubmission O- inventory is 0 and the history has an entry for O- x 1, Discarded, Expired |

## 4. Object states used

| Object | States in this diagram | Full state list (all diagrams) |
|---|---|---|
| Hospital inventory | none (quantity changes only) | not defined in the workbook |

## 5. Screens touched

| Actor action | Screen ID |
|---|---|
| | |

## 6. Open questions

| # | Question | Blocking? | Default applied |
|---|---|---|---|
| 1 | [NEEDS CLARIFICATION: Confirm the handling chosen by the team: when the recorded quantity exceeds the hospital inventory or a required field is missing, the system rejects with a reason and Hospital Staff can correct and resubmit. FR-02-01 must be revised to add this IF clause.] | No | Option B, chosen by the team (drawn in section 1); pending confirmation by the lecturer. |
| 2 | [NEEDS CLARIFICATION: Which values make a record valid? Allowed values of blood type, quantity (whole number, minimum 1?), status (Used, Discarded only?), and is the reason mandatory, free text or from a list?] | Yes | None. Test values in section 3 are examples only. |
| 3 | [NEEDS CLARIFICATION: Confirm that units with the earliest expiry date are deducted first, and that this also applies to records with status Discarded.] | No | Earliest expiry first for both Used and Discarded, chosen by the team; BR-xx to be added. |

## 7. Verification log

| Check | Result | By | Date |
|---|---|---|---|
| Renders on mermaid.live | | | |
| Every decision has guards on all outgoing flows, no overlap, no gap | Pass | | |
| No action has two incoming flows (merge used instead) | Pass | | |
| Every fork has a matching join | n/a (no forks) | | |
| Every system action cites an FR | Pass (s3 cites the proposed clause of FR-02-01) | | |
| Every FR of this use case appears in the diagram | 1 of 1 | | |
| Lane names match the Context Diagram and the use case actors; no Web App, API or database lanes | | | |
| Related sequence diagram uses the same names and FR IDs; its alt fragments appear here as decisions | | | |
| AI critic prompt run, findings resolved or logged as open questions | | | |
