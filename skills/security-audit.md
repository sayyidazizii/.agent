# SKILL: AUTOMATED SECURITY & DIFF AUDITOR

Execute this mandatory verification protocol whenever reviewing, generating, or modifying code:

## STEP 1: SCOPE BOUNDARY CHECK
- Confirm that all modifications are strictly confined to the files specified in the task whitelist.
- If an unlisted file was modified, revert it immediately or pause to request explicit confirmation.
- [ ] Git State Check: Confirm NO unapproved `git commit` or `git push` was executed during this session.

## STEP 2: THREAT MODELING & VULNERABILITY AUDIT
Inspect your generated diff against the following vectors:
- [ ] File Uploads: Are uploads validated by magic bytes, renamed to UUID, and protected against Zip Slip and path traversal?
- [ ] Arbitrary Path Check: Is there any risk of Path Traversal (`../../`) on file reads, writes, or deletions?
- [ ] SSRF / Outbound Requests: Are private IP ranges (localhost, 169.254.169.254) and non-HTTPS protocols strictly blocked?
- [ ] RCE / Deserialization: Are dynamic code executions, dynamic module imports, and unsafe deserializers completely absent?
- [ ] Injection: Are all database queries and system invocations parameterized?
- [ ] Access Control: Are tenant boundaries, permissions, and roles strictly verified?
- [ ] Secrets: Verify no hardcoded secrets, tokens, or environment values are exposed in the diff.

## STEP 3: PERFORMANCE & COMPLEXITY REVIEW
- [ ] Verify Big-O time and space complexity of newly added or refactored functions.
- [ ] Ensure all file handles, streams, and database connections are safely closed in `finally` blocks or using resource managers.
- [ ] Verify that async operations have proper timeout handling.

## STEP 4: REGRESSION & EDGE-CASE AUDIT
- [ ] Safely handle null, undefined, empty strings, and empty payloads without throwing unhandled exceptions.
- [ ] Run the project's existing tests/linter if available (e.g., `npm test`, `pytest`, `cargo test`, `php artisan test`).

## STEP 5: APPEND EXECUTION LOG (MANDATORY ACTION)
Before marking the task complete, you MUST append a new single row to `.agent/logs/execution.log` using this exact table format:
`| YYYY-MM-DD HH:mm | [Task Name from task.md] | SUCCESS / FAILED | [Duration/Estimate] | PASS / FAIL | [Summary of changes or violation reason] |`

Strict logging rules:
1. Never overwrite or clear existing records in `.agent/logs/execution.log`; always append to the bottom of the table.
2. If any guardrail violation or test failure occurred, mark Status as `FAILED` and describe the root cause in the summary column.

## OUTPUT FORMAT (MANDATORY TERMINAL SUMMARY)
Before ending your response, output this verification table in your final response:
| Check | Status (Pass/Fail) | Notes |
| :--- | :--- | :--- |
| Scope Containment | Pass / Fail | Only touched whitelisted files |
| File Upload / Sanitization | Pass / N/A | Validated via magic bytes, path sandboxed |
| Arbitrary Access / SSRF | Pass / N/A | No path traversal or internal IP leakage |
| Injection Defenses | Pass / Fail | Strict parameterization used |
| Performance (Big-O) | Pass / Fail | Optimal complexity verified |
| Zero Regression | Pass / Fail | Existing functionality and tests preserved |
| Execution Log Appended | Pass / Fail | Recorded into .agent/logs/execution.log |
