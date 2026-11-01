ARG BUILD_FROM=ghcr.io/home-assistant/base-python:3.14-alpine3.24-2026.10.1
ARG BUILD_ARCH=amd64

# renovate: datasource=repology depName=alpine_3_24/git
ARG GIT_VERSION=2.54.0-r0
# Alpine 3.22 is the last base image generation for the 32-bit architectures.
# renovate: datasource=repology depName=alpine_3_22/git
ARG GIT_VERSION_LEGACY=2.49.1-r0
# renovate: datasource=pypi depName=PyYAML
ARG PYYAML_VERSION=6.0.3

# Current multi-architecture images cover amd64 and aarch64. Keep the last official
# Python images for the older architectures; BuildKit only builds the selected stage.
FROM ${BUILD_FROM} AS base-amd64
FROM ${BUILD_FROM} AS base-aarch64
FROM ghcr.io/home-assistant/armv7-base-python:3.13-alpine3.22-2025.11.1 AS base-armv7
FROM ghcr.io/home-assistant/armhf-base-python:3.13-alpine3.22-2025.11.1 AS base-armhf
FROM ghcr.io/home-assistant/i386-base-python:3.13-alpine3.22-2025.11.1 AS base-i386

FROM base-${BUILD_ARCH}

# Python is supplied by the base image; install PyYAML into that same interpreter.
ARG GIT_VERSION
ARG GIT_VERSION_LEGACY
ARG PYYAML_VERSION
ARG BUILD_ARCH
RUN case "${BUILD_ARCH}" in \
        armv7|armhf|i386) GIT_VERSION="${GIT_VERSION_LEGACY}" ;; \
    esac \
    && apk add --no-cache git=${GIT_VERSION} \
    && python3 -m pip install --no-cache-dir PyYAML==${PYYAML_VERSION}

COPY rootfs/ /
COPY LICENSE THIRD_PARTY_NOTICES.md /usr/share/licenses/git-exporter/

# Git does not keep the executable bit when files are uploaded through the GitHub API.
RUN find /etc/s6-overlay/s6-rc.d -type f \( -name run -o -name finish \) -exec chmod a+x {} +

ARG BUILD_VERSION=2.0.0-beta.2
RUN mkdir -p /etc/git-exporter && echo "${BUILD_VERSION}" >/etc/git-exporter/version
LABEL io.hass.version="${BUILD_VERSION}" \
    io.hass.type="addon" \
    io.hass.arch="${BUILD_ARCH}" \
    org.opencontainers.image.source="https://github.com/seb5594/Home-Assistant-git-exporter-Addon"

ENV PYTHONPATH=/opt/git-exporter \
    PYTHONDONTWRITEBYTECODE=1 \
    PYTHONUNBUFFERED=1

HEALTHCHECK --interval=5m --timeout=10s --start-period=1m --retries=3 \
    CMD ["python3", "-c", "import urllib.request; urllib.request.urlopen('http://127.0.0.1:8099/api/log', timeout=5)"]
