# Jenkins를 활용한 CI/CD 파이프라인 구성 실습

## 실습 환경
### 1. 

---
## 시나리오
### 1. 

---
## Jenkins 설치
```sh
# Jenkins 패키지 추가
yum update -y
sudo wget -O /etc/yum.repos.d/jenkins.repo http://pkg.jenkins.io/redhat/jenkins.repo && 
sudo rpm --import https://pkg.jenkins.io/redhat/jenkins.io.key

# Java, Docker, Git 설치
sudo yum install -y java-1.8.0-openjdk jenkins git docker

# Java Version 8로 설정
alternatives --config java

# Jenkins 서비스 시작
service jenkins start
```

## Jenkins 접속하기
### Jenkins 웹페이지 접속 및 초기 설정
- http://{Jenkins Server public_IP}:8080 접속
- Unlock Jenkins에서 '관리자 패스워드' 입력
```sh
# 아래 파일에서 '관리자 패스워드' 확인 가능
sudo cat /var/lib/jenkins/secrets/initialAdminPassword
```
- 'Install suggested plugins' 선택
  - Jenkins에서 추천해주는 플러그인을 설치하는 옵션

## Git Credential 등록

### 관리자 정보 입력
- Dashboard → Credentials → System → Global credentials (unrestricted)로 이동
  - Github에서 Profile → Settings → Developer Settings → Personal access tokens 텝으로 이동
  - 'Generate new token' 버튼을 클릭
  - 원하는 이름 입력 및 repo 체크 박스 클릭 후, 'Generate Token' 클릭
  - 생성된 Token 값을 복사하여 Jenkins Credential의 Password에 copy & paste
  - Username: 깃허브 아이디
    - ID: 원하는 아이디
    - OK 클릭

## Jenkinsfile 수정
### AWS에서 IAM 사용자 생성
- ACCESS KEY
- SECRET KEY
### 해당 사용자 정보를 Jenkins 서버에 등록
- Jenkins 서버의 Email 발송을 위한 SMTP 서버 도메인 주소 또는 공인 IP를 복사 (copy)
- Jenkins 관리 → 시스템 설정으로 이동
- 최하단에 복사한 SMTP 서버 주소를 붙여넣기 (paste)
- Use SMTP Authentication 체크
  - Jenkinsfile에 등록한 사용자 정보를 입력
    - 사용자명: 사용자 ID ( ex) aa12123@gmail.com)
    - 비밀번호: 사용자 password ( ex) sgwatgwatwq)

## 플러그인 설치
**Jenkins 서버에서 파이프라인 실행에 필요한 플러그인 설치**
### Docker 및 Git 플러그인 설치
  - Jenkins 관리 → 플러그인 관리 → 설치 가능 텝으로 이동
  - 검색창에 'docker' 입력
    - Docker, Docker Pipeline 체크
    - '재시작 없이 설치하기' 버튼 클릭

## Docker를 ec2-user 및 jenkins 사용자 그룹으로 등록
### Jenkins Server CLI 접속
```sh
sudo usermod -aG docker $USER
sudo usermod -aG docker jenkins
sudo service docker start
```
### CLI 로그아웃 후 재실행
```sh
logout
ssh -i ~/.ssh/{pem 파일 이름}.pem {리눅스 사용자 명}@{공인 IP 주소}
```

### 새로운 item 만들기
- Dashboard → 새로운 item 클릭
  - item name 입력 후 'Pipeline' 클릭
- item 구성 설정
  - Poll SCM 기능 활성화
    - Jenkinsfile에 triggers-pollSCM 명령어를 적용하기 위함
  - Pipleline → Definition에서 'Pipeline script from SCM' 클릭
    - SCM은 'Git'으로 설정
    - 파이프라인이 적용될 깃허브 repository 주소 입력
    - Github Credential 입력
### 파이프라인 실행
- 화면 좌측 'Build Now' 클릭
