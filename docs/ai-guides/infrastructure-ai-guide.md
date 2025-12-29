# Infrastructure & DevOps AI Guide

> **Version**: 3.0
> **Last Updated**: December 26, 2025
> **Author**: Sam
> **Target Audience**: DevOps, Infrastructure Engineers, Claude Code

---

## 1. AI-Powered Infrastructure Development

### 1.1 Development Tools

```typescript
interface InfrastructureTools {
  claudeCode: {
    purpose: string[];
    useCases: string[];
  };
  githubCopilot: {
    purpose: string[];
    useCases: string[];
  };
}

const tools: InfrastructureTools = {
  claudeCode: {
    purpose: ["Pulumi configuration (TypeScript)", "GitHub Actions", "Shell scripts", "IaC"],
    useCases: ["AWS setup", "CI/CD pipelines", "Database migrations", "Monitoring"]
  },
  githubCopilot: {
    purpose: ["Code review", "Real-time suggestions"],
    useCases: ["Script completion", "Configuration validation", "Syntax correction"]
  }
};
```

### 1.2 Infrastructure Workflow

```mermaid
graph LR
    A[Define Requirements] --> B[Write Pulumi TypeScript]
    B --> C[pulumi preview]
    C --> D[pulumi up]
    D --> E[Configure Monitoring]
    E --> F[Setup CI/CD]
```

**선택 근거: Pulumi vs Terraform**
- **TypeScript 통일**: Backend/Frontend/IaC 모두 동일 언어
- **타입 안정성**: IDE 자동완성, 컴파일 타임 검증
- **빠른 반복**: `pulumi up` 한 번에 preview + apply (Terraform은 plan/apply 2단계)
- **Lambda 통합**: TypeScript Lambda를 인라인으로 IaC에 정의 가능

---

## 2. Common Infrastructure Tasks

### 2.1 Pulumi Configuration (TypeScript)

**Infrastructure Requirements**:
```typescript
interface AWSInfrastructure {
  compute: {
    ecs: {
      cluster: string;
      service: string;
      taskDefinition: string;
    };
  };
  networking: {
    vpc: {
      cidrBlock: string;
      subnets: {
        public: number;   // 2 subnets
        private: number;  // 2 subnets
        database: number; // 2 subnets (isolated)
        azCount: number;  // 2 AZs
      };
    };
    loadBalancer: string;
  };
  database: {
    rds: {
      engine: "postgres";
      version: "18";      // PostgreSQL 18
      instance: string;
    };
    cache: {
      engine: "valkey";   // Valkey 8.x (Redis-compatible)
      version: "8.0";
      instance: string;
    };
  };
  security: {
    securityGroups: string[];
    secretsManager: {
      anthropicApiKey: string;
      discordWebhook: string;
    };
  };
}
```

**Infrastructure Architecture (3-Tier VPC)**:
```mermaid
graph TB
    subgraph Internet
        A[User]
    end

    subgraph Public_Subnet[Public Subnet - 10.0.1.0/24, 10.0.2.0/24]
        B[ALB]
        C[NAT Gateway]
    end

    subgraph Private_Subnet[Private Subnet - 10.0.11.0/24, 10.0.12.0/24]
        D[ECS Fargate]
    end

    subgraph Database_Subnet[Database Subnet - 10.0.21.0/24, 10.0.22.0/24]
        E[RDS PostgreSQL 18]
        F[Valkey 8.x]
    end

    A --> B
    B --> D
    D --> C
    D --> E
    D --> F
```

**Prompt Template**:
```
Create Pulumi TypeScript configuration for Sage.ai backend infrastructure:

Requirements:
- AWS ECS Fargate cluster
- Application Load Balancer (ALB)
- RDS PostgreSQL 18 (5-year LTS, JSON 30% faster)
- Valkey 8.x cluster (Redis-compatible, Linux Foundation OSS)
- VPC 3-tier architecture:
  * Public subnets (10.0.1.0/24, 10.0.2.0/24) - ALB, NAT Gateway
  * Private subnets (10.0.11.0/24, 10.0.12.0/24) - ECS tasks
  * Database subnets (10.0.21.0/24, 10.0.22.0/24) - RDS, Valkey
- Security groups with least privilege
- AWS Secrets Manager for API keys (Anthropic, Discord webhook)
- Discord alert integration (3 webhooks: errors, performance, business)
- Output: ALB DNS name

Region: us-west-2
Use TypeScript for type safety.
```

