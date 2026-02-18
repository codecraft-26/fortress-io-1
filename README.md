# 🏰 Fortress.io — Secure Digital Asset Management System

> A hackathon-grade, fully-featured DAM system with file locking, auto-expiry, QR sharing, and activity auditing.

---

## 🚀 Quick Start (Local Development)

### Step 1: Prerequisites
Make sure you have these installed:
- [Node.js 18+](https://nodejs.org/) — download and install
- [MongoDB](https://www.mongodb.com/try/download/community) — OR use free cloud MongoDB (see below)

### Step 2: Clone & Install

```bash
# Install backend dependencies
cd backend
npm install

# Install frontend dependencies
cd ../frontend
npm install
```

### Step 3: Configure Environment

```bash
# In the backend folder, copy the example env file
cd backend
cp .env.example .env
```

Open `.env` and set your values:
```
PORT=5000
MONGO_URI=mongodb://localhost:27017/fortressio   # or your MongoDB Atlas URI
JWT_SECRET=change_this_to_a_long_random_string
JWT_EXPIRES_IN=7d
QR_JWT_EXPIRES_IN=300
UPLOAD_DIR=uploads
CLIENT_URL=http://localhost:5173
```

### Step 4: Run Both Servers

**Terminal 1 — Backend:**
```bash
cd backend
npm run dev
```

**Terminal 2 — Frontend:**
```bash
cd frontend
npm run dev
```

Open http://localhost:5173 in your browser. Done! 🎉

---

## ☁️ Free Cloud MongoDB (If You Don't Want Local MongoDB)

1. Go to https://cloud.mongodb.com and sign up free
2. Create a free cluster (M0 tier)
3. Click "Connect" → "Drivers" → copy the connection string
4. It looks like: `mongodb+srv://user:password@cluster.mongodb.net/fortressio`
5. Paste that as your `MONGO_URI` in `.env`

---

## 🌐 Deployment (Railway — Free & Easy)

Railway is the easiest free hosting platform. Here's exactly how to deploy:

### Deploy Backend

1. Go to https://railway.app and sign up with GitHub
2. Click **"New Project"** → **"Deploy from GitHub repo"**
3. Connect your GitHub account and select your repo
4. Click on the deployed service → **"Settings"** → set **Root Directory** to `backend`
5. Go to **"Variables"** tab and add all your `.env` values:
   - `MONGO_URI` = your MongoDB Atlas URI
   - `JWT_SECRET` = your secret
   - `CLIENT_URL` = your frontend URL (add this after deploying frontend)
   - `PORT` = 5000
6. Railway will auto-deploy. Copy the generated URL (like `fortress-backend.up.railway.app`)

### Deploy Frontend

1. In Railway, click **"New"** → **"GitHub Repo"** again
2. Set Root Directory to `frontend`
3. Add variable: `VITE_API_URL` = your backend Railway URL
4. Update `frontend/vite.config.js` proxy to point to your Railway backend URL

### Quick Deploy Alternative: Render.com

**Backend on Render:**
1. Go to https://render.com → New → Web Service
2. Connect repo → Root Directory: `backend`
3. Build command: `npm install`
4. Start command: `node server.js`
5. Add environment variables

**Frontend on Render:**
1. New → Static Site
2. Root Directory: `frontend`
3. Build command: `npm install && npm run build`
4. Publish directory: `dist`

---

## 🐳 Docker Compose (One Command Deploy)

Create a `docker-compose.yml` in the root:

```yaml
version: '3.8'
services:
  mongo:
    image: mongo:7
    volumes:
      - mongo_data:/data/db
    ports:
      - "27017:27017"

  backend:
    build: ./backend
    ports:
      - "5000:5000"
    environment:
      - MONGO_URI=mongodb://mongo:27017/fortressio
      - JWT_SECRET=your_secret_here
      - CLIENT_URL=http://localhost:5173
    depends_on:
      - mongo
    volumes:
      - ./backend/uploads:/app/uploads

  frontend:
    build: ./frontend
    ports:
      - "5173:80"
    depends_on:
      - backend

volumes:
  mongo_data:
```

Add `Dockerfile` to backend:
```dockerfile
FROM node:18-alpine
WORKDIR /app
COPY package*.json ./
RUN npm install
COPY . .
EXPOSE 5000
CMD ["node", "server.js"]
```

Add `Dockerfile` to frontend:
```dockerfile
FROM node:18-alpine AS builder
WORKDIR /app
COPY package*.json ./
RUN npm install
COPY . .
RUN npm run build

FROM nginx:alpine
COPY --from=builder /app/dist /usr/share/nginx/html
EXPOSE 80
```

Then run:
```bash
docker-compose up --build
```

---

## 📡 API Reference

### Auth
| Method | Endpoint | Description |
|--------|----------|-------------|
| POST | `/auth/register` | Register new user |
| POST | `/auth/login` | Login, get JWT token |

### Files
| Method | Endpoint | Description |
|--------|----------|-------------|
| POST | `/files/upload` | Upload file (multipart/form-data) |
| GET | `/files` | List all your files |
| GET | `/files/:id` | Download file |
| DELETE | `/files/:id` | Delete file |
| POST | `/files/:id/lock` | Lock file |
| POST | `/files/:id/unlock` | Unlock file |
| GET | `/files/:id/qr` | Get QR code for temporary download |
| GET | `/files/download/:token` | Download via QR token (no auth) |

### Folders
| Method | Endpoint | Description |
|--------|----------|-------------|
| POST | `/folders` | Create folder |
| GET | `/folders` | List all folders |
| GET | `/folders/:id/files` | Get files in folder |

### Activity
| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/activity/recent` | Get last 20 activity logs |

---

## 🔑 Key Features Explained

### 🔒 Conflict-Free File Locking
- Any user can lock a file they own
- Only the user who locked it can unlock or modify it
- Admins can override locks
- Locked files display who locked them and when

### 💥 Auto-Expiry (Self-Destruct)
- Set a timer (in minutes) during upload
- `node-cron` runs every minute, checks for expired files
- Deletes physical file from disk AND database record
- Activity log records the auto-deletion

### 📱 QR-Based Secure Sharing
- Generates a signed JWT embedded in a QR code
- Link expires after 5 minutes (configurable via `QR_JWT_EXPIRES_IN`)
- Anyone with the QR can download — no auth required
- Activity log records QR generation

### 📊 Visual Activity Audit
- Every action is logged: upload, download, delete, lock, unlock, auto_delete, qr_share
- Activity feed shows user email, action, file name, and timestamp
- Admins see all activity; users see their own

---

## 🏗️ Project Structure

```
fortress-io/
├── backend/
│   ├── models/
│   │   ├── User.js
│   │   ├── File.js
│   │   ├── Folder.js
│   │   └── ActivityLog.js
│   ├── routes/
│   │   ├── auth.js
│   │   ├── files.js
│   │   ├── folders.js
│   │   └── activity.js
│   ├── middleware/
│   │   ├── auth.js        # JWT verification
│   │   └── upload.js      # Multer config
│   ├── uploads/           # Uploaded files stored here
│   ├── server.js          # Entry point + cron jobs
│   ├── .env.example
│   └── package.json
│
└── frontend/
    ├── src/
    │   ├── pages/
    │   │   ├── Login.jsx
    │   │   └── Dashboard.jsx  # Main app
    │   ├── App.jsx
    │   ├── AuthContext.jsx
    │   ├── api.js
    │   └── index.css
    ├── index.html
    ├── vite.config.js
    └── package.json
```

---

## 🛡️ Security Notes

- Passwords hashed with bcrypt (10 rounds)
- JWT tokens for all protected routes
- File access restricted to owner + admins
- Lock enforcement prevents unauthorized modification
- QR tokens are short-lived and signed

---

## ⚡ Hackathon Tips

- Run `npm run dev` in both folders simultaneously
- Use Postman or Thunder Client to test APIs
- MongoDB Compass is a great GUI for viewing data
- The cron job runs every minute — set short destruct timers (1-2 min) for demos
