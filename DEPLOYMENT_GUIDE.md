# 🚀 Deployment Guide

Complete deployment guide for ShareUpTime social media platform

## 📋 Table of Contents

1. [Deployment Overview](#deployment-overview)
2. [Development Environment](#development-environment)
3. [Staging Environment](#staging-environment)
4. [Production Environment](#production-environment)
5. [Docker Deployment](#docker-deployment)
6. [Cloud Platform Deployments](#cloud-platform-deployments)
7. [CI/CD Pipeline Setup](#cicd-pipeline-setup)
8. [Monitoring and Maintenance](#monitoring-and-maintenance)

---

## 🌐 Deployment Overview

ShareUpTime supports multiple deployment strategies:

### Deployment Environments

| Environment | Purpose | URL Example |
|-------------|---------|-------------|
| Development | Local development | http://localhost:3000 |
| Staging | Pre-production testing | https://staging.shareuptime.com |
| Production | Live application | https://shareuptime.com |

### Deployment Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                        Load Balancer                         │
│                    (NGINX/CloudFlare)                       │
└─────────────────────────┬───────────────────────────────────┘
                          │
        ┌─────────────────┼─────────────────┐
        │                 │                 │
        ▼                 ▼                 ▼
┌─────────────┐    ┌─────────────┐    ┌─────────────┐
│Web Frontend │    │API Gateway  │    │Mobile App   │
│(Vercel/     │    │(Railway/    │    │(App Store/  │
│ Netlify)    │    │ Heroku)     │    │ Play Store) │
└─────────────┘    └─────────────┘    └─────────────┘
                          │
        ┌─────────────────┼─────────────────┐
        │                 │                 │
        ▼                 ▼                 ▼
┌─────────────┐    ┌─────────────┐    ┌─────────────┐
│Microservice │    │Microservice │    │Microservice │
│Cluster      │    │Cluster      │    │Cluster      │
│(K8s/Docker) │    │(K8s/Docker) │    │(K8s/Docker) │
└─────────────┘    └─────────────┘    └─────────────┘
                          │
        ┌─────────────────┼─────────────────┐
        │                 │                 │
        ▼                 ▼                 ▼
┌─────────────┐    ┌─────────────┐    ┌─────────────┐
│PostgreSQL   │    │MongoDB      │    │Redis        │
│(AWS RDS)    │    │(Atlas)      │    │(ElastiCache)│
└─────────────┘    └─────────────┘    └─────────────┘
```

---

## 🛠️ Development Environment

### Local Development Setup

**Prerequisites**:
- Node.js 18+
- Docker Desktop
- Git

**Quick Start**:
```bash
# Clone repository
git clone https://github.com/ruhaverse/Ruhaverse-mehmet.git
cd Ruhaverse-mehmet

# Start all services with Docker Compose
docker-compose up -d

# Or start manually
npm run dev:all
```

### Docker Compose for Development

Create `docker-compose.dev.yml`:

```yaml
version: '3.8'
services:
  # Database Services
  postgres:
    image: postgres:15
    environment:
      POSTGRES_DB: shareuptime
      POSTGRES_USER: dev_user
      POSTGRES_PASSWORD: dev_password
    ports:
      - "5432:5432"
    volumes:
      - postgres_data:/var/lib/postgresql/data

  mongodb:
    image: mongo:6
    ports:
      - "27017:27017"
    volumes:
      - mongo_data:/data/db

  redis:
    image: redis:7-alpine
    ports:
      - "6379:6379"

  # Backend Services
  api-gateway:
    build: ./backend/services/api-gateway
    ports:
      - "3000:3000"
    environment:
      - NODE_ENV=development
      - JWT_SECRET=dev-secret-key
    depends_on:
      - postgres
      - redis

  auth-service:
    build: ./backend/services/auth-service
    ports:
      - "3001:3001"
    environment:
      - NODE_ENV=development
      - DATABASE_URL=postgresql://dev_user:dev_password@postgres:5432/shareuptime
    depends_on:
      - postgres

  user-service:
    build: ./backend/services/user-service
    ports:
      - "3002:3002"
    environment:
      - NODE_ENV=development
      - DATABASE_URL=postgresql://dev_user:dev_password@postgres:5432/shareuptime
    depends_on:
      - postgres

  # Frontend Services
  web-frontend:
    build: ./shareuptime-frontend
    ports:
      - "3007:3000"
    environment:
      - NEXT_PUBLIC_API_URL=http://localhost:3000
    depends_on:
      - api-gateway

volumes:
  postgres_data:
  mongo_data:
```

### Development Commands

```bash
# Start all services
docker-compose -f docker-compose.dev.yml up

# Start specific service
docker-compose -f docker-compose.dev.yml up auth-service

# View logs
docker-compose -f docker-compose.dev.yml logs -f

# Stop all services
docker-compose -f docker-compose.dev.yml down
```

---

## 🧪 Staging Environment

### Staging Setup

Staging environment mimics production but with test data and debugging enabled.

**Environment Variables** (`.env.staging`):
```env
NODE_ENV=staging
API_BASE_URL=https://staging-api.shareuptime.com
WEB_BASE_URL=https://staging.shareuptime.com

# Database (staging replicas)
DATABASE_URL=postgresql://staging_user:password@staging-db.example.com:5432/shareuptime_staging
MONGODB_URL=mongodb://staging-mongo.example.com:27017/shareuptime_staging
REDIS_URL=redis://staging-redis.example.com:6379

# Security (use different keys than production)
JWT_SECRET=staging-jwt-secret-key
API_KEY=staging-api-key

# Feature flags
ENABLE_DEBUG_LOGS=true
ENABLE_ANALYTICS=false
RATE_LIMIT_ENABLED=true

# External services (sandbox/test environments)
STRIPE_API_KEY=pk_test_...
SENDGRID_API_KEY=SG.test...
```

### Staging Deployment Script

```bash
#!/bin/bash
# deploy-staging.sh

set -e

echo "🚀 Deploying to Staging Environment..."

# Build and test
npm run test
npm run build

# Deploy backend services
echo "📦 Deploying Backend Services..."
docker build -t shareuptime/api-gateway:staging ./backend/services/api-gateway
docker build -t shareuptime/auth-service:staging ./backend/services/auth-service
docker build -t shareuptime/user-service:staging ./backend/services/user-service

# Push to staging registry
docker push shareuptime/api-gateway:staging
docker push shareuptime/auth-service:staging
docker push shareuptime/user-service:staging

# Deploy frontend
echo "🌐 Deploying Frontend..."
cd shareuptime-frontend
npm run build
npm run deploy:staging

# Run smoke tests
echo "🔍 Running Smoke Tests..."
npm run test:staging

echo "✅ Staging Deployment Complete!"
echo "🌐 Staging URL: https://staging.shareuptime.com"
```

---

## 🏭 Production Environment

### Production Architecture

**High Availability Setup**:
- Load balancer (NGINX/HAProxy)
- Multiple service instances
- Database replicas
- CDN for static assets
- Monitoring and logging

### Production Environment Variables

```env
NODE_ENV=production
API_BASE_URL=https://api.shareuptime.com
WEB_BASE_URL=https://shareuptime.com

# Database (production with replicas)
DATABASE_URL=postgresql://prod_user:secure_password@prod-db-cluster.amazonaws.com:5432/shareuptime
MONGODB_URL=mongodb+srv://prod_user:secure_password@prod-cluster.mongodb.net/shareuptime
REDIS_URL=rediss://prod-redis-cluster.amazonaws.com:6380

# Security (strong production keys)
JWT_SECRET=super-secure-production-jwt-secret-key-256-bits
API_KEY=prod-api-key-with-high-entropy
ENCRYPTION_KEY=aes-256-encryption-key-for-sensitive-data

# Performance
ENABLE_COMPRESSION=true
ENABLE_CACHING=true
CACHE_TTL=3600
MAX_REQUEST_SIZE=50mb

# Monitoring
ENABLE_METRICS=true
LOG_LEVEL=info
SENTRY_DSN=https://sentry-dsn.sentry.io/project-id

# External services (production)
STRIPE_API_KEY=pk_live_...
SENDGRID_API_KEY=SG.live...
AWS_ACCESS_KEY_ID=AKIA...
AWS_SECRET_ACCESS_KEY=...
CDN_URL=https://cdn.shareuptime.com
```

### Production Deployment Script

```bash
#!/bin/bash
# deploy-production.sh

set -e

echo "🏭 Starting Production Deployment..."

# Pre-deployment checks
echo "🔍 Running Pre-deployment Checks..."
npm run test:all
npm run security:audit
npm run performance:test

# Backup current production
echo "💾 Creating Production Backup..."
kubectl create backup prod-backup-$(date +%Y%m%d-%H%M%S)

# Build production images
echo "🏗️ Building Production Images..."
docker build --no-cache -t shareuptime/api-gateway:latest ./backend/services/api-gateway
docker build --no-cache -t shareuptime/auth-service:latest ./backend/services/auth-service
# ... build all services

# Deploy with zero-downtime
echo "🔄 Performing Zero-Downtime Deployment..."
kubectl set image deployment/api-gateway api-gateway=shareuptime/api-gateway:latest
kubectl rollout status deployment/api-gateway --timeout=300s

# Deploy frontend
echo "🌐 Deploying Frontend to CDN..."
cd shareuptime-frontend
npm run build:production
aws s3 sync ./out s3://shareuptime-frontend-bucket --delete
aws cloudfront create-invalidation --distribution-id E1234567890 --paths "/*"

# Health checks
echo "🏥 Running Health Checks..."
./scripts/health-check.sh

echo "✅ Production Deployment Complete!"
echo "🌐 Live URL: https://shareuptime.com"
```

---

## 🐳 Docker Deployment

### Production Dockerfiles

**API Gateway Dockerfile**:
```dockerfile
FROM node:18-alpine AS builder

WORKDIR /app
COPY package*.json ./
RUN npm ci --only=production && npm cache clean --force

FROM node:18-alpine AS runtime

# Add security and performance optimizations
RUN addgroup -g 1001 -S nodejs
RUN adduser -S nextjs -u 1001

WORKDIR /app

# Copy node_modules from builder stage
COPY --from=builder --chown=nextjs:nodejs /app/node_modules ./node_modules
COPY --chown=nextjs:nodejs . .

USER nextjs

EXPOSE 3000

# Health check
HEALTHCHECK --interval=30s --timeout=3s --start-period=5s --retries=3 \
  CMD curl -f http://localhost:3000/health || exit 1

CMD ["node", "index.js"]
```

**Frontend Dockerfile**:
```dockerfile
FROM node:18-alpine AS builder

WORKDIR /app
COPY package*.json ./
RUN npm ci

COPY . .
RUN npm run build

FROM node:18-alpine AS runner

WORKDIR /app

ENV NODE_ENV production

RUN addgroup --system --gid 1001 nodejs
RUN adduser --system --uid 1001 nextjs

COPY --from=builder /app/public ./public
COPY --from=builder --chown=nextjs:nodejs /app/.next/standalone ./
COPY --from=builder --chown=nextjs:nodejs /app/.next/static ./.next/static

USER nextjs

EXPOSE 3000

ENV PORT 3000

CMD ["node", "server.js"]
```

### Kubernetes Deployment

**API Gateway Deployment**:
```yaml
# k8s/api-gateway-deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: api-gateway
  labels:
    app: api-gateway
spec:
  replicas: 3
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 1
      maxUnavailable: 0
  selector:
    matchLabels:
      app: api-gateway
  template:
    metadata:
      labels:
        app: api-gateway
    spec:
      containers:
      - name: api-gateway
        image: shareuptime/api-gateway:latest
        ports:
        - containerPort: 3000
        env:
        - name: NODE_ENV
          value: "production"
        - name: JWT_SECRET
          valueFrom:
            secretKeyRef:
              name: app-secrets
              key: jwt-secret
        resources:
          requests:
            memory: "256Mi"
            cpu: "250m"
          limits:
            memory: "512Mi"
            cpu: "500m"
        livenessProbe:
          httpGet:
            path: /health
            port: 3000
          initialDelaySeconds: 30
          periodSeconds: 10
        readinessProbe:
          httpGet:
            path: /health
            port: 3000
          initialDelaySeconds: 5
          periodSeconds: 5
---
apiVersion: v1
kind: Service
metadata:
  name: api-gateway-service
spec:
  selector:
    app: api-gateway
  ports:
    - protocol: TCP
      port: 80
      targetPort: 3000
  type: LoadBalancer
```

### Docker Compose for Production

```yaml
# docker-compose.prod.yml
version: '3.8'
services:
  nginx:
    image: nginx:alpine
    ports:
      - "80:80"
      - "443:443"
    volumes:
      - ./nginx/nginx.conf:/etc/nginx/nginx.conf
      - ./ssl:/etc/nginx/ssl
    depends_on:
      - api-gateway
      - web-frontend

  api-gateway:
    image: shareuptime/api-gateway:latest
    deploy:
      replicas: 3
      resources:
        limits:
          memory: 512M
        reservations:
          memory: 256M
    environment:
      - NODE_ENV=production
    secrets:
      - jwt_secret
    depends_on:
      - postgres
      - redis

  auth-service:
    image: shareuptime/auth-service:latest
    deploy:
      replicas: 2
    environment:
      - NODE_ENV=production
    secrets:
      - database_url
    depends_on:
      - postgres

  postgres:
    image: postgres:15
    environment:
      POSTGRES_DB: shareuptime
    secrets:
      - postgres_user
      - postgres_password
    volumes:
      - postgres_data:/var/lib/postgresql/data
    deploy:
      resources:
        limits:
          memory: 2G

secrets:
  jwt_secret:
    external: true
  database_url:
    external: true
  postgres_user:
    external: true
  postgres_password:
    external: true

volumes:
  postgres_data:
    driver: local
```

---

## ☁️ Cloud Platform Deployments

### AWS Deployment

**Using AWS ECS**:
```json
{
  "family": "shareuptime-api-gateway",
  "networkMode": "awsvpc",
  "requiresCompatibilities": ["FARGATE"],
  "cpu": "512",
  "memory": "1024",
  "executionRoleArn": "arn:aws:iam::account:role/ecsTaskExecutionRole",
  "taskRoleArn": "arn:aws:iam::account:role/ecsTaskRole",
  "containerDefinitions": [
    {
      "name": "api-gateway",
      "image": "shareuptime/api-gateway:latest",
      "portMappings": [
        {
          "containerPort": 3000,
          "protocol": "tcp"
        }
      ],
      "environment": [
        {
          "name": "NODE_ENV",
          "value": "production"
        }
      ],
      "secrets": [
        {
          "name": "JWT_SECRET",
          "valueFrom": "arn:aws:ssm:region:account:parameter/shareuptime/jwt-secret"
        }
      ],
      "logConfiguration": {
        "logDriver": "awslogs",
        "options": {
          "awslogs-group": "/ecs/shareuptime",
          "awslogs-region": "us-east-1",
          "awslogs-stream-prefix": "api-gateway"
        }
      },
      "healthCheck": {
        "command": ["CMD-SHELL", "curl -f http://localhost:3000/health || exit 1"],
        "interval": 30,
        "timeout": 5,
        "retries": 3
      }
    }
  ]
}
```

### Vercel Deployment (Frontend)

**vercel.json**:
```json
{
  "version": 2,
  "builds": [
    {
      "src": "package.json",
      "use": "@vercel/next"
    }
  ],
  "env": {
    "NEXT_PUBLIC_API_URL": "https://api.shareuptime.com"
  },
  "routes": [
    {
      "src": "/api/(.*)",
      "dest": "https://api.shareuptime.com/api/$1"
    }
  ],
  "headers": [
    {
      "source": "/(.*)",
      "headers": [
        {
          "key": "X-Content-Type-Options",
          "value": "nosniff"
        },
        {
          "key": "X-Frame-Options",
          "value": "DENY"
        },
        {
          "key": "X-XSS-Protection",
          "value": "1; mode=block"
        }
      ]
    }
  ]
}
```

### Railway Deployment (Backend)

**railway.json**:
```json
{
  "$schema": "https://railway.app/railway.schema.json",
  "build": {
    "builder": "DOCKERFILE",
    "dockerfilePath": "Dockerfile"
  },
  "deploy": {
    "startCommand": "node index.js",
    "healthcheckPath": "/health",
    "healthcheckTimeout": 100,
    "restartPolicyType": "ON_FAILURE",
    "restartPolicyMaxRetries": 10
  }
}
```

### Heroku Deployment

**Procfile**:
```
web: node backend/services/api-gateway/index.js
auth: node backend/services/auth-service/index.js
user: node backend/services/user-service/index.js
```

**app.json**:
```json
{
  "name": "ShareUpTime API",
  "description": "Social media platform backend",
  "repository": "https://github.com/ruhaverse/Ruhaverse-mehmet",
  "logo": "https://shareuptime.com/logo.png",
  "keywords": ["node", "express", "social-media"],
  "env": {
    "NODE_ENV": {
      "description": "Node environment",
      "value": "production"
    },
    "JWT_SECRET": {
      "description": "JWT secret key",
      "generator": "secret"
    }
  },
  "addons": [
    "heroku-postgresql:hobby-dev",
    "heroku-redis:hobby-dev"
  ],
  "buildpacks": [
    {
      "url": "heroku/nodejs"
    }
  ]
}
```

---

## 🔄 CI/CD Pipeline Setup

### GitHub Actions Workflow

**.github/workflows/deploy.yml**:
```yaml
name: Deploy ShareUpTime

on:
  push:
    branches: [ main ]
  pull_request:
    branches: [ main ]

jobs:
  test:
    runs-on: ubuntu-latest
    
    services:
      postgres:
        image: postgres:15
        env:
          POSTGRES_PASSWORD: test_password
          POSTGRES_DB: test_db
        options: >-
          --health-cmd pg_isready
          --health-interval 10s
          --health-timeout 5s
          --health-retries 5

    steps:
    - uses: actions/checkout@v3
    
    - name: Setup Node.js
      uses: actions/setup-node@v3
      with:
        node-version: '18'
        cache: 'npm'
    
    - name: Install dependencies
      run: |
        npm ci
        cd backend && npm ci --ignore-scripts
        cd ../shareuptime-frontend && npm ci
    
    - name: Run tests
      run: |
        npm run test:backend
        npm run test:frontend
        npm run test:integration
    
    - name: Run security audit
      run: npm audit --production
    
    - name: Build applications
      run: |
        npm run build:backend
        npm run build:frontend

  deploy-staging:
    needs: test
    runs-on: ubuntu-latest
    if: github.ref == 'refs/heads/develop'
    
    steps:
    - uses: actions/checkout@v3
    
    - name: Deploy to staging
      run: |
        echo "Deploying to staging..."
        # Deploy to staging environment
    
  deploy-production:
    needs: test
    runs-on: ubuntu-latest
    if: github.ref == 'refs/heads/main'
    
    steps:
    - uses: actions/checkout@v3
    
    - name: Deploy to production
      env:
        AWS_ACCESS_KEY_ID: ${{ secrets.AWS_ACCESS_KEY_ID }}
        AWS_SECRET_ACCESS_KEY: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
      run: |
        echo "Deploying to production..."
        # Production deployment steps
```

### GitLab CI/CD Pipeline

**.gitlab-ci.yml**:
```yaml
stages:
  - test
  - build
  - deploy

variables:
  DOCKER_DRIVER: overlay2
  DOCKER_TLS_CERTDIR: "/certs"

test:
  stage: test
  image: node:18
  services:
    - postgres:15
  variables:
    POSTGRES_DB: test_db
    POSTGRES_USER: test_user
    POSTGRES_PASSWORD: test_password
  script:
    - npm ci
    - npm run test:all
    - npm audit --production
  coverage: '/Lines\s*:\s*(\d+\.\d+)%/'
  artifacts:
    reports:
      coverage_report:
        coverage_format: cobertura
        path: coverage/cobertura-coverage.xml

build:
  stage: build
  image: docker:latest
  services:
    - docker:dind
  script:
    - docker build -t $CI_REGISTRY_IMAGE/api-gateway:$CI_COMMIT_SHA ./backend/services/api-gateway
    - docker push $CI_REGISTRY_IMAGE/api-gateway:$CI_COMMIT_SHA

deploy_staging:
  stage: deploy
  image: alpine:latest
  script:
    - apk add --no-cache curl
    - curl -X POST "$WEBHOOK_URL/deploy/staging"
  environment:
    name: staging
    url: https://staging.shareuptime.com
  only:
    - develop

deploy_production:
  stage: deploy
  image: alpine:latest
  script:
    - apk add --no-cache curl
    - curl -X POST "$WEBHOOK_URL/deploy/production"
  environment:
    name: production
    url: https://shareuptime.com
  when: manual
  only:
    - main
```

---

## 📊 Monitoring and Maintenance

### Application Monitoring

**Health Check Endpoints**:
```javascript
// Enhanced health check with metrics
app.get('/health', async (req, res) => {
  const health = {
    status: 'healthy',
    timestamp: new Date().toISOString(),
    version: process.env.npm_package_version,
    uptime: process.uptime(),
    memory: process.memoryUsage(),
    connections: {
      database: await checkDatabaseConnection(),
      redis: await checkRedisConnection(),
      external: await checkExternalServices()
    },
    metrics: {
      requests: requestCount,
      errors: errorCount,
      responseTime: averageResponseTime
    }
  };
  
  const isHealthy = Object.values(health.connections).every(conn => conn.status === 'connected');
  
  res.status(isHealthy ? 200 : 503).json(health);
});
```

**Prometheus Metrics**:
```javascript
const prometheus = require('prom-client');

const httpRequestsTotal = new prometheus.Counter({
  name: 'http_requests_total',
  help: 'Total number of HTTP requests',
  labelNames: ['method', 'route', 'status']
});

const httpRequestDuration = new prometheus.Histogram({
  name: 'http_request_duration_seconds',
  help: 'Duration of HTTP requests in seconds',
  labelNames: ['method', 'route'],
  buckets: [0.1, 0.5, 1, 2, 5]
});
```

### Logging Configuration

**Winston Logger Setup**:
```javascript
const winston = require('winston');

const logger = winston.createLogger({
  level: process.env.LOG_LEVEL || 'info',
  format: winston.format.combine(
    winston.format.timestamp(),
    winston.format.errors({ stack: true }),
    winston.format.json()
  ),
  defaultMeta: { service: 'api-gateway' },
  transports: [
    new winston.transports.File({ filename: 'error.log', level: 'error' }),
    new winston.transports.File({ filename: 'combined.log' }),
    new winston.transports.Console({
      format: winston.format.simple()
    })
  ]
});
```

### Database Maintenance

**Automated Backup Script**:
```bash
#!/bin/bash
# backup-database.sh

DB_NAME="shareuptime"
BACKUP_DIR="/backups"
TIMESTAMP=$(date +%Y%m%d_%H%M%S)

# PostgreSQL backup
pg_dump $DB_NAME | gzip > $BACKUP_DIR/postgres_backup_$TIMESTAMP.sql.gz

# MongoDB backup
mongodump --db $DB_NAME --out $BACKUP_DIR/mongo_backup_$TIMESTAMP

# Upload to S3
aws s3 sync $BACKUP_DIR s3://shareuptime-backups/

# Cleanup old backups (keep last 7 days)
find $BACKUP_DIR -name "*.gz" -mtime +7 -delete
```

### Maintenance Scripts

**Update Dependencies**:
```bash
#!/bin/bash
# update-dependencies.sh

echo "🔄 Updating Dependencies..."

# Backend services
for service in backend/services/*; do
  if [ -d "$service" ]; then
    echo "Updating $service..."
    cd "$service"
    npm update
    npm audit fix
    cd - > /dev/null
  fi
done

# Frontend
cd shareuptime-frontend
npm update
npm audit fix

echo "✅ Dependencies Updated!"
```

**Performance Optimization**:
```bash
#!/bin/bash
# optimize-performance.sh

echo "⚡ Running Performance Optimizations..."

# Clear Redis cache
redis-cli FLUSHALL

# Restart services with optimized settings
export NODE_OPTIONS="--max-old-space-size=2048"

# Run database optimizations
psql -d shareuptime -c "VACUUM ANALYZE;"
psql -d shareuptime -c "REINDEX DATABASE shareuptime;"

echo "✅ Performance Optimizations Complete!"
```

This comprehensive deployment guide covers all aspects of deploying ShareUpTime from development to production environments with proper monitoring and maintenance procedures.