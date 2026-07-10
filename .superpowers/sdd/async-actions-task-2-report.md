# Async Operations Task 2 Report

## Status

DONE

## Scope Delivered

- Operations callbacks now render from the stored `DiagnosticReport` snapshot and return before fresh diagnosis, recovery execution, or Gateway restart work begins.
- `recheck`, `confirm_repair`, and `confirm_restart` schedule tracked work only from `_AfterEofJsonResponse`; repeated clicks retain exactly-once behavior.
- Background recheck/repair/restart work uses the bounded executor path with a 12.0-second diagnostic timeout and PATCHes only the delivery owned by the completing operation.
- Repair and restart finish helpers atomically complete their original inflight record before creating a successor, preventing late workers from reclaiming delivery ownership or consuming inflight capacity.
- Failure cards retain `重新检测`; failed PATCH attempts are recorded with a safe diagnostic and operation result marker.

## RED Evidence

- `test_confirm_repair_returns_executing_card_before_fresh_evidence_and_runs_once` initially failed after adding the assertion that completed work leaves no `executing` record. This exposed the predecessor lifecycle leak; the finish helpers now call `store.complete` before successor creation.
- `test_operations_patch_failure_is_recorded_on_the_completed_card` initially failed because the assertion expected the raw fake-client error. The runtime correctly uses the existing sanitized error formatter; the test now verifies the sanitized diagnostic record instead.

## GREEN Evidence

- Each newly added core test was run alone and completed within 0.11-0.33 seconds; the restart regression completed in 0.47 seconds.
- `python -m pytest tests/integration/test_server.py -q -x`: 131 passed in 11.23 seconds.
- `python -m pytest tests/integration/test_server.py tests/unit/test_operations.py -q`: 185 passed in 11.56 seconds.
- `git diff --check`: passed.

## Commits

- `4b54f8b feat: complete operations actions in background` - Task 2 implementation and integration coverage.

## Concerns

- Cancelling an asyncio diagnostic task cannot forcibly stop a synchronous report-builder thread that is already executing. The callback and operation state remain bounded by the 12-second coroutine timeout, cleanup cancels tracked tasks/futures, and the test fixtures explicitly release their blocked builders. A genuinely uninterruptible third-party report builder could still occupy an executor worker until it returns.
- No real Feishu smoke run was performed in this task; coverage is HTTP integration plus fake-client PATCH behavior.
