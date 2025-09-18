# 🚀 ShareUpTime Complete Setup Guide

**A beginner-friendly guide to set up the complete ShareUpTime social media platform**

## 📋 Table of Contents
1. [Prerequisites](#prerequisites)
2. [Project Structure Overview](#project-structure-overview)
3. [Backend Setup (Microservices)](#backend-setup-microservices)
4. [Frontend Setup (Web & Mobile)](#frontend-setup-web--mobile)
5. [Running the Complete Platform](#running-the-complete-platform)
6. [Verification Steps](#verification-steps)
7. [Common Issues & Solutions](#common-issues--solutions)

---

## 🔧 Prerequisites

Before starting, ensure you have the following installed:

### Required Software
- **Node.js** (v18 or higher): [Download here](https://nodejs.org/)
- **Git**: [Download here](https://git-scm.com/)
- **Java Development Kit (JDK 17)**: [Download here](https://adoptium.net/)
- **Android Studio**: [Download here](https://developer.android.com/studio)
- **Visual Studio Code**: [Download here](https://code.visualstudio.com/)

### Database Requirements (Optional for Basic Setup)
- **PostgreSQL**: For structured data storage
- **MongoDB**: For content and media storage
- **Redis**: For caching and session management
- **Neo4j**: For social graph relationships

### System Requirements
- **Operating System**: Windows 10/11, macOS, or Linux
- **RAM**: Minimum 8GB (16GB recommended)
- **Storage**: At least 5GB free space
- **Internet**: Stable connection for package downloads

---

## 🏗️ Project Structure Overview

```
ShareUpTime Platform/
├── backend/                    # Microservices backend
│   ├── services/
│   │   ├── api-gateway/       # Main API gateway (Port 3000)
│   │   ├── auth-service/      # Authentication service (Port 3001)
│   │   ├── user-service/      # User management (Port 3002)
│   │   ├── post-service/      # Content management (Port 3003)
│   │   ├── feed-service/      # Timeline generation (Port 3004)
│   │   ├── media-service/     # File uploads (Port 3005)
│   │   └── notification-service/ # Notifications (Port 3006)
│   └── package.json           # Backend dependencies
├── shareuptime-frontend/      # Next.js web application
│   ├── src/
│   │   ├── app/              # Next.js 15 app directory
│   │   ├── components/       # Reusable UI components
│   │   └── services/         # API service layers
│   └── package.json          # Frontend dependencies
├── ShareUpTimeMobile/         # React Native mobile app
│   ├── src/
│   │   ├── components/       # Mobile UI components
│   │   ├── screens/          # App screens
│   │   └── services/         # Mobile API services
│   └── package.json          # Mobile dependencies
└── .env files                 # Environment configurations
```

---

## 🔄 Backend Setup (Microservices)

### Step 1: Clone the Repository
```bash
git clone https://github.com/ruhaverse/Ruhaverse-mehmet.git
cd Ruhaverse-mehmet
```

### Step 2: Install Backend Dependencies
```bash
# Navigate to backend directory
cd backend

# Install main backend dependencies
npm install --ignore-scripts

# Install dependencies for each service
cd services/api-gateway && npm install --ignore-scripts
cd ../auth-service && npm install --ignore-scripts
cd ../user-service && npm install --ignore-scripts
cd ../post-service && npm install --ignore-scripts
cd ../feed-service && npm install --ignore-scripts
cd ../media-service && npm install --ignore-scripts
cd ../notification-service && npm install --ignore-scripts
```

### Step 3: Environment Configuration
Create `.env` files for each service with the following templates:

#### API Gateway (.env)
```env
PORT=3000
NODE_ENV=development
JWT_SECRET=your-jwt-secret-key
AUTH_SERVICE_URL=http://localhost:3001
USER_SERVICE_URL=http://localhost:3002
POST_SERVICE_URL=http://localhost:3003
FEED_SERVICE_URL=http://localhost:3004
MEDIA_SERVICE_URL=http://localhost:3005
NOTIFICATION_SERVICE_URL=http://localhost:3006
```

#### Auth Service (.env)
```env
PORT=3001
NODE_ENV=development
JWT_SECRET=your-jwt-secret-key
JWT_EXPIRES_IN=24h
DATABASE_URL=postgresql://username:password@localhost:5432/shareuptime_auth
```

#### User Service (.env)
```env
PORT=3002
NODE_ENV=development
DATABASE_URL=postgresql://username:password@localhost:5432/shareuptime_users
REDIS_URL=redis://localhost:6379
```

#### Post Service (.env)
```env
PORT=3003
NODE_ENV=development
DATABASE_URL=postgresql://username:password@localhost:5432/shareuptime_posts
MONGODB_URL=mongodb://localhost:27017/shareuptime_content
```

#### Feed Service (.env)
```env
PORT=3004
NODE_ENV=development
REDIS_URL=redis://localhost:6379
POST_SERVICE_URL=http://localhost:3003
USER_SERVICE_URL=http://localhost:3002
```

#### Media Service (.env)
```env
PORT=3005
NODE_ENV=development
MONGODB_URL=mongodb://localhost:27017/shareuptime_media
UPLOAD_PATH=./uploads
MAX_FILE_SIZE=50MB
```

#### Notification Service (.env)
```env
PORT=3006
NODE_ENV=development
REDIS_URL=redis://localhost:6379
WEBSOCKET_PORT=3007
```

### Step 4: Start Backend Services

**Option A: Start All Services Manually**
```bash
# Terminal 1: API Gateway
cd backend/services/api-gateway
node index.js

# Terminal 2: Auth Service
cd backend/services/auth-service
node index.js

# Terminal 3: User Service
cd backend/services/user-service
node index.js

# Terminal 4: Post Service
cd backend/services/post-service
node index.js

# Terminal 5: Feed Service
cd backend/services/feed-service
node index.js

# Terminal 6: Media Service
cd backend/services/media-service
node index.js

# Terminal 7: Notification Service
cd backend/services/notification-service
node index.js
```

**Option B: Use Background Processes (Windows)**
```powershell
# Start all services in background
cd backend/services/api-gateway; Start-Process node -ArgumentList "index.js" -WindowStyle Hidden
cd backend/services/auth-service; Start-Process node -ArgumentList "index.js" -WindowStyle Hidden
cd backend/services/user-service; Start-Process node -ArgumentList "index.js" -WindowStyle Hidden
cd backend/services/post-service; Start-Process node -ArgumentList "index.js" -WindowStyle Hidden
cd backend/services/feed-service; Start-Process node -ArgumentList "index.js" -WindowStyle Hidden
cd backend/services/media-service; Start-Process node -ArgumentList "index.js" -WindowStyle Hidden
cd backend/services/notification-service; Start-Process node -ArgumentList "index.js" -WindowStyle Hidden
```

---

## 💻 Frontend Setup (Web & Mobile)

### Web Application Setup (Next.js)

#### Step 1: Install Dependencies
```bash
cd shareuptime-frontend
npm install
```

#### Step 2: Environment Configuration
Create `.env.local` in the frontend directory:
```env
NEXT_PUBLIC_API_URL=http://localhost:3000
NEXT_PUBLIC_APP_NAME=ShareUpTime
NEXT_PUBLIC_APP_VERSION=1.0.0
```

#### Step 3: Start the Web Application
```bash
npm run dev
```
The web app will be available at: `http://localhost:3007`

### Mobile Application Setup (React Native)

#### Step 1: Install Dependencies
```bash
cd ShareUpTimeMobile
npm install
```

#### Step 2: Android Environment Setup
1. **Install Android Studio**
2. **Set up Android SDK**:
   ```bash
   # Add to your system environment variables
   ANDROID_HOME=C:\Users\YourUsername\AppData\Local\Android\Sdk
   JAVA_HOME=C:\Program Files\Eclipse Adoptium\jdk-17.0.x-hotspot\
   ```

3. **Add to PATH**:
   ```bash
   C:\Users\YourUsername\AppData\Local\Android\Sdk\platform-tools
   C:\Users\YourUsername\AppData\Local\Android\Sdk\emulator
   ```

#### Step 3: Start Android Emulator
```bash
# List available emulators
emulator -list-avds

# Start an emulator
emulator -avd YOUR_EMULATOR_NAME
```

#### Step 4: Run the Mobile App
```bash
# Start Metro bundler
npx react-native start

# In another terminal, run the app
npx react-native run-android
```

---

## 🏃‍♂️ Running the Complete Platform

### Quick Start Checklist

1. ✅ **Start all 7 backend services**
2. ✅ **Start the Next.js web application**
3. ✅ **Start the React Native mobile app**
4. ✅ **Verify all services are healthy**

### Service Health Check
```bash
# Check all services are running
curl http://localhost:3000/health  # API Gateway
curl http://localhost:3001/health  # Auth Service
curl http://localhost:3002/health  # User Service
curl http://localhost:3003/health  # Post Service
curl http://localhost:3004/health  # Feed Service
curl http://localhost:3005/health  # Media Service
curl http://localhost:3006/health  # Notification Service
```

Expected response from each service:
```json
{"status":"healthy","timestamp":"2025-09-18T10:00:00.000Z"}
```

---

## ✅ Verification Steps

### 1. Backend Services Verification
- [ ] API Gateway responds at `http://localhost:3000/health`
- [ ] All 7 services return healthy status
- [ ] No error messages in service logs

### 2. Web Application Verification
- [ ] Web app loads at `http://localhost:3007`
- [ ] Login/register pages are accessible
- [ ] Dashboard loads without errors

### 3. Mobile Application Verification
- [ ] App installs on Android emulator
- [ ] App launches without crashes
- [ ] Basic navigation works

### 4. Integration Verification
- [ ] Web app can communicate with backend API
- [ ] Mobile app can fetch data from backend
- [ ] Authentication works across platforms

---

## 🚨 Common Issues & Solutions

### Issue 1: Port Already in Use
**Error**: `EADDRINUSE: address already in use :::3000`

**Solution**:
```bash
# Find process using the port
netstat -ano | findstr :3000
# Kill the process
taskkill /PID <process-id> /F
```

### Issue 2: Java Version Conflicts
**Error**: `Could not determine java version from 'x.x.x'`

**Solution**:
```bash
# Set correct JAVA_HOME
set JAVA_HOME=C:\Program Files\Eclipse Adoptium\jdk-17.0.x-hotspot\
# Verify Java version
java -version
```

### Issue 3: Android SDK Path Issues
**Error**: `SDK location not found`

**Solution**:
1. Create `local.properties` in Android project root:
```properties
sdk.dir=C:\\Users\\YourUsername\\AppData\\Local\\Android\\Sdk
```

### Issue 4: Metro Bundler Issues
**Error**: `Metro bundler failed to start`

**Solution**:
```bash
# Clear Metro cache
npx react-native start --reset-cache
# Clear npm cache
npm start -- --reset-cache
```

### Issue 5: Database Connection Errors
**Error**: `Connection refused` for databases

**Solution**:
1. Start database services manually or use Docker:
```bash
# For development, you can skip database setup initially
# Services will use in-memory storage as fallback
```

### Issue 6: Windows Path Length Limitations
**Error**: Path too long errors

**Solution**:
1. Move project to shorter path (e.g., `C:\ShareUpTime\`)
2. Enable long path support in Windows

---

## 🎯 Next Steps

Once you have the basic setup running:

1. **Database Setup**: Configure PostgreSQL, MongoDB, Redis, Neo4j
2. **Production Deployment**: Follow deployment guides
3. **Feature Development**: Start building custom features
4. **Testing**: Set up automated testing
5. **Security**: Implement production security measures

---

## 📚 Additional Resources

- [Backend Architecture Documentation](./BACKEND_ARCHITECTURE.md)
- [Frontend Development Guide](./FRONTEND_GUIDE.md)
- [API Reference](./API_DOCUMENTATION.md)
- [Deployment Guide](./DEPLOYMENT.md)
- [Troubleshooting Guide](./TROUBLESHOOTING.md)

---

## 🆘 Support

If you encounter issues not covered in this guide:

1. Check the [Troubleshooting Guide](./TROUBLESHOOTING.md)
2. Review service logs for error messages
3. Ensure all prerequisites are properly installed
4. Verify environment variables are correctly set

---

**🎉 Congratulations!** You now have a complete ShareUpTime social media platform running locally with microservices backend, modern web frontend, and mobile application.