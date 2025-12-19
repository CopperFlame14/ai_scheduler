# 🚀 Complete Deployment Guide for AI Scheduler

This guide will walk you through setting up **Supabase** (Database), **Railway** (Backend), and **Vercel** (Frontend) for the AI Scheduler project.

---

## 📋 Table of Contents

1. [Supabase Setup (Database)](#1-supabase-setup-database)
2. [Railway Setup (Backend)](#2-railway-setup-backend)
3. [Vercel Setup (Frontend)](#3-vercel-setup-frontend)
4. [Connecting Everything Together](#4-connecting-everything-together)
5. [Testing Your Deployment](#5-testing-your-deployment)

---

## 1. Supabase Setup (Database)

### Step 1.1: Create a Supabase Project

1. Go to [https://supabase.com](https://supabase.com) and sign in
2. Click **"New Project"**
3. Fill in:
   - **Project name**: `ai-scheduler`
   - **Database password**: Choose a strong password (save it!)
   - **Region**: Choose the closest to your users
4. Click **"Create new project"** and wait ~2 minutes

### Step 1.2: Get Your API Keys

1. In your Supabase dashboard, go to **Settings** → **API**
2. Copy these values:
   - **Project URL** → This is your `SUPABASE_URL`
   - **anon public** key → This is your `SUPABASE_ANON_KEY`
   - **service_role** key → This is your `SUPABASE_SERVICE_KEY`

> ⚠️ **IMPORTANT**: Never expose your `service_role` key in frontend code!

### Step 1.3: Create Database Tables

1. Go to **SQL Editor** in your Supabase dashboard
2. Click **"New query"**
3. Copy and paste the entire contents of `supabase_schema.sql` from this project
4. Click **"Run"** to execute

This creates:
- `user_profiles` - User profile data
- `tasks` - Task management
- `calendar_events` - Calendar events
- `email_notifications` - Email tracking
- `reminders` - Reminder system

### Step 1.4: Enable Authentication

1. Go to **Authentication** → **Providers**
2. Enable **Email** provider (already enabled by default)
3. (Optional) Enable **Google** OAuth:
   - Go to [Google Cloud Console](https://console.cloud.google.com)
   - Create OAuth credentials
   - Add the Client ID and Secret in Supabase

### Step 1.5: Configure Site URL

1. Go to **Authentication** → **URL Configuration**
2. Set **Site URL** to your Vercel frontend URL (after Step 3)
3. Add **Redirect URLs**:
   - `http://localhost:5000` (for local development)
   - `https://your-vercel-app.vercel.app` (your production URL)

---

## 2. Railway Setup (Backend)

### Step 2.1: Create a Railway Account

1. Go to [https://railway.app](https://railway.app) and sign in with GitHub
2. Click **"New Project"**
3. Select **"Deploy from GitHub repo"**
4. Connect your GitHub account if not already connected
5. Select the `ai_scheduler` repository

### Step 2.2: Configure the Service

Railway will automatically detect the project. Now configure it:

1. Click on your newly created service
2. Go to the **Settings** tab
3. Under **Deploy**, verify:
   - **Start Command**: `gunicorn --chdir backend app:app --bind 0.0.0.0:$PORT`
   - This is already configured in `railway.json`

### Step 2.3: Add Environment Variables

1. Go to the **Variables** tab
2. Add each of these variables:

```
SUPABASE_URL=https://bbaiwauzaxwszxwgutdg.supabase.co
SUPABASE_ANON_KEY=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...
SUPABASE_SERVICE_KEY=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...
GEMINI_API_KEY=your_gemini_api_key_here
SECRET_KEY=60a2adbb6fd6df4b58196ed15c040bcdf0a4bdfbdcaef03ad05ddc71f06a5ffe
FLASK_ENV=production
```

> 💡 **Get a Gemini API Key**: Go to [https://aistudio.google.com/app/apikey](https://aistudio.google.com/app/apikey)

### Step 2.4: Deploy

1. Railway will auto-deploy when you push to GitHub
2. Wait for the deployment to complete (~2-3 minutes)
3. Click **"Generate Domain"** to get your public URL
4. Your backend URL will look like: `https://your-app.up.railway.app`

### Step 2.5: Test Backend

Visit `https://your-app.up.railway.app/api/health` - you should see:
```json
{"status": "healthy", "database": "connected"}
```

---

## 3. Vercel Setup (Frontend)

### Step 3.1: Create a Vercel Account

1. Go to [https://vercel.com](https://vercel.com) and sign in with GitHub
2. Click **"Add New..."** → **"Project"**
3. Import the `ai_scheduler` repository

### Step 3.2: Configure Build Settings

Vercel will auto-detect the configuration from `vercel.json`. Verify:

1. **Framework Preset**: Other
2. **Root Directory**: `.` (root)
3. **Build Command**: (leave empty)
4. **Output Directory**: `frontend`

### Step 3.3: Update Frontend Config

Before deploying, update `frontend/config.js` with your Railway backend URL:

```javascript
const config = {
    API_URL: window.location.hostname === 'localhost'
        ? 'http://localhost:5000'
        : 'https://YOUR-RAILWAY-APP.up.railway.app',  // Replace this!
    
    OAUTH_REDIRECT: window.location.origin + '/auth/google/callback',
    APP_NAME: 'ARIA',
    VERSION: '1.0.0'
};
```

### Step 3.4: Add Environment Variables (Optional)

In Vercel dashboard → Settings → Environment Variables:

```
NEXT_PUBLIC_SUPABASE_URL=https://bbaiwauzaxwszxwgutdg.supabase.co
NEXT_PUBLIC_SUPABASE_ANON_KEY=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...
```

### Step 3.5: Deploy

1. Click **"Deploy"**
2. Wait for deployment (~1-2 minutes)
3. Your frontend URL will be: `https://your-app.vercel.app`

---

## 4. Connecting Everything Together

### Update Supabase Redirect URLs

1. Go to Supabase → **Authentication** → **URL Configuration**
2. Add your Vercel URL to **Redirect URLs**:
   - `https://your-app.vercel.app`
   - `https://your-app.vercel.app/dashboard.html`

### Update CORS in Backend

The backend already has CORS configured, but if you face issues, update `backend/app.py`:

```python
CORS(app, origins=[
    "http://localhost:5000",
    "http://localhost:3000",
    "https://your-app.vercel.app",  # Add your Vercel URL
    "https://your-app.up.railway.app"
])
```

---

## 5. Testing Your Deployment

### Test 1: Health Check
```
GET https://your-railway-app.up.railway.app/api/health
```
Expected: `{"status": "healthy"}`

### Test 2: Frontend Loads
Visit `https://your-app.vercel.app` - You should see the landing page

### Test 3: Authentication
1. Click "Sign Up"
2. Enter email and password
3. Should redirect to dashboard

### Test 4: Create a Task
1. In chat, type: "Create a task to review documents tomorrow"
2. Task should appear in the task list

---

## 🔑 Environment Variables Summary

| Variable | Where Used | Description |
|----------|------------|-------------|
| `SUPABASE_URL` | Backend (Railway) | Supabase project URL |
| `SUPABASE_ANON_KEY` | Backend + Frontend | Public API key |
| `SUPABASE_SERVICE_KEY` | Backend only | Admin key (keep secret!) |
| `GEMINI_API_KEY` | Backend (Railway) | Google Gemini API key |
| `SECRET_KEY` | Backend (Railway) | Flask session secret |
| `FLASK_ENV` | Backend (Railway) | `production` for live |

---

## 🔧 Troubleshooting

### "Failed to fetch" errors
- Check CORS settings in backend
- Verify API_URL in frontend/config.js matches Railway URL

### Authentication not working
- Verify Redirect URLs in Supabase match your Vercel domain
- Check browser console for specific errors

### Database errors
- Verify `supabase_schema.sql` was executed successfully
- Check RLS policies are enabled

### Railway deployment fails
- Check build logs for missing dependencies
- Ensure `requirements.txt` has all needed packages

---

## 📞 Support

If you encounter issues:
1. Check Railway logs: Railway Dashboard → Deployments → View Logs
2. Check Vercel logs: Vercel Dashboard → Deployments → View Logs
3. Check Supabase logs: Supabase Dashboard → Database → Logs

Good luck with your deployment! 🎉
