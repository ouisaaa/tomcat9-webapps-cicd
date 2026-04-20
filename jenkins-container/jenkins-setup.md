# Jenkins 설치 및 설정 가이드

개발 서버 리소스 제약 환경을 고려하여 경량 Alpine 기반 Jenkins를 Docker Compose로 구성한 가이드입니다.

## 구성 개요

| 항목 | 내용 |
|------|------|
| Jenkins 이미지 | `jenkins/jenkins:lts-alpine` (경량 Alpine 기반) |
| 빌드 방식 | 일회용 빌드 컨테이너 (`gradle:8.5-jdk11`) 분리 |
| Docker 접근 | 호스트 소켓 마운트 (`/var/run/docker.sock`) |
| 메모리 제한 | 1g (JVM 힙: 512m ~ 768m) |
| 접속 포트 | 9001 |

> **설계 배경**: 개발 서버 리소스가 제한적인 환경에서 Jenkins 자체의 메모리 사용량을 최소화하고, 빌드 시에만 일회용 Gradle 컨테이너를 띄워 리소스를 효율적으로 사용하는 구조로 구성했습니다.

---

## 사전 요구사항

- Docker, Docker Compose 설치
- Docker 그룹 GID 확인 필요

```bash
# Docker 그룹 GID 확인
cat /etc/group | grep docker
# 출력 예시: docker:x:979:
# → 979가 GID
```

---

## 디렉토리 구조

```
/cicd/
├── data/          # Jenkins 홈 디렉토리 (볼륨 마운트)
└── build_lib/     # 빌드 공유 라이브러리 (선택)

conf/
├── Dockerfile          # Jenkins 커스텀 이미지
└── jenkins_builder.yml # Docker Compose 파일
```

---

## 설치

### 1. 데이터 디렉토리 생성

```bash
mkdir -p /cicd/data /cicd/build_lib
```

### 2. docker-compose GID 수정

`jenkins_builder.yml`의 `group_add` 값을 실제 docker 그룹 GID로 변경합니다.

```yaml
group_add:
  - "979"   # 실제 GID로 변경
```

### 3. 이미지 빌드 및 컨테이너 실행

```bash
# conf/ 디렉토리에서 실행
docker compose -f jenkins_builder.yml up -d --build
```

### 4. 초기 관리자 비밀번호 확인

```bash
docker exec jenkins cat /var/jenkins_home/secrets/initialAdminPassword
```

### 5. 초기 설정

1. `http://<서버IP>:9001` 접속
2. 초기 비밀번호 입력
3. **Install suggested plugins** 선택
4. 관리자 계정 생성

---

## Credentials 등록

Jenkins UI → **Dashboard > Manage Jenkins > Credentials > System > Global**

| 항목 | 값 |
|------|-----|
| Kind | Username with password |
| Username | GitHub 계정명 |
| Password | GitHub Personal Access Token |
| ID | `github-credentials` |

> GitHub Personal Access Token은 `repo` 권한 필요

---

## Pipeline Job 생성

1. **New Item** → Pipeline 선택
2. **Pipeline** 탭 → Definition: `Pipeline script from SCM`
3. SCM: `Git`
4. Repository URL: `https://github.com/<your-org>/<your-repo>.git`
5. Credentials: `github-credentials`
6. Script Path: `Jenkinsfile`
7. 저장

---

## 트러블슈팅

**Jenkins에서 docker 명령어 Permission denied**
```bash
# docker 소켓 권한 확인
ls -la /var/run/docker.sock

# group_add GID가 실제 docker 그룹 GID와 일치하는지 확인
cat /etc/group | grep docker
```

**메모리 부족으로 Jenkins 기동 불안정**
```yaml
# jenkins_builder.yml에서 힙 사이즈 조정
environment:
  - JAVA_OPTS=-Xms256m -Xmx512m ...
```

**플러그인 설치 실패 (이미지 빌드 시)**
```bash
# 이미지 캐시 없이 재빌드
docker compose -f jenkins_builder.yml build --no-cache
```
