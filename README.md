# AWS·Docker 기반 웹 서비스 인프라 구축

AWS EC2와 Docker Compose를 활용하여 Nginx Reverse Proxy와 Python Backend로 구성된 웹 서비스 인프라를 구축하고, 장애 상황을 직접 발생시켜 로그를 확인하고 복구하는 과정을 실습한 프로젝트입니다.

## 1. 프로젝트 개요

- **프로젝트명:** AWS·Docker 기반 웹 서비스 인프라 구축
- **목적:** AWS 클라우드 환경에서 Linux 서버와 Docker 기반 웹 서비스 인프라 구축 및 운영 경험 확보
- **환경:** AWS EC2, Ubuntu Linux
- **컨테이너:** Docker, Docker Compose
- **Web Server:** Nginx
- **Backend:** Python HTTP Server

## 2. 시스템 구성

```text
Internet
   │
   ▼
AWS EC2
Ubuntu Linux
   │
   ▼
Docker Compose
   │
   ├── Nginx Container
   │       │
   │       │ Reverse Proxy
   │       ▼
   └── Python Backend Container
           │
           ▼
        Port 8080
```

Nginx는 외부의 HTTP 요청을 받아 Backend 컨테이너의 8080 포트로 전달하도록 구성했습니다.

## 3. 주요 구성 요소

| 구성 요소 | 역할 |
|---|---|
| AWS EC2 | 클라우드 서버 환경 제공 |
| Ubuntu Linux | 서버 운영체제 |
| Docker | 애플리케이션 컨테이너 실행 |
| Docker Compose | 여러 컨테이너 관리 |
| Nginx | Web Server 및 Reverse Proxy |
| Python HTTP Server | Backend 서버 |

## 4. Docker Compose 구성

### Nginx

- 외부 포트 `80` 사용
- `nginx.conf`를 컨테이너에 연결
- Backend 컨테이너로 요청 전달
- `restart: unless-stopped` 적용

### Backend

- Python 기반 HTTP Server
- 컨테이너 내부 `8080` 포트 사용
- 외부에는 직접 포트를 공개하지 않고 Docker Compose 네트워크를 통해 Nginx에서 접근
- `restart: unless-stopped` 적용

## 5. 프로젝트 구조

```text
aws-docker-infra-project/
├── backend/
│   ├── Dockerfile
│   └── index.html
├── compose.yml
├── nginx.conf
└── README.md
```

## 6. 주요 작업

### AWS EC2 서버 구축

Ubuntu Linux 기반 EC2 인스턴스를 생성하고 SSH를 이용하여 서버에 접속했습니다.

### Docker 환경 구축

Docker를 설치하고 컨테이너의 생성, 실행, 중지, 재시작 및 로그 확인을 실습했습니다.

```bash
docker ps
docker ps -a
docker start
docker stop
docker logs
docker exec
```

### Nginx Reverse Proxy 구성

Nginx가 외부 요청을 받아 Backend 컨테이너로 전달하도록 Reverse Proxy를 구성했습니다.

```text
Client
  ↓
Nginx :80
  ↓
Backend :8080
```

### Docker Compose 적용

기존에 개별적으로 실행하던 컨테이너를 Docker Compose를 이용하여 하나의 구성으로 관리했습니다.

```bash
docker compose up -d --build
docker compose ps
```

### 컨테이너 자동 재시작

`restart: unless-stopped`를 적용하여 컨테이너 장애 또는 서버 재부팅 이후에도 컨테이너가 자동으로 다시 실행되도록 구성했습니다.

## 7. 장애 상황 테스트

인프라 운영 과정에서 발생할 수 있는 장애 상황을 직접 재현했습니다.

### Backend 장애 발생

Backend 서버를 중지한 후 웹 페이지에 접속하여 Nginx에서 `502 Bad Gateway`가 발생하는 것을 확인했습니다.

```text
Client
  ↓
Nginx
  ↓
Backend 연결 실패
  ↓
502 Bad Gateway
```

### 로그 확인

Nginx 로그를 확인하여 Backend 연결 실패 원인을 분석했습니다.

```bash
docker logs --tail 30 infra-nginx
```

로그에서 다음과 같은 연결 거부 메시지를 확인했습니다.

```text
connect() failed (111: Connection refused)
```

이를 통해 Nginx 자체의 문제가 아니라 Backend 서버가 실행되지 않은 것이 원인임을 확인했습니다.

이후 Backend를 다시 실행하여 서비스를 정상 상태로 복구했습니다.

## 8. 서버 재부팅 테스트

EC2 서버를 재부팅한 후 Docker 컨테이너의 상태를 확인했습니다.

`restart: unless-stopped` 설정에 따라 Nginx와 Backend 컨테이너가 다시 실행되는 것을 확인했습니다.

이를 통해 서버 재부팅 상황에서도 Docker 기반 서비스가 자동으로 복구되는 것을 확인했습니다.

## 9. 사용 기술

- AWS EC2
- Ubuntu Linux
- Docker
- Docker Compose
- Nginx
- Python
- Git
- GitHub

## 10. 배운 점

이번 프로젝트를 통해 단순히 서버를 실행하는 것뿐만 아니라 웹 서비스가 어떤 구조로 연결되는지 직접 확인할 수 있었습니다.

특히 Nginx Reverse Proxy를 구성하면서 Client의 요청이 Nginx를 거쳐 Backend로 전달되는 과정을 이해했고, Backend 장애 상황에서 `502 Bad Gateway`를 직접 발생시켜 로그를 분석하고 원인을 찾아 복구하는 과정을 경험했습니다.

또한 Docker Compose를 이용하여 여러 컨테이너를 하나의 구성으로 관리하고, 서버 재부팅 이후 컨테이너가 자동으로 복구되는 환경을 구성하면서 기본적인 인프라 운영 과정을 경험했습니다.
