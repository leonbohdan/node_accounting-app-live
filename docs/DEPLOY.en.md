# Deployment Guide — Render.com

## Table of Contents

- [Project Preparation](#project-preparation)
- [Step 1: Prepare the Repository](#step-1-prepare-the-repository)
- [Step 2: Sign Up on Render](#step-2-sign-up-on-render)
- [Step 3: Create a PostgreSQL Database](#step-3-create-a-postgresql-database)
- [Step 4: Create a Web Service](#step-4-create-a-web-service)
- [Step 5: Configure Environment Variables](#step-5-configure-environment-variables)
- [Step 6: Initialize Database Tables](#step-6-initialize-database-tables)
- [Verify the Deployment](#verify-the-deployment)
- [Updating the App](#updating-the-app)
- [Troubleshooting](#troubleshooting)

---

## Project Preparation

Before deploying, make sure:

- The `.env` file is **not committed** to the Git repository (it must be listed in `.gitignore`)
- The `.env.example` file **is present** in the repository as a variable template
- `package.json` contains the script `"start": "node src/index.js"`

### Check `.gitignore`

Open `.gitignore` and confirm these lines exist:

```
.env
.env.local
```

---

## Step 1: Prepare the Repository

1. Initialize a Git repository (if you haven't already):
   ```bash
   git init
   git add .
   git commit -m "initial commit"
   ```

2. Push to GitHub:
   ```bash
   # Create a repository on github.com, then:
   git remote add origin https://github.com/<your-username>/<repo-name>.git
   git push -u origin main
   ```

---

## Step 2: Sign Up on Render

1. Go to [render.com](https://render.com) and sign up
2. Click **"Sign in with GitHub"** — this allows Render to automatically access your repositories

---

## Step 3: Create a PostgreSQL Database

> Render provides a free PostgreSQL database (limitation: 90 days on the free plan).

1. In the dashboard, click **"New +"** → **"PostgreSQL"**
2. Fill in the form:
   - **Name**: `accounting-db` (or any name you prefer)
   - **Database**: `accounting`
   - **User**: `accounting_user`
   - **Region**: Frankfurt (EU Central) — closest to Europe
   - **Plan**: Free
3. Click **"Create Database"**
4. Wait 1–2 minutes for the database to be created
5. **Save the connection credentials** from the database page — you'll need them in the next step:
   - **Host** (use the Internal Host for connections within Render)
   - **Port**: `5432`
   - **Database**
   - **Username**
   - **Password**

---

## Step 4: Create a Web Service

1. In the dashboard, click **"New +"** → **"Web Service"**
2. Choose **"Build and deploy from a Git repository"**
3. Connect your GitHub repository
4. Fill in the settings:

   | Field | Value |
   |-------|-------|
   | **Name** | `accounting-api` |
   | **Region** | Frankfurt (EU Central) |
   | **Branch** | `main` |
   | **Runtime** | Node |
   | **Build Command** | `npm install --omit=dev` |
   | **Start Command** | `node src/index.js` |
   | **Plan** | Free |

5. **Do not click "Create Web Service" yet** — first configure environment variables (Step 5)

---

## Step 5: Configure Environment Variables

On the same Web Service creation page, scroll down to the **"Environment Variables"** section and add the following:

| Key | Value |
|-----|-------|
| `PORT` | `3000` |
| `POSTGRES_HOST` | Internal Host from the Render PostgreSQL page |
| `POSTGRES_PORT` | `5432` |
| `POSTGRES_USER` | Username from the Render PostgreSQL page |
| `POSTGRES_PASSWORD` | Password from the Render PostgreSQL page |
| `POSTGRES_DB` | Database name from the Render PostgreSQL page |
| `POSTGRES_DIALECT` | `postgres` |
| `NODE_ENV` | `production` |

> **Important:** Use the **Internal Host** (not External) to keep the connection between services fast and free within Render's network.

After filling in the variables, click **"Create Web Service"**.

---

## Step 6: Initialize Database Tables

After the first deployment, you need to run the database setup once to create all tables.

### Option A: Via Render Shell (recommended)

1. Open your Web Service on Render
2. Go to the **"Shell"** tab
3. Run the following command:
   ```bash
   node src/setupDb.js
   ```
4. You should see:
   ```
   --- The table for the todos model was just created! ---
   --- The table for the users model was just created! ---
   --- The table for the expenses model was just created! ---
   ```

### Option B: Via External Connection

Use the **External Database URL** from the Render PostgreSQL settings to connect via `psql` or any GUI client (TablePlus, DBeaver, pgAdmin).

---

## Verify the Deployment

After a successful deployment, Render will provide a URL like `https://accounting-api.onrender.com`.

Test the API:

```bash
# Check server availability
curl https://accounting-api.onrender.com/todos

# Or open in the browser
https://accounting-api.onrender.com/users
```

Expected result — a JSON response (`[]` if the database is empty).

---

## Updating the App

Render automatically deploys a new version every time you push changes to the `main` branch:

```bash
git add .
git commit -m "feat: describe your changes"
git push origin main
```

Render will automatically:
1. Detect the new commit
2. Run the Build Command
3. Replace the previous version with the new one (zero downtime)

---

## Troubleshooting

### ❌ `Error: Dialect needs to be explicitly supplied`
**Cause:** The `POSTGRES_DIALECT` variable is missing or empty.  
**Fix:** Add `POSTGRES_DIALECT=postgres` to the Environment Variables on Render.

---

### ❌ `Error: connect ECONNREFUSED` or `Connection refused`
**Cause:** Wrong host or the database is not ready yet.  
**Fix:** Make sure you are using the **Internal Host** (not External) from the Render PostgreSQL settings.

---

### ❌ Server "sleeps" after 15 minutes of inactivity
**Cause:** Render's free plan automatically spins down inactive services.  
**Fix:** The server will "wake up" on the next request (takes ~30 seconds). For always-on uptime, upgrade to a paid plan or use an external pinger like [UptimeRobot](https://uptimerobot.com).

---

### ❌ Tables not created — `relation does not exist` errors
**Cause:** `setupDb.js` was not run after deployment.  
**Fix:** Run `node src/setupDb.js` via the Shell tab in Render (Step 6).

---

### 📋 Checking Logs

Application logs are available under the **"Logs"** tab of your Web Service on Render — this is the first place to check for any issues.
