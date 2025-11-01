# 🚀 CI/CD 자동화 및 Terraform IaC 구성


## 1️⃣ 할 일

AWS 인프라를 코드로 선언하고, GitHub Actions를 통해 빌드부터 배포까지 완전 자동화된 CI/CD 파이프라인을 구성한다. 또한, 무중단(롤링) 배포와 모니터링 통합까지 구현하여 실서비스 수준의 자동화 환경을 구축한다.

---

## 2️⃣ 실습 목표

- **Terraform**으로 VPC, EC2, RDS, S3, ALB 등 인프라를 코드화 (IaC)
- **GitHub Actions**로 Docker 이미지 빌드 → DockerHub/ECR 푸시 → AWS EC2 자동 배포
- **SSM Agent + Shell Script**를 통한 무중단(롤링) 배포 구현
- **CloudWatch + Grafana + Slack Webhook**으로 실시간 배포 상태 모니터링
- **Secrets Manager**로 환경변수 및 인증정보 안전하게 관리

---

## 3️⃣ 인프라 구조 (트리 형태)

```
GitHub Actions (CI/CD)
   ├── Docker Build & Push (ECR/DockerHub)
   │
   ├── AWS CLI (배포 자동화)
   │      ├→ Terraform Apply (VPC, EC2, ALB, RDS, S3 등 IaC)
   │      └→ SSM (EC2 배포 명령)
   │
   ├── EC2 (배포 대상 서버)
   │      ├→ SSM Agent (원격 명령 실행)
   │      ├→ Docker Compose (멀티 컨테이너 서비스)
   │      ├→ 무중단 배포 (Rolling Update)
   │      └→ Health Check (Nginx, Spring 등)
   │
   ├── CloudWatch (로그 및 메트릭)
   │      └→ Grafana Dashboard 시각화
   │
   └── Slack Webhook (배포 결과 알림)

```

---

## 4️⃣ 실습 상세 내용

### (1) IaC 구성 – Terraform

- Terraform을 설치하고 AWS IAM 사용자 자격 증명을 연동한다.
- `main.tf` 파일에 **VPC, EC2, RDS, ALB, S3** 등의 리소스를 선언형으로 정의한다.
- `terraform plan`으로 변경 사항을 미리 검토하고 `terraform apply`로 실제 인프라를 배포한다.
- **AMI**를 사용해 미리 세팅된 서버 이미지를 자동으로 프로비저닝하여 배포 속도를 향상시킨다.
- **Secrets Manager**를 통해 DB 비밀번호, API 키 등 민감 정보를 안전하게 관리하고 Terraform 변수로 불러온다.

### (2) CI/CD 파이프라인 구축 – GitHub Actions

- `.github/workflows/deploy.yml`을 작성하여 다음 단계를 자동화한다.
    - **Build 단계:** `Dockerfile`을 기반으로 이미지 빌드.
    - **Push 단계:** DockerHub 또는 AWS ECR에 이미지 업로드.
    - **Deploy 단계:** AWS CLI를 통해 SSM 명령 실행.
- **AWS CLI**로 `aws ssm send-command`를 호출해 EC2에 배포 스크립트를 실행한다.

### (3) 무중단(롤링) 배포 및 모니터링

- 배포 시 **기존 컨테이너를 종료하지 않고**, 새 컨테이너를 띄운 후 Health Check 통과 시 트래픽 전환.
- 배포 실패 시 **이전 버전으로 자동 롤백**할 수 있도록 스크립트를 구성.
- **CloudWatch** 로그를 Grafana 대시보드와 연동하여 배포 및 서버 상태를 실시간 모니터링.

### (4) 보안 및 자동화 강화

- **SSM Agent**를 통한 SSH 비활성화 원격 배포 환경 구성.
- **SSM Session Manager**로 보안 그룹에서 22번 포트를 열지 않고도 진단 및 로그 확인.
- **Slack Webhook** 연동으로 배포 성공/실패 알림 자동화.
- 배포 전 자동 **단위 테스트 (Unit Test)** 단계 추가.

---

## 5️⃣ 추가 사항

- **Blue-Green 배포** 전략을 추가 적용하여 다운타임을 완전히 제거할 예정.
- *Parameter Store (SSM)**로 환경별 설정값 관리 개선.
- **SNS 연동**으로 장애 알림을 실시간 Slack 및 이메일로 전송.
- *Tracing (X-Ray, Jaeger 등)**을 통한 배포 후 애플리케이션 트랜잭션 분석.
- AWS CodePipeline

---

