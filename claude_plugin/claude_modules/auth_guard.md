# AUTH GUARD CHECKLIST
# Triggered when: adding @authenticate_user to any endpoint

## STEP 1 — Verify imports present in the file
The file must have ALL of:
  from api.middlewares.auth_middleware import authenticate_user
  from fastapi import ..., Security, ...
  from fastapi.security import HTTPAuthorizationCredentials, HTTPBearer
  auth_scheme = HTTPBearer(auto_error=False)

If any missing → add them before adding the decorator.

## STEP 2 — Apply in correct order
Decorator order must be:
  @router.<method>(...)     ← route decorator FIRST
  @authenticate_user        ← auth decorator SECOND
  async def <func>(request: Request, token: HTTPAuthorizationCredentials = Security(auth_scheme)):

## STEP 3 — Verify after applying
  grep -n "authenticate_user\|Security\|save-login-config" <file>
Must show all three: the import, the decorator, and the Security param.

## STEP 4 — Check it is NOT on pre-auth endpoints
Never add @authenticate_user to:
  - /login, /refresh (no token exists yet)
  - /login/get-config, /login/get-token (called before login page loads)
  - /user/generate-code, /user/validate-otp, /user/change-password (OTP flow)
  - /sse/*, /app/*/logo (browser technical constraints)

## STEP 5 — If multi-branch task → follow multi_branch.md protocol
