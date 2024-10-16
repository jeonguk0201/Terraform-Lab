# Terraform-Lab 🚀  
이 리포지토리는 Terraform 학습 과정을 정리한 프로젝트입니다. Terraform 설치부터 S3 버킷과 EC2 생성, 정책 설정, 트러블슈팅까지 실습하며 배운 내용을 기록했습니다.

---

## 📋 목차  
1. [설치 방법](#-설치-방법)  
2. [S3 업로드](#-s3-업로드)  
3. [EC2 생성](#-ec2-생성)  
4. [정책 관련 주의할 점](#-정책-관련-주의할-점)  
5. [트러블 슈팅](#-트러블-슈팅)  
6. [결론](#-결론)  

---

## 🛠 설치 방법  
Terraform 설치 및 설정 방법입니다.

```bash
$ sudo su -

# wget -O- https://apt.releases.hashicorp.com/gpg | sudo gpg --dearmor -o /usr/share/keyrings/hashicorp-archive-keyring.gpg


# echo "deb [signed-by=/usr/share/keyrings/hashicorp-archive-keyring.gpg] https://apt.releases.hashicorp.com $(lsb_release -cs) main" | sudo tee /etc/apt/sources.list.d/hashicorp.list

# apt-get update && apt-get install terraform -y

# terraform -version
```

---

## 📦 S3 업로드  
Terraform을 사용해 S3 버킷을 생성하고 HTML 파일을 업로드합니다.

**Terraform 코드 예시 (S3 버킷 생성 및 파일 업로드):**
```hcl
resource "aws_s3_bucket" "example" {
  bucket = "example-bucket"
}

resource "aws_s3_object" "index" {
  bucket        = aws_s3_bucket.example.id
  key           = "index.html"
  source        = "index.html"
  content_type  = "text/html"
}
```
- **S3 웹 호스팅 설정**:
  - S3 버킷에 `index.html`과 `main.html`을 업로드.
  - 캐시 문제로 인해 변경된 파일이 바로 반영되지 않을 수 있음 (트러블 슈팅 참고).

---

## 🖥 EC2 생성  
Terraform을 사용해 EC2 인스턴스를 생성합니다.

**Terraform 코드 예시 (EC2 생성):**
```hcl
resource "aws_instance" "ec2 이름" {
  ami           = "[ami 값]"
  instance_type = "t2.micro"

  tags = {
    Name = "MyEC2Instance"
  }

  key_name = "example-key"  # EC2 접속을 위한 키페어 이름
}
```
- **키 페어**: EC2에 접속할 수 있도록 AWS 콘솔에서 키 페어를 생성하고 Terraform 코드에 반영해야 합니다.

---

## 🔐 정책 관련 주의할 점  
1. **S3 버킷 정책 설정**:  
   - S3 버킷에 public read 정책을 적용해야 웹 호스팅이 가능합니다.

```hcl
resource "aws_s3_bucket_policy" "public_read" {
  bucket = aws_s3_bucket.example.id

  policy = <<EOF
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": "*",
      "Action": "s3:GetObject",
      "Resource": "arn:aws:s3:::example-bucket/*"
    }
  ]
}
EOF
}
```

2. **EC2 보안 그룹**:  
   - 인바운드 및 아웃바운드 트래픽을 허용하는 보안 그룹 설정이 필요합니다.

---

## 🛠 트러블 슈팅  
### **1. S3 업로드 후 캐시 문제**  
- **문제**: S3에 `index.html` 파일을 덮어썼으나, 브라우저에서 여전히 이전 버전이 보임.
- **원인**: 브라우저가 기존 HTML 파일을 캐시하고 있음.

**해결 방법**:
1. **강제 새로고침**: `Ctrl + Shift + R` (Windows/Linux), `Cmd + Shift + R` (Mac).
2. **S3 객체의 Cache-Control 설정**:  
   - Terraform 코드에서 Cache-Control 헤더를 설정해 캐시를 방지합니다.

   ```hcl
   resource "aws_s3_object" "index" {
     bucket        = aws_s3_bucket.example.id
     key           = "index.html"
     source        = "index.html"
     content_type  = "text/html"
     metadata = {
       "Cache-Control" = "no-cache, no-store, must-revalidate"
     }
   }
   ```

---

## 📝 결론  
Terraform을 사용해 AWS 리소스를 생성하고 관리하는 방법을 학습했습니다. 이번 프로젝트를 통해 다음과 같은 점을 익혔습니다:
- Terraform 설치 및 기본 사용법.
- S3 버킷 생성과 정적 웹사이트 파일 업로드.
- EC2 인스턴스 생성과 키 페어 설정.
- S3 정책 설정 및 웹 호스팅을 위한 퍼블릭 접근 허용.
- 트러블 슈팅: 브라우저 캐시 문제 해결 방법.

