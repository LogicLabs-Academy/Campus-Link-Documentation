# Issue: Admin and Student Dashboard Separation

## Narrative

After solving the initial authentication problem with matric numbers, the next challenge arose:

- The system still treated **all users the same**.
- Both admin users and students could potentially access the same routes.
- This posed a serious **security** and **usability** problem: admins need special privileges, while students should be restricted to their own space.

Therefore, we decided to separate dashboards for **Admins** and **Students**.

---

## Possible Cause

- Lack of role-based access control (RBAC).
- No protected routes in frontend routing.
- No way to distinguish admin vs student sessions in the URL structure.

---

## Step-by-Step Solution

### 1. Define Clear Roles

- **Admin**: Has full control (e.g., manage users, verify students).
- **Student**: Limited to their own dashboard.

### 2. Create Two Dashboards

- **`/dashboard/admin` → Admin Dashboard**
- **`/dashboard/student` → Student Dashboard**

### 3. Protected Routes

- Implemented a `ProtectedRoute.jsx` component.
- Checks if the user’s role = "admin" or "student".
- Redirects unauthorized users automatically.

Example:

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

### 4. Session-based URLs

To further enforce separation, we updated routes to include session identifiers:

- Admin → campuslink/MOUAU123ere/dashboard
- Student → campuslink/#rdc12133-DC/dashboard
  This prevents students from simply copying admin URLs.
  Example snippet:
- Generating Unique IDs

```js
const generateDashboardId = (userType) => {
  if (userType === "admin") {
    return `MOUAU${crypto.randomBytes(4).toString("hex").toUpperCase()}`;
  } else {
    return crypto.randomBytes(8).toString("hex");
  }
};

const generateTokens = (user) => {
  const payload = { id: user.id, type: user.type };
  const accessToken = jwt.sign(payload, process.env.JWT_SECRET, {
    expiresIn: process.env.JWT_EXPIRES_IN || "1h",
  });
  const refreshToken = crypto.randomBytes(32).toString("hex");
  return { accessToken, refreshToken };
};
```

- For `Admin`

```js
const isMatch = await bcrypt.compare(password, admin.password);
if (!isMatch) return res.status(401).json({ message: "Invalid credentials" });

const dashboardId = generateDashboardId("admin");
const { accessToken, refreshToken } = generateTokens({
  id: admin.id,
  type: "admin",
});
```

### 5. Easter Egg Access for Admin Register

- To prevent exposing campuslink/admin/register, we hid it.
- Instead, admins can only access registration by clicking the logo 4 times.
- This prevents students from peeking admin routes.

Example snippet:

```jsx
import { useState } from "react";
import { useNavigate, Link } from "react-router-dom";

const handleLogoTap = () => {
  setTapCount((prev) => {
    const next = prev + 1;
    if (next >= 4) navigate("/register"); // back to student register
    return next;
  });
};
```

---

## What We Implemented

- ✅ Two distinct dashboards with strict role separation.
- ✅ Protected routes to enforce authentication + role validation.
- ✅ Session IDs in URL for better tracking & obfuscation.
- ✅ Hidden admin register route (via Easter Egg).

---

## Future Improvements

- Use JWT tokens with role claims instead of just frontend checks.
- Add audit logging for admin actions.
- Make the admin register path configurable via .env for extra security.

---

## What We Learned

- Assigning demarcated or differentiated route path for multiple type login is the way to go.
- In securing of dashboard from intrusion of unauthorized users, it is advisable for wrap component in a ProtectedRoute.jsx.
- Using JWT tokens to store checks.

---

## Verification

- Clicking on the Logo 4(Four times) on either the Login or Register page redirects and reroutes user to Admin path of the same page.
- Signing Up or Logging In on the Admin page redirects you to the Admin dashboard and based on the position or role of the admin, priviledges are unlocked.

---

## Troubleshooting details

- Creation of 2 different dashboards: `AdmnDashboard.jsx` and `StudentDashboard.jsx`.
- Wrapped both Dashboards in a component `ProtectedRoutes.jsx` which wraps dshboard pages and checks for role to access either of them.
- Added session-based unique Ids to each route url to enforce visual and browsser seperation.
- Used `useEffect` and `navigate` to trigger Easter Egg Effect.

---

## References

- [Routing Guide]](../Routing.md)
- [Database Setup]()
- [Security Fixes](../Security%20and%20Authentication%20Documentation/README.md)

---

