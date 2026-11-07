# 개발 워크스테이션 구축 및 Docker 실습

## 1. 프로젝트 소개

본 프로젝트는 Windows 환경에서 개발 워크스테이션을 구축하고 Git 및 Docker의 기본 사용 방법을 실습하는 것을 목적으로 한다.

Docker를 이용하여 Nginx 웹 서버를 실행하고, 포트 매핑, Bind Mount, Docker Volume을 통해 컨테이너의 데이터 관리 및 영속성을 확인하였다.

## 2. 개발 환경

| 항목 | 환경 |
|---|---|
| 운영체제 | Windows |
| 터미널 | PowerShell |
| 개발 도구 | Visual Studio Code |
| 컨테이너 도구 | Docker Desktop |
| Docker 실행 환경 | Linux / WSL2 |
| Docker 버전 | 29.9.2 |
| Git 버전 | 2.56.0.windows.4 |
| Git 브랜치 | main |

## 3. 프로젝트 구조

```text
dev-workstation-new/
├── .git/
├── .gitignore
├── Dockerfile
├── app/
│   └── index.html
├── logs/
│   └── docker-operations.txt
└── screenshots/
    └── Docker 및 Git 실습 증거 이미지
```

## 4. Docker 웹 서버 구축

### 4-1. Dockerfile 작성

Nginx Alpine 이미지를 기반으로 웹 서버를 구성하였다.

```dockerfile
FROM nginx:alpine

COPY app/index.html /usr/share/nginx/html/index.html

EXPOSE 80
```

### 4-2. Docker 이미지 빌드

```powershell
docker build -t my-web .
```

### 4-3. 컨테이너 실행 및 포트 매핑

```powershell
docker run -d -p 8080:80 --name my-web-container my-web
```

호스트의 8080 포트와 컨테이너의 80 포트를 연결하였다.

브라우저에서 다음 주소로 접속하여 웹 서버 실행을 확인하였다.

`http://localhost:8080`

## 5. Bind Mount 검증

Bind Mount를 이용하여 로컬의 HTML 파일을 컨테이너 내부 웹 서버 디렉터리와 연결하였다.

```powershell
docker run -d -p 8080:80 --name my-web-bind -v "${PWD}\app:/usr/share/nginx/html" nginx:alpine
```

로컬의 `app/index.html` 파일을 수정한 뒤 브라우저를 새로고침하여 변경 내용이 반영되는 것을 확인하였다.

이를 통해 이미지를 다시 빌드하지 않고도 로컬 파일 변경 사항을 컨테이너에서 확인할 수 있음을 검증하였다.

## 6. Docker Volume 영속성 검증

### 6-1. Volume 생성

```powershell
docker volume create web-data
```

### 6-2. 데이터 저장

```powershell
docker run --name volume-test-1 -v web-data:/data alpine sh -c "echo 'Docker Volume persistence test' > /data/persistence.txt && cat /data/persistence.txt"
```

### 6-3. 기존 컨테이너 삭제

```powershell
docker rm volume-test-1
```

### 6-4. 새 컨테이너에서 데이터 확인

```powershell
docker run --rm -v web-data:/data alpine cat /data/persistence.txt
```

실행 결과:

```text
Docker Volume persistence test
```

기존 컨테이너를 삭제한 뒤에도 데이터가 유지되는 것을 확인하였다.

## 7. Docker 운영 로그

Docker 명령어 및 실행 결과는 다음 파일에 기록하였다.

- [Docker 운영 로그](logs/docker-operations.txt)

## 8. 트러블슈팅

### 문제 1. Dockerfile을 파일이 아닌 디렉터리로 생성

**문제 상황**

처음 Dockerfile을 생성하는 과정에서 파일이 아닌 디렉터리로 생성되어 Docker 이미지 빌드에 실패하였다.

**해결 방법**

PowerShell에서 `Get-Item` 명령어로 파일 유형을 확인하고, 기존 디렉터리 이름을 변경한 후 올바른 Dockerfile을 생성하였다.

```powershell
Get-Item .\Dockerfile
```

`PSIsContainer` 속성을 확인하여 디렉터리인지 파일인지 구분하였다.

### 문제 2. Docker 컨테이너 이름 충돌

**문제 상황**

이미 존재하는 `volume-test-2`라는 컨테이너 이름을 다시 사용하려 하여 이름 충돌 오류가 발생하였다.

**해결 방법**

기존 컨테이너 목록을 확인하여 이름 중복 여부를 파악하고, 기존 컨테이너를 삭제하거나 다른 이름을 사용하도록 하였다.

```powershell
docker ps -a
```

일회성 테스트에는 `--rm` 옵션을 사용하여 테스트 종료 후 컨테이너가 자동으로 삭제되도록 구성하였다.

## 9. Git 및 GitHub 관리

Git을 이용하여 프로젝트의 버전 관리를 수행하였다.

주요 작업:

- Git 저장소 초기화
- `main` 브랜치 설정
- Git 사용자 정보 설정
- 프로젝트 파일 커밋
- 스크린샷 변경 사항 추가 커밋
- GitHub 원격 저장소 연결
- 원격 저장소에 Push

주요 명령어:

```powershell
git init
git branch -M main
git add .
git commit -m "Initial development workstation setup"

git remote add origin <GitHub Repository URL>
git push -u origin main
```

## 10. 결과 및 결론

본 프로젝트를 통해 Windows 환경에서 Git과 Docker를 활용한 기본 개발 워크플로를 실습하였다.

Docker 이미지 빌드 및 웹 서버 실행, 포트 매핑, Bind Mount를 통한 파일 변경 반영, Docker Volume을 이용한 데이터 영속성을 검증하였다.

또한 Git을 이용하여 프로젝트 파일과 실습 증거 자료를 버전 관리하고 GitHub 원격 저장소에 업로드하였다.

이를 통해 개발 환경 구축부터 컨테이너 운영 및 프로젝트 버전 관리까지의 기본 과정을 경험하였다.