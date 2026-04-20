# 경량 Jenkins 이미지 빌드
FROM jenkins/jenkins:lts-alpine
USER root

# Docker CLI 설치 (Alpine 패키지 매니저 apk 사용)
RUN apk add --no-cache docker-cli curl tar && \
    curl -fsSL https://github.com/cli/cli/releases/download/v2.40.1/gh_2.40.1_linux_amd64.tar.gz | tar xz && \
    mv gh_*/bin/gh /usr/local/bin/ && \
    rm -rf gh_*

# Jenkins 유저로 복귀
USER jenkins

# 필수 플러그인 설치
RUN jenkins-plugin-cli --plugins \
    git \
    docker-workflow \
    workflow-aggregator
