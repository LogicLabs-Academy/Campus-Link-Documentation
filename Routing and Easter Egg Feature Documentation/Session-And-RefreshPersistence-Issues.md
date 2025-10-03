# Sesssion and Refresh Persistence Documentation

This document explains the problems and solutions to session persistence issues that were faced in the CampusLink Project which includes all related issues to Login Memory, Unwanted reload loops & autologout on close tabe or close window. This issue was and is still one of the most frequently face issues among devs, so it has been designed in a detailed step-by-step process with code snippets where neccessary.

---

# Issue 1 - User Logs Out On Refresh

## Narrative

After Logging In successfully, if the user refreshed the page (F5), the session was erased and the app redirected user back to login.

## Possible Cause

- Tokens were stored only in React state (ie. `useState`), not in persistent storage.
- On reload, Browser refresh clears runtime state if not saved in a persistent storage.
- Uncentralized token and `ProtectedRoutes` not checking `localstorage` for tokens.

## Step-by-step Solution

### 1. Store **JWT accessToken, refreshToken and UserType** in `localstorage` on login.

```jsx
//AdminLogin.jsx and StudentLogin.jsx
// On successful login
localStorage.setItem("accessToken", response.data.accessToken);
localStorage.setItem("refreshToken", response.data.refreshToken);
localStorage.setItem("userType", response.data.userType);
```

### 2. Ensure ProtectedRoutes checks localstorage, not just `useState`.

### 3. Add a hook (`useAuth`) to centralize token handling.

```js
//routes/auth.js
export function useAuth() {
  const token = localStorage.getItem("accessToken");
  const refreshToken = localStorage.getItem("refreshToken");
  const userType = localStorage.getItem("userType");
  return { token, refreshToken, userType };
}
```

---

## What We Implemented

- Moved from **state-only** to **localstorage-based authentication persistence**.
  -Wrapped routes in `ProtectedRoutes`.

## Future Improvements

- Use **httpsOnly cookies** instead of localstorage for better security.
- Add automatic refresh with refreshToken instead of forcing relogin.

## What We Learned

Refresh Persistent is a mandatory part or feature in any SPA-based Authentication application that must be checked to ensure seemless flow in the application.

---

## Verification

- Login, refresh page => still logged in.
- Close and reopen browser => still logged in.

## Troubleshooting Details

- If persistence fails, confirm `localstorage` keys match exactly (`"accessToken"`, `"refreshToken"`, `"userType"`).
- Check that login response contains tokens before saving.

## References

- Refer to [Protected Routes and Related Issues.md](./ProtectedRoutes-Related-Issues.md) on issues concerninng failed login state or incorrected dashboard.

---

# Issue 2 - Refresh Loop (Page Keeps Reloading)

## Narrative

On refresh, instead of staying Logged In, the app would sometimes enter and infinite redirect loop or reload unexpectedly.

---

## Possible Causes

- ProtectedRoute misconfiguration (checking before `localStorage` was populated).
- Typo error in code.
- Using `navigate()` inside render phase, causing re-renders.

## Step-by-step Solutions

### 1. Ensure ProtectedRoutes runs **after localStorage check**, not in render phase.

```jsx
//ProtectedRoutes.jsx
React.useEffect(() => {
  const token = localStorage.getItem("accessToken");
  const userType = localStorage.getItem("userType");
}, [allowedRoles]);
```

### 2. Wrap redirects in conditions only when token is missing.

```jsx
//ProtectedRoutes.jsx
React.useEffect(() => {
  const token = localStorage.getItem("accessToken");
  const userType = localStorage.getItem("userType");

  if (!token) {
    setIsValid(false);
  } else if (allowedRoles.includes(userType)) {
    setIsValid(true);
  }
  setLoading(false);
}, [allowedRoles]);
```

### 3. Add a **loading state** while checking auth.

```jsx
//ProtectedRoutes.jsx
if (loading) return <p>Loading...</p>;
if (!isValid) return <Navigate to="/login" replace />;
return children;
```

### 4. Check for any Uncaught Typo erros in your code that could cause such errors.

Example snippet:
Final Code

```jsx
//ProtectedRouutes.jsx
export default function ProtectedRoute({ children, allowedRoles }) {
  const [loading, setLoading] = React.useState(true);
  const [isValid, setIsValid] = React.useState(false);

  React.useEffect(() => {
    const token = localStorage.getItem("accessToken");
    const userType = localStorage.getItem("userType");

    if (!token) {
      setIsValid(false);
    } else if (allowedRoles.includes(userType)) {
      setIsValid(true);
    }
    setLoading(false);
  }, [allowedRoles]);

  if (loading) return <p>Loading...</p>;
  if (!isValid) return <Navigate to="/login" replace />;
  return children;
}
```

