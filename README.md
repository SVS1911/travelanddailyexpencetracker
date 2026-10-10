# Expense Tracker

The frontend is React/Vite. A FastAPI backend handles authentication and API routing, and SQLite stores accounts, expenses, travelers, spending limits, and the notification inbox in a local database file. Passwords are Argon2-hashed; the browser keeps only a signed session token.

## Requirements

- Node.js 20 or newer
- Python 3.10 or newer
- No separate database server is required; SQLite is included with Python.

## Run locally

1. Install frontend packages: `npm install`
2. Create an isolated Python environment: `python -m venv .venv`
3. Activate it in PowerShell: `.venv\Scripts\Activate.ps1`
4. Install API packages: `python -m pip install -r requirements.txt`
5. Copy `.env.example` to `.env` (PowerShell: `Copy-Item .env.example .env`) and set `JWT_SECRET` to a random value with at least 32 characters. `DATABASE_PATH` defaults to `server/expense_tracker.db`.
6. Start the React frontend and FastAPI API: `npm run dev`
7. Open `http://localhost:5173` and register an account, or import the legacy account and expenses as described below. The API health endpoint is `http://localhost:8000/api/health`.

## Share on the same Wi-Fi/LAN

The Vite development server listens on your laptop's network interfaces. Find your laptop's IPv4 address with `ipconfig`, then share `http://<laptop-ip>:5173` with people on the same trusted Wi-Fi/LAN. Keep `npm run dev` running, and allow the Node.js app through Windows Firewall only on private networks if prompted. This address is not a public internet link.

FastAPI automatically creates and updates the SQLite tables at startup. The database file remains on disk between restarts; back it up regularly. The former PostgreSQL database is not automatically copied into SQLite. Export any existing PostgreSQL data before switching, or use the legacy JSON import described below.

## Spending limits

The regular and travel spending limits compare against the cumulative total of all saved expenses in their respective categories. They are not monthly limits.

## Email notifications

Adding an expense creates an in-app notification only; it does not send email. Add a traveler with their name and email on the Travel overview; submitting an existing traveler's name again updates their email. Click **Send email** beside a traveler's total to send that person an itemized list of all recorded travel expenses assigned to them, with the total. Sending requires SMTP configuration.

Configure SMTP in `.env` to send email:

```dotenv
SMTP_HOST=smtp.example.com
SMTP_PORT=587
SMTP_USER=your-smtp-username
SMTP_PASSWORD=your-smtp-password
MAIL_FROM=expenses@example.com
SMTP_USE_TLS=false
SMTP_START_TLS=true
```

For implicit TLS (commonly port 465), set `SMTP_USE_TLS=true` and `SMTP_START_TLS=false`. Keep SMTP credentials out of source control. If SMTP is not configured, in-app notifications continue to work; pressing **Send email** displays a configuration error.

## Import existing JSON data

The original `db.json` is retained as a one-time import source. It contains legacy account data, including plaintext passwords; do not use those credentials for a public deployment. Run `npm run migrate:json` to import users, expenses, and travel companions without modifying the source file. The importer hashes legacy passwords and tracks the source file hash in SQLite, so repeating the command for the same file will not duplicate records. Imported travelers have no email address; add their email in the Travel overview before sending them a summary.

## Verify the app

Run `npm run build` to check and compile the frontend, and `python -m compileall -q server scripts` to check Python syntax. The API and database must be running and configured in `.env` for end-to-end account and expense flows.

## Production

Set `DATABASE_PATH`, `JWT_SECRET`, and optional SMTP settings in the hosting environment, then run `npm install`, create and activate a Python virtual environment, install `requirements.txt`, `npm run build`, and `npm start`. FastAPI serves the compiled React app from `dist`. Persist and back up the SQLite database file; use a persistent volume for it in hosted environments.


## Deploy on Render (React + FastAPI)

This repository includes `render.yaml` for a React Static Site and a FastAPI Web Service. In Render, choose **New → Blueprint**, connect this existing GitHub repository, and apply the blueprint. Review the service names and settings before creating the services.

- `expense-tracker-frontend`: builds with `npm install && npm run build` and publishes `dist`.
- `expense-tracker-api`: installs `requirements.txt`, starts Uvicorn, and checks `/api/health`.
- The API uses `/var/data/expense_tracker.db` on a persistent disk. Persistent disks may require a paid Render plan; confirm the current plan and disk availability in Render before applying the Blueprint.
- The frontend API URL is configured from the API service hostname. The API CORS allowlist initially includes localhost; after the frontend service is created, add its deployed `https://...onrender.com` origin to the API service's `CORS_ORIGINS` environment variable, then redeploy the API. Do not include a path such as `/api` in `CORS_ORIGINS`.
- Ensure `JWT_SECRET` is a strong, private value. Never commit `.env` or real credentials.

The local `.env`, Git history, `node_modules`, virtual environments, SQLite files, and `db.json` were intentionally not included in the deployment archive. The `db.json` file in the original project contained legacy plaintext credentials; do not deploy or publish it. If legacy records need to be migrated, sanitize and securely migrate them separately rather than committing the original file.
