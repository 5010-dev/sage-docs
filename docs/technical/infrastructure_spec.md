# Sage.ai Infrastructure Specification

> **문서 버전**: 1.0
> **최종 수정**: 2025년 12월 19일
> **작성자**: Sam
> **대상 독자**: DevOps, Infrastructure 팀

---

## Infrastructure Overview

### Architecture Diagram

```
Internet
   │
   ├─> CloudFront (CDN)
   │     └─> S3 (Frontend Static Files)
   │
   └─> Application Load Balancer
         └─> ECS Fargate (Backend)
               ├─> RDS PostgreSQL 16
               ├─> ElastiCache Redis 7.x
               └─> External APIs (Anthropic, CoinGecko)
```

### Technology Stack

| 컴포넌트 | 기술 | 버전/설정 |
|---------|------|----------|
| **Compute** | AWS ECS Fargate | 1.4.0 |
| **Database** | AWS RDS PostgreSQL | 16 |
| **Cache** | AWS ElastiCache Redis | 7.x |
| **Storage** | AWS S3 | - |
| **CDN** | AWS CloudFront | - |
| **Load Balancer** | AWS ALB | - |
| **IaC** | Terraform | 1.6+ |
| **CI/CD** | GitHub Actions | - |
| **Monitoring** | Sentry + CloudWatch | - |
| **Logs** | CloudWatch Logs | - |

---

## AWS Services

### Compute: ECS Fargate

#### Cluster Configuration

```hcl
# terraform/ecs.tf
resource "aws_ecs_cluster" "sage" {
  name = "sage-cluster"

  setting {
    name  = "containerInsights"
    value = "enabled"
  }
}
```

#### Task Definition (Backend)

```hcl
resource "aws_ecs_task_definition" "backend" {
  family                   = "sage-backend"
  network_mode             = "awsvpc"
  requires_compatibilities = ["FARGATE"]
  cpu                      = "512"   # 0.5 vCPU (MVP)
  memory                   = "1024"  # 1 GB (MVP)
  execution_role_arn       = aws_iam_role.ecs_execution.arn
  task_role_arn            = aws_iam_role.ecs_task.arn

  container_definitions = jsonencode([
    {
      name      = "backend"
      image     = "${aws_ecr_repository.backend.repository_url}:latest"
      cpu       = 512
      memory    = 1024
      essential = true

      portMappings = [
        {
          containerPort = 3000
          protocol      = "tcp"
        }
      ]

      environment = [
        { name = "NODE_ENV", value = "production" },
        { name = "DATABASE_URL", value = "postgresql://..." },
        { name = "REDIS_URL", value = "redis://..." }
      ]

      secrets = [
        {
          name      = "ANTHROPIC_API_KEY"
          valueFrom = "${aws_secretsmanager_secret.anthropic_key.arn}"
        }
      ]

      logConfiguration = {
        logDriver = "awslogs"
        options = {
          "awslogs-group"         = "/ecs/sage-backend"
          "awslogs-region"        = "us-west-2"
          "awslogs-stream-prefix" = "ecs"
        }
      }

      healthCheck = {
        command     = ["CMD-SHELL", "curl -f http://localhost:3000/health || exit 1"]
        interval    = 30
        timeout     = 5
        retries     = 3
        startPeriod = 60
      }
    }
  ])
}
```

#### Service Configuration

```hcl
resource "aws_ecs_service" "backend" {
  name            = "sage-backend"
  cluster         = aws_ecs_cluster.sage.id
  task_definition = aws_ecs_task_definition.backend.arn
  desired_count   = 2  # MVP: 2 tasks

  launch_type = "FARGATE"

  network_configuration {
    subnets         = aws_subnet.private[*].id
    security_groups = [aws_security_group.backend.id]
  }

  load_balancer {
    target_group_arn = aws_lb_target_group.backend.arn
    container_name   = "backend"
    container_port   = 3000
  }

  # Auto Scaling
  lifecycle {
    ignore_changes = [desired_count]
  }
}
```

#### Auto Scaling

```hcl
resource "aws_appautoscaling_target" "backend" {
  max_capacity       = 10  # Max 10 tasks
  min_capacity       = 2   # Min 2 tasks
  resource_id        = "service/${aws_ecs_cluster.sage.name}/${aws_ecs_service.backend.name}"
  scalable_dimension = "ecs:service:DesiredCount"
  service_namespace  = "ecs"
}

resource "aws_appautoscaling_policy" "backend_cpu" {
  name               = "backend-cpu-scaling"
  policy_type        = "TargetTrackingScaling"
  resource_id        = aws_appautoscaling_target.backend.resource_id
  scalable_dimension = aws_appautoscaling_target.backend.scalable_dimension
  service_namespace  = aws_appautoscaling_target.backend.service_namespace

  target_tracking_scaling_policy_configuration {
    target_value       = 70.0  # Target 70% CPU
    predefined_metric_specification {
      predefined_metric_type = "ECSServiceAverageCPUUtilization"
    }
    scale_in_cooldown  = 300
    scale_out_cooldown = 60
  }
}
```

