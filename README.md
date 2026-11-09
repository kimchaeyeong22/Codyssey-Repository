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

##Docker 버전 및 데몬 상태 증빙

VERSION

docker --version

Docker version 29.9.2, build a82e528

SERVER STATUS

docker info

Server Version: 29.9.2

Containers: 2

Running: 1

Stopped: 1

Images: 3

SYSTEM INFORMATION

Operating System: Docker Desktop

OSType: linux

Architecture: x86_64

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


### 이하 추가 및 보완 내용


### 파일 시스템 관리 실습

PowerShell을 사용하여 기본적인 파일 및 폴더 관리 작업을 수행하였다.



**수행 내용**
- `New-Item`: 폴더 및 파일 생성
- `Set-Content`: 파일에 텍스트 작성
- `Get-Content`: 파일 내용 확인
- `Get-ChildItem`: 폴더 내부 파일 및 하위 폴더 확인
- `Move-Item`: 파일을 다른 폴더로 이동
- `Remove-Item`: 파일 및 폴더 삭제
- `Test-Path`: 파일 및 폴더의 존재 여부 확인

**검증 결과**
- 파일 및 폴더 생성 완료
- 파일 내용 작성 및 읽기 확인
- 파일 이동 후 대상 폴더에서 파일 확인
- 파일 삭제 후 `Test-Path` 결과 `False` 확인
- 폴더 삭제 후 `Test-Path` 결과 `False` 확인

**실행 기록**

실제 PowerShell 명령어와 실행 결과는 다음 로그 파일에 기록하였다.

- [파일 시스템 작업 로그](logs/filesystem-operations.txt)

`Start-Transcript`와 `Stop-Transcript`를 사용하여 명령어 및 실행 결과를 기록하였다.
## 파일 시스템 관리 실습

PowerShell을 활용하여 파일 및 폴더의 생성, 조회, 이동, 삭제 작업을 수행하였다.

### 수행한 작업

| 작업 | 명령어 | 검증 결과 |
|---|---|---|
| 폴더 생성 | `New-Item -ItemType Directory` | 생성 결과 확인 |
| 파일 생성 | `New-Item -ItemType File` | 생성 결과 확인 |
| 파일 내용 작성 | `Set-Content` | 텍스트 작성 완료 |
| 파일 내용 조회 | `Get-Content` | 작성한 텍스트 출력 확인 |
| 폴더 내부 조회 | `Get-ChildItem` | 파일 및 폴더 목록 확인 |
| 파일 이동 | `Move-Item` | 이동한 파일 확인 |
| 파일 삭제 | `Remove-Item` | `Test-Path` 결과 `False` |
| 폴더 삭제 | `Remove-Item -Recurse -Force` | `Test-Path` 결과 `False` |

### 실행 결과 및 증빙

PowerShell의 `Start-Transcript` 기능을 사용하여 실제 명령어와 실행 결과를 로그 파일에 기록하였다.

- [파일 시스템 작업 전체 로그](logs/filesystem-operations.txt)

로그에는 파일과 폴더의 생성, 파일 이동 및 삭제, 삭제 여부 검증 결과가 포함되어 있다.




Add-Content -Path .\README.md -Value @'

## 파일 권한 관리 실습

Windows PowerShell에서 `icacls`, `Get-Acl`, `Set-Acl` 명령어를 사용하여 파일 접근 권한을 확인하고 변경하였다.

### 주요 작업
- `Get-Acl`: 파일의 기존 접근 권한 확인
- `icacls`: 현재 파일 권한 확인 및 읽기 권한 부여
- `Set-Acl`: 변경 전 백업한 ACL을 이용하여 원래 권한 복구
- `Get-Acl`: 권한 복구 결과 확인

### 실습 결과
- 테스트 파일: `permission-test/sample.txt`
- 읽기 권한 `(R)` 부여 성공
- 변경된 ACL 확인 성공
- 원래 권한 복구 성공
- 최종 권한 확인 성공

### 작업 로그
[파일 권한 작업 전체 로그](logs/permission-operations.txt)
'@







Add-Content -Path .\README.md -Value @'

## Docker Hello-world 실행 실습

Docker의 기본 실행 테스트를 위해 `docker run hello-world` 명령어를 실행하였다.

### 수행 결과
- 로컬에 이미지가 없어 Docker Hub에서 `hello-world` 이미지 다운로드
- 컨테이너 생성 및 실행 성공
- `Hello from Docker!` 출력 확인
- `docker ps -a` 명령어로 컨테이너 상태 확인
- 컨테이너 상태: `Exited (0)` (정상 종료)

