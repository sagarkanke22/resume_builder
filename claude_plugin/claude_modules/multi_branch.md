# MULTI-BRANCH TASK PROTOCOL
# Triggered when: user says "apply to both branches", "do same on Dev/PROD", or names 2+ branches

## STEP 1 — List all target branches upfront
Before touching any code, confirm the full branch list with the user.
Example: "I'll apply this to: PROD_saml_fix, Dev_saml_fix. Correct?"

## STEP 2 — Apply change to Branch 1
- Checkout branch
- Make the change
- Immediately verify with grep that the change is present:
    grep -n "<added_symbol>" <changed_file>
- Do NOT proceed to Branch 2 until grep confirms Branch 1.

## STEP 3 — Apply change to Branch 2
- Checkout branch
- Make the same change
- Immediately verify with grep that the change is present.

## STEP 4 — Cross-branch parity check (MANDATORY)
After all branches done, run:
  git diff branch1 branch2 -- <changed_file>

Expected result: zero diff on the changed lines.
If diff shows the change missing on any branch → fix that branch before reporting complete.

## STEP 5 — Report
Only say "done on both branches" after Step 4 passes.

## NEVER
- Never use git stash to carry changes across branches. Stash drops silently.
  Always re-apply the change manually on each branch and verify.
- Never assume a change applied to Branch 1 is present on Branch 2.

## LESSON LEARNED
@authenticate_user was applied to save_login_config on PROD_saml_fix via stash.
Stash applied to Dev_saml_fix but the change was missing in committed state.
Discovered only when user explicitly asked to check Dev_saml_fix again.