**Expected Implementation**:
```typescript
// infra/index.ts
import * as pulumi from "@pulumi/pulumi";
import * as aws from "@pulumi/aws";

const config = new pulumi.Config();
const region = "us-west-2";

// VPC
const vpc = new aws.ec2.Vpc("sage-vpc", {
  cidrBlock: "10.0.0.0/16",
  enableDnsHostnames: true,
  enableDnsSupport: true,
  tags: { Name: "sage-vpc" },
});

// Public Subnets (ALB, NAT Gateway)
const publicSubnets = ["10.0.1.0/24", "10.0.2.0/24"].map((cidr, i) =>
  new aws.ec2.Subnet(`sage-public-${i + 1}`, {
    vpcId: vpc.id,
    cidrBlock: cidr,
    availabilityZone: `us-west-2${String.fromCharCode(97 + i)}`, // us-west-2a, us-west-2b
    mapPublicIpOnLaunch: true,
    tags: { Name: `sage-public-${i + 1}`, Tier: "Public" },
  })
);

// Private Subnets (ECS Tasks)
const privateSubnets = ["10.0.11.0/24", "10.0.12.0/24"].map((cidr, i) =>
  new aws.ec2.Subnet(`sage-private-${i + 1}`, {
    vpcId: vpc.id,
    cidrBlock: cidr,
    availabilityZone: `us-west-2${String.fromCharCode(97 + i)}`,
    tags: { Name: `sage-private-${i + 1}`, Tier: "Private" },
  })
);

// Database Subnets (RDS, Valkey)
const dbSubnets = ["10.0.21.0/24", "10.0.22.0/24"].map((cidr, i) =>
  new aws.ec2.Subnet(`sage-db-${i + 1}`, {
    vpcId: vpc.id,
    cidrBlock: cidr,
    availabilityZone: `us-west-2${String.fromCharCode(97 + i)}`,
    tags: { Name: `sage-db-${i + 1}`, Tier: "Database" },
  })
);

// RDS PostgreSQL 18
const dbSubnetGroup = new aws.rds.SubnetGroup("sage-db-subnet", {
  subnetIds: dbSubnets.map(s => s.id),
  tags: { Name: "sage-db-subnet" },
});

const rds = new aws.rds.Instance("sage-postgres", {
  engine: "postgres",
  engineVersion: "18",
  instanceClass: "db.t3.micro",
  allocatedStorage: 20,
  dbName: "sage",
  username: "sage_admin",
  password: config.requireSecret("dbPassword"),
  dbSubnetGroupName: dbSubnetGroup.name,
  skipFinalSnapshot: true,
  tags: { Name: "sage-postgres-18" },
});

// Valkey 8.x (ElastiCache)
const cacheSubnetGroup = new aws.elasticache.SubnetGroup("sage-cache-subnet", {
  subnetIds: dbSubnets.map(s => s.id),
  tags: { Name: "sage-cache-subnet" },
});

const valkey = new aws.elasticache.Cluster("sage-valkey", {
  engine: "valkey",
  engineVersion: "8.0",
  nodeType: "cache.t3.micro",
  numCacheNodes: 1,
  subnetGroupName: cacheSubnetGroup.name,
  tags: { Name: "sage-valkey-8" },
});

// AWS Secrets Manager - Anthropic API Key
const anthropicSecret = new aws.secretsmanager.Secret("anthropic-api-key", {
  name: "sage/anthropic-api-key",
  description: "Anthropic Claude API key for Sage.ai",
});

// Discord Webhooks
const discordErrorsSecret = new aws.secretsmanager.Secret("discord-errors", {
  name: "sage/discord-webhook-errors",
  description: "Discord webhook for error alerts",
});

const discordPerfSecret = new aws.secretsmanager.Secret("discord-performance", {
  name: "sage/discord-webhook-performance",
  description: "Discord webhook for performance alerts",
});

const discordBusinessSecret = new aws.secretsmanager.Secret("discord-business", {
  name: "sage/discord-webhook-business",
  description: "Discord webhook for business metrics",
});

// Exports
export const vpcId = vpc.id;
export const albDns = ""; // Will be created in ECS setup
export const rdsEndpoint = rds.endpoint;
export const valkeyEndpoint = valkey.cacheNodes[0].address;
```

