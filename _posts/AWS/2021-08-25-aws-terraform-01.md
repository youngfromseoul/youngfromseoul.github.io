---
title: "[IaC]Terraform 사용하기 with AWS #01"
excerpt: "Terraform으로 AWS 인프라 구성하기"
categories: 
  - AWS
last_modified_at: 2021-08-25T15:44:00+09:00
tags: 
    - AWS
    - Terraform
    - DevOps
author_profile: true
read_time: true
toc_label: "Terraform" 
toc_icon: "cog" 
toc: true
toc_sticky: true
---

## Infrastructure as Code

IaC, 즉 코드로서의 인프라는 인프라를 이루는 서버, 미들웨어 그리고 서비스 등,
인프라 구성요소들을 코드를 통해 구축하는 것.
IaC는 코드로서의 장점, 즉 작성 용이성, 재사용성, 유지보수 등의 장점을 가진다.

<br>
## Terraform

가장 많이 쓰이는 IaC 도구로, 표준으로 보아도 무방하다.
<br>
### Terraform 구성 요소

> **provider** \- 테라폼으로 생성할 인프라의 종류를 의미
> 
> **resource** \- 테라폼으로 실제로 생성할 인프라의 자원을 의미
> 
> **state** \- 테라폼을 통해 생성한 자원을 상태를 의미
> 
> **output** \- 테라폼으로 만든 자원을 변수 형태로 state에 저장하는 것을 의미
> 
> **module** \- 공통적으로 활용할 수 있는 코드를 문자 그대로 모듈 형태로 정의하는 것을 의미
> 
> **remote -** 다른 경로의 state를 참조하는 것을 의미, Output 변수를 불러올 때 주로 사용

<br>
### Terraform 명령어

> **init** \- 테라폼 명령어 사용을 위해 각종 설정을 진행
> 
> **plan** \- 테라폼으로 작성한 코드가 실제로 어떻게 만들어질지에 대한 예측 실행
> 
> **apply** \- 테라폼 코드로 실제 인프라를 생성
> 
> **import** \- 이미 만들어진 자원을 테라폼 state 파일로 옮겨주는 명령어
> 
> **state** \- 테라폼 state를 다루는 명령어\, 하위 명령어로 mv\, push와 같은 명령어가 있다\.
> 
> **destroy** \- 생성된 자원\, state 파일 기준으로 모두 삭제하는 명령어

<br>
<br>
## AWS Configure

> AWS와 Terraform 작업 환경에서 서로 API 통신이 가능하도록 환경 설정이 필요

