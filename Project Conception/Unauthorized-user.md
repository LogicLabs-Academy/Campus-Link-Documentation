# Issue: Fake Email and Unverified Student Authentication

## Narrative

At the very beginning of CampusLink’s conception, the login/register flow was designed with a simple **email + password** form.

This introduced a major security and authenticity flaw:

- Anyone (even non-students) could register with any fake email address.
- There was no way to differentiate between a real student and an outsider.
- As a result, the system could not guarantee that accounts represented actual students.

This was the **first big issue** we faced in the project.

---

## Possible Cause

- Lack of validation schema for student identities.
- Using only email + password for authentication (too generic).
- No database checks against real student records.

---

## Step-by-Step Solution

### 1. Define the Problem Clearly

We needed a way to **verify if a registering user is actually a student of the institution**.

### 2. Introduce a database storage table ie `schema.sql` for storing students credentials

- Created a **schema.sql** file for structured database rules.
- The schema enforced **matric number + password** instead of email.
- Each student’s matric number was tied to real school records (cannot be faked).

Example snippet:

```sql
CREATE TABLE students (
  id INT PRIMARY KEY AUTO_INCREMENT,
  matric_number VARCHAR(20) UNIQUE NOT NULL,
  password_hash VARCHAR(255) NOT NULL,
  full_name VARCHAR(100) NOT NULL,
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```

### 3. Update Backend Authentication

- Modified backend API to accept `matric_number` instead of email.
- Registration endpoint cross-checks matric numbers against the database.
- Example snippet:

```js
router.post("/login", async (req, res)) => {
  const { matric_number, password } = req.body;
  try {
    const { rows } = await pool.query(
      "SELECT * FROM students WHERE matric_number = $1",
      [matric_number]
    );
    const student = rows[0];
    if (!student)
      return res.status(401).json({ message: "Invalid credentials" });

    const isMatch = await bcrypt.compare(password, student.password);
    if (!isMatch)
      return res.status(401).json({ message: "Invalid credentials" });
    else {
    await pool.query(
      `INSERT INTO sessions (user_type, user_id, session_token, dashboard_id, created_at, expires_at)
       VALUES ('student', $1, $2, $3, NOW(), NOW() + INTERVAL '${
         process.env.REFRESH_EXPIRES_IN || "3d"
       }')`,
      [student.id, refreshToken, dashboardId]
    );
    res.json({
      accessToken,
      refreshToken,
      dashboardId,
      fullname: `${student.firstname} ${student.surname}`,
      type: "student",
    })};
  } catch (err) {
    console.error("Student login error:", err.message);
    res.status(500).json({ message: "Server error" });
  }
```

### Update Frontend Ui

- Changed login/register forms from `email → matric number`.
- Updated API calls.
  Example snippet:

```jsx
await api.post("/auth/register", { matric, password });
```

## What We Implemented

- ✅Student registration now requires a valid matric number.
- ✅ Backend checks against the SQL schema to reject fake entries.
- ✅ Admin accounts remain separate (handled later in routing/security).

---

## Future Improvements

- Add an institutional email verification layer (e.g., verify @mouau.edu.ng).
- Implement OTP/email confirmation for additional verification.
- Integrate with the school’s official database for real-time matric validation.

---

## What We Learned

- Using only email/password is not sufficient in an academic portal.
- Grounding authentication in real institutional identifiers (matric number) ensures authenticity.
- Always start database schema design early to avoid patchwork fixes later.

---

## Verification

- Logged in with an invalid credential that is not in the database: should receive an `Invalid Credential` error message.
- Using a registered students database logged user in to their respective dashboard.

---

## Troubleshooting details

1. Created a database storage table using schema.sql to stored students data.
2. Modified backend API to accept `matric_number` instead of email.
3. Updated the Api call path of backend.
4. Updated Frontend Api.

---

## References

- [Routing Guide]](../Routing.md)
- [Database Setup]()
- [Security Fixes](../Security%20and%20Authentication%20Documentation/README.md)

---