### 🔑 핵심 키워드

> IaC, Terraform, AMI, Secrets Manager, GitHub Actions, Docker, Docker Compose, 무중단(롤링) 배포, SSM, SSM Agent, SSM Session Manager, 쉘(스크립트), AWS CLI, Health Check, CloudWatch, Grafana, Webhook
> 

```jsx
ROOT: 클라우드 기반 DevOps 시스템 아키텍처 (IaC + CI/CD)
│
├── I. 인프라스트럭처 정의 및 관리 (IaC - Terraform)
│   │
│   ├── 1️⃣ 네트워크 및 진입점
│   │     ├── Route 53 (도메인 → ALB)
│   │     ├── IGW (인터넷 게이트웨이)
│   │     ├── ALB (Application Load Balancer)
│   │     │     └── Target Group (헬스체크 기반 트래픽 분산)
│   │     └── Public / Private Subnet (보안영역 분리)
│   │
│   ├── 2️⃣ 컴퓨팅 및 스토리지 리소스
│   │     ├── EC2 (Spring / Flask / Nginx)
│   │     │     └── AMI (Docker, Python, SSM Agent 사전 설치)
│   │     ├── RDS (데이터베이스, 별도 서브넷 내 구성)
│   │     └── S3 버킷 (3분리)
│   │           ├── S3-1: 정적 자산 (Frontend)
│   │           ├── S3-2: 백업 / 로그 / 미디어
│   │           └── S3-3: 설정 및 배포 스크립트 (Compose, Shell)
│   │
│   ├── 3️⃣ 보안 및 설정 관리
│   │     ├── IAM Role / Instance Profile
│   │     │     └── 권한: SSM, S3, Secrets Manager 접근 허용
│   │     └── Secrets Manager
│   │           └── DB 비밀번호, API Key 등 민감정보 암호화 저장
│   │
│   └── 4️⃣ 초기 자동화
│         └── EC2 User Data
│              └── 부팅 시 Docker 설치, S3 설정파일 자동 다운로드
│
│
├── II. 지속적 통합 및 배포 (CI/CD Pipeline)
│   │
│   ├── 1️⃣ 지속적 통합 (CI)
│   │     ├── GitHub (소스 저장소)
│   │     ├── GitHub Actions (자동 빌드/푸시)
│   │     │     ├── Docker Build
│   │     │     └── DockerHub/ECR Push
│   │     └── Workflow Trigger: main 브랜치 푸시 시 자동 실행
│   │
│   ├── 2️⃣ 지속적 배포 (CD)
│   │     ├── SSM (AWS Systems Manager)
│   │     │     ├── SSM Agent (EC2 내 명령 실행 담당)
│   │     │     └── SSM Session Manager (SSH 없는 원격 접속)
│   │     │
│   │     ├── 배포 로직
│   │     │     ├── deploy.sh (핵심 스크립트)
│   │     │     │     ├── S3에서 compose/env 파일 다운로드
│   │     │     │     ├── Docker Pull → Run → Clean
│   │     │     │     └── 무중단(롤링) 교체 로직 포함
│   │     │     └── Docker Compose (멀티 컨테이너 정의)
│   │     │
│   │     ├── AWS CLI
│   │     │     └── GitHub Actions → AWS API 호출 (SSM RunCommand)
│   │     │
│   │     └── 배포 원칙
│   │           ├── Backend-First 전략
│   │           ├── Health Check 기반 롤링 배포
│   │           └── 서비스 중단 없는 무중단 운영
│   │
│   └── 3️⃣ 자동 알림 및 연동
│         ├── Slack Webhook (배포 성공/실패 알림)
│         ├── CloudWatch Alarm → Slack 경고 전송
│         └── IaC 기반 자동복구 (Terraform Plan/Apply 재실행)
│
│
└── III. 운영 및 관찰성 (Observability)
    │
    ├── 1️⃣ 모니터링 시스템
    │     ├── CloudWatch (로그, 메트릭 수집)
    │     ├── Grafana (CloudWatch + Prometheus 통합 시각화)
    │     └── Prometheus / Loki (선택적 로그 파이프라인)
    │
    ├── 2️⃣ 알림 시스템
    │     ├── Slack Webhook (실시간 알림)
    │     └── CloudWatch EventBridge → 알림 자동 트리거
    │
    └── 3️⃣ 유지보수 및 복구
          ├── 백업 자동화 (S3 + RDS Snapshot)
          ├── 정기 검증 (cron / EventBridge)
          └── 장애 시 Terraform IaC 복구

```
