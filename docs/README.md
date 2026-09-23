# Project overview

마지막 확인일: 2026-09-23

이 저장소는 환경별 Terraform 실행기와 재사용 가능한 AWS Terraform 모듈을 제공합니다. 현재 환경 디렉터리에는 실제 Root Module이 없으므로 AWS 리소스를 생성할 수 있는 단계는 아닙니다.

## 새 세션에서 확인할 순서

1. [`AGENTS.md`](../AGENTS.md)에서 AI-DLC 절차와 문서화 규칙을 확인합니다.
2. 이 문서에서 전체 진행 상태와 다음 작업을 확인합니다.
3. 관련 AI-DLC 문서와 모듈 README를 확인합니다.
4. 현재 코드와 실제 검증 결과가 문서보다 우선합니다.

## 저장소 구조

```text
tf                         환경별 Terraform 실행기
env/dev                    dev Root Module 위치
env/stg                    stg Root Module 위치
env/prd                    prd Root Module 위치
modules/ec2                단일 EC2 인스턴스 모듈
modules/network            VPC와 Subnet, NAT Gateway 모듈
modules/s3                 단일 비공개 S3 버킷 모듈
modules/security-group     VPC Security Group과 규칙 모듈
modules/iam/policy         고객 관리 IAM Policy 모듈
modules/iam/role           IAM Role과 선택적 EC2 Instance Profile 모듈
modules/alb                단일 ALB와 0개 이상의 Target Group, Listener 모듈
tests/iam-composition      IAM Policy, Role, EC2 결합 mock 테스트 구성
docs/ai-dlc                작업 단위별 AI-DLC 기록
```

## 진행 현황

| 작업 단위 | 구현 | 로컬 검증 | AWS Plan과 Apply | Git 상태 | 상세 기록 |
|---|---|---|---|---|---|
| `tf` 실행기 | 완료 | `bash -n` 통과, 자동 기능 테스트 없음 | 해당 없음 | 커밋 `d051afd` | [`tf`](../tf) |
| EC2 모듈 | 완료 | 포맷과 `terraform validate` 통과, 자동 테스트 없음 | 미수행 | 커밋 `776d3dd`, `3532c5f` | [EC2 AI-DLC](./ai-dlc/ec2-module.md) |
| Network 모듈 | 완료 | 포맷과 `terraform validate` 통과, mock 테스트 7개 통과 | 실제 AWS 기준 미수행 | 커밋 `3587c82` | [Network AI-DLC](./ai-dlc/network-module.md) |
| S3 모듈 | 완료 | 포맷과 `terraform validate` 통과, mock 테스트 4개 통과 | 실제 AWS 기준 미수행 | 구현 커밋 `e2cf4ef` 원격 main 확인 | [S3 AI-DLC](./ai-dlc/s3-module.md) |
| Security Group 모듈 | 완료 | 포맷과 `terraform validate` 통과, mock 테스트 8개 통과 | 실제 AWS 기준 미수행 | 구현 커밋 `8949e16` 원격 main 확인 | [Security Group AI-DLC](./ai-dlc/security-group-module.md) |
| IAM Policy와 Role 모듈 | 완료 | 포맷과 세 구성의 `terraform validate` 통과, mock 테스트 9개 통과 | 실제 AWS 기준 미수행 | 구현 커밋 `2190a91` 원격 main 확인 | [IAM AI-DLC](./ai-dlc/iam-role-policy-modules.md) |
| ALB 모듈 | Listener 0개·여러 개 확장까지 완료 | 포맷과 `terraform validate` 통과, 확장 mock 테스트 15개 통과 | 실제 AWS 기준 미수행 | 구현 미커밋 | [ALB AI-DLC](./ai-dlc/alb-module.md) |
| 환경별 Root Module | 미구현 | 검증 대상 없음 | 미수행 | `.gitkeep`만 존재 | `env/` |
| 지속 문서화 | ALB 구현 기록 반영 | 문서 공백 점검 통과 | 해당 없음 | ALB 문서 미커밋 | 이 문서 |

IAM 구현 커밋 `2190a91`을 푸시한 직후 로컬 `HEAD`, `origin/main`, 원격 main의 SHA가 모두 `2190a91b4e6023492d71023f6c69a959d1a825b3`인 것을 확인했습니다.

## 구성 요소

### Terraform 실행기

[`tf`](../tf)는 실행 파일 위치를 기준으로 `env/<환경>`을 찾습니다. 명령 직접 실행, 번호 선택 TUI, Bash와 zsh 자동완성을 지원하며 `format`을 Terraform의 `fmt`로 변환합니다.