---

### Database: RDS PostgreSQL

#### Instance Configuration

```hcl
# terraform/rds.tf
resource "aws_db_instance" "postgres" {
  identifier     = "sage-postgres"
  engine         = "postgres"
  engine_version = "16.1"

  instance_class    = "db.t4g.micro"  # MVP: Free tier
  allocated_storage = 20               # 20 GB (MVP)
  storage_type      = "gp3"
  storage_encrypted = true

  db_name  = "sage"
  username = "sage_admin"
  password = random_password.db_password.result

  vpc_security_group_ids = [aws_security_group.rds.id]
  db_subnet_group_name   = aws_db_subnet_group.main.name

  backup_retention_period = 7
  backup_window          = "03:00-04:00"
  maintenance_window     = "mon:04:00-mon:05:00"

  enabled_cloudwatch_logs_exports = ["postgresql", "upgrade"]

  deletion_protection = true
  skip_final_snapshot = false
  final_snapshot_identifier = "sage-postgres-final-${timestamp()}"

  tags = {
    Name        = "sage-postgres"
    Environment = "production"
  }
}
```

#### Read Replica (Phase 2+)

```hcl
resource "aws_db_instance" "postgres_replica" {
  count = var.enable_read_replica ? 1 : 0

  identifier          = "sage-postgres-replica"
  replicate_source_db = aws_db_instance.postgres.id

  instance_class = "db.t4g.micro"

  # Inherit most settings from primary
}
```

---

### Cache: ElastiCache Redis

#### Cluster Configuration

```hcl
# terraform/elasticache.tf
resource "aws_elasticache_replication_group" "redis" {
  replication_group_id       = "sage-redis"
  replication_group_description = "Sage.ai Redis cluster"

  engine         = "redis"
  engine_version = "7.0"
  node_type      = "cache.t4g.micro"  # MVP: Free tier

  num_cache_clusters = 2  # 1 primary + 1 replica

  port                     = 6379
  parameter_group_name     = "default.redis7"
  subnet_group_name        = aws_elasticache_subnet_group.main.name
  security_group_ids       = [aws_security_group.redis.id]

  automatic_failover_enabled = true
  at_rest_encryption_enabled = true
  transit_encryption_enabled = true

  maintenance_window = "sun:05:00-sun:06:00"
  snapshot_window    = "03:00-04:00"
  snapshot_retention_limit = 5

  tags = {
    Name        = "sage-redis"
    Environment = "production"
  }
}
```

---

### Storage & CDN

#### S3 Bucket (Frontend)

```hcl
# terraform/s3.tf
resource "aws_s3_bucket" "frontend" {
  bucket = "sage-frontend-prod"

  tags = {
    Name        = "sage-frontend"
    Environment = "production"
  }
}

resource "aws_s3_bucket_versioning" "frontend" {
  bucket = aws_s3_bucket.frontend.id

  versioning_configuration {
    status = "Enabled"
  }
}

resource "aws_s3_bucket_public_access_block" "frontend" {
  bucket = aws_s3_bucket.frontend.id

  block_public_acls       = true
  block_public_policy     = true
  ignore_public_acls      = true
  restrict_public_buckets = true
}
```

#### CloudFront Distribution

```hcl
# terraform/cloudfront.tf
resource "aws_cloudfront_distribution" "frontend" {
  origin {
    domain_name = aws_s3_bucket.frontend.bucket_regional_domain_name
    origin_id   = "S3-sage-frontend"

    s3_origin_config {
      origin_access_identity = aws_cloudfront_origin_access_identity.frontend.cloudfront_access_identity_path
    }
  }

  enabled             = true
  is_ipv6_enabled     = true
  default_root_object = "index.html"

  aliases = ["app.sage.ai", "www.sage.ai"]

  default_cache_behavior {
    allowed_methods  = ["GET", "HEAD", "OPTIONS"]
    cached_methods   = ["GET", "HEAD"]
    target_origin_id = "S3-sage-frontend"

    forwarded_values {
      query_string = false
      cookies {
        forward = "none"
      }
    }

    viewer_protocol_policy = "redirect-to-https"
    min_ttl                = 0
    default_ttl            = 3600    # 1 hour
    max_ttl                = 86400   # 1 day

    compress = true
  }

  # SPA routing
  custom_error_response {
    error_code         = 404
    response_code      = 200
    response_page_path = "/index.html"
  }

  restrictions {
    geo_restriction {
      restriction_type = "none"
    }
  }

  viewer_certificate {
    acm_certificate_arn      = aws_acm_certificate.frontend.arn
    ssl_support_method       = "sni-only"
    minimum_protocol_version = "TLSv1.2_2021"
  }

  tags = {
    Name        = "sage-frontend-cdn"
    Environment = "production"
  }
}
```