---

### 2.2 GitHub Actions CI/CD

**Pipeline Requirements**:
```typescript
interface CICDPipeline {
  trigger: {
    branch: string;
    paths: string[];
  };
  steps: {
    checkout: boolean;
    awsAuth: boolean;
    build: {
      dockerfile: string;
      registry: "ECR";
    };
    deploy: {
      target: "ECS";
      cluster: string;
      service: string;
    };
  };
}
```

**Deployment Flow**:
```mermaid
graph LR
    A[Git Push] --> B[GitHub Actions]
    B --> C[Build Docker Image]
    C --> D[Push to ECR]
    D --> E[Update ECS Service]
    E --> F[Rolling Deployment]
```

**Prompt Template**:
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

**Expected Implementation**:
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

### 2.3 Dockerfile Optimization

**Dockerfile Requirements**:
```typescript
interface DockerfileSpec {
  baseImage: "node:20-alpine";
  stages: {
    dependencies: {
      packageManager: "pnpm";
      prisma: boolean;
    };
    build: {
      compile: boolean;
    };
    production: {
      user: "non-root";
      healthcheck: boolean;
      sizeTarget: "<200MB";
    };
  };
}
```

**Multi-stage Build Flow**:
```mermaid
graph LR
    A[Stage 1: Dependencies] --> B[Stage 2: Build]
    B --> C[Stage 3: Production]
    C --> D[Final Image <200MB]
```

**Prompt Template**:
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

**Expected Implementation**:
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

### 2.4 Database Migration Script

**Migration Requirements**:
```typescript
interface MigrationScript {
  preChecks: {
    connectivity: boolean;
    backup: boolean;
  };
  execution: {
    command: "prisma migrate deploy";
    rollbackPlan: boolean;
  };
  logging: {
    path: string;
    level: "info" | "error";
  };
}
```

**Migration Flow**:
```mermaid
graph TB
    A[Check Connectivity] --> B{Connected?}
    B -->|No| C[Exit with Error]
    B -->|Yes| D[Create Backup]
    D --> E[Deploy Migration]
    E --> F{Success?}
    F -->|Yes| G[Log Success]
    F -->|No| H[Rollback Available]
```

**Prompt Template**:
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

**Expected Implementation**:
```bash
#!/bin/bash
# scripts/migrate.sh

set -e  # Exit on error

LOG_FILE="/var/log/backup.log"
S3_BUCKET="sage-backups"
TIMESTAMP=$(date +%Y%m%d_%H%M%S)
BACKUP_FILE="backup_${TIMESTAMP}.sql"

log() {
  echo "[$(date +'%Y-%m-%d %H:%M:%S')] $1" | tee -a $LOG_FILE
}

log "Checking database connectivity..."
if ! npx prisma db execute --stdin <<< "SELECT 1" &> /dev/null; then
  log "Database connection failed"
  exit 1
fi

log "Database connected"

log "Creating backup..."
BACKUP_FILE="backup_$(date +%Y%m%d_%H%M%S).sql"
pg_dump $DATABASE_URL > $BACKUP_FILE
log "Backup created: $BACKUP_FILE"

log "Deploying migration..."
if npx prisma migrate deploy; then
  log "Migration deployed successfully"
else
  log "Migration failed. Consider rollback."
  log "To rollback, restore from: $BACKUP_FILE"
  exit 1
fi

log "Migration complete!"
```

---

### 2.5 Monitoring & Alerting

