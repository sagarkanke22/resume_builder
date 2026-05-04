# ROUTE DELETION CHECKLIST
# Triggered when: deleting any backend route file or removing a router.include_router() line

## MANDATORY — run ALL steps before deleting

### STEP 1 — Extract every endpoint path from the file
grep all @router.get / @router.post / @router.put / @router.delete decorators.
List every path string found.

### STEP 2 — Check frontend for each path
For every path found in Step 1, run:
  grep -rn "<path>" docker/services/product-client/src --include="*.js" --include="*.jsx" --include="*.ts" --include="*.tsx"

### STEP 3 — Evaluate results
- If ANY path has frontend hits → STOP. Do NOT delete. Report to user:
  "Endpoint <path> is still called from <file>:<line>. Deleting will break <feature>."
  Ask user to confirm before proceeding.

- If ALL paths have zero frontend hits → safe to delete. Proceed.

### STEP 4 — After deletion, verify import removed from base_route.py
  grep -n "<route_module_name>" api/routes/base_route.py
  Must return zero results before marking step complete.

## LESSON LEARNED
saml_login_route.py was deleted on 2026-05-01 without this check.
GET /login/get-config was still called on every login page load.
Result: full production outage, emergency revert (PR #529).