### 작업 로그
[Docker Hello-world 실행 전체 로그](logs/docker-hello-world.txt)
'@



## Docker 이미지 빌드

- Dockerfile 기반 이미지 빌드 실행
- 실행 명령어: `docker build -t my-web .`
- 빌드 결과: 성공
- PowerShell 작업 기록: [docker-build-operations.txt](./logs/docker-build-operations.txt)
- 실제 Docker 빌드 출력: [docker-build-output.txt](./logs/docker-build-output.txt)



## Git 사용자 정보 설정

- 사용자 이름 설정: `git config --local user.name`
- 이메일 설정: `git config --local user.email`
- 설정 확인: `git config --local --list`
- 설정 범위: 현재 저장소(Local)
- 실행 기록: [git-config-operations.txt](./logs/git-config-operations.txt)



### 프로젝트 디렉터리 설계 원칙

본 프로젝트는 **기능별 파일 분리와 작업 기록의 체계적인 관리**를 기준으로 디렉터리를 구성하였다.

- `app/`: 웹 애플리케이션의 소스 코드를 보관한다. 실제 서비스에 사용되는 파일을 Docker 설정 및 문서와 분리하여 관리하기 위해 구성하였다.
- `logs/`: 파일 시스템, 권한 관리, Git 및 Docker 명령어의 실행 기록과 결과를 보관한다. 실습 과정의 재현성과 검증 가능성을 확보하기 위해 구성하였다.
- `screenshots/`: PowerShell 명령어 실행 화면과 웹 브라우저 접속 결과 등 시각적 증거를 보관한다. 텍스트 로그만으로 확인하기 어려운 작업 결과를 보완하기 위해 구성하였다.

### 루트 디렉터리 파일의 역할

- `Dockerfile`: 웹 애플리케이션을 Docker 이미지로 빌드하기 위한 설정을 정의한다.
- `README.md`: 프로젝트 개요, 디렉터리 구조, 실행 방법 및 실습 결과를 문서화한다.
- `.gitignore`: 버전 관리에 포함하지 않을 파일과 디렉터리를 지정하여 불필요한 파일이 저장소에 추가되는 것을 방지한다.

### 디렉터리 구성 기준

1. **관심사 분리**: 소스 코드, 실행 기록, 시각적 증거를 각각 별도의 폴더에서 관리한다.
2. **유지보수성**: 파일의 목적에 따라 위치를 구분하여 필요한 자료를 쉽게 찾고 수정할 수 있도록 한다.
3. **재현성과 검증**: 명령어 실행 로그와 스크린샷을 함께 보관하여 실습 결과를 확인할 수 있도록 한다.
4. **버전 관리 효율성**: Git으로 관리할 프로젝트 파일과 제외할 파일을 구분하여 저장소를 체계적으로 유지한다.

### Docker 이미지와 컨테이너의 개념

Docker 이미지(Image)는 애플리케이션 실행에 필요한 파일과 환경을 포함하는 읽기 전용 템플릿이다. Docker 컨테이너(Container)는 해당 이미지를 기반으로 생성되어 실제 프로그램을 실행하는 독립적인 환경이다.

- **이미지(Image):** `docker build` 명령으로 생성하며, 컨테이너를 실행하기 위한 기반이 된다.
- **컨테이너(Container):** `docker run` 명령으로 이미지를 실행한 인스턴스이다. 실행 중인 컨테이너에는 별도의 쓰기 영역이 존재한다.
- **저장 방식:** 컨테이너 내부의 변경 사항은 기본적으로 컨테이너의 쓰기 영역에 저장된다. 컨테이너 삭제 후에도 데이터를 유지하려면 Docker Volume 또는 Bind Mount를 사용한다.

### Docker 이미지 빌드 및 컨테이너 실행 과정

본 프로젝트는 Nginx 기반 이미지를 생성하고 이를 컨테이너로 실행하여 웹 서비스를 제공한다.

1. **소스 파일 작성 및 수정:** `app/index.html` 파일을 수정한다.
2. **이미지 빌드:** `docker build -t my-web .` 명령으로 Dockerfile을 기반으로 이미지를 생성한다. 이 과정에서 `COPY` 명령을 통해 HTML 파일이 이미지 내부의 Nginx 웹 루트에 복사된다.
3. **컨테이너 실행:** `docker run -d -p 8080:80 --name my-web-container my-web` 명령으로 컨테이너를 실행하고 호스트의 8080 포트와 컨테이너의 80 포트를 연결한다.
4. **서비스 확인:** 브라우저에서 `http://localhost:8080`에 접속하여 웹페이지가 제공되는지 확인한다.

