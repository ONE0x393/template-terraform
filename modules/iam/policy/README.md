# IAM Policy 모듈

고객 관리 IAM Policy 하나를 만듭니다. Policy는 Role과 독립적으로 생성되므로 출력 ARN을 여러 Role에 연결할 수 있습니다. 정책의 권한 범위는 호출자가 정합니다.

## 입력 속성

| 속성 | 타입 | 기본값 | 역할 |
|---|---|---|---|
| `name` | `string` | 필수 | 생성할 고객 관리 IAM Policy 이름입니다. |
| `policy_json` | `string` | 필수 | 호출자가 작성한 IAM 권한 정책 JSON입니다. JSON 문법을 검사합니다. |
| `description` | `string` | `null` | Policy 설명입니다. |
| `tags` | `map(string)` | `{}` | Policy에 붙일 태그입니다. `Name`은 Policy 이름이 우선합니다. |

## 출력 속성

| 속성 | 역할 |
|---|---|
| `policy_name` | 생성된 Policy 이름입니다. |
| `policy_arn` | Role에 연결할 Policy ARN입니다. |

## 사용 예시

```hcl
module "object_read_policy" {
  source = "../../modules/iam/policy"

  name        = "sample-object-read"
  policy_json = jsonencode({
    Version = "2012-10-17"
    Statement = [{
      Effect   = "Allow"
      Action   = ["s3:GetObject"]
      Resource = ["arn:aws:s3:::example-data/*"]
    }]
  })

  tags = {
    Service = "sample"
  }
}
```

`policy_json`에는 `aws_iam_policy_document`의 `json` 출력도 전달할 수 있습니다. 모듈은 JSON 형식만 검사하며 IAM 권한의 의미와 실제 리소스 접근 가능 여부는 AWS 적용 전까지 확인하지 않습니다. 출력값은 `policy_name`과 `policy_arn`입니다.

## 2026-10-19 최신화 검증

이번 변경의 포맷 검사, `terraform validate`와 mock Plan 테스트 3개가 통과했습니다. 입력 및 출력 속성 표도 코드와 대조했습니다. 명령, 도구 버전과 실제 AWS 미검증 범위는 [AI-DLC 기록](../../../docs/ai-dlc/iam-role-policy-modules.md)을 참고합니다.
