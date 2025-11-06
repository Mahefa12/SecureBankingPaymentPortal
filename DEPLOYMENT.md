# Payment Portal Deployment Guide

This guide provides comprehensive instructions for deploying the Payment Portal application (both frontend and backend) to various platforms and environments.

## 📋 Overview

The Payment Portal consists of two main components:
- **Backend**: Node.js/Express API with Firebase integration
- **Frontend**: React TypeScript application

Both components need to be deployed and configured to work together in production.

## 🔧 Pre-Deployment Checklist

### Backend Requirements
- [ ] Firebase project created and configured
- [ ] Environment variables configured
- [ ] Database (Firestore) set up
- [ ] SSL certificates ready (for custom domains)
- [ ] Domain/subdomain configured (if applicable)

### Frontend Requirements
- [ ] Backend API URL configured
- [ ] Environment variables set
- [ ] Build process tested locally
- [ ] SSL certificates ready (for custom domains)
- [ ] CDN configured (optional but recommended)

### Security Checklist
- [ ] All secrets rotated for production
- [ ] CORS configured properly
- [ ] Rate limiting enabled
- [ ] Security headers configured
- [ ] HTTPS enforced
- [ ] Environment variables secured

## 🚀 Deployment Options

### Option 1: Cloud Platform Deployment (Recommended)

#### Backend Deployment

##### Heroku
```bash
# 1. Install Heroku CLI and login
heroku login

# 2. Create Heroku app
heroku create your-payment-portal-api

# 3. Set environment variables
heroku config:set NODE_ENV=production
heroku config:set PORT=5001
heroku config:set FIREBASE_PROJECT_ID=your-firebase-project-id
heroku config:set FIREBASE_PRIVATE_KEY="-----BEGIN PRIVATE KEY-----\nYOUR_PRIVATE_KEY\n-----END PRIVATE KEY-----\n"
heroku config:set FIREBASE_CLIENT_EMAIL=your-service-account@your-project.iam.gserviceaccount.com
heroku config:set JWT_SECRET=your-super-secret-jwt-key-min-32-chars
heroku config:set JWT_REFRESH_SECRET=your-super-secret-refresh-jwt-key-min-32-chars
heroku config:set ENCRYPTION_KEY=your-super-secret-encryption-key-32-chars
heroku config:set CSRF_SECRET=your-super-secret-csrf-key-min-32-chars

# 4. Deploy
git push heroku main

# 5. Check logs
heroku logs --tail
```

##### Railway
```bash
# 1. Install Railway CLI
npm install -g @railway/cli

# 2. Login and initialize
railway login
railway init

# 3. Set environment variables in Railway dashboard
# Navigate to your project → Variables tab

# 4. Deploy
railway up
```

##### Google Cloud Platform (App Engine)
```yaml
# app.yaml
runtime: nodejs16

env_variables:
  NODE_ENV: production
  FIREBASE_PROJECT_ID: your-firebase-project-id
  # Add other environment variables

automatic_scaling:
  min_instances: 1
  max_instances: 10
```

```bash
# Deploy
gcloud app deploy
```

#### Frontend Deployment

##### Netlify
```bash
# 1. Build the app
npm run build

# 2. Install Netlify CLI
npm install -g netlify-cli

# 3. Login and deploy
netlify login
netlify deploy --prod --dir=build

# Or connect GitHub repository in Netlify dashboard
# Build command: npm run build
# Publish directory: build
```

Environment variables in Netlify:
```
REACT_APP_API_BASE_URL=https://your-payment-portal-api.herokuapp.com
REACT_APP_ENCRYPTION_KEY=your-strong-production-encryption-key
```

##### Vercel
```bash
# 1. Install Vercel CLI
npm i -g vercel

# 2. Deploy
vercel --prod

# Set environment variables in Vercel dashboard
```

##### AWS S3 + CloudFront
```bash
# 1. Build the app
npm run build

# 2. Create S3 bucket
aws s3 mb s3://your-payment-portal-frontend

# 3. Configure bucket for static website hosting
aws s3 website s3://your-payment-portal-frontend --index-document index.html --error-document error.html

# 4. Upload files
aws s3 sync build/ s3://your-payment-portal-frontend --delete

# 5. Create CloudFront distribution (optional but recommended)
# Configure in AWS Console or use AWS CLI
```

