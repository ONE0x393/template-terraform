# Project overview

마지막 확인일: 2026-10-19

환경별 Terraform 실행기와 재사용 가능한 AWS 모듈 15개를 제공합니다. `env/dev`, `env/stg`, `env/prd`에는 `.gitkeep`만 있어 실제 배포용 Root Module은 없습니다.

## 현재 작업

참고 저장소의 [커밋 b46bc8b](https://github.com/softbank-hackathon-2026/platform-terraform/tree/b46bc8ba0031778eefff0c65eed3424a36d24676/modules)를 기준으로 기존 8개 모듈을 갱신하고 ECS 및 CloudFront를 추가했습니다. 전체 범위의 Ideation, Inception과 Unit별 Construction 계획은 2026-10-19 사용자의 구현 요청으로 승인되었습니다.

**10개 모듈 최신화와 모듈별 커밋 완료. Terraform mock 테스트 95개와 JavaScript 테스트 14개 통과.**

| 구분 | 현재 상태 |
|---|---|
| 작업 폴더 | 현재 프로젝트. Git 워크트리 미사용 |
| 브랜치 | `chore/sync-platform-modules` 생성 완료 |
| 구현 | 10개 모듈 코드 및 원본 테스트 반영 |
| 정적 확인 | 전체 Terraform 포맷, README 입력/출력 대조, 로컬 링크 및 신규 금지 문자 점검 통과 |
| JavaScript | CloudFront 경로 테스트 14개 통과 |
| Terraform init | 변경 모듈 10개와 IAM 결합 및 Aurora 구성 모두 통과 |
| Terraform validate | 변경 모듈 10개와 결합 및 회귀 구성 통과. RDS는 Terraform 1.15.4와 1.17.0-rc2 모두 확인 |
| Terraform mock 테스트 | 총 95개 통과: 변경 모듈 80개, IAM 결합 1개, Aurora 회귀 14개 |
| Git 커밋 | 모듈별 10개 완료. 공통 문서에 실제 SHA 및 검증 결과 반영 |
| AWS Plan/Apply, 배포 | 미수행 |
| 원격 푸시 | 미수행 |

초기 자동 승인 검토는 `Selected model is at capacity` 오류로 실행되지 않았습니다. 이후 사용자가 직접 승인하는 방식으로 전환해 브랜치 생성, 전체 로컬 검증과 모듈별 커밋을 완료했습니다.

## 저장소 구조

```text
tf                         환경별 Terraform 실행기
env/dev, env/stg, env/prd   환경 Root Module 위치
modules/network            VPC, Subnet, 선택적 NAT
modules/security-group     Security Group과 개별 규칙
modules/iam/policy         고객 관리 IAM Policy
modules/iam/role           Role과 선택적 Instance Profile
modules/alb                ALB, Target Group과 Listener
modules/ecr                비공개 ECR 저장소
modules/s3                 비공개 S3 버킷
modules/rds/instance       일반 RDS와 Read Replica
modules/ecs                ECS Cluster와 선택적 Fargate 서비스
modules/cloudfront         S3/ALB 비공개 Origin과 SPA 라우팅
modules/ec2                EC2 인스턴스
modules/rds/aurora          Provisioned Aurora와 reader
modules/kms                대칭 KMS 키
modules/eks                EKS 클러스터와 관리형 노드 그룹
modules/eks/addons         관리형 EKS Add-On
tests/iam-composition      IAM Policy, Role, EC2 결합 mock 테스트
docs/ai-dlc                작업 단위별 승인 및 검증 기록
```

## 최신화한 모듈

아래 표에 모듈별 변경과 이번 mock 테스트 결과를 정리했습니다.

| 모듈 | 변경 | mock 테스트 | 상세 기록 |
|---|---|---|---|
| `network` | Private Subnet 생략과 enable_nat_route를 지원하고 NAT 대상 계산, Name 태그와 AWS Provider 상한을 참고 버전에 맞춥니다. | 13개 통과 | [Network AI-DLC](./ai-dlc/network-module.md) |
| `security-group` | 인바운드와 아웃바운드 규칙에 공통 태그와 규칙별 Name 태그를 적용합니다. | 8개 통과 | [Security Group AI-DLC](./ai-dlc/security-group-module.md) |
| `iam/policy` | Policy에 입력 태그를 유지하면서 Policy 이름을 Name 태그로 적용합니다. | 3개 통과 | [IAM Policy AI-DLC](./ai-dlc/iam-role-policy-modules.md) |
| `iam/role` | Role과 선택적 Instance Profile에 이름 기반 Name 태그를 적용합니다. | 5개 통과 | [IAM Role AI-DLC](./ai-dlc/iam-role-policy-modules.md) |
| `alb` | Forward와 Redirect Listener에 공통 태그와 Listener 키 기반 Name 태그를 적용합니다. | 15개 통과 | [ALB AI-DLC](./ai-dlc/alb-module.md) |
| `ecr` | Repository에 입력 태그를 유지하면서 저장소 이름을 Name 태그로 적용합니다. | 6개 통과 | [ECR AI-DLC](./ai-dlc/ecr-module.md) |
| `s3` | CloudFront 연결에 사용하는 bucket_regional_domain_name 출력을 추가합니다. | 4개 통과 | [S3 AI-DLC](./ai-dlc/s3-module.md) |
| `rds/instance` | Secret과 DB의 write-only 버전 번호를 52비트로 제한하고 기본 DB의 copy_tags_to_snapshot을 활성화합니다. | 16개 통과 | [RDS Instance AI-DLC](./ai-dlc/rds-module.md) |
| `ecs` | Cluster와 CloudWatch Logs 그룹, 선택적 Linux X86_64 On-Demand Fargate 서비스 하나를 구성하는 모듈을 추가합니다. | 6개 통과 | [ECS AI-DLC](./ai-dlc/ecs-module.md) |
| `cloudfront` | S3 OAC와 Internal ALB VPC Origin을 연결하는 Distribution, SPA Function과 선택적 사용자 도메인을 추가합니다. | 4개 통과 | [CloudFront AI-DLC](./ai-dlc/cloudfront-module.md) |

Network의 Name 태그 형식 변경과 NAT 경로 선택, RDS의 52비트 버전 번호 전환에 따른 일회성 암호 변경 가능성은 각 모듈 README에 설명했습니다. 기존 리소스에 적용하기 전 실제 Plan이 필요합니다.

## 코드가 바뀌지 않은 모듈

아래 검증 결과는 기존 AI-DLC 기록입니다. 이번에 새로 실행한 결과가 아닙니다.

| 모듈 | 기존 기록 | 상세 기록 |
|---|---|---|
| EC2 | 포맷 및 validate 통과, 자동 테스트 없음 | [EC2 AI-DLC](./ai-dlc/ec2-module.md) |
| Aurora | mock 테스트 14개 통과 | [RDS AI-DLC](./ai-dlc/rds-module.md) |
| KMS | mock 테스트 10개 통과 | [KMS AI-DLC](./ai-dlc/kms-module.md) |
| EKS | Unit 1 mock 테스트 16개 통과. Unit 3~5 미시작 | [EKS AI-DLC](./ai-dlc/eks-module.md) |
| EKS Addons | Unit 2 mock 테스트 16개 통과 | [EKS AI-DLC](./ai-dlc/eks-module.md) |

`tf` 실행기는 환경 디렉터리에서 명령을 실행하고 TUI와 셸 자동완성을 지원합니다. 기존 `bash -n tf` 검증 기록은 유지하며 이번 최신화에서 실행기 코드는 변경하지 않았습니다.

## 이번 검증 기록

| 날짜 | 명령 또는 대상 | 결과 |
|---|---|---|
| 2026-10-19 | `terraform fmt -check -recursive modules tests` | PASS |
| 2026-10-19 | 변경 모듈 10개에서 `terraform init -backend=false -input=false -no-color`, `terraform validate -no-color` | PASS |
| 2026-10-19 | 변경 모듈별 `terraform test -test-directory=tests -no-color`. RDS는 `tests-terraform-1.17` 사용 | PASS, 총 80 passed and 0 failed |
| 2026-10-19 | `tests/iam-composition`에서 init, fmt, validate, mock Plan | PASS, 1 passed and 0 failed |
| 2026-10-19 | `modules/rds/aurora`에서 init, fmt, validate, mock Plan | PASS, 14 passed and 0 failed |
| 2026-10-19 | `node --test modules/cloudfront/tests/spa.test.cjs` | PASS, 14 passed and 0 failed |
| 2026-10-19 | 시스템 Terraform 1.15.4로 일반 RDS `terraform validate -no-color` | PASS |
| 2026-10-19 | `shasum -a 256 -c SHA256SUMS.darwin-arm64` | PASS, 공식 Terraform 1.17.0-rc2 바이너리 검증 |
| 2026-10-19 | `python3 /private/tmp/tf-module-sync-dwhkg5j7/audit.py /private/tmp/tf-module-sync-dwhkg5j7` | PASS. 92개 참고 파일, 15개 README의 입력 139개, 출력 80개, 내부 속성 74개 및 로컬 링크 점검 |
| 2026-10-19 | `git diff 8f00e10 --check` 및 모듈 커밋별 경로 대조 | PASS, 모듈 코드와 관련 문서만 각 커밋에 포함 |

일반 검증은 Terraform 1.15.4와 AWS Provider 6.58.0, RDS mock 테스트는 임시 Terraform 1.17.0-rc2와 Random Provider 3.9.1을 사용했습니다. Node는 26.7.0입니다. 시스템 Terraform은 변경하지 않았습니다.

초기 sandbox 실행의 Provider IPC 차단, Random Provider 미확보와 자동 승인 검토 서버 오류는 사용자 승인과 필요한 도구 확보로 해소했습니다. Provider와 검증 복사본은 임시 경로를 사용했습니다. 프로젝트에 Provider 바이너리, lock 파일이나 상태 파일을 추가하지 않았고 Git 워크트리는 생성하지 않았습니다.

## 모듈별 커밋 결과

각 모듈의 코드, 테스트, README와 관련 AI-DLC를 독립 커밋했습니다. 커밋별 변경 경로를 대조했으며 원격 푸시는 수행하지 않았습니다.

| 순서 | 모듈 | 커밋 | 메시지 |
|---|---|---|---|
| 1 | `network` | `f80ccdb` | `feat(network): sync subnet and NAT options` |
| 2 | `security-group` | `4e9979e` | `feat(security-group): tag ingress and egress rules` |
| 3 | `iam/policy` | `89b6eba` | `feat(iam-policy): add policy Name tag` |
| 4 | `iam/role` | `8f86a20` | `feat(iam-role): tag role and instance profile` |
| 5 | `alb` | `1e7ad68` | `feat(alb): tag forward and redirect listeners` |
| 6 | `ecr` | `4d59553` | `feat(ecr): add repository Name tag` |
| 7 | `s3` | `5f3a656` | `feat(s3): expose regional bucket domain` |
| 8 | `rds/instance` | `eb1a309` | `fix(rds-instance): keep write-only versions within 52 bits` |
| 9 | `ecs` | `3812fcd` | `feat(ecs): add optional Fargate service module` |
| 10 | `cloudfront` | `da1007e` | `feat(cloudfront): add private origins and SPA routing` |

공통 검증 결과와 실제 모듈 SHA는 별도의 문서 커밋으로 기록합니다.

## 다음 작업

1. EKS Unit 3 Controller 및 Gateway API, Unit 4 시작 템플릿과 Spot, Unit 5 결합 검증과 Lambda 모듈은 기존 후속 작업으로 남깁니다.
2. 환경별 Root Module 구성과 실제 AWS Plan/Apply는 별도 작업입니다. Network 이름 및 NAT 경로 변경과 RDS 암호 갱신 가능성은 실제 적용 전 Plan에서 검토해야 합니다.
3. RDS 모듈 소유 Secret을 운영에 사용하기 전 암호 회전 및 복구 절차를 설계합니다.
4. 원격 푸시는 이번 범위에서 제외하며 요청이 있을 때 진행합니다.

새 세션은 [AGENTS.md](../AGENTS.md), 이 문서, 해당 모듈 README와 AI-DLC 순서로 확인합니다. 현재 코드와 실제 실행 결과가 이전 기록보다 우선합니다.
