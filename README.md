# 📋 Attendance Tracker — Azure Deployment Guide

Sign in to everything below using your **Outlook / Microsoft account**.

---

## Overview of What You're Setting Up

| Service | Purpose | Cost |
|---|---|---|
| **Azure Cosmos DB** | Stores all attendance data | Free tier (serverless) |
| **Azure Static Web Apps** | Hosts the app + API | Free tier |
| **GitHub** | Stores your code (deployment trigger) | Free |

Total cost: **$0/month** on free tiers.

---

## Step 1 — Create a GitHub Account & Upload Code

1. Go to [https://github.com](https://github.com) → Sign up (use any email)
2. Click **"New repository"** → name it `attendance-tracker` → **Create repository**
3. Install Git from [https://git-scm.com](https://git-scm.com)
4. Open a terminal/command prompt in this project folder and run:

```bash
git init
git add .
git commit -m "Initial commit"
git branch -M main
git remote add origin https://github.com/YOUR_USERNAME/attendance-tracker.git
git push -u origin main
```

---

## Step 2 — Create Azure Cosmos DB

1. Go to [https://portal.azure.com](https://portal.azure.com) — sign in with your **Outlook account**
2. Search **"Azure Cosmos DB"** in the top bar → Click **Create**
3. Choose **"Azure Cosmos DB for NoSQL"** → Click **Create**
4. Fill in:
   - **Subscription:** Your subscription (or "Free Trial")
   - **Resource Group:** Click "Create new" → name it `attendance-rg`
   - **Account Name:** `attendance-tracker-db` (must be globally unique — add numbers if taken)
   - **Location:** Choose nearest to you (e.g. UAE North, Southeast Asia)
   - **Capacity mode:** Select **Serverless** ← important for free usage
5. Click **Review + Create** → **Create** (takes ~3-5 minutes)

### Create the Database and Containers

6. Once deployed, click **"Go to resource"**
7. In the left menu click **"Data Explorer"**
8. Click **"New Container"** and create these one by one:

| Database ID | Container ID | Partition key |
|---|---|---|
| `AttendanceDB` | `clients` | `/id` |
| `AttendanceDB` | `attendance` | `/id` |
| `AttendanceDB` | `members` | `/id` |

> For the first container, check **"Create new database"** and type `AttendanceDB`.
> For the remaining two, select the existing `AttendanceDB`.

### Copy Your Keys

9. In the left menu click **"Keys"**
10. Copy these two values — you will need them in Step 3:
    - **URI** (looks like `https://attendance-tracker-db.documents.azure.com:443/`)
    - **PRIMARY KEY** (long string of characters)

---

## Step 3 — Deploy to Azure Static Web Apps

1. In [portal.azure.com](https://portal.azure.com), search **"Static Web Apps"** → Click **Create**
2. Fill in:
   - **Resource Group:** Select `attendance-rg`
   - **Name:** `attendance-tracker-app`
   - **Plan type:** Free
   - **Region:** Choose nearest to you
   - **Source:** GitHub
3. Click **"Sign in with GitHub"** → authorize Azure
4. Select your repository: `attendance-tracker`, branch: `main`
5. **Build Details:**
   - Build Preset: **React**
   - App location: `/`
   - Api location: `api`
   - Output location: `build`
6. Click **Review + Create** → **Create**

Azure will automatically build and deploy your app and add a GitHub Action to your repo.

### Add Your Cosmos DB Keys as Environment Variables

7. Once created, click **"Go to resource"**
8. In the left menu, click **"Configuration"**
9. Click **"+ Add"** and add these two entries:

| Name | Value |
|---|---|
| `COSMOS_ENDPOINT` | Your URI from Step 2 |
| `COSMOS_KEY` | Your PRIMARY KEY from Step 2 |

10. Click **Save**

---

## Step 4 — You're Live! 🎉

1. In your Static Web App resource, find the **URL** at the top
   (e.g. `https://attendance-tracker-app.azurestaticapps.net`)
2. Open it in your browser — the app should load
3. Click **"Create new client workspace"** → enter a client name → receive your access code
4. Share the **URL + access code** with your team members

---

## How Access Codes Work

| Who | Action |
|---|---|
| **You (Admin)** | Create a workspace → receive an 8-character code |
| **Team members** | Open the URL → enter name + code → joined instantly |
| **Different client** | Create a new separate workspace → separate code |

Each client's data is completely isolated from others.

---

## Updating the App in the Future

Any time you push changes to GitHub, Azure automatically rebuilds and redeploys:

```bash
git add .
git commit -m "Your update description"
git push
```

---

## Project Structure

```
attendance-tracker/
├── api/                          ← Azure Functions (backend/API)
│   ├── shared/
│   │   └── cosmosClient.js       ← Cosmos DB connection (uses your env keys)
│   ├── clients/                  ← Create and look up client workspaces
│   ├── attendance/               ← Save and load monthly attendance data
│   ├── employees/                ← Manage the resource/employee list
│   ├── members/                  ← Manage team members per workspace
│   └── host.json
├── public/
│   ├── index.html
│   └── staticwebapp.config.json  ← Azure routing config
├── src/
│   ├── App.js                    ← Root component
│   ├── AccessScreen.js           ← Login / create workspace screen
│   ├── AttendanceTracker.js      ← Main attendance grid + summary
│   └── MembersPanel.js           ← Team member management panel
└── package.json
```

---

## Azure Free Tier Limits

| Resource | Free Allowance |
|---|---|
| Cosmos DB Serverless | 1,000 RU/s · 25 GB storage |
| Static Web Apps | 100 GB bandwidth/month · 2 custom domains |

Both are well within limits for a team attendance tracker.

---

## Troubleshooting

| Problem | Fix |
|---|---|
| App shows blank page | Check GitHub Actions tab — look for build errors |
| "Connection error" on login | Verify COSMOS_ENDPOINT and COSMOS_KEY are saved in Configuration |
| Access code says invalid | Make sure Cosmos DB containers were created with the exact names above |
| Changes not reflecting | Wait ~2 minutes after `git push` for Azure to rebuild |
