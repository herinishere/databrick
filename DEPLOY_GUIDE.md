# 🚆 Rail Drishti — Render + Vercel Deploy Guide

---

## Architecture

```
GitHub Repo
    │
    ├── Render (Backend + API)     → https://rail-drishti.onrender.com
    │   └── Docker container
    │       ├── FastAPI backend
    │       ├── ML models
    │       ├── All data files
    │       └── Frontend (also served here at /app)
    │
    └── Vercel (Frontend only)     → https://rail-drishti.vercel.app
        └── Static index.html → calls Render API
```

You can use **either**:
- **Render only** → backend + frontend both at `https://rail-drishti.onrender.com/app`
- **Render + Vercel** → backend on Render, frontend on Vercel (faster load)

---

## STEP 1 — Prepare your data files

Make sure these 4 files exist in the `data/` folder:
```
data/stations.json               (~1.8 MB)
data/train_delay_data.csv        (~122 KB)
data/Train_details_22122017.csv  (~16 MB)
data/schedules.json              (~79 MB)  ← LARGE
```

Also add the rich training data if you have it:
```
data/train_delay_data_rich.csv
```

---

## STEP 2 — Push to GitHub

### Install Git
Download from: https://git-scm.com/download/win
After install, close and reopen PowerShell.

### Push your project

```powershell
cd C:\Users\herin\Downloads\rail_drishti_deploy_ready\rail_drishti
```

```powershell
git init
git add .
git commit -m "Rail Drishti - ready for deploy"
```

Go to https://github.com/new → create repo named `rail-drishti` → copy the URL.

```powershell
git remote add origin https://github.com/YOUR_USERNAME/rail-drishti.git
git branch -M main
git push -u origin main
```

### If push fails (files too large) — use Git LFS

Install Git LFS from: https://git-lfs.com

```powershell
git lfs install
git lfs track "data/schedules.json"
git lfs track "data/Train_details_22122017.csv"
git add .gitattributes
git add .
git commit -m "Add large files via LFS"
git push -u origin main
```

---

## STEP 3 — Deploy Backend on Render

1. Go to https://render.com → Sign up (free, no credit card)
2. Click **New +** → **Web Service**
3. Click **Connect a repository** → Connect GitHub → Select `rail-drishti`
4. Fill in settings:

| Field | Value |
|---|---|
| Name | `rail-drishti` |
| Region | Singapore (closest to India) |
| Branch | `main` |
| Runtime | **Docker** |
| Instance Type | **Free** |

5. Click **Advanced** → Add environment variables:

| Key | Value |
|---|---|
| `RAILWAYS_DATA_DIR` | `/app/data` |
| `PYTHONPATH` | `/app` |

6. Click **Create Web Service**

### Wait for deployment (5-10 minutes)

Watch the build logs. You'll see:
```
Loading datasets...
Training ML models...
System ready
```

### Your Render URLs:
- **Frontend:** `https://rail-drishti.onrender.com/app`
- **API Docs:** `https://rail-drishti.onrender.com/docs`
- **Health:** `https://rail-drishti.onrender.com/health`

> ⚠️ Free tier sleeps after 15 min of no traffic. First request after sleep takes ~30 seconds.

---

## STEP 4 — Deploy Frontend on Vercel (Optional)

Only do this if you want a faster-loading frontend URL.

### First: Update your Render URL in index.html

Open `frontend/index.html` and find line:
```javascript
const RENDER_BACKEND = 'https://rail-drishti.onrender.com';
```
Replace `rail-drishti` with your actual Render service name if different.

Then commit and push:
```powershell
git add frontend/index.html
git commit -m "Set Render backend URL"
git push
```

### Deploy on Vercel

1. Go to https://vercel.com → Sign up (free)
2. Click **Add New Project** → Import from GitHub → Select `rail-drishti`
3. Set **Root Directory** to `frontend`
4. Framework Preset: **Other**
5. Click **Deploy**

### Your Vercel URL:
- `https://rail-drishti.vercel.app`

---

## STEP 5 — Verify Everything Works

```
https://rail-drishti.onrender.com/health         → {"status":"ok","loaded":true}
https://rail-drishti.onrender.com/app            → Full UI
https://rail-drishti.onrender.com/docs           → API docs
https://rail-drishti.vercel.app                  → Frontend (if using Vercel)
```

Test an API call:
```
https://rail-drishti.onrender.com/api/user/train-info?train_no=12301
```

---

## Common Problems & Fixes

| Problem | Fix |
|---|---|
| Render build fails: `COPY data/schedules.json` | The file must exist in your git repo. Use Git LFS. |
| `{"loaded":false}` from /health | Data still loading — wait 2-3 more minutes |
| Render site not loading | Free tier is asleep — wait 30 seconds after first request |
| Vercel shows blank page | Check browser console — likely RENDER_BACKEND URL is wrong |
| CORS error on Vercel | Already fixed — backend has `allow_origins=["*"]` |
| 512MB RAM exceeded on Render | Upgrade to Starter ($7/mo) or reduce data loaded |

---

## Local Run (Windows) — Quick Reference

```powershell
cd rail_drishti\backend
$env:RAILWAYS_DATA_DIR = "../data"
python -m uvicorn app:app --host 0.0.0.0 --port 8000 --reload
```

Open: http://localhost:8000/app
