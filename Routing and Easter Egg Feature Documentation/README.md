# 📂 Routing – Overview

This folder contains documentation of all issues, fixes, and improvements related to **routing** in the CampusLink project.

Routing was one of the most challenging aspects of the project because we had to separate **student** and **admin** paths, enforce **protected routes**, and handle persistence across refreshes without breaking the user experience.

---

## ⚡ Quick Notes

- Always make sure **API endpoints call their respective units**.
  - Example: student endpoints should never respond to admin routes.
- Use `ProtectedRoute.jsx` to enforce **role-based access** (student/admin).
- Avoid direct exposure of **admin login/register** pages by using Easter Egg redirects.
- For **refresh persistence**, ensure `localStorage` is checked before re-rendering routes.
- Use URL session identifiers (`/#rdc12133-DC/dashboard`) for students and (`/MOUAU123ere)/dashboard`) for admins.
- Handle Easter Egg logo multi-tap carefully to avoid unwanted reload loops.

---

## 📑 Related Documentation

- [ProtectedRoutes-Related-Issues.md](./ProtectedRoutes-Related-Issues.md)
- [RefreshPersistence.md](./RefreshPersistence.md)
- [EasterEggRouting.md](./EasterEggRouting.md)
- [ApiEndpoints.md](./ApiEndpoints.md)

---

## 🛠 Troubleshooting

If you encounter routing issues:

- Verify the `App.jsx` router setup matches your user role logic.
- Ensure tokens + `userType` in `localStorage` are being read correctly.
- Check the `navigate()` paths in `StudentLogin.jsx` and `AdminLogin.jsx`.
- See the specific `.md` files above for **step-by-step fixes**.
