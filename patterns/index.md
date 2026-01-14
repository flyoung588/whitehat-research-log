# Failure Pattern Index

This index catalogs recurring **rule-level failure patterns** observed across smart contract systems.
Each pattern abstracts a structural failure mode that can manifest across multiple protocols and implementations.

---

## Pattern List

### Pattern-001 — Delegatecall Storage Collision

**Category:** Architectural / Rule-Level
**Summary:**
Stateful contracts using `delegatecall` to external strategy logic risk critical storage collisions when caller-owned packed variables (e.g. owner, paused flags) share slots with callee-declared state.

**Keywords:**
`delegatecall`, `storage collision`, `state corruption`, `paused`, `owner`, `strategy pattern`, `DeFi architecture`

**File:**
[`Pattern-001-Delegatecall-Storage-Collision.md`](./Pattern-001-Delegatecall-Storage-Collision.md)

---

## How to Use This Index

* Use this index as a **map of known failure structures**
* Reference pattern IDs (e.g. `Pattern-001`) in:

  * Bug bounty reports
  * Audit notes
  * Design reviews
* New patterns should be appended following the same format

---

*Maintained by Invaribreak*
