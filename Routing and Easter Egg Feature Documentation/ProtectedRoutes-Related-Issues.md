# Protected Routes Related Issues.md

This document details the issues, debugging process, and solutions related to implementing Protected Routes in the CampusLink project.

---

---

# Issues 1 - Unauthorized access to Admin/Student Pages

## Narrative

## Initially, both students and admins could access each other’s dashboards by simply typing or pasting the URL (e.g., `/admin/dashboard` or `/student/dashboard`). This was a major security flaw.

## Possible Cause

- **React Router** allowing direct navigation to page if someone knew the url.
- Routes not guarded with proper authentication & role validation checks.

---

## Step-by-step Solution

### 1. Implement a `ProtectedRoute` component that wraps private routes.

### 2. Generate a valid accessToken that would be stored in localStorage when logged in.

```js
//Backend .routes/auth.js
const generateTokens = (user) => {
  const payload = { id: user.id, type: user.type };
  const accessToken = jwt.sign(payload, process.env.JWT_SECRET, {
    expiresIn: process.env.JWT_EXPIRES_IN || "1h",
  });
  const refreshToken = crypto.randomBytes(32).toString("hex");
  return { accessToken, refreshToken };
};
```

### 3. In the `Protected Routes.jsx`, confirm the stored userType through the sored token in localstorage (e.g., "admin" or "student").

If the user’s type doesn’t match the required route, redirect them to their correct dashboard.
Example snippet:
In `ProtectedRoutes.jsx`

```jsx
// Example ProtectedRoute
export default function ProtectedRoute({ children, allowedRoles }) {
  const token = localStorage.getItem("accessToken");
  const userType = localStorage.getItem("userType");

  if (!token) return <Navigate to="/login" replace />;
  if (!allowedRoles.includes(userType)) {
    return <Navigate to={`/${userType}/dashboard`} replace />;
  }
  return children;
}
```

In `App.jsx`

```jsx
<Route
 path="/dashboard/admin"
 element={
   <ProtectedRoute role="admin">
     <AdminDashboard />
   </ProtectedRoute>
 }
/>

<Route
 path="/dashboard/student"
 element={
   <ProtectedRoute role="student">
     <StudentDashboard />
   </ProtectedRoute>
 }
/>
```

---

## What We Implemented

- We wrapped each of the different role based path in a ProtectedRoute components which ensures acces is role-based
- We stored our accesstokens in localstorage.

## Future Improvements

- Move from localStorage to httpOnly cookies for better security.
- Use JWT role claims instead of storing role in localStorage.

## What We Learned

- **ProtectedRoutes** are the backbone of role seperation; without them, security is superficial.
- Storing role based accesstokend or JWT token in httpOnly or localstorage adds another level of security.

---

## Verification

- Admin cannot access student pages unless the role is checked or accestoken is present.
- Students cannot access Admin Page.
- Unauthorized Users get redirect to /login (ie **student login**)

## Troubleshooting Details

If routes keep refreshing ensure:

- `useType` is saved at login
- **ProtectedRoute** checks the same naming convention ( `"admin"` vs `"student"`)

## References

