# 🛠️ SmartSociety — Comprehensive Installation & Troubleshooting Guide

Hey there! 👋 This guide walks you through setting up **SmartSociety** (both the Node.js/Express Backend and the React 19 Frontend) from scratch. 

It includes real-world, human-written troubleshooting steps to fix every common issue you might run into when starting the servers.

---

## 📋 System Prerequisites

Before starting, make sure you have the following installed on your machine:

1. **Node.js**: `v18.0.0` or higher (we recommend `v20.x LTS`).
   - Check version: `node -v`
2. **npm** (comes with Node) or **yarn**:
   - Check version: `npm -v`
3. **MongoDB**:
   - **Local:** MongoDB Community Edition installed and running on port `27017`.
   - **OR Cloud:** A free [MongoDB Atlas](https://www.mongodb.com/atlas) connection string (e.g., `mongodb+srv://...`).
4. **Git**:
   - Check version: `git --version`

---

## 🚀 Quick Setup (Happy Path)

### Step 1: Clone the Project
Open your terminal (PowerShell, Bash, or Command Prompt) and run:
```bash
git clone https://github.com/your-username/ContestAzm-26.git
cd ContestAzm-26
```

---

### Step 2: Configure & Start the Backend

Open a terminal window in the `backend/` folder:

```bash
cd backend

# 1. Install all dependencies
npm install

# 2. Create your .env configuration file
# On Windows PowerShell:
Copy-Item .env.example .env
# On Mac/Linux:
cp .env.example .env
```

Open `.env` in your code editor and verify these values:
```env
PORT=5000
NODE_ENV=development
MONGO_URI=mongodb://localhost:27017/smartsociety
JWT_SECRET=smartsociety_super_secret_jwt_access_key_2026!
JWT_REFRESH_SECRET=smartsociety_super_secret_jwt_refresh_key_2026!
JWT_EXPIRES_IN=1d
JWT_REFRESH_EXPIRES_IN=7d
CORS_ORIGIN=http://localhost:5173
```

Now, launch the backend development server:
```bash
npm run dev
```

You should see:
```
[Server] Connected to MongoDB database: smartsociety
[Server] SmartSociety API Server running on port 5000
```

---

### Step 3: Configure & Start the Frontend

Open a **second terminal window** and navigate to `frontend/`:

```bash
cd frontend

# 1. Install dependencies
npm install

# 2. Create your frontend .env file
# On Windows PowerShell:
Copy-Item .env.example .env
# On Mac/Linux:
cp .env.example .env
```

Verify your `frontend/.env` contains:
```env
VITE_API_URL=http://localhost:5000/api/v1
```

Now, launch the Vite dev server:
```bash
npm run dev
```

Open your browser and navigate to: **`http://localhost:5173`** 🎉

---

## 🔑 Pre-Configured Demo Login Accounts

Once both servers are running, you can log into any of the 4 role panels using these credentials:

| Role | Email | Password | What You Can Do |
|---|---|---|---|
| 👑 **Executive Admin** | `admin@smartsociety.com` | `Admin123!` | Manage society flats, users, billing engine, circulars, audits |
| 👤 **Resident Owner** | `resident@smartsociety.com` | `Resident123!` | Generate QR visitor passes, pay dues, book amenities, submit complaints |
| 🛡️ **Security Guard** | `guard@smartsociety.com` | `Guard123!` | Holographic QR camera scanner, numeric keypads, walk-in register |
| 🔧 **Maintenance Staff** | `staff@smartsociety.com` | `Staff123!` | View assigned work orders (Plumbing, Electrical), SLA timers, remarks |

---

## 🆘 Backend Troubleshooting Guide (When Things Go Wrong)

If your backend doesn't start or gives an error, don't worry! Here is a step-by-step breakdown of the most common issues and how to fix them quickly.

---

### 🔴 Problem 1: `ECONNREFUSED 127.0.0.1:27017` or `MongooseServerSelectionError`

#### Why it happens:
Your Node.js app is trying to connect to MongoDB on port 27017, but the MongoDB service is not running on your computer.

#### How to fix:
* **On Windows:**
  1. Press `Win + R`, type `services.msc` and hit Enter.
  2. Scroll down to find **MongoDB Server** (or `MongoDB`).
  3. Right-click on it and click **Start**.
  4. Alternatively, open PowerShell as Administrator and run:
     ```powershell
     net start MongoDB
     ```
* **On macOS (Homebrew):**
  ```bash
  brew services start mongodb-community
  ```
* **On Linux (Ubuntu/Debian):**
  ```bash
  sudo systemctl start mongod
  sudo systemctl status mongod
  ```
* **Using MongoDB Atlas (Cloud Database — Easiest Alternative):**
  If you don't want to run MongoDB locally, create a free database at [MongoDB Atlas](https://www.mongodb.com/atlas) and update your `backend/.env`:
  ```env
  MONGO_URI=mongodb+srv://<username>:<password>@cluster0.mongodb.net/smartsociety?retryWrites=true&w=majority
  ```

---

### 🔴 Problem 2: `Error: listen EADDRINUSE: address already in use :::5000`

#### Why it happens:
Port `5000` is already being occupied by a previous zombie Node process, Apple AirPlay (on Mac), or another background application.

#### How to fix:
* **Option A — Kill whatever is on port 5000:**
  * **On Windows (PowerShell):**
    ```powershell
    # Find the Process ID (PID)
    netstat -ano | findstr :5000
    
    # Kill the process using its PID (replace <PID> with the actual number from previous command)
    taskkill /PID <PID> /F
    ```
    Or simply use `npx`:
    ```powershell
    npx kill-port 5000
    ```
  * **On macOS / Linux:**
    ```bash
    lsof -ti :5000 | xargs kill -9
    ```

* **Option B — Change the Backend Port:**
  1. In `backend/.env`, change:
     ```env
     PORT=5001
     ```
  2. In `frontend/.env`, update the URL accordingly:
     ```env
     VITE_API_URL=http://localhost:5001/api/v1
     ```
  3. Restart both servers.

---

### 🔴 Problem 3: `JWT_SECRET is missing` or `jwt.sign() requires a secret or private key`

#### Why it happens:
The backend started without reading the `.env` file, or the `.env` file was placed in the wrong folder.

#### How to fix:
1. Make sure your `.env` file is located directly in the root of the **`backend/`** folder (i.e. `ContestAzm-26/backend/.env`), NOT inside `backend/src/`.
2. Check that your `.env` has no quotes or extra spaces around secrets:
   ```env
   JWT_SECRET=smartsociety_super_secret_jwt_access_key_2026!
   JWT_REFRESH_SECRET=smartsociety_super_secret_jwt_refresh_key_2026!
   ```
3. Restart the backend server.

---

### 🔴 Problem 4: `Access to XMLHttpRequest has been blocked by CORS policy` (Frontend Red Error)

#### Why it happens:
The backend is rejecting requests coming from the frontend because the frontend URL doesn't match the allowed origin.

#### How to fix:
1. Check what port your frontend is running on (usually `http://localhost:5173`).
2. Open `backend/.env` and verify:
   ```env
   CORS_ORIGIN=http://localhost:5173
   ```
3. If your Vite server started on a different port like `http://localhost:5174` (because 5173 was busy), update `CORS_ORIGIN=http://localhost:5174` in `backend/.env` and restart the backend.

---

### 🔴 Problem 5: `ENOENT: no such file or directory, open 'uploads/...'` (Avatar / Image Uploads)

#### Why it happens:
When a user uploads a profile picture or complaint photo, Multer looks for the `uploads/` directory inside `backend/`. If it doesn't exist yet, it might throw an error on older Node versions.

#### How to fix:
Create the `uploads/` folder manually inside `backend/`:
```bash
# Inside backend/ directory:
mkdir uploads
```

---

### 🔴 Problem 6: `node-gyp rebuild failed` or Native Module Errors during `npm install`

#### Why it happens:
Occurs if your Node.js version is too new/unstable (e.g. Node 22+ with uncompiled C++ dependencies) or cache is corrupted.

#### How to fix:
```bash
# 1. Clean npm cache
npm cache clean --force

# 2. Delete existing node_modules and lock file
# On Windows PowerShell:
Remove-Item -Recurse -Force node_modules, package-lock.json
# On Mac/Linux:
rm -rf node_modules package-lock.json

# 3. Reinstall cleanly
npm install
```

---

## 🎯 Verification Checklist

Before reporting any issue, verify these 4 quick points:
- [ ] MongoDB is running (`localhost:27017` or Atlas URI connects).
- [ ] Backend is running on `http://localhost:5000` with `[Server] Connected to MongoDB`.
- [ ] Frontend `.env` has `VITE_API_URL=http://localhost:5000/api/v1`.
- [ ] Both terminal windows are kept open while testing.

Happy coding! If you need further help, feel free to reach out to the project maintainers. 🚀
