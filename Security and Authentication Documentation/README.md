# Security Documentation- detailed diary style documentation

This repository markdown file contains all related security issues and fixs associated with the app during development and deployment. This serves to act as your first go to documentation on issues concerning authentication, authorization, accesstoken, session IDs and other related security issues especially related to the Campus Link project.

**This markdown documentation on issues would be in the following format**

- `Issue`
- `Narrative`
- `Possible Cause`
- `Step-by-step Solution`
- `What We Implemented`
- `Future Improvement` ** If possible **
- `What We Learned`
- `Verification`
- `Troubleshooting details`
- `References`

---

# Example

## Issue: Failed AutoLogin to dashboard even after selecting the remember me feature

## Narrative

While testing "remember me" we noticed: after closing the browser and reopening, users could not be taken back to the correct dashboa...

## Possible causes

1. **Token storage had no explicit user-type or session id attached**. Storing only `accessToken`/`refreshToken` is insufficient to infer user type reliably.
2. **Used hash routes** in places (e.g. `/#id/dashboard`) leading to refresh handling issues.
3. **LocalStorage vs httpOnly cookies** — by using `localStorage` we made tokens easier to access but introduced security concerns.

---

## Step-by-Step Solutions

- Generate a **dashboardId** per session (server-generated, tied to refresh token).
  - Admin IDs start with `MOUAU` (e.g. `MOUAUAB12C3`) — makes visual inspection easier.
  - Student IDs are random (e.g. `rdc4f5a1b`) or prefixed `#rdc...` if you want the symbol.
- Persist to `localStorage`:
  - `accessToken`
  - `refreshToken`
  - `userType` (`student` | `admin`)
  - `dashboardId`
- Navigation uses **path segments** not hashes: `/<dashboardId>/dashboard` — so refresh lands on the same route and React Router can take over client-side.
- Implemented `ProtectedRoute` that:
  - verifies `accessToken` exists
  - verifies `userType` matches expected
  - verifies `sessionId` in URL equals the saved `dashboardId`

**Future**: Switch to `httpOnly` cookies (for refresh token) — recommended for production.

---

## What We Implemented

### Backend

1. On successful login, generate:
   - `dashboardId = (admin ? 'MOUAU' + randomHex(6) : randomHex(12))`
   - `refreshToken = randomBytes(32).toString('hex')`
2. Persist session in `sessions` table:
   ```sql
   INSERT INTO sessions (user_type, user_id, session_token, dashboard_id, created_at, expires_at)
   VALUES ($1, $2, $3, $4, NOW(), NOW() + INTERVAL '3 days');
   ```
3. Return to frontend
   ```json
   { "accessToken": "...", "refreshToken": "...", "dashboardId": "MOUAU..." }
   ```
   1. On success
   ```js
   localStorage.setItem("accessToken", res.data.accessToken);
   localStorage.setItem("refreshToken", res.data.refreshToken);
   localStorage.setItem("userType", "student" | "admin");
   localStorage.setItem("dashboardId", res.data.dashboardId);
   ```
   2. Navigate using React Router (not hash)
   ```js
   navigate(`/${res.data.dashboardId}/dashboard`, { replace: true });
   ```

### ProtectedRoute.jsx

    ```jsx
    // src/components/ProtectedRoute.jsx
    import { Navigate, useParams } from 'react-router-dom';
    export default function ProtectedRoute({ userType, children }) {
    const token = localStorage.getItem('accessToken');
    const storedType = localStorage.getItem('userType');
    const storedSessionId = localStorage.getItem('dashboardId');
    const { sessionId } = useParams();

    if (!token || storedType !== userType) return <Navigate to="/login" replace />;
    if (sessionId && storedSessionId && sessionId !== storedSessionId) {
    return <Navigate to="/login" replace />;
    }
    return children;
    }
    ```

---

## Future Improvements

- Move refresh token to httpOnly cookie (rotate refresh tokens on use).
- Implement device/session listing and revocation in dashboard (admin actions).
- Add rate limit and brute-force protection on login endpoints.

---

## What We Learned

- We Learnt to add Refresh Tokens to httpOnly cookies
  -...

---

## Verification

- Log in as a student: `localStorage` should contain `dashboardId`.
- Refresh the dashboard page (`/<dashboardId>/dashboard`) — you should stay logged in.
- Manually modify `dashboardId` in URL → ProtectedRoute should redirect to `login`.

## Troubleshooting details

1. Check localStorage entries with DevTools (Application -> Local Storage).
2. Verify backend returned dashboardId in login response.
3. Ensure api.js used 127.0.0.1 if backend bound to that host.
4. Inspect server logs for session insertion errors (DB constraints, duplicate keys).
5. If using httpOnly cookies in the future, ensure withCredentials and CORS credentials: true are set.

---

## References

- `[Routing Guide]](../Routing.md)`
- `[Database Setup]()`
- `[Security Fixes](../Security%20and%20Authentication%20Documentation/README.md)`

---
