# EC2 module

EC2 인스턴스 하나를 생성하는 로컬 Terraform 모듈입니다.

이 모듈은 VPC, Subnet, Security Group, IAM Role, IAM Instance Profile, Key Pair, Elastic IP를 생성하지 않습니다. 호출하는 Root Module이 필요한 값을 전달해야 합니다.

## 기본 보안 설정

- Root EBS 볼륨은 gp3, 암호화, 종료 시 삭제로 고정됩니다.
- Instance Metadata Service는 IMDSv2만 허용합니다.
- Public IP 자동 연결은 기본적으로 꺼져 있습니다.
- 구매 옵션은 기본적으로 On-Demand입니다.
- `user_data`는 Terraform state에 평문으로 남을 수 있으므로 비밀번호와 토큰을 넣지 마세요.

## 사용 예시

```hcl
provider "aws" {
  region = "ap-northeast-2"
}

module "app" {
  source = "../../modules/ec2"

  name               = "sample-dev-app"
  ami_id             = "ami-0123456789abcdef0"
  instance_type      = "t3.micro"
  subnet_id          = "subnet-0123456789abcdef0"
  security_group_ids = ["sg-0123456789abcdef0"]

  iam_instance_profile = "sample-dev-app"
  root_volume_size     = 30

  tags = {
    Environment = "dev"
    Project     = "sample"
  }
}
```

## Spot 사용

`purchase_option`을 `spot`으로 지정하면 일회성 Spot Instance를 생성합니다. AWS가 Spot Instance를 회수하면 종료되며, 이 단일 인스턴스 모듈은 자동으로 대체 인스턴스를 만들지 않습니다.

```hcl
module "batch" {
  source = "../../modules/ec2"

  name               = "sample-dev-batch"
  ami_id             = "ami-0123456789abcdef0"
  instance_type      = "t3.micro"
  purchase_option    = "spot"
  subnet_id          = "subnet-0123456789abcdef0"
  security_group_ids = ["sg-0123456789abcdef0"]
}
```

`purchase_option`을 변경하면 기존 EC2 인스턴스가 교체됩니다. 중단 후 자동 복구가 필요한 워크로드에는 Auto Scaling Group 또는 EC2 Fleet 사용을 검토하세요.

여러 인스턴스가 필요하면 모듈 내부를 바꾸지 말고 호출하는 Root Module에서 `for_each`를 사용하세요.

## 검증

호출하는 Root Module에서 다음 명령을 실행하세요.

```sh
terraform init -backend=false
terraform validate
terraform fmt -check -recursive
```
