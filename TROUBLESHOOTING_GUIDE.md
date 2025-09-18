# 🔧 Troubleshooting Guide

Complete troubleshooting guide for ShareUpTime platform based on real integration experience

## 📋 Table of Contents

1. [Common Setup Issues](#common-setup-issues)
2. [Backend Service Problems](#backend-service-problems)
3. [Frontend Application Issues](#frontend-application-issues)
4. [Mobile App Problems](#mobile-app-problems)
5. [Database Connection Issues](#database-connection-issues)
6. [Network and API Problems](#network-and-api-problems)
7. [Performance Issues](#performance-issues)
8. [Development Environment Problems](#development-environment-problems)

---

## ⚠️ Common Setup Issues

### Issue 1: Port Already in Use

**Error Message**:
```
Error: listen EADDRINUSE: address already in use :::3000
```

**Cause**: Another process is using the required port

**Solutions**:

**Windows**:
```powershell
# Find process using port 3000
netstat -ano | findstr :3000

# Kill the process (replace PID with actual process ID)
taskkill /PID 1234 /F
```

**macOS/Linux**:
```bash
# Find process using port 3000
lsof -ti :3000

# Kill the process
kill -9 $(lsof -ti :3000)
```

**Alternative**: Change port in environment variables:
```env
PORT=3001
```

### Issue 2: Node.js Version Compatibility

**Error Message**:
```
node: --openssl-legacy-provider is not allowed in NODE_OPTIONS
```

**Cause**: Node.js version compatibility issues

**Solution**:
```bash
# Check Node.js version
node --version

# Install compatible version (18 or higher)
nvm install 18
nvm use 18

# Or update package.json scripts
"scripts": {
  "dev": "NODE_OPTIONS='--openssl-legacy-provider' next dev"
}
```

### Issue 3: Package Installation Failures

**Error Message**:
```
npm ERR! peer dep missing
npm ERR! ERESOLVE unable to resolve dependency tree
```

**Solutions**:
```bash
# Clear npm cache
npm cache clean --force

# Delete node_modules and package-lock.json
rm -rf node_modules package-lock.json

# Install with legacy peer deps
npm install --legacy-peer-deps

# Or use yarn
yarn install

# For husky git hooks issues
npm install --ignore-scripts
```

### Issue 4: Git Repository Issues

**Error Message**:
```
warning: adding embedded git repository
```

**Cause**: Nested git repositories from cloned external projects

**Solution**:
```bash
# Remove nested .git directories
find . -name ".git" -type d -not -path "./.git*" -exec rm -rf {} +

# Or add to .gitignore
echo "**/.git" >> .gitignore

# Clean and re-add files
git rm -r --cached .
git add .
```

---

## 🔧 Backend Service Problems

### Issue 1: Service Won't Start

**Error Message**:
```
Error: Cannot find module 'express'
```

**Cause**: Missing dependencies in service directory

**Solution**:
```bash
# Navigate to specific service
cd backend/services/api-gateway

# Install dependencies
npm install

# Check package.json exists
ls -la package.json

# If missing, copy from parent or reinstall
cp ../../package.json ./
npm install
```

### Issue 2: Database Connection Failures

**Error Message**:
```
Error: connect ECONNREFUSED 127.0.0.1:5432
```

**Cause**: Database service not running or incorrect connection string

**Solutions**:

**For Development (Skip Database)**:
```javascript
// Modify service to use in-memory storage temporarily
const useDatabase = process.env.DATABASE_URL ? true : false;

if (!useDatabase) {
  console.log('Running without database - using in-memory storage');
  // Implement mock data storage
}
```

**Start PostgreSQL**:
```bash
# Windows (if installed)
net start postgresql-x64-14

# macOS
brew services start postgresql

# Linux
sudo systemctl start postgresql

# Docker
docker run -d --name postgres -p 5432:5432 -e POSTGRES_PASSWORD=password postgres
```

### Issue 3: Service Health Check Failures

**Error Message**:
```
GET http://localhost:3001/health net::ERR_CONNECTION_REFUSED
```

**Debugging Steps**:
```bash
# Check if service is running
netstat -tlnp | grep :3001

# Check service logs
cd backend/services/auth-service
node index.js

# Verify service file exists
ls -la index.js

# Test direct connection
curl http://localhost:3001/health
```

### Issue 4: JWT Token Issues

**Error Message**:
```
JsonWebTokenError: invalid signature
```

**Cause**: JWT_SECRET mismatch between services

**Solution**:
```bash
# Ensure same JWT_SECRET across all services
JWT_SECRET="your-secret-key-here"

# Update all .env files
echo "JWT_SECRET=your-secret-key-here" >> backend/services/*/\.env

# Restart all services after updating
```

### Issue 5: CORS Errors

**Error Message**:
```
Access to fetch at 'http://localhost:3000' blocked by CORS policy
```

**Solution**:
```javascript
// Add to API Gateway index.js
const cors = require('cors');

app.use(cors({
  origin: ['http://localhost:3007', 'http://localhost:3000'],
  credentials: true
}));
```

---

## 💻 Frontend Application Issues

### Issue 1: Next.js Build Errors

**Error Message**:
```
Module not found: Can't resolve './components/...'
```

**Solutions**:
```bash
# Check file paths and case sensitivity
ls -la src/components/

# Update tsconfig.json paths
{
  "compilerOptions": {
    "baseUrl": ".",
    "paths": {
      "@/*": ["./src/*"],
      "@/components/*": ["./src/components/*"]
    }
  }
}

# Clear Next.js cache
rm -rf .next
npm run dev
```

### Issue 2: API Connection Problems

**Error Message**:
```
TypeError: Failed to fetch
```

**Debugging**:
```javascript
// Check API base URL in browser console
console.log('API URL:', process.env.NEXT_PUBLIC_API_URL);

// Test API manually
fetch('http://localhost:3000/health')
  .then(res => res.json())
  .then(data => console.log('API Health:', data))
  .catch(err => console.error('API Error:', err));
```

**Solution**:
```env
# Ensure correct API URL in .env.local
NEXT_PUBLIC_API_URL=http://localhost:3000
```

### Issue 3: Hydration Errors

**Error Message**:
```
Warning: Text content did not match. Server: "..." Client: "..."
```

**Solution**:
```typescript
// Use dynamic imports for client-only components
import dynamic from 'next/dynamic';

const ClientOnlyComponent = dynamic(
  () => import('../components/ClientOnly'),
  { ssr: false }
);

// Or use useEffect for client-side only code
useEffect(() => {
  // Client-side only logic here
}, []);
```

### Issue 4: Authentication State Issues

**Error Message**:
```
Cannot read property 'user' of null
```

**Solution**:
```typescript
// Add proper null checks
const { user, isAuthenticated } = useAuthStore();

if (!isAuthenticated || !user) {
  return <LoginForm />;
}

// Or use optional chaining
return <div>Welcome, {user?.name}</div>;
```

---

## 📱 Mobile App Problems

### Issue 1: Android SDK Issues

**Error Message**:
```
ANDROID_HOME is not set and "android" command not in your PATH
```

**Solution**:
```bash
# Windows
set ANDROID_HOME=C:\Users\%USERNAME%\AppData\Local\Android\Sdk
set PATH=%PATH%;%ANDROID_HOME%\tools;%ANDROID_HOME%\platform-tools

# macOS/Linux  
export ANDROID_HOME=$HOME/Library/Android/sdk
export PATH=$PATH:$ANDROID_HOME/emulator
export PATH=$PATH:$ANDROID_HOME/platform-tools
```

### Issue 2: Java Version Conflicts

**Error Message**:
```
Could not determine java version from '11.0.2'
```

**Solutions**:
```bash
# Check Java versions
java -version
javac -version

# Set JAVA_HOME to JDK 17
# Windows
set JAVA_HOME=C:\Program Files\Eclipse Adoptium\jdk-17.0.8.101-hotspot\

# macOS
export JAVA_HOME=/Library/Java/JavaVirtualMachines/temurin-17.jdk/Contents/Home

# Verify
echo $JAVA_HOME
```

### Issue 3: Metro Bundler Issues

**Error Message**:
```
Metro bundler failed to start
```

**Solutions**:
```bash
# Clear Metro cache
npx react-native start --reset-cache

# Clear watchman (macOS)
watchman watch-del-all

# Clear npm cache
npm cache clean --force

# Restart with clean slate
rm -rf node_modules
npm install
npx react-native start --reset-cache
```

### Issue 4: Android Emulator Problems

**Error Message**:
```
No Android emulators available
```

**Solutions**:
```bash
# List available AVDs
emulator -list-avds

# Create new AVD if none exist
# Open Android Studio > AVD Manager > Create Virtual Device

# Start emulator manually
emulator -avd Pixel_4_API_30

# Check if emulator is detected
adb devices
```

### Issue 5: App Registration Errors

**Error Message**:
```
Application shareup has not been registered
```

**Solution**:
```javascript
// Check index.js registration
import { AppRegistry } from 'react-native';
import App from './App';
import { name as appName } from './app.json';

// Register multiple component names for safety
AppRegistry.registerComponent(appName, () => App);
AppRegistry.registerComponent('shareup', () => App);
AppRegistry.registerComponent('ShareUpTimeMobile', () => App);
```

### Issue 6: Windows Path Length Issues

**Error Message**:
```
ENAMETOOLONG: name too long
```

**Solution**:
```bash
# Move project to shorter path
move "C:\very\long\path\ShareUpTimeMobile" "C:\ShareUpTime"

# Enable long paths in Windows (as admin)
New-ItemProperty -Path "HKLM:\SYSTEM\CurrentControlSet\Control\FileSystem" -Name "LongPathsEnabled" -Value 1 -PropertyType DWORD -Force
```

---

## 💾 Database Connection Issues

### Issue 1: PostgreSQL Connection Failed

**Error Message**:
```
error: password authentication failed for user "postgres"
```

**Solutions**:
```bash
# Reset PostgreSQL password
sudo -u postgres psql
ALTER USER postgres PASSWORD 'newpassword';

# Update connection string
DATABASE_URL=postgresql://postgres:newpassword@localhost:5432/shareuptime

# Create database if missing
createdb shareuptime
```

### Issue 2: MongoDB Connection Issues

**Error Message**:
```
MongoNetworkError: failed to connect to server
```

**Solutions**:
```bash
# Start MongoDB service
# Windows
net start MongoDB

# macOS
brew services start mongodb-community

# Linux
sudo systemctl start mongod

# Docker
docker run -d -p 27017:27017 --name mongo mongo:latest
```

### Issue 3: Redis Connection Problems

**Error Message**:
```
Redis connection to 127.0.0.1:6379 failed
```

**Solutions**:
```bash
# Start Redis
# Windows (if installed)
redis-server

# macOS
brew services start redis

# Linux
sudo systemctl start redis

# Docker
docker run -d -p 6379:6379 --name redis redis:alpine
```

---

## 🌐 Network and API Problems

### Issue 1: API Gateway Not Routing

**Error Message**:
```
404 Not Found - GET /api/users
```

**Debug Steps**:
```javascript
// Check API Gateway routing configuration
console.log('Available routes:', app._router.stack);

// Add logging middleware
app.use((req, res, next) => {
  console.log(`${req.method} ${req.url}`);
  next();
});
```

### Issue 2: Service Discovery Issues

**Error Message**:
```
ECONNREFUSED 127.0.0.1:3001
```

**Solution**:
```javascript
// Add health check before routing
const checkServiceHealth = async (serviceUrl) => {
  try {
    const response = await fetch(`${serviceUrl}/health`);
    return response.ok;
  } catch {
    return false;
  }
};

// Implement circuit breaker pattern
const circuitBreaker = require('opossum');
const options = {
  timeout: 3000,
  errorThresholdPercentage: 50,
  resetTimeout: 30000
};
```

### Issue 3: WebSocket Connection Failed

**Error Message**:
```
WebSocket connection to 'ws://localhost:3007/ws' failed
```

**Solution**:
```javascript
// Add WebSocket server to notification service
const WebSocket = require('ws');
const wss = new WebSocket.Server({ port: 3007 });

wss.on('connection', (ws) => {
  console.log('Client connected');
  
  ws.on('message', (message) => {
    console.log('Received:', message);
  });
});
```

---

## ⚡ Performance Issues

### Issue 1: Slow API Responses

**Symptoms**: API calls taking over 5 seconds

**Solutions**:
```javascript
// Add response time logging
app.use((req, res, next) => {
  const start = Date.now();
  res.on('finish', () => {
    const duration = Date.now() - start;
    console.log(`${req.method} ${req.url} - ${duration}ms`);
  });
  next();
});

// Implement caching
const NodeCache = require('node-cache');
const cache = new NodeCache({ stdTTL: 600 }); // 10 minutes

app.get('/api/feed', (req, res) => {
  const cacheKey = `feed_${req.user.id}`;
  const cachedFeed = cache.get(cacheKey);
  
  if (cachedFeed) {
    return res.json(cachedFeed);
  }
  
  // Fetch fresh data and cache it
});
```

### Issue 2: Memory Leaks

**Symptoms**: Increasing memory usage over time

**Solution**:
```javascript
// Monitor memory usage
setInterval(() => {
  const usage = process.memoryUsage();
  console.log('Memory usage:', {
    rss: Math.round(usage.rss / 1024 / 1024) + ' MB',
    heapTotal: Math.round(usage.heapTotal / 1024 / 1024) + ' MB',
    heapUsed: Math.round(usage.heapUsed / 1024 / 1024) + ' MB'
  });
}, 30000);

// Implement proper cleanup
process.on('SIGINT', () => {
  console.log('Cleaning up...');
  // Close database connections
  // Clear intervals/timeouts
  process.exit(0);
});
```

---

## 🔨 Development Environment Problems

### Issue 1: Hot Reload Not Working

**Problem**: Changes not reflecting in browser

**Solutions**:
```bash
# Next.js
# Clear .next cache
rm -rf .next
npm run dev

# React Native
# Clear Metro cache
npx react-native start --reset-cache

# Check file watchers limit (Linux)
echo fs.inotify.max_user_watches=524288 | sudo tee -a /etc/sysctl.conf
sudo sysctl -p
```

### Issue 2: Environment Variables Not Loading

**Problem**: `process.env.VARIABLE_NAME` returns undefined

**Solutions**:
```bash
# Check .env file location and naming
ls -la .env*

# For Next.js, use NEXT_PUBLIC_ prefix for client-side vars
NEXT_PUBLIC_API_URL=http://localhost:3000

# Restart development server after env changes
# Check if variables are loaded
console.log('Environment:', process.env.NODE_ENV);
console.log('API URL:', process.env.NEXT_PUBLIC_API_URL);
```

### Issue 3: TypeScript Errors

**Error Message**:
```
Property 'user' does not exist on type '{}'
```

**Solutions**:
```typescript
// Define proper types
interface User {
  id: string;
  name: string;
  email: string;
}

interface AuthState {
  user: User | null;
  isAuthenticated: boolean;
}

// Use type assertions when necessary
const authData = response.data as AuthState;

// Or create declaration files
// types/global.d.ts
declare global {
  interface Window {
    gtag: Function;
  }
}
```

---

## 🆘 Emergency Debugging Steps

When everything seems broken, follow these steps:

### 1. **Health Check All Services**

```bash
# Check all service health endpoints
curl http://localhost:3000/health  # API Gateway
curl http://localhost:3001/health  # Auth Service
curl http://localhost:3002/health  # User Service
curl http://localhost:3003/health  # Post Service
curl http://localhost:3004/health  # Feed Service
curl http://localhost:3005/health  # Media Service
curl http://localhost:3006/health  # Notification Service
```

### 2. **Check Process Status**

```bash
# Windows
netstat -ano | findstr :3000
netstat -ano | findstr :3001
# ... check all ports

# macOS/Linux
lsof -ti :3000,:3001,:3002,:3003,:3004,:3005,:3006
```

### 3. **Restart Everything**

```bash
# Kill all Node processes
# Windows
taskkill /f /im node.exe

# macOS/Linux
killall node

# Restart services one by one
cd backend/services/api-gateway && node index.js &
cd backend/services/auth-service && node index.js &
# ... start all services
```

### 4. **Check Logs**

```bash
# Enable debug logging
DEBUG=* node index.js

# Check system logs
# Windows
Get-EventLog -LogName Application -Source "Node.js"

# macOS
log show --predicate 'process == "node"' --last 1h

# Linux
journalctl -u your-service-name -f
```

### 5. **Reset Development Environment**

```bash
# Nuclear option - reset everything
rm -rf node_modules package-lock.json .next
npm cache clean --force
npm install
npm run dev
```

---

## 📞 Getting Help

### Before Asking for Help

1. **Check logs** for specific error messages
2. **Try the solutions** in this troubleshooting guide
3. **Search for similar issues** online
4. **Isolate the problem** to specific component/service

### What to Include When Reporting Issues

1. **Exact error message** (copy-paste, don't paraphrase)
2. **Steps to reproduce** the issue
3. **Environment details**:
   - Operating system and version
   - Node.js version (`node --version`)
   - npm/yarn version
   - Browser (for frontend issues)
4. **Relevant code snippets**
5. **Log outputs**

### Useful Commands for Diagnostics

```bash
# System information
node --version
npm --version
git --version

# Check running processes
ps aux | grep node

# Check network connections
netstat -tulnp

# Check disk space
df -h

# Check memory usage
free -m
```

This troubleshooting guide covers the most common issues encountered during ShareUpTime platform setup and development. Keep this handy during development for quick problem resolution!