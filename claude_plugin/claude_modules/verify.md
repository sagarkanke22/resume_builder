# POST-CHANGE VERIFICATION PROTOCOL
# Do NOT run all tiers every time.
# Check each tier's TRIGGER line — if condition not met, SKIP that tier entirely.
# Only Tier 1 and Tier 2 always run. All others are conditional.

================================================================================
TIER 1 — DID THE CHANGE LAND? (Always run)
================================================================================

For every file edited:
  grep -n "<key symbol or string you added/removed>" <file>

Pass: grep returns the expected result.
Fail: grep returns nothing or wrong line → re-apply, do not proceed.

If multi-branch task: run this grep on EVERY branch before moving on.
Never assume a write succeeded. Never assume stash carried changes correctly.

================================================================================
TIER 2 — DID THE CHANGE BREAK ANY IMPORTS? (Always run)
================================================================================

For every file edited, check:
  - All imports at top of file still resolve (no dangling references)
  - If you ADDED an import → grep the codebase to confirm the source exists
  - If you REMOVED an import → grep the file to confirm nothing still uses it

Pass: no dangling imports.
Fail: import added whose source doesn't exist, or import removed but symbol still used → fix before proceeding.

================================================================================
TIER 3 — DID YOU REMOVE SOMETHING STILL IN USE? (Run when deleting or removing)
================================================================================

Triggers: deleting a file, removing a function, removing a route, removing an import.

For every removed symbol/path/endpoint:
  grep -rn "<removed_name_or_path>" <entire_relevant_codebase>

Scope:
  - Removing backend route → grep frontend src/
  - Removing a function → grep entire backend
  - Removing a config key → grep all files that read config

Pass: zero hits (or only hits in the file being deleted itself).
Fail: any external file still references it → STOP. Report to user before proceeding.
"<symbol> is still referenced in <file>:<line>. Removing it will break <feature>."

================================================================================
TIER 4 — SECURITY IMPACT CHECK (Run when change touches auth, routes, or middleware)
================================================================================

Triggers: adding/removing @authenticate_user, adding/removing routes, editing middleware.

Checks:
A. For every NEW endpoint added:
   → Is it guarded? If not, is there a documented reason it must stay open?

B. For every REMOVED guard:
   → Is the endpoint now publicly accessible? Is that intentional?

C. For every auth decorator added:
   → Is the import present?
   → Is the Security(auth_scheme) param in the function signature?
   → Is the decorator in the right position (below @router, above async def)?

D. For any route file change:
   → Run Tier 3 (frontend dependency scan) on all removed endpoints.

Pass: all endpoints are either guarded or have a documented justification.
Fail: any endpoint is unguarded without justification → flag to user before closing task.

================================================================================
TIER 5 — SYNTAX / PARSE CHECK (Always run on backend Python files)
================================================================================

After editing any .py file, run:
  python -m py_compile <file>

Pass: command exits with no output (no errors).
Fail: any SyntaxError or ImportError printed → fix before reporting complete.
Never assume Python is valid just because the edit looked correct.

For frontend .js / .jsx files:
  Check for obvious unclosed brackets, missing imports, or broken JSX tags by reading
  the edited section. If uncertain → flag to user.

================================================================================
TIER 6 — BEHAVIOUR REGRESSION CHECK (Run after bug fixes)
================================================================================

Triggers: any fix described as "fixing a bug", "it was broken", "wasn't working".

Steps:
1. State what the original symptom was:
   "Before fix: <describe what was broken>"

2. State what the fix changes:
   "Fix: <what was changed and why>"

3. Confirm the symptom is gone — not just that the code changed:
   - If testable via grep/logic → verify inline
   - If testable via API call or UI → instruct user to test the specific flow
   - If not immediately testable → explicitly say:
     "Cannot auto-verify. Please test: <exact steps to reproduce original symptom>"

Pass: symptom is confirmed gone OR user is given exact steps to verify.
Fail: fix applied but original symptom not addressed → reopen and investigate root cause.

Do NOT mark a bug fix complete just because the code changed.
The bug being gone is the definition of done, not the code being different.

================================================================================
TASK COMPLETE GATE
================================================================================

Only report a task as complete when:
  [ ] Tier 1 passed — change landed correctly
  [ ] Tier 2 passed — no broken imports
  [ ] Tier 3 passed — nothing removed that is still in use (if deleting)
  [ ] Tier 4 passed — no new security gaps introduced (if touching auth/routes)
  [ ] Tier 5 passed — syntax check clean (if editing .py files)
  [ ] Tier 6 passed — original symptom confirmed gone (if bug fix)

If any tier fails → state which tier, what failed, and what the fix is.
Do not silently skip a tier.