---

### Load Balancer

#### Application Load Balancer

```hcl
# terraform/alb.tf
resource "aws_lb" "main" {
  name               = "sage-alb"
  internal           = false
  load_balancer_type = "application"
  security_groups    = [aws_security_group.alb.id]
  subnets            = aws_subnet.public[*].id

  enable_deletion_protection = true

  tags = {
    Name        = "sage-alb"
    Environment = "production"
  }
}

resource "aws_lb_target_group" "backend" {
  name        = "sage-backend-tg"
  port        = 3000
  protocol    = "HTTP"
  vpc_id      = aws_vpc.main.id
  target_type = "ip"

  health_check {
    path                = "/health"
    protocol            = "HTTP"
    matcher             = "200"
    interval            = 30
    timeout             = 5
    healthy_threshold   = 2
    unhealthy_threshold = 3
  }

  deregistration_delay = 30
}

resource "aws_lb_listener" "https" {
  load_balancer_arn = aws_lb.main.arn
  port              = "443"
  protocol          = "HTTPS"
  ssl_policy        = "ELBSecurityPolicy-TLS-1-2-2017-01"
  certificate_arn   = aws_acm_certificate.backend.arn

  default_action {
    type             = "forward"
    target_group_arn = aws_lb_target_group.backend.arn
  }
}

resource "aws_lb_listener" "http" {
  load_balancer_arn = aws_lb.main.arn
  port              = "80"
  protocol          = "HTTP"

  default_action {
    type = "redirect"
    redirect {
      port        = "443"
      protocol    = "HTTPS"
      status_code = "HTTP_301"
    }
  }
}
```

---

## Networking

### VPC Configuration

```hcl
# terraform/vpc.tf
resource "aws_vpc" "main" {
  cidr_block           = "10.0.0.0/16"
  enable_dns_hostnames = true
  enable_dns_support   = true

  tags = {
    Name = "sage-vpc"
  }
}

# Public subnets (2 AZs)
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

# Private subnets (2 AZs)
resource "aws_subnet" "private" {
  count             = 2
  vpc_id            = aws_vpc.main.id
  cidr_block        = "10.0.${count.index + 10}.0/24"
  availability_zone = data.aws_availability_zones.available.names[count.index]

  tags = {
    Name = "sage-private-${count.index + 1}"
  }
}

# Internet Gateway
resource "aws_internet_gateway" "main" {
  vpc_id = aws_vpc.main.id

  tags = {
    Name = "sage-igw"
  }
}

# NAT Gateway (for private subnets)
resource "aws_eip" "nat" {
  count  = 2
  domain = "vpc"

  tags = {
    Name = "sage-nat-eip-${count.index + 1}"
  }
}

resource "aws_nat_gateway" "main" {
  count         = 2
  allocation_id = aws_eip.nat[count.index].id
  subnet_id     = aws_subnet.public[count.index].id

  tags = {
    Name = "sage-nat-${count.index + 1}"
  }
}
```

### Security Groups

```hcl
# terraform/security_groups.tf

# ALB Security Group
resource "aws_security_group" "alb" {
  name        = "sage-alb-sg"
  description = "Security group for ALB"
  vpc_id      = aws_vpc.main.id

  ingress {
    from_port   = 80
    to_port     = 80
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }

  ingress {
    from_port   = 443
    to_port     = 443
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }

  egress {
    from_port   = 0
    to_port     = 0
    protocol    = "-1"
    cidr_blocks = ["0.0.0.0/0"]
  }

  tags = {
    Name = "sage-alb-sg"
  }
}

# Backend (ECS) Security Group
resource "aws_security_group" "backend" {
  name        = "sage-backend-sg"
  description = "Security group for ECS backend"
  vpc_id      = aws_vpc.main.id

  ingress {
    from_port       = 3000
    to_port         = 3000
    protocol        = "tcp"
    security_groups = [aws_security_group.alb.id]
  }

  egress {
    from_port   = 0
    to_port     = 0
    protocol    = "-1"
    cidr_blocks = ["0.0.0.0/0"]
  }

  tags = {
    Name = "sage-backend-sg"
  }
}

# RDS Security Group
resource "aws_security_group" "rds" {
  name        = "sage-rds-sg"
  description = "Security group for RDS"
  vpc_id      = aws_vpc.main.id

  ingress {
    from_port       = 5432
    to_port         = 5432
    protocol        = "tcp"
    security_groups = [aws_security_group.backend.id]
  }

  tags = {
    Name = "sage-rds-sg"
  }
}

# Redis Security Group
resource "aws_security_group" "redis" {
  name        = "sage-redis-sg"
  description = "Security group for Redis"
  vpc_id      = aws_vpc.main.id

  ingress {
    from_port       = 6379
    to_port         = 6379
    protocol        = "tcp"
    security_groups = [aws_security_group.backend.id]
  }

  tags = {
    Name = "sage-redis-sg"
  }
}
```

