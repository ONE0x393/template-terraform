# EC2 module

EC2 인스턴스 하나를 생성하는 로컬 Terraform 모듈입니다.

이 모듈은 VPC, Subnet, Security Group, IAM Role, IAM Instance Profile, Key Pair, Elastic IP를 생성하지 않습니다. 호출하는 Root Module이 필요한 값을 전달해야 합니다.

## 기본 보안 설정

- Root EBS 볼륨은 gp3, 암호화, 종료 시 삭제로 고정됩니다.
- Instance Metadata Service는 IMDSv2만 허용합니다.
- Public IP 자동 연결은 기본적으로 꺼져 있습니다.
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

여러 인스턴스가 필요하면 모듈 내부를 바꾸지 말고 호출하는 Root Module에서 `for_each`를 사용하세요.

## 검증

호출하는 Root Module에서 다음 명령을 실행하세요.

```sh
terraform init -backend=false
terraform validate
terraform fmt -check -recursive
```
