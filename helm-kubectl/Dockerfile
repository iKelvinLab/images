# syntax=docker/dockerfile:1

# helm-kubectl —— 同时提供 kubectl 与 helm 的轻量 CLI 镜像
#
# 构建:
#   docker build -t helm-kubectl:latest .
#   docker build --build-arg HELM_VERSION=v4.3.1 --build-arg KUBECTL_VERSION=v1.37.2 \
#     -t helm-kubectl:v4.3.1-v1.37.2 .
#   docker buildx build --platform linux/amd64,linux/arm64 -t <registry>/helm-kubectl:latest --push .
#
# 使用:
#   docker run --rm helm-kubectl kubectl version --client
#   docker run --rm -v ~/.kube:/home/kube/.kube helm-kubectl kubectl get nodes
#   docker run --rm -v ~/.kube:/home/kube/.kube helm-kubectl helm list

ARG ALPINE_VERSION=3.22

# ---------- 第一阶段:下载并校验 ----------
FROM alpine:${ALPINE_VERSION} AS downloader

ARG KUBECTL_VERSION=v1.37.2
ARG HELM_VERSION=v4.3.1
# buildx 多架构构建时自动注入(amd64/arm64);本地构建时回退到 uname
ARG TARGETARCH

RUN apk add --no-cache curl

WORKDIR /tmp

RUN set -eux; \
    arch="${TARGETARCH:-$(uname -m | sed -e 's/x86_64/amd64/' -e 's/aarch64/arm64/')}"; \
    # kubectl
    curl -fsSLo kubectl "https://dl.k8s.io/release/${KUBECTL_VERSION}/bin/linux/${arch}/kubectl"; \
    curl -fsSLo kubectl.sha256 "https://dl.k8s.io/release/${KUBECTL_VERSION}/bin/linux/${arch}/kubectl.sha256"; \
    echo "$(cat kubectl.sha256)  kubectl" | sha256sum -c -; \
    chmod +x kubectl; \
    # helm
    curl -fsSLo "helm-${HELM_VERSION}-linux-${arch}.tar.gz" "https://get.helm.sh/helm-${HELM_VERSION}-linux-${arch}.tar.gz"; \
    curl -fsSLo "helm-${HELM_VERSION}-linux-${arch}.tar.gz.sha256sum" "https://get.helm.sh/helm-${HELM_VERSION}-linux-${arch}.tar.gz.sha256sum"; \
    sha256sum -c "helm-${HELM_VERSION}-linux-${arch}.tar.gz.sha256sum"; \
    tar -xzf "helm-${HELM_VERSION}-linux-${arch}.tar.gz"; \
    mv "linux-${arch}/helm" ./helm; \
    chmod +x helm; \
    # 冒烟测试
    ./kubectl version --client >/dev/null; \
    ./helm version >/dev/null

# ---------- 第二阶段:运行镜像 ----------
FROM alpine:${ALPINE_VERSION}

LABEL org.opencontainers.image.title="helm-kubectl" \
      org.opencontainers.image.description="Lightweight image shipping the kubectl and helm CLIs" \
      org.opencontainers.image.base.name="docker.io/library/alpine"

RUN apk add --no-cache ca-certificates bash; \
    addgroup -S kube; \
    adduser -S kube -G kube -h /home/kube -s /bin/bash; \
    mkdir -p /home/kube/.kube; \
    chown -R kube:kube /home/kube

COPY --from=downloader /tmp/kubectl /usr/local/bin/kubectl
COPY --from=downloader /tmp/helm /usr/local/bin/helm

USER kube
WORKDIR /home/kube

# 不设置 ENTRYPOINT,直接把 kubectl / helm 作为命令传入即可
CMD ["/bin/bash"]
