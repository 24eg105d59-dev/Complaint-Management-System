# ResolveHub Production Deployment Guide

Follow this guide to deploy the complete ResolveHub MERN stack application to production using **MongoDB Atlas**, **Render (Backend)**, and **Vercel (Frontend)**.

---

## Prerequisites
- A GitHub repository containing the project files.
- Accounts on:
  - [MongoDB Atlas](https://www.mongodb.com/cloud/atlas)
  - [Render](https://render.com/)
  - [Vercel](https://vercel.com/)

---

## Step 1: Database Setup (MongoDB Atlas)

1. **Log in** to MongoDB Atlas.
2. **Create a Database Cluster** (the free M0 tier is sufficient).
3. **Database Access setup**:
   - Go to **Security > Database Access**.
   - Click **Add New Database User**.
   - Create a user with **Read and Write to any database** permissions. Save the username and password.
4. **Network Access setup**:
   - Go to **Security > Network Access**.
   - Click **Add IP Address**.
   - Select **Allow Access From Anywhere** (`0.0.0.0/0`). This is necessary since Render Web Services use dynamic outbound IP addresses.
5. **Get Connection String**:
   - Go to **Database > Clusters**.
   - Click **Connect** on your cluster.
   - Choose **Drivers** under "Connect to your application".
   - Copy the Connection String (URI). Replace `<password>` with your created database user's password.

---

## Step 2: Backend Deployment (Render)

1. **Log in** to Render and click **New > Web Service**.
2. **Connect your GitHub repository**.
3. **Configure Service Parameters**:
   - **Name**: `resolvehub-backend`
   - **Root Directory**: `backend`
   - **Runtime**: `Node`
   - **Build Command**: `npm install`
   - **Start Command**: `node server.js`
4. **Configure Environment Variables**:
   Click **Advanced > Add Environment Variable** and add:
   - `NODE_ENV`: `production`
   - `PORT`: `5000` (Render will override this, but standardizing it is recommended)
   - `MONGODB_URI`: *[Your MongoDB Atlas Connection String]*
   - `JWT_SECRET`: *[A long, secure random string]*
   - `FRONTEND_URL`: `https://resolvehub-frontend.vercel.app` (Replace with your actual Vercel URL once generated in Step 3)
   - `EMAIL_SERVICE`: `gmail` *(optional)*
   - `EMAIL_USERNAME`: *[Your Gmail address]* *(optional)*
   - `EMAIL_PASSWORD`: *[Your Gmail App Password]* *(optional)*
5. **Deploy**: Click **Create Web Service**. Save the generated service URL (e.g. `https://resolvehub-backend.onrender.com`).

---

## Step 3: Frontend Deployment (Vercel)

1. **Log in** to Vercel and click **Add New > Project**.
2. **Import your GitHub repository**.
3. **Configure Project Settings**:
   - **Framework Preset**: `Vite` (Vercel auto-detects this)
   - **Root Directory**: Click Edit and select `frontend`.
   - **Build Command**: `npm run build`
   - **Output Directory**: `dist`
   - **Install Command**: `npm install --legacy-peer-deps` (Required to resolve React 19 dependencies!)
4. **Configure Environment Variables**:
   Under "Environment Variables", add:
   - `VITE_API_URL`: `https://resolvehub-backend.onrender.com/api` (Replace with your actual Render URL + `/api`)
   - `VITE_SOCKET_URL`: `https://resolvehub-backend.onrender.com` (Replace with your actual Render URL)
5. **Deploy**: Click **Deploy**. Vercel will build and assign your production domain.

> [!NOTE]
> SPA routing rewrites are already configured for you inside [vercel.json](file:///c:/Users/vishn/OneDrive/Documents/Complaint%20Management%20System/frontend/vercel.json) in the `frontend/` folder. This guarantees that direct browser refreshes on routes like `/dashboard` work perfectly.

---

## Step 4: Circular Configurations Sync
Once Vercel completes deployment, copy the production Vercel frontend URL (e.g., `https://resolvehub-frontend.vercel.app`) and update the `FRONTEND_URL` environment variable inside your Render Web Service configurations dashboard. Restart the Render service to apply CORS mapping.
