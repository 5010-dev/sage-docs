# Infrastructure & DevOps AI Guide

> **문서 버전**: 1.0
> **최종 수정**: 2025년 12월 19일
> **작성자**: Sam
> **대상 독자**: DevOps, Infrastructure 팀

---

## AI-Powered Infrastructure Development

### Tools

- **Claude Code**: Terraform, GitHub Actions, Shell scripts
- **Cursor**: IaC (Infrastructure as Code), 빠른 스크립트 생성

---

## Prompts for Common Tasks

### 1. Terraform Configuration

**Prompt**:
```
Create Terraform configuration for Sage.ai backend infrastructure:

Requirements:
- AWS ECS Fargate cluster
- Application Load Balancer (ALB)
- RDS PostgreSQL 16
- ElastiCache Redis 7.x
- VPC with 2 public + 2 private subnets (2 AZs)
- Security groups (ALB, ECS, RDS, Redis)
- Output: ALB DNS name

Region: us-west-2
Use best practices for production.
```

**Expected Output**:
```hcl
# terraform/main.tf

provider "aws" {
  region = "us-west-2"
}

# VPC
resource "aws_vpc" "main" {
  cidr_block           = "10.0.0.0/16"
  enable_dns_hostnames = true
  enable_dns_support   = true

  tags = {
    Name = "sage-vpc"
  }
}

# Public Subnets
resource "aws_subnet" "public" {
  count             = 2
  vpc_id            = aws_vpc.main.id
  cidr_block        = "10.0.${count.index}.0/24"
  availability_zone = data.aws_availability_zones.available.names[count.index]
  map_public_ip_on_launch = true

  tags = {
    Name = "sage-public-${count.index + 1}"
  }
}

# ... (rest of configuration)
```

---

### 2. GitHub Actions CI/CD

**Prompt**:
```
Create a GitHub Actions workflow for deploying Sage.ai backend to ECS:

Steps:
1. Checkout code
2. Configure AWS credentials (secrets)
3. Login to ECR
4. Build Docker image
5. Push to ECR (tag: git SHA + latest)
6. Deploy to ECS (force new deployment)

Trigger: Push to main branch, paths: apps/backend/**
```

**Expected Output**:
```yaml
# .github/workflows/deploy-backend.yml
name: Deploy Backend to ECS

on:
  push:
    branches: [main]
    paths: ['apps/backend/**']

jobs:
  deploy:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout code
        uses: actions/checkout@v4

      - name: Configure AWS credentials
        uses: aws-actions/configure-aws-credentials@v4
        with:
          aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID }}
          aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
          aws-region: us-west-2

      - name: Login to Amazon ECR
        id: login-ecr
        uses: aws-actions/amazon-ecr-login@v2

      - name: Build, tag, and push image
        env:
          ECR_REGISTRY: ${{ steps.login-ecr.outputs.registry }}
          ECR_REPOSITORY: sage-backend
          IMAGE_TAG: ${{ github.sha }}
        run: |
          docker build -t $ECR_REGISTRY/$ECR_REPOSITORY:$IMAGE_TAG apps/backend
          docker tag $ECR_REGISTRY/$ECR_REPOSITORY:$IMAGE_TAG $ECR_REGISTRY/$ECR_REPOSITORY:latest
          docker push $ECR_REGISTRY/$ECR_REPOSITORY:$IMAGE_TAG
          docker push $ECR_REGISTRY/$ECR_REPOSITORY:latest

      - name: Deploy to ECS
        run: |
          aws ecs update-service \
            --cluster sage-cluster \
            --service sage-backend \
            --force-new-deployment \
            --region us-west-2
```

---

### 3. Dockerfile Optimization

**Prompt**:
```
Create an optimized multi-stage Dockerfile for Sage.ai Nest.js backend:

Requirements:
- Node.js 20 Alpine base image
- Multi-stage build (dependencies, build, production)
- pnpm as package manager
- Prisma client generation
- Non-root user for security
- Final image size < 200MB
- Health check endpoint
```

**Expected Output**:
```dockerfile
# apps/backend/Dockerfile

# Stage 1: Dependencies
FROM node:20-alpine AS deps

RUN apk add --no-cache libc6-compat openssl

WORKDIR /app

# Install pnpm
RUN corepack enable && corepack prepare pnpm@latest --activate

# Copy package files
COPY package.json pnpm-lock.yaml ./
COPY prisma ./prisma

# Install dependencies
RUN pnpm install --frozen-lockfile

# Generate Prisma Client
RUN pnpm prisma generate

# Stage 2: Build
FROM node:20-alpine AS builder

WORKDIR /app

# Copy dependencies from deps stage
COPY --from=deps /app/node_modules ./node_modules
COPY --from=deps /app/prisma ./prisma

# Copy source code
COPY . .

# Build application
RUN pnpm run build

# Stage 3: Production
FROM node:20-alpine AS runner

RUN apk add --no-cache libc6-compat openssl

WORKDIR /app

# Create non-root user
RUN addgroup --system --gid 1001 nodejs
RUN adduser --system --uid 1001 nestjs

# Copy built application
COPY --from=builder --chown=nestjs:nodejs /app/dist ./dist
COPY --from=builder --chown=nestjs:nodejs /app/node_modules ./node_modules
COPY --from=builder --chown=nestjs:nodejs /app/prisma ./prisma

USER nestjs

EXPOSE 3000

# Health check
HEALTHCHECK --interval=30s --timeout=5s --start-period=60s --retries=3 \
  CMD node -e "require('http').get('http://localhost:3000/health', (r) => {if (r.statusCode !== 200) process.exit(1)})"

CMD ["node", "dist/main.js"]
```