**Monitoring Requirements**:
```typescript
interface MonitoringSpec {
  alarms: {
    ecsCpu: {
      threshold: 80;
      evaluationPeriods: 2;
      period: 300;
    };
    rdsCpu: {
      threshold: 80;
      evaluationPeriods: 2;
    };
    apiErrorRate: {
      threshold: 5;
      period: 60;
    };
    responseTime: {
      metric: "p95";
      threshold: 2000;
    };
  };
  notifications: {
    snsTopicName: string;
  };
}
```

**Alert Flow**:
```mermaid
graph LR
    A[CloudWatch Metrics] --> B{Threshold Exceeded?}
    B -->|Yes| C[CloudWatch Alarm]
    C --> D[SNS Topic]
    D --> E[Notification]
```

**Prompt Template**:
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

**Expected Implementation**:
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

## 3. Shell Scripts with AI

### 3.1 Database Backup Script

**Backup Requirements**:
```typescript
interface BackupScript {
  database: {
    tool: "pg_dump";
    compression: "gzip";
  };
  storage: {
    destination: "S3";
    bucket: string;
    retention: {
      days: 7;
      autoCleanup: boolean;
    };
  };
  logging: {
    path: string;
    format: "timestamp + message";
  };
}
```

**Backup Flow**:
```mermaid
graph LR
    A[Start] --> B[Check Connection]
    B --> C[Create Dump]
    C --> D[Compress]
    D --> E[Upload to S3]
    E --> F[Cleanup Old Backups]
    F --> G[Success]
```

**Prompt Template**:
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

**Expected Implementation**:
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

log "Starting database backup..."

# Dump database
log "Creating database dump..."
if pg_dump $DATABASE_URL | gzip > /tmp/$BACKUP_FILE; then
  log "Dump created: /tmp/$BACKUP_FILE"
else
  log "Dump failed"
  exit 1
fi

# Upload to S3
log "Uploading to S3..."
if aws s3 cp /tmp/$BACKUP_FILE s3://$S3_BUCKET/$BACKUP_FILE; then
  log "Uploaded to S3: s3://$S3_BUCKET/$BACKUP_FILE"
else
  log "Upload failed"
  exit 1
fi

# Cleanup local file
rm /tmp/$BACKUP_FILE

# Delete old backups (keep last 7)
log "Cleaning up old backups..."
aws s3 ls s3://$S3_BUCKET/ | \
  grep "backup_" | \
  sort -r | \
  tail -n +8 | \
  awk '{print $4}' | \
  while read file; do
    aws s3 rm s3://$S3_BUCKET/$file
    log "Deleted old backup: $file"
  done

log "Backup complete!"
exit 0
```

---

## 4. Docker Compose for Local Development

**Local Development Requirements**:
```typescript
interface LocalEnvironment {
  services: {
    postgres: {
      version: "16";
      port: 5432;
      persistence: boolean;
    };
    valkey: {
      version: "8.x";
      port: 6379;
    };
    backend: {
      framework: "Nest.js";
      port: 3000;
      hotReload: boolean;
    };
    frontend: {
      build: "Vite";
      port: 5173;
      hotReload: boolean;
    };
  };
  networking: {
    network: string;
  };
}
```

**Local Architecture**:
```mermaid
graph TB
    A[Frontend :5173] --> B[Backend :3000]
    B --> C[PostgreSQL :5432]
    B --> D[Valkey :6379]

    subgraph Docker Network
        A
        B
        C
        D
    end
```

**Prompt Template**:
```
Create docker-compose.yml for Sage.ai local development:

Services:
- PostgreSQL 18 (port 5432, persistent volume)
- Valkey 8.x (port 6379)
- Backend (Nest.js, port 3000, hot reload)
- Frontend (Vite, port 5173, hot reload)

Environment variables from .env
Networks: sage-network
```

**Expected Implementation**:
```yaml
# docker-compose.yml
version: '3.9'

services:
  postgres:
    image: postgres:18-alpine
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

  valkey:
    image: valkey/valkey:8-alpine
    container_name: sage-valkey
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
      - VALKEY_URL=valkey://valkey:6379
      - ANTHROPIC_API_KEY=${ANTHROPIC_API_KEY}
    ports:
      - "3000:3000"
    volumes:
      - ./apps/backend:/app
      - /app/node_modules
    depends_on:
      - postgres
      - valkey
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

