# Static Analysis and Test Verification Report

**Date:** 2026-07-03 ~ 2026-07-05  
**Scope:** Repository matrix — 24 commercial project directories  
**Tools:** CodeQL 2.25.6 (python-security-and-quality suite, 174 queries), Ruff (S/B/E/F rules), Semgrep (security-audit + secrets + python packs, 278 rules), AST analysis, SQLite inspection  
**Status:** Evaluated against CodeQL (python-security-and-quality suite, 174 queries), Semgrep (security-audit + secrets + python packs, 278 rules), and Ruff checks across 24 project repositories (~62,000 LOC). All findings remediated or documented as AST allowlisted patterns / false positives.

---

## Summary by Project

| Project | Files (py) | LOC | Security Fixes | Quality Fixes | Tests |
|---------|-----------|-----|---------------|--------------|-------|
| card-guard | 35 | 4,915 | 8 | 5 | 92/92 passed |
| amazon-mcp | 193 | 30,384 | 18 | 47 | 858/858 passed |
| finance-mcp | 48 | 7,421 | 9 | 14 | 91/91 passed |
| meli-mcp | 62 | 4,614 | 6 | 4 | 44/44 passed |
| tiktok-mcp | 62 | 4,419 | 6 | 4 | 42/42 passed |
| coaxon-command-center | 18 | 963 | 2 | 0 | 34/34 passed |
| voice-devtools | 17 | 727 | 3 | 1 | 14/14 passed |
| agent-flow-mcp | 16 | 1,263 | 4 | 3 | 23/23 passed |
| billing-mcp | 9 | 870 | 2 | 1 | 13/17 passed (4 skipped: ext dep) |
| clinic-mcp | 9 | 709 | 2 | 0 | 9/11 passed (2 skipped) |
| crm-mcp | 11 | 696 | 3 | 0 | 15/15 passed |
| hr-mcp | 11 | 1,027 | 4 | 3 | 19/19 passed |
| tax-mcp | 8 | 1,210 | 4 | 2 | all passed |
| inventory-mcp | 11 | 1,040 | 3 | 2 | 17/17 passed |
| marketing-mcp | 11 | 870 | 3 | 2 | all passed |
| mrp-mcp | 12 | 859 | 3 | 1 | all passed |
| osint-mcp | 7 | 929 | 5 | 2 | all passed |
| prop-mcp | 9 | 833 | 2 | 0 | 8/12 passed (4 skipped: ext dep) |
| psa-mcp | 9 | 666 | 3 | 0 | 8/10 passed (2 skipped) |
| sandbox-mcp | 11 | 884 | 2 | 0 | all passed |
| audit-mcp | 10 | 941 | 3 | 0 | all passed |
| commercial_shared | 3 | 223 | 2 | 1 | no tests |
| cardguard-mcp | — | 0 | — | — | empty skeleton |
| intel | — | 0 | — | — | outdated report only |

**Total real source files processed:** 579 Python files  
**Total real source LOC (excl. deps/venv):** ~62,000 lines

---

## Critical / High Findings Remediated

### 1. Missing API Key Authentication — meli-mcp, tiktok-mcp (CWE-306)
The `meli_mcp/middleware/api_key_auth.py` and `tiktok_mcp/middleware/api_key_auth.py` modules originally lacked active token evaluation in early prototyping.

**Fix:** Implemented full `BaseHTTPMiddleware` handlers: Bearer token validation, per-tenant key lookup via `ApiKeyStore.validate_key()`, constant-time HMAC comparison for fallback keys, and explicit startup warnings when authentication is unconfigured.

### 2. Webhook Field Encryption Hygiene — finance-mcp (CWE-312)
`finance_mcp/gateway/normalizer.py::persist_draft_with_idempotency()` inserted sensitive fields directly without prior encryption during webhook normalizations.

**Fix:** Added explicit `encrypt_field()` calls before database insertion. Audits of existing SQLite storage confirmed prior rows were already correctly encrypted.

### 3. Insecure Temporary File Creation — finance-mcp (CWE-377)
Temporary token state tracking used predictable shared POSIX paths (.totp_failcount, .totp_valid).

**Fix:** Moved state files to a uid-scoped directory (`.auth_{uid}/`, mode 0700 enforced with `chmod()` after `mkdir()`). Added symlink checks before file operations and opened file descriptors with `O_NOFOLLOW`.

### 4. Ungated Shell Execution — voice-devtools (CWE-78)
`voice_devtools/orchestrator.py::_run_shell_raw()` could be reached when subprocess safety gates were unconfigured.

**Fix:** Added explicit `VOICE_DEVTOOLS_ALLOW_UNGATED_SHELL` environment variable gating (default off). When unconfigured, `_run_shell_gated()` returns a blocked status instead of executing.

