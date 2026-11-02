# Activity Diagram: AD-UC-03 Confirm Blood Received

| Field | Value |
|---|---|
| AD ID | `AD-UC-03` |
| Use case | `UC-03 Confirm Blood Received` |
| Feature | `F-01 Blood Ordering & Fulfillment` (W0 Table 2) |
| Primary actor | Hospital Staff |
| Lanes | Hospital Staff · NBSP (name taken from the workbook, confirm against the Context Diagram) · no external systems |
| Inputs (L3-L5) | Context Diagram v<n> · Use Case Diagram v<n> · Sequence: `SD-UC-03` · Architecture/ADR: not used · W1 rows: FR-01-05 (IF clause proposed, see Q1) |
| Trigger | Hospital Staff confirms arrival of a dispatched shipment (FR-01-05). |
| Preconditions | Hospital Staff is signed in (assumed; UC-06 / FR-04-01). The shipment was dispatched (FR-01-03). |
| Success end state | The request is Received and the received units are in the hospital inventory. |
| Failure end state(s) | The shipment is not Dispatched: the confirmation is rejected, nothing changes (proposed, Q1). |
| FRs covered | `FR-01-05` |
| Business rules applied | none |
| Version / status | v0.1 · Draft |
| Author | <owner of F-01, W0 Table 2> |

## 1. Diagram

```mermaid
swimlane-beta TB
  subgraph HS["Hospital Staff"]
    start@{ shape: sm-circ }
    a1("Confirm arrival of dispatched shipment")
  end
  subgraph SYS["NBSP"]
    d1{"Shipment dispatched?"}
    s1("Mark request Received (FR-01-05)")
    o1["Blood request [Received]"]
    s2("Add received units to hospital inventory (FR-01-05)")
    s3("Reject confirmation and state that the shipment is not Dispatched (FR-01-05 proposed)")
    stop1@{ shape: fr-circ }
  end
  start --> a1 --> d1
  d1 -->|"[dispatched]"| s1 --> o1 --> s2 --> stop1
  d1 -->|"[not dispatched]"| s3 --> stop1
```

Reading notes:

- **The `[not dispatched]` branch is a proposal, not an approved requirement.** FR-01-05 says only what happens for a dispatched shipment. The rejection is drawn as option A of Q1. The row FR-01-05 in W1 needs this IF clause (EARS): *IF the shipment is not in Dispatched status, the system SHALL reject the confirmation and SHALL change neither the request nor the hospital inventory.*
- Marking Received and adding units are drawn in sequence, not as a fork, because they must succeed or fail together. Whether they run as one atomic operation is a design matter and is not shown here.
- The main reason for the decision is protecting the inventory: a second confirmation must not add the same units twice.

## 2. Node trace

| Node | Kind | Text | Traces to |
|---|---|---|---|
| d1 | Decision | Shipment dispatched? | FR-01-05 (confirms a dispatched shipment); proposed IF clause (Q1) |
| s1 | Action | Mark request Received | FR-01-05 |
| o1 | Object node | Blood request [Received] | FR-01-05 |
| s2 | Action | Add received units to hospital inventory | FR-01-05 |
| s3 | Action | Reject confirmation and state that the shipment is not Dispatched | FR-01-05, proposed IF clause (Q1) |

## 3. Branches and acceptance tests

Values in these rows are examples.

| Branch | Given | When | Then |
|---|---|---|---|
| d1 [dispatched] | A request has status Dispatched with 3 assigned units of A+; hospital A+ inventory is 5 | Hospital Staff confirms arrival | Request status is Received; hospital A+ inventory is 8 |
| d1 [not dispatched] | The request is already Received and hospital A+ inventory is 8 | Hospital Staff confirms arrival again | The confirmation is rejected with the reason (proposed, Q1); status stays Received; A+ inventory stays 8 |

## 4. Object states used

| Object | States in this diagram | Full state list (all diagrams) |
|---|---|---|
| Blood request | Received | Pending, Dispatched, Received (from FR-01-01, FR-01-03, FR-01-05; confirm whether Approved is a separate value) |

## 5. Screens touched

| Actor action | Screen ID |
|---|---|
| | |

## 6. Open questions

| # | Question | Blocking? | Default applied |
|---|---|---|---|
| 1 | [NEEDS CLARIFICATION: What happens when the shipment is not in Dispatched status, for example a second confirmation of an already Received request? Reject with a reason, or show the current status without an error? FR-01-05 must be revised to add this IF clause.] | No | Option A: reject with a reason, change nothing (drawn in section 1), pending confirmation. |
| 2 | [NEEDS CLARIFICATION: Can Hospital Staff confirm only some of the assigned units, for example when some units arrive damaged or missing?] | No | All assigned units are treated as received. |

## 7. Verification log

| Check | Result | By | Date |
|---|---|---|---|
| Renders on mermaid.live | | | |
| Every decision has guards on all outgoing flows, no overlap, no gap | Pass | | |
| No action has two incoming flows (merge used instead) | Pass | | |
| Every fork has a matching join | n/a (no forks) | | |
| Every system action cites an FR | Pass (s3 cites the proposed clause of FR-01-05) | | |
| Every FR of this use case appears in the diagram | 1 of 1 | | |
| Lane names match the Context Diagram and the use case actors; no Web App, API or database lanes | | | |
| Related sequence diagram uses the same names and FR IDs; its alt fragments appear here as decisions | | | |
| AI critic prompt run, findings resolved or logged as open questions | | | |