## 5. AI-Powered Debugging

### 5.1 Log Analysis

**Log Analysis Process**:
```mermaid
graph LR
    A[Collect Logs] --> B[Identify Patterns]
    B --> C[Analyze Root Cause]
    C --> D[Generate Solution]
    D --> E[Verify Fix]
```

**Prompt Template**:
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

### 5.2 Expected AI Response

```typescript
interface DebugResponse {
  rootCause: {
    issue: "Database connection pool exhausted";
    details: [
      "98/100 connections used",
      "ECS tasks creating too many connections",
      "No connection cleanup or reuse"
    ];
  };
  immediateFix: {
    action: "Increase RDS max_connections";
    commands: string[];
  };
  longTermFix: {
    solutions: [
      "Enable PgBouncer (connection pooler)",
      "Optimize Prisma client instantiation",
      "Set connection pool size per ECS task"
    ];
  };
}
```

**Fix Implementation**:
```sql
-- Immediate Fix: Increase max_connections
ALTER SYSTEM SET max_connections = 200;
SELECT pg_reload_conf();
```

```typescript
// Long-term Fix: Connection pooling
DATABASE_URL="postgresql://user:pass@host:5432/db?connection_limit=5"
```

---

## 6. Best Practices

### 6.1 Infrastructure Management

```typescript
interface BestPractices {
  testing: {
    bad: "terraform apply directly in production";
    good: "Test in staging environment first";
  };
  security: {
    bad: "Hardcode credentials";
    good: "Use environment variables, AWS Secrets Manager";
  };
  versionControl: {
    bad: "Manual AWS Console changes";
    good: "All infrastructure changes via Terraform + Git";
  };
}
```

### 6.2 Change Management Flow

```mermaid
graph LR
    A[Write Terraform] --> B[terraform plan]
    B --> C[Review Changes]
    C --> D[Apply to Staging]
    D --> E{Test Pass?}
    E -->|Yes| F[Apply to Production]
    E -->|No| A
```

---

---

## 7. PostgreSQL 18 & Valkey Best Practices

### 7.1 Why PostgreSQL 18 for Infrastructure?

**선택 근거**:
- **5-year LTS support**: 2030년까지 장기 지원 (운영 안정성)
- **JSON processing 30% faster**: Market data 저장/조회 성능 향상
- **Improved query optimizer**: N+1 query 자동 최적화 향상
- **Security updates**: CVE 대응 장기 지원

**RDS Configuration**:
```typescript
// Pulumi - RDS PostgreSQL 18
const rds = new aws.rds.Instance("sage-postgres", {
  engine: "postgres",
  engineVersion: "18",
  instanceClass: "db.t3.micro",  // MVP: 1 vCPU, 1GB RAM
  allocatedStorage: 20,           // 20GB SSD
  maxAllocatedStorage: 100,       // Auto-scaling up to 100GB
  multiAz: false,                 // MVP: Single AZ (Phase 2: Multi-AZ)
  backupRetentionPeriod: 7,       // 7-day backup
  preferredBackupWindow: "03:00-04:00",  // UTC 3-4AM (Korea 12-1PM)
  tags: { Name: "sage-postgres-18", Version: "18" },
});
```

### 7.2 Why Valkey 8.x for Caching?

**선택 근거**:
- **100% Redis-compatible**: 기존 Redis 클라이언트 라이브러리 그대로 사용
- **Linux Foundation OSS**: 라이센스 안정성 (BSD 3-Clause vs Redis SSPL)
- **Active development**: Redis Labs 이탈 후 커뮤니티 주도 개발
- **AWS ElastiCache 지원**: 2024년 11월부터 공식 지원