- [React Router Protected Routes Docs](https://reactrouter.com/en/main/start/tutorial)
- [CampusLink Project Implemetation](https://github.com/LogicLabs-Academy/Campus-Link-Backend/src/routes)

---

# Issue 2 - Refresh Causing Logout, Route reset or Error 404 on deployment

## Narrative

On refreshing the browser, users were sometimes redirected to `/login` on development and **Error 404: Not Found** on deployment even when they had a valid token.

## Possible Cause

- `localstorage` or session handling wasn't persisted correctly
- The app reloaded before ProtectedRoute could validate the token
- Web server or hosting service wasn't set to handle client-sided routing.

---

## Step-by-step Solution

### 1. Add a fallback route to enable ReactRouter take over and handle all route request

**Note this should be added at the last after app other routes**

```js
//server.js
const path = require("path");
// ... other routes

app.get("*", (req, res) => {
  res.sendFile(path.resolve(__dirname, "build", "index.html"));
});
```

### 2. On login, store both `accessToken`, `refreshToken`and `userType` in `localstorage`.

### 3. Modify `ProtectedRoutes.jsx` to check for these tokens before redirecting

```jsx
//ProtectedRoutes.jsx
const token = localStorage.getItem("accessToken");
const refresh = localStorage.getItem("refreshToken");
if (!token || !refresh) navigate("/login");
```

### 4. Use a small "loading" state while verifying token existence

---

## What We Implemented

- Added a fallback route handle issue of 404 on deployment.
- Made sure protectedroutes checked for presence of tokens before redirecting during reloads.

## Future Improvement

- Add silent refresh mechanism with refresh tokens.
- Validation not just checking of accessToken before redirecting.
- Possibly move to using **vercel.json** or **Nginx** for handling Client-side routing.

## What We Learned

## Persistence is crucial: without the proper use of `localstorage` and `rememberMeTokens` ie (`refreshTokens`), sessions and seemless user experience breaks all together.

## Verification

-If `rememberMe` choice was checked, close browser and reopen: Still Logged in .

- Refresh doesn't force logout.

## Troubleshooting Details

- If always redirected, check if `localstorage.clear()` is called anywhere
- Ensure API BASE URL is correct (`http://127.0.0.1/api`)

## References

- [Stack Overflow Question and Answer Forum](https://stackoverflow.com/questions/63462828/404-error-on-refresh-with-spa-react-router-app-in-github-pages)
- [Github Facebook Support Forum](https://github.com/facebook/react/issues/26669)

---

# Issue 3 - Role-based URL Seperation

## Narrative

Single `/dashboard` route was used for both roles, making it confusing. Needed a way to seperate **Admin** vs **Student** dashboard URLs. Example

- Admin => `/MOUAU1d3%ere/dashboard`
- Student => `/#rdc12133-DC/dashboard`

## Possible Cause

- Lack of Id added in **path** in App.jsx
- Both URLs linking to same `/dashboard` on login.
- ProtectedRoute not validating URL befoore granting access

## Step-by-step Solution

### 1. Create two seperate `dashboard.jsx` =>

- `AdminDashboard.jsx`
- `StudentDashboard.jsx`

### 2. Assign Ids in route path in `App.jsx`

```jsx
<Route
  path="/MOUAU123ere)/dashboard"
  element={
    <ProtectedRoute allowedRoles={["admin"]}>
      <AdminDashboard />
    </ProtectedRoute>
  }
/>
<Route
  path="/#rdc12133-DC/dashboard"
  element={
    <ProtectedRoute allowedRoles={["student"]}>
      <StudentDashboard />
    </ProtectedRoute>
  }
/>
```

### 3. Updated `ProtectedRoutes.jsx` to validate before granting access.

```jsx
//App.jsx
<ProtectedRoute allowedRoles={["admin"]}>
  <AdminDashboard />
</ProtectedRoute>
```

---

## What We Implemented

- We added the unique identifiers to the route path to each dashboard for cleaner URL seperation.
- We updated ProtectedRoute to validate the role before acces to each dashboard.
- Had two dashboards: each for each role.

## Future Improvement

- Generate random session identifiers per logijn instead of static ones.
- Integrate with backend session validation.

## What We Learned

Custom URL identifiers doesn't really act as a real security approach especially with static imputed URL Id prefixes. Rather it acts as an easy role identifier through URLs. Randomingly generating the ID, linking the ID to session tokens and validating can act as some layer of security instead.

## Verification

- Direct copying and Pasting of URL fails if `userType` is not matched.
- Logging In is now seemless with role-defined dashboards and access.

## Troubleshooting Details

- If always redirected to `/dashboard` or `/login` even after userType token is valid, ensure that the route path is updated and the dashboard components are imported correctly.

## References

- Refer to the documentation on [Authorization-guide.md](../Project%20Conception/Authorization-guide.md) for codes snippets on generating unique Ids as prefixes to `/dashboard`.
- Also refer to documentation on [Session And RefreshPersistence Issues.md](./Session-And-RefreshPersistence-Issues.md)

---
