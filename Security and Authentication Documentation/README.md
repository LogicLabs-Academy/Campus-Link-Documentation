# Security Documentation- detailed diary style documentation and overview

This repository folder markdown file contains all related security issues and fixs associated with the app during development and deployment. This serves to act as your first go to documentation on issues concerning authentication, authorization, accesstoken, session IDs and other related security issues especially related to the Campus Link project.

Security issues we faced include:

- Token storage and refresh handling.
- Differentiating student/admin session IDs.
- Preventing token misuse via localStorage.
- Handling login persistence across refresh/browser close.

---

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

## Quick Notes

- Always store refresh tokens securely (prefer **httpOnly cookies** over localStorage).
- Each login session should generate a **unique session ID** tied to the refresh token.
- Never expose admin routes directly – route protection is key.
- Add validation for student email/ID at registration to prevent fake signups.

---

## Related Documentation

For step-by-step fixes, see the detailed files linked below.

- [AuthTokenHandling.md](./AuthTokenHandling.md)
- [SessionIdManagement.md](./SessionIdManagement.md)
- [PasswordVisibility.md](./PasswordVisibility.md)
- [LocalStorageVsCookies.md](./LocalStorageVsCookies.md)
