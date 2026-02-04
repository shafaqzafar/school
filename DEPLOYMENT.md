# 🚀 VPS Deployment Guide

## 📋 Prerequisites
- Ubuntu/Debian VPS with SSH access
- Node.js 18+ & npm
- PostgreSQL 13+
- Nginx (optional but recommended)
- PM2 for process management

## 🛠️ Setup Steps

### 1. Clone Repository
```bash
git clone https://github.com/shafaqzafar/school.git
cd school
```

### 2. Install Dependencies
```bash
# Frontend dependencies
npm install

# Backend dependencies
cd backend
npm install
cd ..
```

### 3. Environment Configuration
Create `.env` files for both frontend and backend:

#### Backend `.env` (in `backend/` folder)
```env
NODE_ENV=production
PORT=59201
DB_HOST=localhost
DB_PORT=5432
DB_NAME=sms_db
DB_USER=postgres
DB_PASSWORD=your_password
JWT_SECRET=your_jwt_secret_key
VITE_API_URL=http://your-vps-ip:59201/api
```

#### Frontend `.env` (in root folder)
```env
VITE_API_URL=http://your-vps-ip:59201/api
VITE_TOKEN_STORAGE=local
VITE_ENABLE_DEMO_AUTH=false
```

### 4. Database Setup
```bash
# Create database
sudo -u postgres createdb sms_db

# Run migrations (if available)
cd backend
npm run migrate
cd ..
```

### 5. Build Frontend
```bash
npm run build
```

### 6. Start Services with PM2
```bash
# Install PM2 globally
npm install -g pm2

# Start backend
cd backend
pm2 start src/server.js --name "sms-backend"

# Serve frontend (if not using Nginx)
cd ..
pm2 serve dist 3000 --name "sms-frontend" --spa

# Save PM2 configuration
pm2 save
pm2 startup
```

### 7. Nginx Configuration (Optional)
Create `/etc/nginx/sites-available/sms`:
```nginx
server {
    listen 80;
    server_name your-domain.com;

    # Frontend
    location / {
        root /path/to/school/dist;
        try_files $uri $uri/ /index.html;
    }

    # Backend API
    location /api {
        proxy_pass http://localhost:59201;
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
```

Enable site:
```bash
sudo ln -s /etc/nginx/sites-available/sms /etc/nginx/sites-enabled/
sudo nginx -t
sudo systemctl restart nginx
```

## 🔧 Important Notes

### Session Persistence
- ✅ Fixed: Users stay logged in after page refresh
- ✅ All roles supported: admin, owner, student, teacher, driver
- ✅ No blank screens on refresh

### Features Included
- ✅ QR Attendance with logs and export
- ✅ Results generation (CSV upload/manual entry)
- ✅ Class merit list with PDF/CSV export
- ✅ Student marksheet/result card with PDF/CSV export

### Troubleshooting
- If frontend shows blank screen: Check `VITE_API_URL` in `.env`
- If backend fails: Check database connection in backend `.env`
- Check logs: `pm2 logs sms-backend` and `pm2 logs sms-frontend`

## 🌐 Access
- Frontend: `http://your-vps-ip:3000` or your domain
- Backend API: `http://your-vps-ip:59201/api`
- Default admin credentials: Check backend seed data or create via registration

## 🔄 Updates
To update to latest version:
```bash
cd /path/to/school
git pull origin main
npm install
cd backend && npm install && cd ..
npm run build
pm2 restart sms-backend sms-frontend
```
