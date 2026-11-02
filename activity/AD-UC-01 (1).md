# Activity Diagram: AD-UC-01 Request Blood Provision

| Field | Value |
|---|---|
| AD ID | `AD-UC-01` |
| Use case | `UC-01 Request Blood Provision` |
| Feature | `F-01 Blood Ordering & Fulfillment` (W0 Table 2) |
| Primary actor | Hospital Staff |
| Lanes | Hospital Staff · NBSP (name taken from the workbook, confirm against the Context Diagram) · no external systems |
| Inputs (L3-L5) | Context Diagram v<n> · Use Case Diagram v<n> · Sequence: `SD-UC-01` · Architecture/ADR: not used · W1 rows: FR-01-01 (IF clause proposed, see Q1) |
| Trigger | Hospital Staff submits a blood type, quantity and urgency (FR-01-01). |
| Preconditions | Hospital Staff is signed in (assumed; sign-in is UC-06 / FR-04-01, not stated in FR-01-01). |
| Success end state | A Pending blood request exists and is in the NBC request queue. |
| Failure end state(s) | None in this flow: invalid input is rejected with reasons and the flow returns to validation until the input is valid. Abandoning the request is not modeled (no FR covers it). |
| FRs covered | `FR-01-01` |
| Business rules applied | none yet; the rule defining valid input is missing (Q2) |
| Version / status | v0.3 · Draft |
| Author | Nhàn |

## 1. Diagram

```mermaid
swimlane-beta TB
  subgraph HS["Hospital Staff"]
    start@{ shape: sm-circ }
    a1("Submit blood request with type, quantity and urgency")
    a2("Correct the invalid fields and resubmit")
  end
  subgraph SYS["NBSP"]
    m1{" "}
    s1("Validate request input (FR-01-01)")
    d1{"Input valid?"}
    s4("Reject request and show the reason for each invalid field (FR-01-01 proposed)")
    s2("Create Pending blood request (FR-01-01)")
    o1["Blood request [Pending]"]
    s3("Place request in NBC request queue (FR-01-01)")
    stop1@{ shape: fr-circ }
  end
  start --> a1 --> m1 --> s1 --> d1
  d1 -->|"[input valid]"| s2 --> o1 --> s3 --> stop1
  d1 -->|"[input invalid]"| s4 --> a2 --> m1
```

Reading notes:

- **The invalid-input branch is a proposal, not yet an approved requirement.** FR-01-01 today says only validate, create, queue. The loop is drawn as option B of Q1. The workbook row FR-01-01 needs this IF clause added (EARS): *IF the input fails validation, the system SHALL create no request and SHALL show the reason for each invalid field, and Hospital Staff SHALL be able to correct and resubmit.*
- `m1` is a merge. The first submission and every resubmission enter validation through it, so the loop returns through a merge and `s1` has one incoming flow.
- What counts as "valid" is not defined yet (Q2). Until it is, `d1` traces only to FR-01-01, not to a business rule.

## 2. Node trace

| Node | Kind | Text | Traces to |
|---|---|---|---|
| m1 | Merge | (first submission or resubmission) | none (control node) |
| s1 | Action | Validate request input | FR-01-01 |
| d1 | Decision | Input valid? | FR-01-01; BR to be added (Q2) |
| s4 | Action | Reject request and show the reason for each invalid field | FR-01-01, proposed IF clause (Q1) |
| s2 | Action | Create Pending blood request | FR-01-01 |
| o1 | Object node | Blood request [Pending] | FR-01-01 |
| s3 | Action | Place request in NBC request queue | FR-01-01 |

## 3. Branches and acceptance tests

Values in these rows are examples. Replace them when Q2 is answered.

| Branch | Given | When | Then |
|---|---|---|---|
| d1 [input valid] | Hospital Staff of one hospital is signed in | Staff submits blood type O-, quantity 4, urgency Urgent | One request with status Pending exists for O- x 4, Urgent, and it appears in the NBC request queue |
| d1 [input invalid] | Hospital Staff is signed in | Staff submits blood type O-, quantity 0, urgency Urgent, then corrects quantity to 4 and resubmits | After the first submission no request exists and the reason for the quantity field is shown (proposed, Q1; whether 0 is invalid: Q2). After the resubmission one Pending request exists for O- x 4, Urgent |

## 4. Object states used

| Object | States in this diagram | Full state list (all diagrams) |
|---|---|---|
| Blood request | Pending | Pending, Dispatched, Received (from FR-01-01, FR-01-03, FR-01-05; confirm whether Approved is a separate value) |

## 5. Screens touched

| Actor action | Screen ID |
|---|---|
| | |

## 6. Open questions

| # | Question | Blocking? | Default applied |
|---|---|---|---|
| 1 | [NEEDS CLARIFICATION: Confirm that invalid input is rejected with a reason for each invalid field and that Hospital Staff can correct and resubmit. Is a reason per field required, or is one general message enough? FR-01-01 must be revised to add this IF clause.] | No | Option B: reject with reasons, correct and resubmit (drawn in section 1), pending confirmation. |
| 2 | [NEEDS CLARIFICATION: Which values are valid for each field? Blood type: which allowed values, and whole blood only or also components? Quantity: whole number, minimum, maximum per request, unit? Urgency: which levels, and is it required? Which fields are mandatory, and are duplicate requests blocked?] | Yes | None. Test values in section 3 are examples only. |

## 7. Verification log

| Check | Result | By | Date |
|---|---|---|---|
| Renders on mermaid.live | | | |
| Every decision has guards on all outgoing flows, no overlap, no gap | Pass | | |
| No action has two incoming flows (merge used instead) | Pass | | |
| Every fork has a matching join | n/a (no forks) | | |
| Every system action cites an FR | Pass (s4 cites the proposed clause of FR-01-01) | | |
| Every FR of this use case appears in the diagram | 1 of 1 | | |
| Lane names match the Context Diagram and the use case actors; no Web App, API or database lanes | | | |
| Related sequence diagram uses the same names and FR IDs; its alt fragments appear here as decisions | | | |
| AI critic prompt run, findings resolved or logged as open questions | | | |
