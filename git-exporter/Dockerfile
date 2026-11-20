ARG BUILD_FROM=ghcr.io/seb5594/home-assistant-base-python:3.14-alpine3.24-2026.11.0

# renovate: datasource=pypi depName=PyYAML
ARG PYYAML_VERSION=6.0.3

FROM ${BUILD_FROM}

# Python is supplied by the base image; install PyYAML into that same interpreter.
ARG PYYAML_VERSION
RUN apk add --no-cache git \
    && python3 -m pip install --no-cache-dir PyYAML==${PYYAML_VERSION}

COPY rootfs/ /
COPY LICENSE THIRD_PARTY_NOTICES.md /usr/share/licenses/git-exporter/

# Git does not keep the executable bit when files are uploaded through the GitHub API.
RUN find /etc/s6-overlay/s6-rc.d -type f \( -name run -o -name finish \) -exec chmod a+x {} +

ARG BUILD_VERSION=2.0.0-beta.3
RUN mkdir -p /etc/git-exporter && echo "${BUILD_VERSION}" >/etc/git-exporter/version
LABEL io.hass.version="${BUILD_VERSION}" \
    io.hass.type="addon" \
    org.opencontainers.image.title="Home Assistant App: Home Assistant Git Exporter" \
    org.opencontainers.image.version="${BUILD_VERSION}" \
    org.opencontainers.image.source="https://github.com/seb5594/Home-Assistant-git-exporter-Addon"

ENV PYTHONPATH=/opt/git-exporter \
    PYTHONDONTWRITEBYTECODE=1 \
    PYTHONUNBUFFERED=1

HEALTHCHECK --interval=5m --timeout=10s --start-period=1m --retries=3 \
    CMD ["python3", "-c", "import urllib.request; urllib.request.urlopen('http://127.0.0.1:8099/api/log', timeout=5)"]