---

## What We Implemented

- Added `loading` state to prevent redirect race conditions.
- Checked and corrected any typo errors in the code.
- Wrapped ProtectedRoutes in `useEffect()` not `navigate()` to avoid rerenders.

## Future Improvements

- Implement backend token validation before finalizing truth for added security.
- **If possible**, implement extension for autocheck or autocorrect keyword typos.

## What We Learned

- Never trigger `navigate()` inside render directly. Always check inside `useEffect()`
- Always check for Uncaught typos

---

## Verification

- Refresh once => no reload loop.
- Multiple refreshes => stable dashboard.

## Troubleshooting Details

- If infinite redirect occurs, check if `navigate()` is rendered unconditionally or directly inside.

## References

- Checkout [Mozilla Documentation]() concerning Persist reload issues.

---

# Issue 3 - `Remember Me` Option Not Working

## Narrative

Added a `remember-me` checkbox on login, but users still got logged out after closing the browser even when tokens where stored.

## Possible Cause

- Tokens were saved in `sessionstorage` instead of `localstorage`
- The checkbox wasn't tied to a storage mechanism even when tokens where stored correctly.

## Step-by-step Solution

### 1. Add a `useEffect` and `useState` to track if the checkbox was checked.

### 2. On login, check if **Remember Me** is true:

- Save tokens in `localStorage` (persistes after closing browser).
- Else, save tokens in `sessionStorage`(clears on close).

Example snippet:

```jsx
if (rememberMe) {
  localStorage.setItem("accessToken", token);
  localStorage.setItem("userType", userType);
} else {
  sessionStorage.setItem("accessToken", token);
  sessionStorage.setItem("userType", userType);
}
```

## What We Implemented

A **Remember Me** toggle that switches between session and localStorage.

## Future Improvements

Allow customizable session timeout(eg. 1hour, 24hours). Maybe an .env file with `refreshToken_Expires_In`check.

## What We Learned

Having a smooth seemless application is as crucial as the security and usage of the application. Users hate being logged out unexpectedly or beinng asked to login every single time especially if the apllication was a productivity app that the users user almost everyday.
**User Experience matters.**

---

## Verification

- With `RememberMe` checked => User stays logged in after browser restarts.
- With `RememberMe` unchecked => Logouts out as soon as browser closes.

## References

- Refer to [Why Ux matters in modern day applications]() by Therms
- Refer to [Docs by Mozilla ]() for more research.

---

# Issue 4 - Backend/ Frontend Sync

## Narrative

Sometimes the frontend thought the user was logged in, but backend rejected API requests with **Error 401 - Unauthorised**

## Possible Cause

- Expired token stored in `localStorage` without refresh logic.
- Frontend never tried to renew access token.

## Step-by-step Solution

### 1. Store both `accessToken` and `refreshToken` correctly in `localStorage`.

Example snippet:

```jsx
//AdminLogin.jsx and StudentLogin.jsx
// On successful login
localStorage.setItem("accessToken", response.data.accessToken);
localStorage.setItem("refreshToken", response.data.refreshToken);
localStorage.setItem("userType", response.data.userType);
```

### 2. Create and Axios Interceptor:

- Attach `accessToken` to every request.
- If **Error 401** occurs, use RefreshToken to get a new `accessToken`.
  Example snippet:

```js
//server.js
import axios from "axios";

const api = axios.create({ baseURL: "http://127.0.0.1/api" });

api.interceptors.request.use((config) => {
  const token = localStorage.getItem("accessToken");
  if (token) config.headers.Authorization = `Bearer ${token}`;
  return config;
});

api.interceptors.response.use(
  (res) => res,
  async (err) => {
    if (err.response.status === 401) {
      const refresh = localStorage.getItem("refreshToken");
      const newToken = await getNewAccessToken(refresh);
      localStorage.setItem("accessToken", newToken);
      err.config.headers.Authorization = `Bearer ${newToken}`;
      return api(err.config);
    }
    return Promise.reject(err);
  }
);
```

## What We Implemented

Added Axios Interceptors to keep tokens in sync with backend.

## Future Improvement

- Add backend logout endpoint to blacklist refresh tokens.

## What We Learned

A persisted token alone isn't enough for security, refresh logic is critical.

---

## Verification

- Api requests remain valid after long sessions
- Expired `accessTokens` auto-refreshes.

## Troubleshooting Details

If requests keep failing:

- Check backend refresh endpoint.
- Check if tokens are stored and accessed correctly

## References

Refer to [API endpoint Issue Documentation.md](./) for issues concerning failed API endpoint communication.
