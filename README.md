# Traefik Reverse Proxy Setup

Traefik v3.4 기반 리버스 프록시 설정입니다.

## 프로젝트 구조

```
.
├── docker-compose.yml    # Traefik 컨테이너 설정
├── traefik.yml          # Traefik 메인 설정
├── dynamic/             # 동적 라우팅 설정
│   ├── routes.yaml      # TCP/HTTP 라우팅 규칙
│   └── tls.yaml         # TLS 인증서 설정
└── certs/               # TLS 인증서 파일 (git ignore)
```

## 주요 기능

- **자동 HTTP → HTTPS 리다이렉션**: 포트 80으로 들어오는 모든 트래픽을 HTTPS로 리다이렉트
- **Docker 통합**: Docker 컨테이너 자동 감지 및 라우팅
- **파일 기반 설정**: `dynamic/` 디렉토리의 YAML 파일로 라우팅 관리
- **대시보드**: `http://localhost:8080`에서 Traefik 관리 UI 접근 가능

## 사전 요구사항

1. Docker 및 Docker Compose 설치
2. TLS 인증서 파일 준비 (`certs/` 디렉토리)
   - `local.pem` (인증서)
   - `local-key.pem` (개인 키)
3. Docker 네트워크 생성:
   ```bash
   docker network create traefik
   ```

## 사용 방법

### 1. Traefik 시작

```bash
docker-compose up -d
```

### 2. 상태 확인

```bash
docker-compose ps
docker-compose logs -f traefik
```

### 3. 대시보드 접속

브라우저에서 `http://localhost:8080` 접속

## 설정 파일 설명

### traefik.yml

- **Entry Points**:
  - `web` (80): HTTP, HTTPS로 자동 리다이렉트
  - `websecure` (443): HTTPS

- **Providers**:
  - File provider: `dynamic/` 디렉토리 감시
  - Docker provider: `traefik` 네트워크의 컨테이너 자동 감지

- **API**: 대시보드 활성화 (insecure mode)

### dynamic/routes.yaml

TCP 라우터 설정 예시:
- 도메인: `my-service.dev`
- TLS Passthrough 활성화
- 백엔드: `my-service:443`

### dynamic/tls.yaml

TLS 인증서 경로 지정:
- 인증서: `/certs/{cert}.pem`
- 개인 키: `/certs/{key}.pem`


## 라우팅 추가/수정

1. `dynamic/routes.yaml` 파일 수정
2. 변경사항 저장 시 Traefik이 자동으로 감지하여 적용
3. 재시작 불필요

## 포트

- **80**: HTTP (→ HTTPS 리다이렉트)
- **443**: HTTPS
- **8080**: Traefik 대시보드

## 보안 고려사항

- `api.insecure: true`는 개발 환경용입니다. 프로덕션에서는 비활성화하고 인증을 추가하세요.
- `certs/` 디렉토리는 `.gitignore`에 포함되어 있습니다.
- Docker 소켓은 읽기 전용(`ro`)으로 마운트됩니다.
- `no-new-privileges` 보안 옵션이 활성화되어 있습니다.

## 문제 해결

### 컨테이너가 시작되지 않는 경우

```bash
# 로그 확인
docker-compose logs traefik

# 네트워크 확인
docker network ls | grep traefik
```

### 라우팅이 작동하지 않는 경우

1. 대시보드(`http://localhost:8080`)에서 라우터 상태 확인
2. `dynamic/` 디렉토리의 YAML 문법 오류 확인
3. Docker 컨테이너 라벨 확인

### 인증서 오류

- `certs/` 디렉토리에 인증서 파일이 존재하는지 확인
- 파일 권한 확인
- 인증서 경로가 `tls.yaml`과 일치하는지 확인

## 참고 자료

- [Traefik 공식 문서](https://doc.traefik.io/traefik/)
- [Docker Provider 설정](https://doc.traefik.io/traefik/providers/docker/)
- [File Provider 설정](https://doc.traefik.io/traefik/providers/file/)