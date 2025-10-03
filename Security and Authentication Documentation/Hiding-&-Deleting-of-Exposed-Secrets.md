# Deleting Exposed Secrets from Clone History

This document explains how to **delete exposed secrets** (API keys, DB passwords, tokens) in Git history or repo leaks.

---

## Issue - Deleting Exposed secret file from github repo, doesn't delete it from previous clones or git repo history

---

## Narrative

Developers sometimes commit `.env` by mistake or paste API keys into code. Simply deleting the file from the latest commit does not erase it from Git history. Attackers scanning GitHub can still find it.

---

## Possible Cause

- `.gitignore` not set up properly for `.env`.
- Developers accidentally committed `.env` before `.gitignore` was added.
- Force pushes without proper history cleaning.

---

## Step-by-step Solution

### 1. **Identify the Leak**

```bash
git log -p | grep "API_KEY"
```

### 2. **Remove File from History**

Using `git-filter-repo` (preferred over `filter-branch`)

```bash
pip install git-filter-repo
git filter-repo --path .env --invert-paths

```

### 3. **Force Push Clean History**

```bash
git push origin main --force

```

### 4. **Rotate Keys**

- Immediately revoke the exposed API keys or DB password
- Regenerate new ones (see [ApiKey Rotation and Recovery.md](Exposed-Secrets-Rotation-&-Recovery.md))

### 5. Invalidate Cached Data

- If JWT Secret was leaked, all existing tokens become invalid.
- Force Users to log in again.

## What We Implemented

- `.env` added to `.gitignore`.
- Used Github's secret scanning alert
- Documentation for using `git-filter-repo`

## Future Improvement

- Pre-commit hook to block commits containinng "API KEYS" or "DB_PASSWORD"
- Enable Github push protection for secrets.

## What We Learned

- Deletiing from the repo is not enough - must clean history.
- Always assume leaked keys are compromised: rotate immediately.

## Verification

- confirm with `git log` that `.env` no longer appears.
- Test system works with new rotated keys.

## Troubleshooting

- if `git-filter-repo` not installed => us BFG Repo cleaner.

```bash
java -jar bfg.jar --delete-files .env
```

- if other collaborators still have old commits, tell them to reclone repo.

## References

- [Github Secret Scanning](https://docs.github.com/en/code-security/secret-scanning/about-secret-scanning)
- [Removing sensitive data from a repo](https://docs.github.com/en/authentication/keeping-your-account-and-data-secure/removing-sensitive-data-from-a-repository)
