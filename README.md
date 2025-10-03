# CampusLink - Documentation

This repository is the operational logbook and troubleshooting guide for the CampusLink project. It contains all technical documentation, setup instructions, and troubleshooting guides for the CampusLink project. It records every error we hit during development, the step-by-step investigation, fixes that worked,
and recommended long-term improvements.
Each section is split into a dedicated .readme file for clarity.

---

**Purpose:**
Make it possible for future contributors (or you in 6 months) to:

- Understand why things were done a certain way
- Reproduce issues locally
- Apply fixes safely and quickly

**Quick note worth knowing**
Most of the issues that arised in the course of development came majorly from the among the following listed out. Hence it is advisable to check them out before venturing into the development work.

- [Github and Related Issues.md](./)
- [ProtectedRoutes and Related Issues.md](./Routing%20and%20Easter%20Egg%20Feature%20Documentation/ProtectedRoutes-Related-Issues.md)
- [Security and Authentication Issues.md](./Security%20and%20Authentication%20Documentation/)
  **Also check out [Project Conception.md](./Project%20Conception)**
- [Database and Related Issues.md](./)
- [Docker Setup and Usage Documentation.md](../Campus-Link-Docker/)
- [CICD Pipeline Documentation.md](./)
- [.env Evironment Uage Guide.md](./)
- [Api and Connectivity Related Issues](./) **Also check out [Routing and Easter Egg Documentation.md](./Routing%20and%20Easter%20Egg%20Feature%20Documentation/)**
  A solid grasp on these should be all it needs to have a smooth sail when such errors are encountered while developing.

---

## Structure (files you will find here)

- `README.md` - This overview
- `Security.md` - Authentication, remember-me, tokens, session IDs
- `Routing.md` - SPA routing, Easter-egg UI toggle, refresh/back button issues
- `Database.md` - Schema, importing SQL, seeding, PostgreSQL in Docker
- `Api.md` - Backend <-> frontend connectivity, CORS, `127.0.0.1` vs `localhost`
- `Docker.md` - Docker / Docker Compose, DNS/registry issues, how to wipe DB/containers
- `Ui.md` - React/Vite issues, service worker, manifest, favicon
- `CICD.md` - basic CI/CD notes & deploy tips
- `Contributors.md` - how to contribute, roles, conventions

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

## Quick start (developer checklist)

1. Ensure Docker is installed (optional but recommended for DB).
2. Copy `.env.example` to `.env.local` and set values (do not commit secrets).
3. Backend:
   ```bash
   cd backend
   npm install
   npm run dev
   ```
4. Frontend:
   ```bash
   cd frontend
   npm install
   npm run dev
   ```
5. Health-check
   Backend: http://127.0.0.1:5000/api/health
   Frontend: http://127.0.0.1:5173/

## Here are the list of the documentations found in this repo

1. [Security & Authentication Documentation](https://github.com/LogicLabs-Academy/Campus-Link-Documentation/Security-and-Authentication-Documentation)
2. [Routing & Easter Egg Feature Documentation](https://github.com/LogicLabs-Academy/Campus-Link-Documentation)
3. [Database & Schema.sql Use Documentation](https://github.com/LogicLabs-Academy/Campus-Link-Documentation)
4. [Api Connectivity Documentation](https://github.com/LogicLabs-Academy/Campus-Link-Documentation)
5. [Docker Usage Documentation](https://github.com/LogicLabs-Academy/Campus-Link-Documentation)
6. [Frontend Setup Documentation](https://github.com/LogicLabs-Academy/Campus-Link-Documentation)
7. [CICD Documentation](https://github.com/LogicLabs-Academy/Campus-Link-Documentation)
8. [Contributor and Role Management Documentation](https://github.com/LogicLabs-Academy/Campus-Link-Documentation)
9. [Github Usage Documentation](https://github.com/LogicLabs-Academy/Campus-Link-Documentation)
10. [Protected Routes Documentation](https://github.com/LogicLabs-Academy/Campus-Link-Documentation)