### Option 2: VPS/Server Deployment

#### Backend on VPS

```bash
# 1. Connect to your server
ssh user@your-server-ip

# 2. Install Node.js and npm
curl -fsSL https://deb.nodesource.com/setup_16.x | sudo -E bash -
sudo apt-get install -y nodejs

# 3. Install PM2 for process management
sudo npm install -g pm2

# 4. Clone and setup your backend
git clone <your-backend-repo>
cd Backend
npm install
npm run build

# 5. Create ecosystem file for PM2
# ecosystem.config.js
module.exports = {
  apps: [{
    name: 'payment-portal-api',
    script: 'dist/server.js',
    instances: 'max',
    exec_mode: 'cluster',
    env: {
      NODE_ENV: 'production',
      PORT: 5001,
      // Add all your environment variables here
    }
  }]
};

# 6. Start with PM2
pm2 start ecosystem.config.js
pm2 save
pm2 startup

# 7. Setup Nginx reverse proxy
sudo apt install nginx

# /etc/nginx/sites-available/payment-portal-api
server {
    listen 80;
    server_name your-api-domain.com;

    location / {
        proxy_pass http://localhost:5001;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        proxy_cache_bypass $http_upgrade;
    }
}

# Enable site
sudo ln -s /etc/nginx/sites-available/payment-portal-api /etc/nginx/sites-enabled/
sudo nginx -t
sudo systemctl restart nginx

# 8. Setup SSL with Let's Encrypt
sudo apt install certbot python3-certbot-nginx
sudo certbot --nginx -d your-api-domain.com
```

#### Frontend on VPS

```bash
# 1. Build locally and upload
npm run build
scp -r build/* user@your-server-ip:/var/www/html/

# Or build on server
git clone <your-frontend-repo>
cd Frontend
npm install
npm run build
sudo cp -r build/* /var/www/html/

# 2. Configure Nginx for React Router
# /etc/nginx/sites-available/payment-portal-frontend
server {
    listen 80;
    server_name your-domain.com;
    root /var/www/html;
    index index.html;

    location / {
        try_files $uri $uri/ /index.html;
    }

    # Cache static assets
    location ~* \.(js|css|png|jpg|jpeg|gif|ico|svg)$ {
        expires 1y;
        add_header Cache-Control "public, immutable";
    }
}

# Enable site and setup SSL
sudo ln -s /etc/nginx/sites-available/payment-portal-frontend /etc/nginx/sites-enabled/
sudo nginx -t
sudo systemctl restart nginx
sudo certbot --nginx -d your-domain.com
```

### Option 3: Docker Deployment

#### Backend Dockerfile
```dockerfile
FROM node:16-alpine

WORKDIR /app

# Copy package files
COPY package*.json ./

# Install dependencies
RUN npm ci --only=production

# Copy source code
COPY . .

# Build the application
RUN npm run build

# Expose port
EXPOSE 5001

# Start the application
CMD ["npm", "start"]
```

#### Frontend Dockerfile
```dockerfile
# Multi-stage build
FROM node:16-alpine as build

WORKDIR /app
COPY package*.json ./
RUN npm ci --only=production
COPY . .
RUN npm run build

# Production stage
FROM nginx:alpine
COPY --from=build /app/build /usr/share/nginx/html

# Copy custom nginx config
COPY nginx.conf /etc/nginx/conf.d/default.conf

EXPOSE 80
CMD ["nginx", "-g", "daemon off;"]
```

