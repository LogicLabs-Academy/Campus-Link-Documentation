# API Endpoint Issues

Another of the major issues and challenges faced among developers in the building of the CampusLink Project and even frontend developers who are stepping into the world of backend and API call Usage. This documentation serves t act as a guide and dev diary that captures all the issues and lessons related to **API endpoint configuration, communication between frontend and backend, and integration with external APIs** in the CampusLink project.

---

# Issue 1: API calls failing due to inconsistent base URLs

## Narrative

During development, some API calls failed because the frontend was configured with `http://localhost:5173` while the backend was running on `http://127.0.0.1:5000`. The mismatch in host/port caused **CORS errors and unreachable endpoints**.

## Possible Cause

- Using `localhost` in one service and `127.0.0.1` in another.
- API base URL not centralized (hardcoded in multiple files).
- Missing proxy configuration during development.

## Step-by-step Solution

### 1. Create a central configuration file for API URLs:

```js
// src/config/api.js
export const API_BASE_URL = "http://127.0.0.1:5000/api";
```

### 2.Update axios calls to import from `/config/api.js`:

Example snippet:

```js
import { API_BASE_URL } from "../config/api";
import axios from "axios";

export const loginUser = async (credentials) => {
  const response = await axios.post(`${API_BASE_URL}/auth/login`, credentials);
  return response.data;
};
```

### 3. Ensure backend has CORS enabled for bross browser and cross-path communication.

Example snippet:

```js
//server.js
// Express backend
const cors = require("cors");
app.use(cors({ origin: "http://127.0.0.1:5173", credentials: true }));
```

## What We Implemented

- Centralized `API_BASE_URL` config file.
- Switched from fetch to axios for consistent error handling.
- Enabled `CORS` properly.

## Future Improvement

- Use `.env` variables for API URLs to avoid hardcoding.
- Add health-check endpoint `/api/health` to verify connectivity.

## What We Learned

- Never hardcode API endpoints across multiple files.
- Always use consistent hostnames(avoid mixing `localhost` and `127.0.0.1`).

---

## Verification

- All routes (`/auth/login`, `/auth/register`, `/auth/admin/register`, etc) tested via **Postman**, **Curl** or **Frontend Login/ Register Pages**
- Confirmed API health check: returned `ERROR 200: Staus OK`.

## Troubleshooting Details

- If API calls fail: check browser console for CORS error vs. `404 error`.
- Ensure backend is running on the correct port.
- Verify `.env` variables are being read correctly in frontend build.

## References

- [MDN: CORS](https://developer.mozilla.org/en-US/docs/Web/HTTP/CORS)
- [Axios Docs](https://axios-http.com/docs/intro)

---

# Issue 2: Error handling with failed API responses

## Narrative

Initially, when a login attempt failed (wrong password), the frontend simply crashed because the error response wasn’t handled gracefully.

## Possible Cause

- No `.catch()` in fetch calls.
- Not parsing error messages from backend properly.
- Missing default UI feedback for API errors.

## Step-by-step Solution

### 1. Add try/catch wrapper for API calls:

Example snippet:

```js
//server.js
try {
  const res = await axios.post(`${API_BASE_URL}/auth/login`, userData);
  return res.data;
} catch (error) {
  throw error.response?.data?.message || "Unexpected error occurred";
}
```

### 2. Display error in UI:

Example snippet:

```js
//server.js
try {
  await loginUser(credentials);
} catch (err) {
  setErrorMessage(err);
}
```

## What We Implemented

- Wrapped all API calls in reusable service functions with error handling.
- Added frontend error banner for failed login/register.

## Future Improvement

- Add retry logic for network-related failures.
- Implement global error handler via React Context.

## What We Learned

- Backend should always return structured **error JSON** (`{ "error": "message" }`).
- Frontend should gracefully display errors, not crash.

## Verification

- Entered wrong password → UI showed "Invalid credentials" instead of blank screen.
- Simulated network failure → UI displayed "Unexpected error occurred".

## Troubleshooting Details

- If `error.response` is undefined, likely a **network error** not a server error.
- Check browser network tab for the request/response payload.

## References

- [React Error Handling Patterns](https://kentcdodds.com/blog/prop-drilling)

---

# Issue 3: External API integration (future concern)

## Narrative

There’s a plan to integrate CampusLink with external services like payment APIs or email services. Poor handling of external API endpoints may cause security risks.

## Possible Cause

- Not securing API keys (hardcoding in frontend).
- Not validating external API responses.
- Missing rate-limit handling.

## Step-by-step Solution

### 1. Store API keys in backend `.env` files only.

Example snippet:

```bash
EMAIL_API_KEY=securedapikey
```

**Note: Never Disclose your Real details in your .env to public, Always add .env files to .gitignore and .dockerignore.**
If you mistakenly release or exposed your `.env` files containing real database or api keys then:

- Firstly, Change passwords or generate another API key if possible. Refer to [Changing of Password.md]() for documentation concerning that.
- Visit our detailed [Documentation](../) on how to clean up/ `reflog` git history. Or visit the written [Documentation]() by Github taking you through step-by-step procedure on that.

### 2. Create backend proxy endpoints for external APIs:

```js
//server.js
app.post("/api/send-email", async (req, res) => {
  const { to, subject, body } = req.body;
  const result = await externalEmailService.send({ to, subject, body });
  res.json(result);
});
```

### 3. Apply rate limiting middleware (eg. `/middleware/rate-limiter.js`)

## What We Implemented

- (Planned) all third-party API calls go through backend proxy.

## Future Improvement

- Implement retry queues for failed external API calls.
- Add monitoring/alerts for API failures.

## What We Learned

- Never expose private keys in frontend.
- Always sanitize and validate external API responses.

## Verification

- Tested backend email proxy endpoint in dev (dummy payload).
- Verified no API keys appear in frontend network requests.

## Troubleshooting Details

- If external API fails: check backend logs first (not frontend).
- Ensure correct API key is loaded from `.env`.

## References

- [OWASP API Security](https://owasp.org/API-Security/)