---

### 4. Database Migration Script

**Prompt**:
```
Create a safe Prisma migration deployment script:

Requirements:
- Check database connectivity
- Backup database before migration
- Deploy migration
- Rollback on failure
- Log all steps
- Use environment variables for DATABASE_URL
```

**Expected Output**:
```bash
#!/bin/bash
# scripts/migrate.sh

set -e  # Exit on error

echo "🔍 Checking database connectivity..."
if ! npx prisma db execute --stdin <<< "SELECT 1" &> /dev/null; then
  echo "❌ Database connection failed"
  exit 1
fi

echo "✅ Database connected"

echo "📦 Creating backup..."
BACKUP_FILE="backup_$(date +%Y%m%d_%H%M%S).sql"
pg_dump $DATABASE_URL > $BACKUP_FILE
echo "✅ Backup created: $BACKUP_FILE"

echo "🚀 Deploying migration..."
if npx prisma migrate deploy; then
  echo "✅ Migration deployed successfully"
else
  echo "❌ Migration failed. Consider rollback."
  echo "To rollback, restore from: $BACKUP_FILE"
  exit 1
fi

echo "🎉 Migration complete!"
```

---

### 5. Monitoring & Alerting

**Prompt**:
```
Create CloudWatch alarms for Sage.ai backend:

Alarms:
1. ECS CPU > 80% (2 consecutive periods of 5 minutes)
2. RDS CPU > 80%
3. API error rate > 5% (1 minute)
4. Response time p95 > 2 seconds

Send alerts to SNS topic: sage-alerts

Use Terraform.
```

**Expected Output**:
```hcl
# terraform/monitoring.tf

# SNS Topic for alerts
resource "aws_sns_topic" "alerts" {
  name = "sage-alerts"

  tags = {
    Name = "sage-alerts"
  }
}

# ECS CPU Alarm
resource "aws_cloudwatch_metric_alarm" "ecs_cpu_high" {
  alarm_name          = "sage-backend-cpu-high"
  comparison_operator = "GreaterThanThreshold"
  evaluation_periods  = 2
  metric_name         = "CPUUtilization"
  namespace           = "AWS/ECS"
  period              = 300
  statistic           = "Average"
  threshold           = 80

  dimensions = {
    ClusterName = aws_ecs_cluster.sage.name
    ServiceName = aws_ecs_service.backend.name
  }

  alarm_description = "ECS CPU utilization is above 80%"
  alarm_actions     = [aws_sns_topic.alerts.arn]
}

# RDS CPU Alarm
resource "aws_cloudwatch_metric_alarm" "rds_cpu_high" {
  alarm_name          = "sage-rds-cpu-high"
  comparison_operator = "GreaterThanThreshold"
  evaluation_periods  = 2
  metric_name         = "CPUUtilization"
  namespace           = "AWS/RDS"
  period              = 300
  statistic           = "Average"
  threshold           = 80

  dimensions = {
    DBInstanceIdentifier = aws_db_instance.postgres.id
  }

  alarm_description = "RDS CPU utilization is above 80%"
  alarm_actions     = [aws_sns_topic.alerts.arn]
}

# API Error Rate Alarm
resource "aws_cloudwatch_metric_alarm" "api_error_rate_high" {
  alarm_name          = "sage-api-error-rate-high"
  comparison_operator = "GreaterThanThreshold"
  evaluation_periods  = 1
  metric_name         = "5XXError"
  namespace           = "AWS/ApplicationELB"
  period              = 60
  statistic           = "Average"
  threshold           = 5

  dimensions = {
    LoadBalancer = aws_lb.main.arn_suffix
  }

  alarm_description = "API error rate is above 5%"
  alarm_actions     = [aws_sns_topic.alerts.arn]
}
```

---

## Shell Scripts with AI

### Database Backup Script

**Prompt**:
```
Create a bash script to backup PostgreSQL database to S3:

Requirements:
- Use pg_dump
- Compress with gzip
- Upload to S3 bucket: sage-backups
- Filename: backup_YYYYMMDD_HHMMSS.sql.gz
- Keep last 7 backups (delete older)
- Log to /var/log/backup.log
- Exit codes: 0 (success), 1 (failure)
```