### 소스 파일 수정 및 변경 사항 반영

Dockerfile의 `COPY` 명령은 이미지 빌드 시점의 소스 파일을 이미지 내부에 복사한다. 따라서 호스트의 `app/index.html`을 수정해도 기존 이미지와 실행 중인 컨테이너에는 변경 사항이 자동으로 반영되지 않는다.

수정된 파일을 서비스에 반영하려면 다음 과정을 수행한다.

1. `app/index.html` 수정
2. `docker build -t my-web .` 명령으로 이미지 재빌드
3. 기존 컨테이너 중지 및 제거
4. 새 이미지로 컨테이너 재생성
5. 브라우저에서 변경 사항 확인

Bind Mount를 사용하는 경우에는 호스트 파일과 컨테이너 내부 경로가 연결되므로 이미지 재빌드 없이 호스트 파일의 변경 사항을 컨테이너에서 확인할 수 있다.

이처럼 이미지 빌드 방식과 컨테이너 실행 방식, 데이터 저장 방식을 구분하여 관리한다.



### Docker 네트워크 네임스페이스와 포트 포워딩

Docker 컨테이너는 독립적인 네트워크 네임스페이스를 사용하여 호스트 PC와 분리된 네트워크 환경을 가진다. 따라서 호스트에서 컨테이너 내부의 웹 서버에 접근하려면 포트 포워딩을 통해 네트워크 연결을 설정해야 한다.

본 프로젝트에서는 `docker run -d -p 8080:80 --name my-web-container my-web` 명령을 사용하였다.

- 호스트 포트 `8080`: 외부 브라우저가 접속하는 포트
- 컨테이너 포트 `80`: Nginx 웹 서버가 요청을 수신하는 포트
- `-p 8080:80`: 호스트의 8080번 포트로 들어온 요청을 컨테이너의 80번 포트로 전달하는 설정

이를 통해 호스트 PC의 브라우저에서 `http://localhost:8080` 주소로 Docker 컨테이너 내부의 웹 서비스에 접근할 수 있다.


### 절대경로와 상대경로의 선택 기준

Docker Bind Mount에서 호스트 디렉터리를 지정할 때 절대경로와 상대경로를 사용할 수 있다.

**1. 절대경로(Absolute Path)**

절대경로는 드라이브 또는 루트 디렉터리부터 시작하는 전체 경로를 의미한다.

- 사용 사례: 특정 폴더의 위치를 명확하게 지정해야 할 때
- 여러 작업 디렉터리에서 동일한 폴더를 참조해야 할 때
- 자동화 스크립트나 배포 환경에서 경로를 명시적으로 관리할 때

예시:
```powershell
docker run -d -p 8080:80 --name my-web-absolute -v "C:\Users\사용자\dev-workstation-new\app:/usr/share/nginx/html" nginx:alpine
```

**2. 상대경로(Relative Path)**

상대경로는 현재 작업 디렉터리를 기준으로 파일이나 폴더의 위치를 지정하는 방식이다.

- 사용 사례: 프로젝트 내부의 폴더를 간단하게 참조할 때
- 프로젝트 루트 디렉터리에서 명령어를 실행할 때
- 프로젝트를 다른 컴퓨터로 옮겨도 폴더 구조가 동일하게 유지되는 경우

예시:
```powershell
docker run -d -p 8080:80 --name my-web-relative -v "${PWD}\app:/usr/share/nginx/html" nginx:alpine
```

**3. 경로 선택 기준**

- 절대경로: 경로의 위치를 명확하게 고정해야 하는 경우 사용한다.
- 상대경로: 프로젝트 내부의 폴더를 간결하게 참조하고 프로젝트의 이동 및 재사용성을 높이고 싶은 경우 사용한다.

본 프로젝트에서는 프로젝트 루트 디렉터리를 기준으로 `app` 폴더를 참조하기 때문에 상대경로 `${PWD}\app`을 사용하는 것이 적합하다. 단, 상대경로를 사용할 때는 명령어를 실행하는 현재 작업 디렉터리가 프로젝트 루트인지 확인해야 한다.