#### Docker Compose
```yaml
version: '3.8'

services:
  backend:
    build: ./Backend
    ports:
      - "5001:5001"
    environment:
      - NODE_ENV=production
      - FIREBASE_PROJECT_ID=${FIREBASE_PROJECT_ID}
      - FIREBASE_PRIVATE_KEY=${FIREBASE_PRIVATE_KEY}
      - FIREBASE_CLIENT_EMAIL=${FIREBASE_CLIENT_EMAIL}
      - JWT_SECRET=${JWT_SECRET}
      - JWT_REFRESH_SECRET=${JWT_REFRESH_SECRET}
      - ENCRYPTION_KEY=${ENCRYPTION_KEY}
      - CSRF_SECRET=${CSRF_SECRET}
    restart: unless-stopped

  frontend:
    build: ./Frontend
    ports:
      - "80:80"
    environment:
      - REACT_APP_API_BASE_URL=http://localhost:5001
      - REACT_APP_ENCRYPTION_KEY=${REACT_APP_ENCRYPTION_KEY}
    depends_on:
      - backend
    restart: unless-stopped
```

Deploy with Docker Compose:
```bash
# Create .env file with all variables
docker-compose up -d
```

## 🔒 Security Configuration

### Environment Variables Security

#### Production Environment Variables Template

**Backend (.env.production)**
```env
NODE_ENV=production
PORT=5001

# Firebase Configuration
FIREBASE_PROJECT_ID=your-production-firebase-project
FIREBASE_PRIVATE_KEY="-----BEGIN PRIVATE KEY-----\nYOUR_PRODUCTION_PRIVATE_KEY\n-----END PRIVATE KEY-----\n"
FIREBASE_CLIENT_EMAIL=your-production-service-account@your-project.iam.gserviceaccount.com

# Security (Generate new strong keys for production)
JWT_SECRET=your-super-strong-production-jwt-secret-min-32-chars
JWT_REFRESH_SECRET=your-super-strong-production-refresh-secret-min-32-chars
ENCRYPTION_KEY=your-super-strong-production-encryption-key-32-chars
CSRF_SECRET=your-super-strong-production-csrf-secret-min-32-chars

# CORS Configuration
CORS_ORIGIN=https://your-production-domain.com

# Rate Limiting
RATE_LIMIT_WINDOW_MS=900000
RATE_LIMIT_MAX_REQUESTS=100
```

**Frontend (.env.production)**
```env
REACT_APP_API_BASE_URL=https://your-production-api-domain.com
REACT_APP_ENCRYPTION_KEY=your-strong-production-frontend-encryption-key
```

### SSL/TLS Configuration

#### Let's Encrypt (Free SSL)
```bash
# Install certbot
sudo apt install certbot python3-certbot-nginx

# Get certificate
sudo certbot --nginx -d your-domain.com -d www.your-domain.com

# Auto-renewal
sudo crontab -e
# Add: 0 12 * * * /usr/bin/certbot renew --quiet
```

#### Custom SSL Certificate
```nginx
server {
    listen 443 ssl http2;
    server_name your-domain.com;

    ssl_certificate /path/to/your/certificate.crt;
    ssl_certificate_key /path/to/your/private.key;
    
    ssl_protocols TLSv1.2 TLSv1.3;
    ssl_ciphers ECDHE-RSA-AES256-GCM-SHA512:DHE-RSA-AES256-GCM-SHA512;
    ssl_prefer_server_ciphers off;
    
    # Security headers
    add_header Strict-Transport-Security "max-age=63072000" always;
    add_header X-Frame-Options DENY;
    add_header X-Content-Type-Options nosniff;
    add_header Referrer-Policy "strict-origin-when-cross-origin";
}
```

## 📊 Monitoring and Logging

### Application Monitoring

#### Backend Monitoring
```javascript
// Add to your backend
const winston = require('winston');

const logger = winston.createLogger({
  level: 'info',
  format: winston.format.combine(
    winston.format.timestamp(),
    winston.format.errors({ stack: true }),
    winston.format.json()
  ),
  transports: [
    new winston.transports.File({ filename: 'error.log', level: 'error' }),
    new winston.transports.File({ filename: 'combined.log' })
  ]
});

// Health check endpoint
app.get('/health', (req, res) => {
  res.status(200).json({
    status: 'healthy',
    timestamp: new Date().toISOString(),
    uptime: process.uptime()
  });
});
```

#### Frontend Monitoring
```javascript
// Error boundary and logging
import * as Sentry from '@sentry/react';

Sentry.init({
  dsn: process.env.REACT_APP_SENTRY_DSN,
  environment: process.env.NODE_ENV
});
```

