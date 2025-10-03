# API Key & Database Password Rotation

This document covers **how to rotate sensitive credentials** (API keys, DB passwords, service tokens) in CampusLink without breaking the system.

---

# Issue - Developer Mistakenly Exposed Api keys and Database Passwords in Code When Deploying To Github

## Narrative

During development, **Database Passwords and External API keys** were stored in `.env` file. After deployment, he noticed that an attacker starts to share passwords and information of his site online.

## Possible Cause

- Exposed `Secrets` or `.env` file during deployment to **Github or respective deployment service**
- Failed to add `.env` to `.gitignore` or typo error in writing of `.env` in `.gitignore`/`.dockerignore`.
- Credentials hardcoded in file that was never added to `.gitignore` or `.dockerignore`.
- Developers sharing `.env` values over chat/email got exposed.

---

## Step-by-step Solution

### 1. **Update the Key/Password in the Provider**

- For DB: Change the password in MySQL/Postgres/Mongo via admin console:

  ```sql
  ALTER USER 'campuslink_user'@'localhost' IDENTIFIED BY 'new_secure_password';
  ```

  **Replace: - `campuslink_user` with your Database Username,- `localhost` with the respective server Postgres was on ie `localhost` or `postgres`. - `new_secure_password` with new strong password.**

- For API: Regenerate key in provider dashboard (e.g., Stripe, SendGrid).

### 2. **Update Backend `.env`**

```env
DB_PASSWORD=new_secure_password
API_KEY=sk_live_newGeneratedKey123
```

### 3. **Restart Services**

#### In Docker, run:

```bash
docker-compose down
docker-compose up -d
#or
pm2 restart all
```

#### In Deployment Services:

- Push the updated codes to Github (this time with `.dockerignore` and `.gitignore` well added and api keys within code removed)
- Now in your deployment Service dashboard
- Under projects, locate the deployed services.
- Click the three dots seen and click `restart` or `delete` project.(Either of them works to deleteing the current project, pulling newest commit on Github and then ready to be hosted again)

### 4. **Update Deployment Environment**

- Push updated secrets to production server(`.env.production`).
- if using Github Actions, add keys in Github Repo=> Settings=> Secrets and Variables.

## What We Implemented

- Centralize `.env` management.
- Documented secret rotation process.
- Added `api/health` endpoint to confirm DB connectivity after changes.

## Future Improvement

- Automate secret rotation via HashiCorp Vault or AWS Secrets Manager.
- Rotate BD/API credentials every 90days by policy.

## What We Learned

- Manual Key rotation is error prone
- Always test after chanaging secrets - many **"outages"** are from outdated keys in deployment.
- Always keep secrets in `.env` files

## Verification

- After password rotation, `npm run dev` still connects to DB
- `/api/health` still connects to database
- External Services stil work even after APIkey Rotation.

## Troubleshooting Details

- If backend crashes => check `.env` mismatch with DB.
- If External API fails => confirm new key is active in provider dashboard.

## References

- [OWASP Secrets Management](https://owasp.org/www-project-cheat-sheets/cheatsheets/Secrets_Management_Cheat_Sheet.html)
- [12 Factor App: Config](https://12factor.net/config)
- Our [Documentation](Hiding-&-Deleting-of-Exposed-Secrets.md) on erasing git history to avoid reclone.