**Expected Script**:
```bash
#!/bin/bash
# scripts/backup-db.sh

set -e

LOG_FILE="/var/log/backup.log"
S3_BUCKET="sage-backups"
TIMESTAMP=$(date +%Y%m%d_%H%M%S)
BACKUP_FILE="backup_${TIMESTAMP}.sql.gz"

log() {
  echo "[$(date +'%Y-%m-%d %H:%M:%S')] $1" | tee -a $LOG_FILE
}

log "🔄 Starting database backup..."

# Dump database
log "📦 Creating database dump..."
if pg_dump $DATABASE_URL | gzip > /tmp/$BACKUP_FILE; then
  log "✅ Dump created: /tmp/$BACKUP_FILE"
else
  log "❌ Dump failed"
  exit 1
fi

# Upload to S3
log "☁️  Uploading to S3..."
if aws s3 cp /tmp/$BACKUP_FILE s3://$S3_BUCKET/$BACKUP_FILE; then
  log "✅ Uploaded to S3: s3://$S3_BUCKET/$BACKUP_FILE"
else
  log "❌ Upload failed"
  exit 1
fi

# Cleanup local file
rm /tmp/$BACKUP_FILE

# Delete old backups (keep last 7)
log "🧹 Cleaning up old backups..."
aws s3 ls s3://$S3_BUCKET/ | \
  grep "backup_" | \
  sort -r | \
  tail -n +8 | \
  awk '{print $4}' | \
  while read file; do
    aws s3 rm s3://$S3_BUCKET/$file
    log "🗑️  Deleted old backup: $file"
  done

log "✅ Backup complete!"
exit 0
```

---

## Docker Compose for Local Development

**Prompt**:
```
Create docker-compose.yml for Sage.ai local development:

Services:
- PostgreSQL 16 (port 5432, persistent volume)
- Redis 7.x (port 6379)
- Backend (Nest.js, port 3000, hot reload)
- Frontend (Vite, port 5173, hot reload)

Environment variables from .env
Networks: sage-network
```

**Expected Output**:
```yaml
# docker-compose.yml
version: '3.9'

services:
  postgres:
    image: postgres:16-alpine
    container_name: sage-postgres
    environment:
      POSTGRES_USER: sage
      POSTGRES_PASSWORD: sage_dev_password
      POSTGRES_DB: sage
    ports:
      - "5432:5432"
    volumes:
      - postgres_data:/var/lib/postgresql/data
    networks:
      - sage-network

  redis:
    image: redis:7-alpine
    container_name: sage-redis
    ports:
      - "6379:6379"
    networks:
      - sage-network

  backend:
    build:
      context: ./apps/backend
      dockerfile: Dockerfile.dev
    container_name: sage-backend
    environment:
      - DATABASE_URL=postgresql://sage:sage_dev_password@postgres:5432/sage
      - REDIS_URL=redis://redis:6379
      - ANTHROPIC_API_KEY=${ANTHROPIC_API_KEY}
    ports:
      - "3000:3000"
    volumes:
      - ./apps/backend:/app
      - /app/node_modules
    depends_on:
      - postgres
      - redis
    networks:
      - sage-network
    command: pnpm run start:dev

  frontend:
    build:
      context: ./apps/frontend
      dockerfile: Dockerfile.dev
    container_name: sage-frontend
    environment:
      - VITE_API_URL=http://localhost:3000
    ports:
      - "5173:5173"
    volumes:
      - ./apps/frontend:/app
      - /app/node_modules
    networks:
      - sage-network
    command: pnpm run dev

volumes:
  postgres_data:

networks:
  sage-network:
    driver: bridge
```

---

## AI-Powered Debugging

### Prompt for Log Analysis

```
Analyze these CloudWatch logs and identify the issue:

Logs:
2024-01-15 10:30:15 [error] PrismaClientKnownRequestError:
Invalid `prisma.chat.findMany()` invocation:
  Error occurred during query execution:
  ConnectorError(ConnectorError { user_facing_error: None, kind: QueryError(PostgresError { code: "53300", message: "too many connections", severity: "FATAL", detail: None, column: None, hint: None }) })

2024-01-15 10:30:16 [error] Repeated 47 times
2024-01-15 10:30:20 [info] ECS CPU: 95%
2024-01-15 10:30:20 [info] RDS connections: 98/100

What's the root cause and how to fix it?
```

**AI Response**:
```
Root Cause:
- Database connection pool exhausted (98/100 connections)
- ECS tasks are creating too many connections
- No connection cleanup or reuse

Immediate Fix:
1. Increase RDS max_connections (temporary):
   ```sql
   ALTER SYSTEM SET max_connections = 200;
   SELECT pg_reload_conf();
   ```

2. Check for connection leaks in code:
   - Ensure Prisma client is singleton
   - Use connection pooling

Long-term Fix:
1. Enable PgBouncer (connection pooler)
2. Optimize Prisma client instantiation
3. Set connection pool size per ECS task:
   ```
   DATABASE_URL="postgresql://user:pass@host:5432/db?connection_limit=5"
   ```
```

---

## Best Practices

### 1. Always Test in Staging

❌ Bad: `terraform apply` directly in production
✅ Good: Test in staging environment first

### 2. Use Variables & Secrets

❌ Bad: Hardcode credentials
✅ Good: Use environment variables, AWS Secrets Manager

### 3. Version Control IaC

❌ Bad: Manual AWS Console changes
✅ Good: All infrastructure changes via Terraform + Git

---

**문서 끝**

_"Between the zeros and ones"_