---

## CI/CD Pipeline

### GitHub Actions Workflow

```yaml
# .github/workflows/deploy-backend.yml
name: Deploy Backend

on:
  push:
    branches: [main]
    paths: ['apps/backend/**']

jobs:
  build-and-deploy:
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

      - name: Build, tag, and push image to Amazon ECR
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

### Frontend Deployment

```yaml
# .github/workflows/deploy-frontend.yml
name: Deploy Frontend

on:
  push:
    branches: [main]
    paths: ['apps/frontend/**']

jobs:
  build-and-deploy:
    runs-on: ubuntu-latest

    steps:
      - uses: actions/checkout@v4

      - uses: actions/setup-node@v4
        with:
          node-version: '20'

      - uses: pnpm/action-setup@v2
        with:
          version: 8

      - name: Install dependencies
        run: pnpm install

      - name: Build
        run: pnpm run build
        working-directory: apps/frontend

      - name: Deploy to S3
        run: |
          aws s3 sync apps/frontend/dist s3://sage-frontend-prod --delete

      - name: Invalidate CloudFront
        run: |
          aws cloudfront create-invalidation \
            --distribution-id ${{ secrets.CLOUDFRONT_DISTRIBUTION_ID }} \
            --paths "/*"
```

---

## Monitoring & Logging

### CloudWatch Alarms

```hcl
# terraform/cloudwatch.tf

# ECS CPU Utilization
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

  alarm_actions = [aws_sns_topic.alerts.arn]
}

# RDS CPU Utilization
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

  alarm_actions = [aws_sns_topic.alerts.arn]
}

# Redis Memory
resource "aws_cloudwatch_metric_alarm" "redis_memory_high" {
  alarm_name          = "sage-redis-memory-high"
  comparison_operator = "GreaterThanThreshold"
  evaluation_periods  = 1
  metric_name         = "DatabaseMemoryUsagePercentage"
  namespace           = "AWS/ElastiCache"
  period              = 300
  statistic           = "Average"
  threshold           = 80

  dimensions = {
    ReplicationGroupId = aws_elasticache_replication_group.redis.id
  }

  alarm_actions = [aws_sns_topic.alerts.arn]
}
```

### Sentry Configuration

```typescript
// apps/backend/src/main.ts
import * as Sentry from '@sentry/node';

Sentry.init({
  dsn: process.env.SENTRY_DSN,
  environment: process.env.NODE_ENV,
  tracesSampleRate: 0.1,  // 10% of transactions
  integrations: [
    new Sentry.Integrations.Http({ tracing: true }),
    new Sentry.Integrations.Prisma({ client: prisma })
  ]
});
```

---

## Cost Optimization

### Monthly Cost Estimate (MVP)

| 서비스 | 리소스 | 월 비용 |
|--------|--------|---------|
| **ECS Fargate** | 2 tasks x 0.5 vCPU x 1 GB | $30 |
| **RDS (t4g.micro)** | 20 GB storage | $15 |
| **ElastiCache (t4g.micro)** | 2 nodes | $20 |
| **ALB** | 1 ALB | $20 |
| **S3** | 10 GB storage + 1M requests | $5 |
| **CloudFront** | 100 GB transfer | $10 |
| **Data Transfer** | 100 GB egress | $10 |
| **CloudWatch Logs** | 10 GB logs | $5 |
| **Total** | - | **~$115/월** |

### Cost Saving Strategies

1. **Reserved Instances** (Phase 2+)
   - RDS 예약으로 40% 절감
   - ElastiCache 예약으로 30% 절감

2. **S3 Lifecycle Policies**
   - 30일 이상 로그 → Glacier

3. **CloudFront Cache 최적화**
   - TTL 증가 → Origin 요청 감소

---

## Disaster Recovery

### Backup Strategy

| 리소스 | 백업 주기 | 보관 기간 | RPO | RTO |
|--------|----------|----------|-----|-----|
| **RDS** | Daily | 7 days | 24h | 1h |
| **S3** | Versioning | - | Immediate | Immediate |
| **Redis** | Daily snapshot | 5 days | 24h | 30m |

### Recovery Procedures

**RDS Restore**:
```bash
aws rds restore-db-instance-from-db-snapshot \
  --db-instance-identifier sage-postgres-restored \
  --db-snapshot-identifier sage-postgres-snapshot-2024-01-15
