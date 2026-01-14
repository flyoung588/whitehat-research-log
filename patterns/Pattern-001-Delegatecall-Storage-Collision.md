# Pattern-001: Delegatecall Storage Collision (Stateful Caller with External Logic)

**Maintained by:** Invaribreak

## Short name
Delegatecall Storage Collision

## Category
Architectural / Rule-Level Failure Pattern

## Description
When a contract that *owns its own storage layout* (often via inheritance of Initializable, Pausable, Ownable, etc.) uses `delegatecall` to execute logic in an external "strategy" or helper contract, writes performed by the callee will target the caller's storage slots. If the external logic declares its own state variables (starting from slot 0), normal writes (e.g., `totalDeposited += amount`) will overwrite the caller's critical packed storage (owner, paused flag, initialization status), resulting in permanent corruption: paused states, lost ownership, or corrupted contract configuration.

This failure is structural: it arises from mixing **independent state ownership** with **delegated execution** and does not require a malicious actor — a standard or well-intentioned strategy implementation is sufficient.

## Typical manifestations
- Protocol becomes permanently paused (whenNotPaused checks block actions).
- Admin/owner address is overwritten, losing upgrade/rescue capability.
- Core configuration (token addresses, strategy pointers) is corrupted, breaking interactions.
- Failure triggered by normal user flow (e.g., deposit), or by governance action switching to an otherwise legitimate strategy.

## Root cause
Mixing `delegatecall` with caller-side state that is not intentionally storage-aligned. The delegatecall contract assumes its own storage layout; the caller assumes different packed variables at the same slots — writes collide.

## Detection signals
- Caller contract uses `delegatecall` (or low-level functionDelegateCall) to an external, potentially stateful contract.
- Caller has inherited/packed storage (Initializable, Pausable, Ownable) with critical variables packed in low slots.
- Strategy or callee code declares state variables (especially `uint256` counters) starting at slot 0.
- Governance or upgrade paths allow swapping strategy addresses to arbitrary implementations.
- Lack of explicit slot reservation (no unused/kept slots for delegatecall safety).

## Proof-of-concept (abstract)
A strategy that declares `uint256 public totalDeposited;` as its first variable and increments it in deposit logic. When invoked via `delegatecall` from a caller that has `_owner` and `_paused` packed into slot 0, the increment writes to slot 0 and corrupts the caller's ownership/paused bits.

## Impact
- Severity: Critical
- Exploitability: High (can be triggered by non-malicious standard implementation or benign user action)
- Affected systems: Strategy-based DeFi protocols, proxy-like architectures mixing external logic and internal storage

## Mitigations
- **Avoid delegatecall for strategy execution.** Use external calls where the strategy maintains its own isolated storage.
- If delegatecall is necessary:
  - Enforce identical, explicitly aligned storage layout across caller and callee (rarely practical across independent contracts).
  - Reserve storage slots explicitly in caller for delegatecall usage (document and enforce slot layout).
  - Limit governance ability to swap in arbitrary, unverified strategy implementations without checks.
  - Use immutable/pure logic or library patterns that do not write storage (where possible).
- Prefer patterns where stateful logic resides in the contract that owns the state; keep delegated code stateless or slot-aligned.

## Related rules
- Rule-001: Incentives Override Declared Design
- Rule-002: State Ownership Must Align with Execution Context

## Examples / historical analogues
- Any system combining proxy-like execution and unaligned storage layouts that suffered state corruption when connecting unverified logic.
- General class analogues: proxy/storage mismatch vulnerabilities, delegatecall-based upgrade anti-patterns.

## Notes
This pattern emphasizes **architectural invariants**: the storage model and execution context are a single invariant — breaking it makes otherwise standard implementations dangerous.
