# tomcat9-webapps-cicd

Tomcat 9 컨테이너에 WAR 파일을 빌드하고 배포하는 Jenkins CI/CD 환경입니다.

개발 서버의 리소스 제약을 고려하여 Jenkins 자체는 경량 Alpine 이미지로 메모리 사용량을 최소화하고,
빌드 시에는 별도의 일회용 빌드 컨테이너(Gradle/Maven)를 생성하여 빌드 후 제거하는 방식으로 구성했습니다.

## 구성 개요

| 항목 | 내용 |
|------|------|
| CI/CD 도구 | Jenkins (Docker 컨테이너) |
| Jenkins 이미지 | `jenkins/jenkins:lts-alpine` (경량) |
| 빌드 방식 | 일회용 빌드 컨테이너 분리 (Gradle / Maven) |
| 빌드 결과물 | WAR 파일 |
| 배포 대상 | Tomcat 9 컨테이너 |
| 파이프라인 수 | 2개 (Gradle, Maven) |

## 디렉토리 구조

```
.
├── jenkins-container/
│   ├── Dockerfile          # 경량 Alpine 기반 Jenkins 커스텀 이미지
│   ├── jenkins_builder.yml # Docker Compose 실행 파일
│   └── jenkins-setup.md    # Jenkins 설치 및 초기 설정 가이드
└── pipelines/
    ├── Jenkinsfile-gradle-deploy  # Gradle 빌드 → WAR 배포 파이프라인
    └── Jenkinsfile-maven-deploy   # Maven 빌드 → WAR 배포 파이프라인
```

## 시작하기

Jenkins 컨테이너 설치 및 초기 설정은 [jenkins-setup.md](jenkins-container/jenkins-setup.md)를 참고하세요.