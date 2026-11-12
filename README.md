# images
Used for building various images.

[![docker](https://github.com/iKelvinLab/images/actions/workflows/docker.yml/badge.svg)](https://github.com/iKelvinLab/images/actions/workflows/docker.yml)

## CI

GitHub Actions ([docker.yml](.github/workflows/docker.yml)) builds every top-level directory containing a `Dockerfile` and pushes to GHCR (`ghcr.io/ikelvinlab/<dir>`). Adding a new image directory requires no CI changes.

| Trigger | Behavior |
| --- | --- |
| Pull request | Multi-arch build, no push |
| Push to `main` | Build + push `<image>:latest` |
| Tag `<image>-v<ver>` (e.g. `helm-kubectl-v1.37.2-v4.3.1`) | Build only that image, push `<image>:v<ver>` |
| Manual dispatch | Build, push optional via input |

Builds are multi-arch (`linux/amd64` + `linux/arm64`) with GHA layer caching.

## helm-kubectl

Lightweight image (Alpine-based, ~133MB) shipping the `kubectl` and `helm` CLIs, running as non-root. Versions are overridable via build args; binaries are SHA-256 verified at build time and support `linux/amd64` + `linux/arm64`.

```bash
docker pull ghcr.io/ikelvinlab/helm-kubectl:latest   # or build locally:
docker build -t helm-kubectl:latest helm-kubectl
docker build --build-arg KUBECTL_VERSION=v1.37.2 --build-arg HELM_VERSION=v4.3.1 -t helm-kubectl:v1.37.2-v4.3.1 helm-kubectl

docker run --rm helm-kubectl kubectl version --client
docker run --rm -v ~/.kube:/home/kube/.kube helm-kubectl helm list
```
