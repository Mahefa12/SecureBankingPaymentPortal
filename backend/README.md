# Payment Portal Backend — Install & Run

This README focuses only on how to install and run the backend API locally.

## Prerequisites
- Node.js v16+ and npm v8+
- Firebase Admin credentials (service account)

## Install
```bash
cd Backend
npm install
```

## Configure
Create an environment file and set required variables:
```bash
cp .env.example .env
```
Edit `.env` and set at minimum:
```env
NODE_ENV=development
PORT=5002
ALLOWED_ORIGINS=http://localhost:3000,http://localhost:3001

# Firebase Admin
FIREBASE_PROJECT_ID=your-project-id
FIREBASE_PRIVATE_KEY="-----BEGIN PRIVATE KEY-----\nYOUR_PRIVATE_KEY\n-----END PRIVATE KEY-----\n"
FIREBASE_CLIENT_EMAIL=service-account@your-project.iam.gserviceaccount.com

# Secrets
JWT_SECRET=change-me-32-chars-min
JWT_REFRESH_SECRET=change-me-32-chars-min
ENCRYPTION_KEY=change-me-32-chars-min
CSRF_SECRET=change-me-32-chars-min
```

## Run (Development)
```bash
npm run dev
```
The API runs on `http://localhost:5002/`.

## Run (Production-like)
```bash
npm run build
PORT=5002 ALLOWED_ORIGINS=http://localhost:3000,http://localhost:3001 node dist/server.js
```

## Notes
- Ensure `ALLOWED_ORIGINS` includes the frontend origin you are using.
- Verify the server starts without errors and the health endpoint responds at `http://localhost:5002/health`.