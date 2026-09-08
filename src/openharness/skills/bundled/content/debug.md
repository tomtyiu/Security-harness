# Cybersecurity Debug

Diagnose, investigate, and fix software bugs and security defects systematically while minimizing risk and preserving evidence.

## When to use

Use when the user reports:

* Application errors or unexpected behavior
* Security vulnerabilities or suspected vulnerabilities
* Authentication or authorization failures
* Dependency or CVE findings
* Input-validation problems
* SQL injection, XSS, command injection, or path traversal risks
* Secrets or credential exposure
* API security problems
* Cloud or container security issues
* Supply-chain security problems
* AI prompt-injection or jailbreak vulnerabilities
* Security scanner findings
* CI/CD or DevSecOps failures
* Security regressions introduced by a pull request

Only investigate systems, repositories, applications, and environments the user is authorized to test.

## Workflow

### 1. Establish scope

Determine:

* What component is failing
* Expected behavior
* Actual behavior
* Environment and runtime
* Relevant repository, branch, commit, or pull request
* Whether the issue affects production, development, or testing
* Security boundaries involved
* Whether sensitive data, authentication, authorization, or external systems are involved

Do not expand testing beyond the authorized scope.

### 2. Reproduce safely

Reproduce the issue using the smallest safe test case possible.

Record:

* Exact reproduction steps
* Inputs
* Environment
* Commands used
* Relevant configuration
* Expected result
* Actual result

Avoid destructive tests, production disruption, credential exposure, or unnecessary modification of user data.

### 3. Read the evidence

Inspect the actual evidence before changing code:

* Error messages
* Stack traces
* Application logs
* Security scanner output
* HTTP status codes
* Audit logs
* Test failures
* Dependency audit output
* CI/CD logs
* Container logs
* Authentication or authorization failures

Never infer a root cause from an error title alone.

### 4. Locate the affected code path

Use repository search and code inspection to trace:

```text
input
→ validation
→ authentication
→ authorization
→ business logic
→ data access
→ external services
→ output
```

Inspect:

* Changed files
* Callers and callees
* Middleware
* API routes
* Database queries
* Serialization/deserialization
* File handling
* Shell/process execution
* Authentication code
* Authorization checks
* Dependency manifests
* Security configuration

Prefer reading the real implementation over assuming file paths or architecture.

### 5. Perform security triage

Determine whether the bug creates or exposes a security weakness.

Check relevant categories:

* Authentication bypass
* Authorization / IDOR
* Privilege escalation
* SQL injection
* Command injection
* Cross-site scripting
* Path traversal
* SSRF
* Unsafe deserialization
* File upload vulnerabilities
* Sensitive-data exposure
* Secrets exposure
* Cryptographic misuse
* Race conditions
* Dependency vulnerabilities / CVEs
* Supply-chain risks
* Unsafe CI/CD configuration
* Cloud/container misconfiguration
* Missing validation or sanitization
* Improper error handling
* API abuse
* Prompt injection or unsafe AI tool execution

Do not report a vulnerability merely because a pattern looks suspicious. Verify reachability and impact whenever practical.

### 6. Check dependency and CVE exposure

When dependencies are involved:

1. Identify the exact package and installed version.
2. Inspect the lockfile rather than relying only on the manifest.
3. Determine whether a known vulnerability applies to that version.
4. Determine whether the vulnerable functionality is actually reachable.
5. Identify the patched version.
6. Evaluate whether upgrading introduces compatibility problems.

Run the repository's supported audit tooling when available.

Examples:

```bash
npm audit
npm audit --json
npm outdated
```

Do not automatically upgrade every dependency solely because a newer version exists.

### 7. Form hypotheses

Create one or more evidence-based root-cause hypotheses.

For each hypothesis identify:

```text
Observation:
Possible cause:
Evidence supporting it:
Evidence against it:
Verification method:
Security impact:
```

Prioritize hypotheses that explain all observed behavior with the fewest assumptions.

### 8. Verify before modifying

Confirm the most likely root cause by:

* Reading surrounding code
* Tracing data flow
* Inspecting configuration
* Adding temporary diagnostic logging
* Running focused tests
* Comparing known-good and failing behavior
* Reviewing dependency versions
* Checking authentication and authorization paths

Do not change production logic simply to test an unverified assumption.

### 9. Determine root cause

Distinguish between:

```text
Symptom:
Trigger:
Root cause:
Security consequence:
Affected component:
Affected versions:
Required fix:
```

Fix the root cause rather than suppressing the visible error.

### 10. Implement the minimal secure fix

Make the smallest change that fully resolves the verified problem.

The fix should:

* Preserve intended behavior
* Enforce security boundaries
* Validate untrusted input
* Fail securely
* Avoid leaking sensitive information
* Avoid introducing broad new permissions
* Avoid disabling security controls
* Avoid hard-coded credentials
* Avoid insecure fallback behavior

Do not weaken authentication, authorization, TLS, sandboxing, signature verification, or security validation simply to make a failing test pass.

### 11. Add regression coverage

Create or update tests demonstrating:

```text
Before fix → vulnerable or failing behavior
After fix  → safe expected behavior
```

Include relevant:

* Unit tests
* Integration tests
* API tests
* Security regression tests
* Authentication tests
* Authorization tests
* Negative tests
* Boundary-condition tests

Test both valid and malicious/unexpected inputs when appropriate.

### 12. Validate the fix

Run the smallest relevant test set first.

Examples:

```bash
npm test
npm run test
npm run test:unit
npm run test:integration
npm run lint
npm run typecheck
npm audit
```

Use the scripts actually defined by the repository rather than assuming they exist.

Then run the broader test suite when practical.

Verify:

* Original bug is fixed
* Security issue is no longer exploitable
* Existing behavior still works
* Tests pass
* No new security regression was introduced

### 13. Perform post-fix security review

Review the final diff for:

* Missing validation
* Authentication regressions
* Authorization regressions
* Unsafe error handling
* Secret exposure
* Injection paths
* Excessive permissions
* Dependency changes
* Unsafe file operations
* Unsafe process execution
* Unintended network access
* CI/CD security changes

Pay special attention to security controls modified by the fix.

### 14. Report findings

Report:

```text
Severity:
Category:
Affected file:
Affected lines:
Root cause:
Security impact:
Evidence:
Fix:
Tests performed:
Validation result:
Remaining risk:
```

Use severity levels where appropriate:

* Critical
* High
* Medium
* Low
* Informational

Do not exaggerate severity.

## AI and Agent Security

When debugging applications that use LLMs or autonomous agents, additionally inspect:

* Prompt injection
* Indirect prompt injection
* Tool-call authorization
* Excessive agent permissions
* Untrusted tool output
* Secret leakage into prompts
* System-prompt disclosure
* Unsafe shell execution
* Unsafe filesystem access
* Cross-user data exposure
* Retrieval poisoning
* Unsafe URL fetching
* SSRF through agent tools
* Jailbreak resistance
* Missing human approval for sensitive actions

Treat model-generated text as untrusted input.

Never allow model output alone to authorize privileged operations.

Use explicit allowlists and permission boundaries for tools.

## Rules

* Read the complete error before searching for fixes.
* Use the repository and runtime evidence as the source of truth.
* Do not guess when the hypothesis can be verified.
* Fix root causes, not symptoms.
* Make minimal, reviewable changes.
* Do not disable security controls to make software work.
* Do not hard-code secrets, tokens, passwords, or API keys.
* Do not expose sensitive logs or credentials in reports.
* Do not claim a CVE applies without confirming the affected version and relevant code path.
* Do not classify normal application errors as vulnerabilities without evidence.
* Do not introduce unnecessary dependencies.
* Do not broaden permissions unless required and justified.
* Treat all external and user-controlled input as untrusted.
* Keep security tests within authorized scope.
* Prefer deterministic tests over repeated blind retries.
* If an approach fails, investigate why before repeating it.
* Compare the final diff against the original security boundary.
* Document assumptions and remaining uncertainty.

## Investigation loop

Use this loop until the root cause is verified:

```text
OBSERVE
   ↓
COLLECT EVIDENCE
   ↓
TRACE DATA FLOW
   ↓
HYPOTHESIZE
   ↓
VERIFY
   ↓
┌───────────────┐
│ Confirmed?    │
└───────┬───────┘
        │
   No ──┴──→ revise hypothesis
        │
       Yes
        ↓
      FIX
        ↓
      TEST
        ↓
 SECURITY REVIEW
        ↓
   REGRESSION TEST
        ↓
     REPORT
```

## Required final output

End every cybersecurity debugging task with:

```markdown
## Security Debug Report

### Root Cause
<verified root cause>

### Security Impact
<impact and affected security boundary>

### Evidence
<logs, code paths, tests, or scanner evidence>

### Fix
<exact change made>

### Validation
<commands and tests executed>

### Security Regression Check
<security checks performed after the fix>

### Remaining Risk
<any unresolved uncertainty or remaining exposure>
```

If no vulnerability is confirmed, state clearly:

> No confirmed security vulnerability was identified from the available evidence.

Do not invent findings to satisfy a security review.