### Server Monitoring
```bash
# Install monitoring tools
sudo apt install htop iotop nethogs

# Setup log rotation
sudo nano /etc/logrotate.d/payment-portal
```

## 🔄 CI/CD Pipeline

### GitHub Actions Example

**.github/workflows/deploy.yml**
```yaml
name: Deploy Payment Portal

on:
  push:
    branches: [main]

jobs:
  deploy-backend:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v2
      
      - name: Setup Node.js
        uses: actions/setup-node@v2
        with:
          node-version: '16'
          
      - name: Install dependencies
        run: |
          cd Backend
          npm ci
          
      - name: Build
        run: |
          cd Backend
          npm run build
          
      - name: Deploy to Heroku
        uses: akhileshns/heroku-deploy@v3.12.12
        with:
          heroku_api_key: ${{secrets.HEROKU_API_KEY}}
          heroku_app_name: "your-payment-portal-api"
          heroku_email: "your-email@example.com"
          appdir: "Backend"

  deploy-frontend:
    runs-on: ubuntu-latest
    needs: deploy-backend
    steps:
      - uses: actions/checkout@v2
      
      - name: Setup Node.js
        uses: actions/setup-node@v2
        with:
          node-version: '16'
          
      - name: Install and build
        run: |
          cd Frontend
          npm ci
          npm run build
        env:
          REACT_APP_API_BASE_URL: ${{secrets.REACT_APP_API_BASE_URL}}
          REACT_APP_ENCRYPTION_KEY: ${{secrets.REACT_APP_ENCRYPTION_KEY}}
          
      - name: Deploy to Netlify
        uses: nwtgck/actions-netlify@v1.2
        with:
          publish-dir: './Frontend/build'
          production-branch: main
          github-token: ${{secrets.GITHUB_TOKEN}}
          deploy-message: "Deploy from GitHub Actions"
        env:
          NETLIFY_AUTH_TOKEN: ${{secrets.NETLIFY_AUTH_TOKEN}}
          NETLIFY_SITE_ID: ${{secrets.NETLIFY_SITE_ID}}
```

## 🧪 Testing Deployment

### Pre-deployment Testing
```bash
# Backend
cd Backend
npm run build
npm start
curl http://localhost:5001/health

# Frontend
cd Frontend
npm run build
npx serve -s build
```

### Post-deployment Testing
```bash
# Test API endpoints
curl https://your-api-domain.com/health
curl https://your-api-domain.com/api/auth/status

# Test frontend
curl -I https://your-domain.com
```

## 🚨 Troubleshooting

### Common Deployment Issues

#### Backend Issues
1. **Environment variables not loading**
   - Check variable names and values
   - Ensure platform-specific configuration

2. **Firebase connection issues**
   - Verify Firebase project settings
   - Check private key formatting

3. **Port binding issues**
   - Use `process.env.PORT` for cloud platforms
   - Check firewall settings for VPS

#### Frontend Issues
1. **API connection failures**
   - Verify CORS configuration
   - Check API URL in environment variables

2. **Build failures**
   - Clear node_modules and reinstall
   - Check for missing dependencies

3. **Routing issues**
   - Configure server for SPA routing
   - Check nginx configuration

### Rollback Procedures

#### Heroku Rollback
```bash
heroku releases
heroku rollback v123
```

#### Manual Rollback
```bash
# Keep previous version
git tag v1.0.0
git checkout v1.0.0
# Deploy previous version
```

## 📞 Support and Maintenance

### Regular Maintenance Tasks
- [ ] Update dependencies monthly
- [ ] Rotate secrets quarterly
- [ ] Monitor logs weekly
- [ ] Check SSL certificate expiry
- [ ] Review security settings
- [ ] Backup database regularly

### Performance Optimization
- Enable gzip compression
- Use CDN for static assets
- Implement caching strategies
- Monitor and optimize database queries
- Use connection pooling

---

**Note:** This deployment guide covers the most common scenarios. Adjust configurations based on your specific requirements and infrastructure setup.