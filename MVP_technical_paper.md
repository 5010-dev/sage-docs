# sage.ai MVP 기술 스택

## 개요

| 항목 | 내용 |
|------|------|
| 프로젝트 | sage.ai - AI 투자 멘토 서비스 |
| 단계 | MVP |
| 목표 | 3개월 내 PMF 검증 |

---

## 백엔드

| 컴포넌트 | 기술 | 버전 | 선택 이유 |
|---------|------|------|-----------|
| 프레임워크 | Nest.js | 10.x | 모듈러 구조, TypeScript 네이티브, 멀티 에이전트 병렬 처리 |
| ORM | Prisma | 5.x | 타입 안정성, 직관적 마이그레이션, 높은 생산성 |
| 데이터베이스 | PostgreSQL | 18 | 5년 LTS 지원, JSON 처리 30% 향상, 쿼리 최적화 개선 |
| 캐시 | Valkey | 8.x | Redis 100% 호환, Linux Foundation OSS (라이센스 안정성) |
| 비동기 작업 | BullMQ | 5.x | Memory 추출, 재시도, 분산 처리 |
| 실시간 | SSE (Nest.js 내장) | - | 종목 띠 업데이트, 경량 구현 |
| 스케줄링 | @nestjs/schedule | - | 주기적 가격 폴링 |

---

## 프론트엔드

| 컴포넌트 | 기술 | 버전 | 선택 이유 |
|---------|------|------|-----------|
| 프레임워크 | React | 18.3 | 생태계, 안정성 |
| 빌드 도구 | Vite | 5.x | 빠른 HMR, 최신 표준 |
| 클라이언트 상태 | Zustand | 4.x | 경량, 직관적 API, 보일러플레이트 최소 |
| 서버 상태 | TanStack Query | 5.x | 캐싱, 동기화, 재시도 자동화 |
| 스타일링 | Tailwind CSS | 3.x | 빠른 UI 개발 |
| 라우팅 | React Router | 6.x | SPA 라우팅 |

### 상태 관리 구분

| 상태 유형 | 도구 | 예시 |
|----------|------|------|
| 클라이언트 상태 | Zustand | 사이드바 토글, 현재 대화 ID, 모달 상태 |
| 서버 상태 | TanStack Query | 대화 목록, 메시지, 포트폴리오, 시장 데이터 |

---

## AI

| 컴포넌트 | 기술 | 용도 |
|---------|------|------|
| 개발 도구 | Claude Code (Sonnet 4) | 코드 작성, 아키텍처 설계, 디버깅 |
| 프로덕션 - 메인 | Claude Sonnet 4 | 월렛 버핏 응답 생성 (Persona Agent) |
| 프로덕션 - 보조 | Claude Haiku 4 | Manager, Analyst, Risk Agent |

---

## 외부 API

| 용도 | 서비스 | 비고 |
|------|--------|------|
| 시장 데이터 | CoinGecko API | 무료 티어 (MVP 충분) |
| 공포탐욕지수 | Alternative.me API | Fear & Greed Index |

---

## 인프라

| 컴포넌트 | 기술 | 선택 이유 |
|---------|------|-----------|
| 컨테이너 | AWS ECS Fargate | 관리형, 오토스케일링 |
| 정적 파일 | S3 + CloudFront | CDN, 프론트엔드 배포 |
| 데이터베이스 | RDS (PostgreSQL 18) | 관리형 DB |
| 캐시 | ElastiCache (Valkey 8.x) | 관리형 Valkey (Redis 호환) |
| IaC | Pulumi | TypeScript 기반 IaC, 풀스택 타입 통일, Terraform보다 빠른 반복 |

---

## 인증

| 컴포넌트 | 기술 | 비고 |
|---------|------|------|
| OAuth | Auth.js (@auth/core) | Google OAuth |
| 세션 | JWT 또는 DB 세션 | Auth.js 내장 지원 |

---

## 모니터링

| 컴포넌트 | 기술 | 용도 |
|---------|------|------|
| 에러 트래킹 | Sentry | 프론트/백엔드 에러 수집, 스택트레이스 UI |
| 로그/메트릭 | CloudWatch | 서버 로그, 커스텀 메트릭 (응답시간, 토큰 사용량, 큐 크기) |
| 팀 알림 | Discord Webhooks | 에러/성능/비즈니스 메트릭별 채널 분류 알림 |

---

## 개발 환경

| 컴포넌트 | 기술 |
|---------|------|
| 패키지 매니저 | pnpm |
| 모노레포 | Turborepo (선택) |
| 린터 | ESLint + Prettier |
| 테스트 | Jest + Supertest |
| CI/CD | GitHub Actions |

---

## 아키텍처

| 항목 | 선택 |
|------|------|
| 아키텍처 스타일 | Layered + Domain 분리 (Clean Lite) |
| API 스타일 | REST |
| 실시간 통신 | SSE |

---

## 비동기 처리 구분

| 작업 | 처리 방식 | 기술 |
|------|----------|------|
| 가격 데이터 폴링 | 주기적 Cron | @nestjs/schedule |
| Memory 추출/압축 | 백그라운드 큐 | BullMQ |
| 급변 알림 발송 | 백그라운드 큐 | BullMQ |

---

## 확장 시 고려 (v2.0 이후)

| 현재 | 확장 시 |
|------|--------|
| ECS Fargate | EKS (트래픽 증가 시) |
| BullMQ | Temporal (워크플로우 복잡화 시) |
| CoinGecko | Binance WebSocket (실시간 필요 시) |
| 단일 리전 | 멀티 리전 (글로벌 확장 시) |

---

---

**Last Updated**: 2025년 12월 26일
**Version**: 5.0 - PostgreSQL 18 + Valkey + Pulumi Stack