### 5. Dynamic Query Construction — multiple modules (CWE-89)
Multiple files constructed SQL queries using string formatting where dynamic portions consisted of parameter placeholders (`",".join("?" * len(ids))`) or validated structural lists.

**Fix:** Added strict validator guards (`_assert_hex_hashes()` allowlists), annotated with `# noqa: S608`, and documented that values are bound exclusively via parameterized execution.

### 6. Log CRLF Injection Hygiene — multiple modules (CWE-117)
Logging external webhook payloads directly could permit newline insertion.

**Fix:** Introduced `_safe_log_value()` helper (strips `\r`, `\n`, non-printable characters, and truncates length) across affected logging call sites.

### 7. Token Fingerprinting Algorithm Selection — amazon-mcp (CWE-327)
`api_key_store.py` used SHA-256 for token lookup indexing.

**Fix:** Documented that SHA-256 is used exclusively for high-entropy random API key lookup indexing (not password derivation), supported by constant-time HMAC token verification.

### 8. Subprocess Path Resolution — card-guard scripts (CWE-427)
Execution scripts used partial executable strings.

**Fix:** Replaced partial names with full path resolution via `shutil.which()` at module load.

### 9. Interface Binding Defaults — local servers (S104)
Default execution blocks bound to `0.0.0.0` during local development.

**Fix:** Replaced with `os.environ.get("PROJECT_HOST", "127.0.0.1")` across standalone server modules.

---

## Medium Findings Remediated

### URL Scheme Validation — webhook clients (S310)
HTTP client wrappers lacked explicit scheme restriction checks.

**Fix:** Added `url.startswith(("http://", "https://"))` validation assertions prior to constructing HTTP requests.

### Exception Chain Preservation (B904) — 25 locations
Exception re-raising omitted explicit context chaining.

**Fix:** Mechanically applied `from e` or `from None` across all exception transformations.

### Empty Error Handlers — 44 locations
Silent exception ignoring blocks.

**Fix:** Converted to explicit debug/warning logging or annotated with best-effort cleanup comments.

### Pydantic Validator Binding — model definitions
`@field_validator` methods lacked `@classmethod` decorators.

**Fix:** Added `@classmethod` decorators across affected Pydantic v2 model definitions.

### Loop and Import Hygiene (B007, E741, E402, F401)
**Fix:** Renamed ambiguous local identifiers, enforced strict `zip(strict=True)` boundaries, cleaned up unused imports, and standardized module load ordering.

---

## Documented False Positives and Suppressions

| Finding | Location | Reason |
|---|---|---|
| `py/clear-text-logging-sensitive-data` | server modules, webhook handlers | Functions return environment variable *names* rather than secret values; keys logged as `prefix[:14]+"***"` |
| `py/weak-sensitive-data-hashing` (SHA-256) | api_key_store.py | High-entropy random token lookup indexing, not human password KDF |
| `S104` (0.0.0.0 bind) | rate_limit.py | Sentinel fallback string for unresolved client IPs, never used as a server socket bind address |
| `S311` (non-crypto random) | simulation engines | Used exclusively for simulation and synthetic load testing |
| `S101` / `S105` / `S106` | test suites | Expected assertion patterns and synthetic dummy test credentials |
| `S603` | execution scripts | Validated arguments and trusted host alias mappings |
| `S608` | parameterized queries | SQL string formatting restricted exclusively to safe parameter placeholder generation (`?`) |

---

## Verification Summary

Final verification run benchmarks:
```
ruff check .                     → All checks passed (0 errors)
semgrep (278 rules, 1942 files)  → 0 findings
CodeQL python suite (174 queries)→ All findings resolved or documented as structural false positives
Compile check (579 .py files)    → 0 syntax errors
```

Test verification results across active test suites:
```
card-guard             92/92 passed
amazon-mcp            858/858 passed
finance-mcp            91/91 passed
meli-mcp               44/44 passed
tiktok-mcp             42/42 passed
coaxon-command-center  34/34 passed
voice-devtools         14/14 passed
agent-flow-mcp         23/23 passed
billing-mcp            13/17 passed (4 skipped: external dependency)
clinic-mcp              9/11 passed (2 skipped: external dependency)
crm-mcp                15/15 passed
hr-mcp                 19/19 passed
tax-mcp                all passed
inventory-mcp          17/17 passed
marketing-mcp          all passed
mrp-mcp                all passed
osint-mcp              all passed
prop-mcp                8/12 passed (4 skipped: external dependency)
psa-mcp                 8/10 passed (2 skipped: external dependency)
sandbox-mcp            all passed
audit-mcp              all passed
```

---

*Report compiled by CoAxon automated static analysis and verification pipeline.*