```

**S3 Restore (Version)**:
```bash
aws s3api list-object-versions \
  --bucket sage-frontend-prod \
  --prefix index.html

aws s3api get-object \
  --bucket sage-frontend-prod \
  --key index.html \
  --version-id <version-id> \
  index.html
```

---

## Security

### IAM Roles

```hcl
# ECS Execution Role (pull images, write logs)
resource "aws_iam_role" "ecs_execution" {
  name = "sage-ecs-execution-role"

  assume_role_policy = jsonencode({
    Version = "2012-10-17"
    Statement = [{
      Action = "sts:AssumeRole"
      Effect = "Allow"
      Principal = {
        Service = "ecs-tasks.amazonaws.com"
      }
    }]
  })
}

resource "aws_iam_role_policy_attachment" "ecs_execution" {
  role       = aws_iam_role.ecs_execution.name
  policy_arn = "arn:aws:iam::aws:policy/service-role/AmazonECSTaskExecutionRolePolicy"
}

# ECS Task Role (access AWS services from app)
resource "aws_iam_role" "ecs_task" {
  name = "sage-ecs-task-role"

  assume_role_policy = jsonencode({
    Version = "2012-10-17"
    Statement = [{
      Action = "sts:AssumeRole"
      Effect = "Allow"
      Principal = {
        Service = "ecs-tasks.amazonaws.com"
      }
    }]
  })
}

# Allow task to access Secrets Manager
resource "aws_iam_policy" "secrets_access" {
  name = "sage-secrets-access"

  policy = jsonencode({
    Version = "2012-10-17"
    Statement = [{
      Action = [
        "secretsmanager:GetSecretValue"
      ]
      Effect = "Allow"
      Resource = [
        aws_secretsmanager_secret.anthropic_key.arn
      ]
    }]
  })
}

resource "aws_iam_role_policy_attachment" "task_secrets" {
  role       = aws_iam_role.ecs_task.name
  policy_arn = aws_iam_policy.secrets_access.arn
}
```

### Secrets Management

```hcl
# terraform/secrets.tf
resource "aws_secretsmanager_secret" "anthropic_key" {
  name = "sage/anthropic-api-key"

  tags = {
    Name = "sage-anthropic-key"
  }
}

resource "aws_secretsmanager_secret_version" "anthropic_key" {
  secret_id     = aws_secretsmanager_secret.anthropic_key.id
  secret_string = var.anthropic_api_key  # From tfvars
}
```

---

## Scaling Strategy

### Phase 1: MVP (MAU 5,000)
- ECS: 2 tasks (0.5 vCPU, 1 GB each)
- RDS: db.t4g.micro (1 vCPU, 1 GB)
- Redis: cache.t4g.micro x2

### Phase 2: Growth (MAU 50,000)
- ECS: 5-10 tasks (auto-scaling)
- RDS: db.t4g.small (2 vCPU, 2 GB) + Read Replica
- Redis: cache.t4g.small x2

### Phase 3: Scale (MAU 100,000+)
- ECS: 10-20 tasks
- RDS: db.r6g.large (2 vCPU, 16 GB) + 2 Read Replicas
- Redis: cache.r6g.large x2
- Multi-Region (US + Asia)

---

**문서 끝**

_"Between the zeros and ones"_

---

## Appendix

### A. Terraform Commands

```bash
# Initialize
terraform init

# Plan
terraform plan -out=tfplan

# Apply
terraform apply tfplan

# Destroy (careful!)
terraform destroy
```

### B. AWS CLI Useful Commands

```bash
# List ECS tasks
aws ecs list-tasks --cluster sage-cluster

# Describe task
aws ecs describe-tasks --cluster sage-cluster --tasks <task-id>

# View logs
aws logs tail /ecs/sage-backend --follow
```

### C. Environment Variables

```bash
# AWS
export AWS_REGION=us-west-2
export AWS_ACCOUNT_ID=123456789012

# Terraform
export TF_VAR_anthropic_api_key="sk-ant-..."
export TF_VAR_db_password="..."
```
