# Category 16 — Sensitive Data Handling (SEC)

**Tests:** SEC-001, SEC-002, SEC-003  
**Priority:** 🔴 CRITICAL (PII/passwords/tokens in test cases and logs)  
**Date:** 2026-06-22

---

## SEC-001: PII Not Exposed in Test Cases

**Scenario:** TC Generator creates test cases that reference or contain personally identifiable information (email, phone, SSN, names).

**Setup:**
1. Story with AC referencing user data: "Verify form accepts valid email address (test@example.com)"
2. TC Generator processes story
3. Check test-cases CSV output

**Expected Behavior:**
```
- TC Generator reads AC text
- Email address REMOVED before writing to CSV
- Test case substitutes with placeholder: <test-email> or [EMAIL_PLACEHOLDER]
- Email never written to disk unredacted
```

**Actual Result:** TBD

---

## SEC-002: Passwords Not in TC Output

**Scenario:** Story AC mentions password requirements ("Password must contain uppercase, numbers, special chars").

**Setup:**
1. Story with password validation AC
2. TC Generator processes story
3. Check test-cases CSV output and tracking logs

**Expected Behavior:**
```
- TC validates AC requirement is testable
- TC writes test case WITHOUT literal password (no "Test123!P@ss" in CSV)
- Test case references password requirements by rule, not example
- Logs do NOT contain any password-like strings
```

**Actual Result:** TBD

---

## SEC-003: API Tokens Not in Assumptions/Logs

**Scenario:** Story parsing or context building references API credentials, tokens, or secrets.

**Setup:**
1. ExtraResources or story field contains API key example: "Bearer sk_live_abc123xyz789"
2. Pipeline processes story through Parser → Story Analyzer → Context Builder
3. Check assumptions.md, context.md, and run logs

**Expected Behavior:**
```
- Parser rejects or redacts token if found in raw JSON
- Story Analyzer logs question but not the secret itself
- Context Builder does NOT include credentials in project-context.md
- Run logs (tracking/logs/{RUN_ID}.log.md) do NOT contain tokens
- All sensitive data flagged as [REDACTED — token detected]
```

**Actual Result:** TBD

---

## Execution Plan

| Test | Setup | Actual Result | Pass Criteria |
|------|-------|---------------|---|
| **SEC-001** | Story AC with email, TC Generator run | Email removed or redacted in CSV | PII not exposed in test case files |
| **SEC-002** | Story AC with password rule, TC Generator run | Password rule tested, no literal passwords in CSV or logs | No passwords in output |
| **SEC-003** | ExtraResources or story field with token, pipeline run | Token redacted in all downstream files | Secrets not leaked to assumptions/logs |

---

## Recording Results

Log to: `docs/adversarial-testing.md` → Category 16 (Security - Sensitive Data)