현재 자동 기능 테스트 파일은 없습니다. `bash -n tf`만 통과했습니다. 환경 디렉터리에는 `.gitkeep`만 있으므로 실제 Terraform 구성 검증과 실행은 아직 할 수 없습니다.

### EC2 모듈

[`modules/ec2`](../modules/ec2/README.md)는 EC2 인스턴스 하나를 생성합니다. On-Demand와 Spot을 선택할 수 있고 암호화된 gp3 Root EBS와 IMDSv2를 기본으로 사용합니다.

VPC, Subnet, Security Group, IAM Role과 IAM Instance Profile은 생성하지 않습니다. 자세한 결정과 검증 상태는 [EC2 AI-DLC](./ai-dlc/ec2-module.md)에 기록합니다.

### Network 모듈

[`modules/network`](../modules/network/README.md)는 VPC, Public과 Private Subnet, Route Table, 선택적 NAT Gateway를 생성합니다. NAT 모드는 `none`, `regional`, `zonal`을 지원합니다.

Zonal NAT는 AZ별 Public Subnet 키를 직접 선택하고 Regional NAT는 Subnet을 입력받지 않습니다. 자세한 결정과 검증 상태는 [Network AI-DLC](./ai-dlc/network-module.md)에 기록합니다.

### S3 모듈

[`modules/s3`](../modules/s3/README.md)는 일반 목적 S3 버킷 하나와 공개 접근 차단을 생성합니다. 버전 관리와 고객 관리 KMS 키는 선택할 수 있습니다. 로컬 검증과 실제 AWS 미검증 범위는 [S3 AI-DLC](./ai-dlc/s3-module.md)에 기록합니다.

### Security Group 모듈

[`modules/security-group`](../modules/security-group/README.md)는 VPC Security Group 하나와 명시한 IPv4 또는 Security Group 참조 규칙을 생성합니다. 기본 인바운드와 아웃바운드 규칙은 없습니다. 로컬 검증과 실제 AWS 미검증 범위는 [Security Group AI-DLC](./ai-dlc/security-group-module.md)에 기록합니다.

### IAM Policy와 Role 모듈

[`modules/iam/policy`](../modules/iam/policy/README.md)는 고객 관리 IAM Policy를 독립적으로 생성합니다. [`modules/iam/role`](../modules/iam/role/README.md)은 서비스별 신뢰 정책으로 Role을 만들고 관리형 Policy를 연결하며, EC2용 Instance Profile을 선택적으로 생성합니다. 결합 mock Plan과 실제 AWS 미검증 범위는 [IAM AI-DLC](./ai-dlc/iam-role-policy-modules.md)에 기록합니다.

### ALB 모듈

[`modules/alb`](../modules/alb/README.md)는 ALB 하나와 논리 키로 구분한 0개 이상의 Target Group, Listener를 생성합니다. Listener별로 같은 Target Group을 공유하거나 다른 Target Group을 선택할 수 있고, HTTP→HTTPS 리디렉션도 설정할 수 있습니다. 공개형 HTTP Listener는 리디렉션만 허용합니다. 검증 범위는 [ALB AI-DLC](./ai-dlc/alb-module.md)에 기록합니다.

## 현재 작업

- AI-DLC 인수인계 문서와 `AGENTS.md` 지속 문서화 규칙 구현, 검증, Review 완료
- 자주 사용하는 AWS 리소스 모듈을 S3, Security Group, IAM Role과 Policy, ALB, RDS, ECR 순서로 추가하기로 결정
- S3 모듈 구현, 로컬 검증, Review 완료. 실제 AWS Plan과 Apply는 미수행
- Security Group 모듈 구현, 로컬 검증, Review 완료. 실제 AWS Plan과 Apply는 미수행
- IAM Policy와 Role 모듈 구현, 로컬 검증, Review 완료. 실제 AWS Plan과 Apply는 미수행
- ALB 모듈의 Listener 0개·여러 개 및 Listener별 Target Group 선택 확장 구현, 로컬 검증, Review 완료. 실제 AWS Plan과 Apply는 미수행

## 다음 작업

1. ALB 확장 구현과 검증 결과를 검토하고 필요하면 커밋 및 푸시합니다.
2. RDS, ECR 순서로 각 모듈의 AI-DLC 단계를 진행합니다.
3. 실제 인프라가 필요해지면 환경별 Root Module 구성을 별도 AI-DLC 작업으로 시작합니다.
4. AWS Plan, Apply, 배포 결과는 실행한 경우에만 상태표와 관련 AI-DLC 문서에 기록합니다.