<br>
<br>
<br>
**AWS CLI 설치 -** [https://docs.aws.amazon.com/ko\_kr/cli/latest/userguide/cli-chap-install.html](https://docs.aws.amazon.com/ko_kr/cli/latest/userguide/cli-chap-install.html)
<br>
* AWS CLI 설치가 완료되었으면, AWS 로그인 후 Terraform용 IAM 계정 생성 및 액세스 키를 만들어준다.

![image.png](https://github.com/youngfromseoul/youngfromseoul.github.io/blob/master/assets/images/aws-01.png?raw=true)
<br>
* 이후, 아래의 커맨드를 통해 액세스 키 정보 및 리전 정보, 출력 포맷 등을 지정해준다.

``` bash
freey-MacBook-Pro ~ aws configure
AWS Access Key ID [None]: 
AWS Secret Access Key [None]: 
Default region name [None]: ap-northeast-2
Default output format [None]: json
```
<br>
* 세팅된 정보는 `cat ~/.aws/credentials`<span style="color:#323232"> 의 Default 쪽에 설정됨</span>

```
freey-MacBook-Pro ~ cat ~/.aws/credentials
[default]
aws_access_key_id = 
aws_secret_access_key =
```
<br>
* 현재 사용자 정보 확인

```
freey-MacBook-Pro ~ aws sts get-caller-identity
{
    "UserId": "",
    "Account": "",
    "Arn": "arn:aws:iam:user/Terraform"
}
(END)
```

## **Terraform 기본 개념**

> **resource** : 실제로 생성할 인프라 자원을 의미합니다.
> ex) aws\_security\_group, aws\_lb, aws\_instance
> 
> **provider** : Terraform으로 정의할 Infrastructure Provider를 의미합니다.
> [https://www.terraform.io/docs/providers/index.html](https://www.terraform.io/docs/providers/index.html)
> 
> **output** : 인프라를 프로비저닝 한 후에 생성된 자원을 output 부분으로 뽑을 수 있습니다.
> Output으로 추출한 부분은 이후에 `remote state`에서 활용할 수 있습니다.
> 
> **backend** : terraform의 상태를 저장할 공간을 지정하는 부분입니다.
> backend를 사용하면 현재 배포된 최신 상태를 외부에 저장하기 때문에 다른 사람과의 협업이 가능합니다. 가장 대표적으로는 AWS S3가 있습니다.
> 
> **module** : 공통적으로 활용할 수 있는 인프라 코드를 한 곳으로 모아서 정의하는 부분입니다.
> Module을 사용하면 변수만 바꿔서 동일한 리소스를 손쉽게 생성할 수 있다는 장점이 있습니다.
> 
> **remote state** : remote state를 사용하면 VPC, IAM 등과 같은 공용 서비스를 다른 서비스에서 참조할 수 있습니다.
> tfstate파일(최신 테라폼 상태정보)이 저장되어 있는 backend 정보를 명시하면, terraform이 해당 backend에서 output 정보들을 가져옵니다.

<br>
## **Terrafrom 작동 원리**

> **Local 코드** : 현재 개발자가 작성/수정하고 있는 코드
> **AWS 인프라** : 실제로 AWS에 배포되어 있는 인프라
> **Backend에 저장된 상태** : 가장 최근에 배포한 테라폼 코드 형상

<span style="color:#323232">위와 같이 Local에서 작성한 테라폼 코드로 인프라를 정의하고, 해당 코드를 실제 AWS 인프라로 프로비저닝하며, backend를 구성하여 최신 코드를 저장합니다.</span>

### Terraform init

* 지정한 backend에 상태 저장을 위한 `.tfstate` 파일을 생성
* init 작업을 완료하면, local에는 `.tfstate`에 정의된 내용을 담은 `.terraform` 파일이 생성된다
* 이미 `.tfstate`에 인프라를 정의한 것이 있다면, init작업을 통해서 local에 sync를 맞출 수 있다.

### Terraform plan

* 정의한 코드에 대한 결과를 보여줍니다. 다만, plan 실행시에는 오류가 없어도, 실제 적용되었을 때는 오류가 발생할 수 있다
* **Plan 명령은 실제로 인프라가 적용되지는 않는다.**

### Terraform apply

* 실제로 인프라를 배포하기 위한 명령어. apply를 완료하면, AWS에 인프라가 생성되고 작업 결과가 backend의 `.tfstate` 파일에 저장된다.
* 해당 결과는 local의 `.terraform` 파일에도 저장됨

### Terraform import

* AWS 인프라에 배포된 리소스를 `terraform state`로 옮겨주는 작업.
* 이는 local의 .terraform에 해당 리소스의 상태 정보를 저장해주는 역할을 한다.
    * Apply 전까지는 backend에 저장되지 않음.
    * Import 이후에 plan을 하면 로컬에 해당 코드가 없기 때문에 리소스가 삭제 또는 변경된다는 결과를 보여준다.

> <span style="color:#323232">만약 기존에 인프라를 AWS에 배포한 상태에서 테라폼을 적용하고 싶으면 모든 리소스를 `terraform import`로 옮겨야 합니다. </span>

<br>
<br>
## Terraform 실습

* AWS provider를 지정하고, terraform init 실행

```
vim provider.tf
provider "aws" {
  region = "ap-northeast-2"
}
```
<br>
* terraform init

```
Initializing the backend...

Initializing provider plugins...
- Checking for available provider plugins...
- Downloading plugin for provider "aws" (hashicorp/aws) 3.37.0...

The following providers do not have any version constraints in configuration,
so the latest version was installed.

To prevent automatic upgrades to new major versions that may contain breaking
changes, it is recommended to add version = "..." constraints to the
corresponding provider blocks in configuration, with the constraint strings
suggested below.

* provider.aws: version = "~> 3.37"


Warning: registry.terraform.io: This version of Terraform has an outdated GPG key and is unable to verify new provider releases. Please upgrade Terraform to at least 0.12.31 to receive new provider updates. For details see: [https://discuss.hashicorp.com/t/hcsec-2021-12-codecov-security-event-and-hashicorp-gpg-key-exposure/23512](https://discuss.hashicorp.com/t/hcsec-2021-12-codecov-security-event-and-hashicorp-gpg-key-exposure/23512)


Terraform has been successfully initialized!

You may now begin working with Terraform. Try running "terraform plan" to see
any changes that are required for your infrastructure. All Terraform commands
should now work.

If you ever set or change modules or backend configuration for Terraform,
rerun this command to reinitialize your working directory. If you forget, other
commands will detect it and remind you to do so if necessary.
```
<br>
* AWS API 관련 라이브러리가 자동으로 설치된다.

```
drwxr-xr-x@ 4 freey  staff        128  8 25 08:34 .
drwxr-xr-x@ 3 freey  staff         96  8 25 08:34 ..
-rwxr-xr-x  1 freey  staff         79  8 25 08:34 lock.json
-rwxr-xr-x  1 freey  staff  200586352  8 25 08:34 terraform-provider-aws_v3.37.0_x5
freey-MacBook-Pro ~ pwd
/Users/freey/icloud/work/git/Terraform/aws_provider/.terraform/plugins/darwin_amd64
```
<br>
* S3 생성

```
vim s3.tf
resource "aws_s3_bucket" "test" {
  bucket = "terraform-test-4343"
}
```
<br>
* terraform plan

```
Refreshing Terraform state in-memory prior to plan...
The refreshed state will be used to calculate this plan, but will not be
persisted to local or remote state storage.


------------------------------------------------------------------------

An execution plan has been generated and is shown below.
Resource actions are indicated with the following symbols:
  + create

Terraform will perform the following actions:

  # aws_s3_bucket.test will be created
  + resource "aws_s3_bucket" "test" {
      + acceleration_status         = (known after apply)
      + acl                         = "private"
      + arn                         = (known after apply)
      + bucket                      = "terraform-test-4343"
      + bucket_domain_name          = (known after apply)
      + bucket_regional_domain_name = (known after apply)
      + force_destroy               = false
      + hosted_zone_id              = (known after apply)
      + id                          = (known after apply)
      + region                      = (known after apply)
      + request_payer               = (known after apply)
      + website_domain              = (known after apply)
      + website_endpoint            = (known after apply)

      + versioning {
          + enabled    = (known after apply)
          + mfa_delete = (known after apply)
        }
    }

Plan: 1 to add, 0 to change, 0 to destroy.

------------------------------------------------------------------------

Note: You didn't specify an "-out" parameter to save this plan, so Terraform
can't guarantee that exactly these actions will be performed if
"terraform apply" is subsequently run.
```
<br>
* terraform apply

```
An execution plan has been generated and is shown below.
Resource actions are indicated with the following symbols:
  + create

Terraform will perform the following actions:

  # aws_s3_bucket.hyoyoung-s3-23422 will be created
  + resource "aws_s3_bucket" "test" {
      + acceleration_status         = (known after apply)
      + acl                         = "private"
      + arn                         = (known after apply)
      + bucket                      = "terraform-test-4343"
      + bucket_domain_name          = (known after apply)
      + bucket_regional_domain_name = (known after apply)
      + force_destroy               = false
      + hosted_zone_id              = (known after apply)
      + id                          = (known after apply)
      + region                      = (known after apply)
      + request_payer               = (known after apply)
      + website_domain              = (known after apply)
      + website_endpoint            = (known after apply)

      + versioning {
          + enabled    = (known after apply)
          + mfa_delete = (known after apply)
        }
    }

Plan: 1 to add, 0 to change, 0 to destroy.

Do you want to perform these actions?
  Terraform will perform the actions described above.
  Only 'yes' will be accepted to approve.

  Enter a value: yes

aws_s3_bucket.test: Creating...
aws_s3_bucket.test: Creation complete after 1s [id=terraform-test-4343]

Apply complete! Resources: 1 added, 0 changed, 0 destroyed.
```

![image.png](https://github.com/youngfromseoul/youngfromseoul.github.io/blob/master/assets/images/aws-02.png?raw=true)
<br>
* 아래 명령을 통해 생성된 aws 자원 확인 가능

```
aws s3 ls
2021-08-25 08:50:56 terraform-test-4343
```
<br>
* terraform import 테스트를 위해 모든 코드 삭제

```
rm -rf s3.tf terraform.tfstate terraform.tfstate.backup .terraform/
terraform init
terraform plan
```
<br>
* 실제로 AWS 상에는 S3 버킷이 있지만, plan에서는 아무런 리소스도 생성되지 않는 것으로 나온다.

```
Refreshing Terraform state in-memory prior to plan...
The refreshed state will be used to calculate this plan, but will not be
persisted to local or remote state storage.


------------------------------------------------------------------------

No changes. Infrastructure is up-to-date.

This means that Terraform did not detect any differences between your
configuration and real physical resources that exist. As a result, no
actions need to be performed.
```
<br>
* 이전과 동일하게 s3.ft 생성

```
vim s3.tf
resource "aws_s3_bucket" "test" {
  bucket = "terraform-test-4343"
}
```
<br>
* 해당 상태에서 terraform plan을 하면 정상 실행되지만, apply 시, 이미 만들어져있기 때문에 생성이 되지 않는다.

```
Error: Error creating S3 bucket: BucketAlreadyOwnedByYou: Your previous request to create the named bucket succeeded and you already own it.
```
<br>
* 이럴 때, terraform import를 통해 일치시켜준다.

```
terraform import aws_s3_bucket.test terraform-test-4343

aws_s3_bucket.test: Importing from ID "terraform-test-4343"...
aws_s3_bucket.test: Import prepared!
  Prepared aws_s3_bucket for import
aws_s3_bucket.test: Refreshing state... [id=terraform-test-4343]

Import successful!

The resources that were imported are shown above. These resources are now in
your Terraform state and will henceforth be managed by Terraform.
```
<br>
* 이후, terraform apply를 통해, 일치시켜준다.

```
aws_s3_bucket.test: Refreshing state... [id=terraform-test-4343]

An execution plan has been generated and is shown below.
Resource actions are indicated with the following symbols:
  ~ update in-place

Terraform will perform the following actions:

  # aws_s3_bucket.test will be updated in-place
  ~ resource "aws_s3_bucket" "test2" {
      + acl                         = "private"
        arn                         = "arn:aws:s3:::terraform-test-4343"
        bucket                      = "terraform-test-4343"
        bucket_domain_name          = "terraform-test-4343.s3.amazonaws.com"
        bucket_regional_domain_name = "terraform-test-4343.s3.ap-northeast-2.amazonaws.com"
      + force_destroy               = false
        hosted_zone_id              = "Z3W03O7B5YMIYP"
        id                          = "terraform-test-4343"
        region                      = "ap-northeast-2"
        request_payer               = "BucketOwner"
        tags                        = {}

        versioning {
            enabled    = false
            mfa_delete = false
        }
    }

Plan: 0 to add, 1 to change, 0 to destroy.

Do you want to perform these actions?
  Terraform will perform the actions described above.
  Only 'yes' will be accepted to approve.

  Enter a value: yes

aws_s3_bucket.test: Modifying... [id=terraform-test-4343]
aws_s3_bucket.test: Modifications complete after 1s [id=terraform-test-4343]
```