**ElastiCache Configuration**:
```typescript
// Pulumi - Valkey 8.x
const valkey = new aws.elasticache.Cluster("sage-valkey", {
  engine: "valkey",
  engineVersion: "8.0",
  nodeType: "cache.t3.micro",  // MVP: 0.5GB RAM
  numCacheNodes: 1,             // MVP: Single node (Phase 2: Replication)
  subnetGroupName: cacheSubnetGroup.name,
  securityGroupIds: [cacheSecurityGroup.id],
  tags: { Name: "sage-valkey-8" },
});

// Cache TTL Strategy
const cacheTTL = {
  marketPrice: 300,       // 5 minutes
  fearGreed: 1800,        // 30 minutes
  userChatList: 30,       // 30 seconds
  aiResponse: 300,        // 5 minutes (identical queries)
};
```

### 7.3 Discord Alert Strategy

**선택 근거**:
- **Channel 분리**: 에러/성능/비즈니스 메트릭 분리 → 노이즈 감소
- **즉각 대응**: CloudWatch Alarm → SNS → Lambda → Discord Webhook
- **비용 효율**: Email SNS보다 Discord가 무료, 실시간성 우수

**Lambda Function for Discord Alerts**:
```typescript
// infra/discord-alert.ts (Pulumi inline Lambda)
import * as aws from "@pulumi/aws";

const discordAlertLambda = new aws.lambda.CallbackFunction("discord-alert", {
  callback: async (event: any) => {
    const message = JSON.parse(event.Records[0].Sns.Message);
    const severity = message.AlarmName.includes("error") ? "🔴 CRITICAL"
                   : message.AlarmName.includes("cpu") ? "🟠 WARNING"
                   : "🟢 INFO";

    const webhookUrl = message.AlarmName.includes("error")
      ? process.env.DISCORD_WEBHOOK_ERRORS
      : message.AlarmName.includes("cpu")
      ? process.env.DISCORD_WEBHOOK_PERFORMANCE
      : process.env.DISCORD_WEBHOOK_BUSINESS;

    await fetch(webhookUrl!, {
      method: "POST",
      headers: { "Content-Type": "application/json" },
      body: JSON.stringify({
        content: `${severity} ${message.AlarmName}`,
        embeds: [{
          title: message.AlarmDescription,
          description: `Threshold: ${message.Trigger.Threshold}`,
          color: severity === "🔴 CRITICAL" ? 0xff0000 : 0xffa500,
        }]
      })
    });
  },
  environment: {
    variables: {
      DISCORD_WEBHOOK_ERRORS: discordErrorsSecret.arn,
      DISCORD_WEBHOOK_PERFORMANCE: discordPerfSecret.arn,
      DISCORD_WEBHOOK_BUSINESS: discordBusinessSecret.arn,
    }
  }
});
```

### 7.4 AWS Secrets Manager Pattern

**선택 근거**:
- **ECS Task Role 통합**: IAM Role 기반 접근 (credential 노출 방지)
- **자동 rotation 지원**: Phase 2에서 API key 자동 갱신
- **Audit trail**: CloudTrail로 접근 기록 추적

**ECS Task Definition with Secrets**:
```typescript
// Pulumi - ECS Task with Secrets Manager
const taskDefinition = new aws.ecs.TaskDefinition("sage-backend-task", {
  family: "sage-backend",
  containerDefinitions: pulumi.interpolate`[{
    "name": "sage-backend",
    "image": "${ecrRepository.repositoryUrl}:latest",
    "secrets": [
      {
        "name": "ANTHROPIC_API_KEY",
        "valueFrom": "${anthropicSecret.arn}"
      },
      {
        "name": "DISCORD_WEBHOOK_ERRORS",
        "valueFrom": "${discordErrorsSecret.arn}"
      }
    ],
    "logConfiguration": {
      "logDriver": "awslogs",
      "options": {
        "awslogs-group": "/ecs/sage-backend",
        "awslogs-region": "us-west-2"
      }
    }
  }]`,
  executionRoleArn: taskExecutionRole.arn,  // Secrets Manager 접근 권한
  taskRoleArn: taskRole.arn,
});
```

---

**Document Version**: 3.0
**Last Updated**: December 26, 2025
**Infrastructure**: AWS ECS + Pulumi (TypeScript)
**Tech Stack**: PostgreSQL 18, Valkey 8.x
**Maintainer**: Sam (dev@5010.tech)

_"Between the zeros and ones"_
