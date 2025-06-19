# Copyright Broadcom, Inc. All Rights Reserved.
# SPDX-License-Identifier: APACHE-2.0

FROM docker.io/bitnami/minideb:bookworm

ARG DOWNLOADS_URL="downloads.bitnami.com/files/stacksmith"
ARG TARGETARCH

LABEL com.vmware.cp.artifact.flavor="sha256:c50c90cfd9d12b445b011e6ad529f1ad3daea45c26d20b00732fae3cd71f6a83" \
      org.opencontainers.image.base.name="docker.io/bitnami/minideb:bookworm" \
      org.opencontainers.image.created="2025-06-13T18:44:56Z" \
      org.opencontainers.image.description="Application originally packaged by Broadcom, Inc. Modified by AndyWebServices" \
      org.opencontainers.image.documentation="https://github.com/AndyWebServices/bitnami-containers/tree/main/bitnami/mongodb/README.md" \
      org.opencontainers.image.ref.name="8.0.10-debian-12-r2" \
      org.opencontainers.image.source="https://github.com/AndyWebServices/bitnami-containers/tree/main/bitnami/mongodb" \
      org.opencontainers.image.title="mongodb" \
      org.opencontainers.image.vendor="AndyWebServices" \
      org.opencontainers.image.version="8.0.10"

ENV HOME="/" \
    OS_ARCH="${TARGETARCH:-amd64}" \
    OS_FLAVOUR="debian-12" \
    OS_NAME="linux"

COPY prebuildfs /
SHELL ["/bin/bash", "-o", "errexit", "-o", "nounset", "-o", "pipefail", "-c"]
# Install required system packages and dependencies
RUN install_packages ca-certificates curl libbrotli1 libcom-err2 libcurl4 libffi8 libgcc-s1 libgmp10 libgnutls30 libgssapi-krb5-2 libhogweed6 libidn2-0 libk5crypto3 libkeyutils1 libkrb5-3 libkrb5support0 libldap-2.5-0 libnettle8 libnghttp2-14 libp11-kit0 libpsl5 librtmp1 libsasl2-2 libssh2-1 libssl3 libtasn1-6 libunistring2 libzstd1 numactl procps zlib1g
RUN mkdir -p /tmp/bitnami/pkg/cache/ ; cd /tmp/bitnami/pkg/cache/ || exit 1 ; \
    COMPONENTS=( \
      "yq-4.45.4-3-linux-${OS_ARCH}-debian-12" \
      "wait-for-port-1.0.8-18-linux-${OS_ARCH}-debian-12" \
      "render-template-1.0.7-18-linux-${OS_ARCH}-debian-12" \
    ) ; \
    if [ "${OS_ARCH}" = "amd64" ]; then \
      COMPONENTS+=( \
        "mongodb-shell-2.5.2-1-linux-${OS_ARCH}-debian-12" \
        "mongodb-8.0.10-2-linux-${OS_ARCH}-debian-12" \
      ) ; \
    fi ; \
    for COMPONENT in "${COMPONENTS[@]}"; do \
      if [ ! -f "${COMPONENT}.tar.gz" ]; then \
        curl -SsLf "https://${DOWNLOADS_URL}/${COMPONENT}.tar.gz" -O ; \
        curl -SsLf "https://${DOWNLOADS_URL}/${COMPONENT}.tar.gz.sha256" -O ; \
      fi ; \
      sha256sum -c "${COMPONENT}.tar.gz.sha256" ; \
      tar -zxf "${COMPONENT}.tar.gz" -C /opt/bitnami --strip-components=2 --no-same-owner ; \
      rm -rf "${COMPONENT}".tar.gz{,.sha256} ; \
    done
RUN apt-get update && apt-get upgrade -y && \
    apt-get clean && rm -rf /var/lib/apt/lists /var/cache/apt/archives

# Only install MongoDB from the repo if OS_ARCH is not amd64
ENV MONGO_PACKAGE=mongodb-org
ENV MONGO_VERSION=8.0.10
RUN if [ "${OS_ARCH}" != "amd64" ]; then \
    set -eux \
    && apt-get update \
    && apt-get install -y --no-install-recommends curl gnupg wget \
	&& (curl -fsSL https://www.mongodb.org/static/pgp/server-8.0.asc | gpg -o /usr/share/keyrings/mongodb-server-8.0.gpg --dearmor;) \
    && echo "deb [ arch=amd64,arm64 signed-by=/usr/share/keyrings/mongodb-server-8.0.gpg ] https://repo.mongodb.org/apt/ubuntu jammy/mongodb-org/8.0 multiverse" | tee "/etc/apt/sources.list.d/mongodb-org-8.0.list" \
    && export DEBIAN_FRONTEND=noninteractive \
	&& apt-get update \
	&& apt-get install -y \
		${MONGO_PACKAGE}=$MONGO_VERSION \
		${MONGO_PACKAGE}-server=$MONGO_VERSION \
		${MONGO_PACKAGE}-shell=$MONGO_VERSION \
		${MONGO_PACKAGE}-mongos=$MONGO_VERSION \
		${MONGO_PACKAGE}-tools=$MONGO_VERSION \
		${MONGO_PACKAGE}-database=$MONGO_VERSION \
		${MONGO_PACKAGE}-database-tools-extra=$MONGO_VERSION \
    && mkdir -p /opt/bitnami/mongodb/bin \
    && ln -s /usr/bin/install_compass            /opt/bitnami/mongodb/bin/ \
    && ln -s /usr/bin/bsondump                   /opt/bitnami/mongodb/bin/ \
    && ln -s /usr/bin/mongo*                     /opt/bitnami/mongodb/bin/ \
    && rm -rf /etc/mysql/mongodb.cnf ; \
fi

RUN chmod g+rwX /opt/bitnami
RUN find / -perm /6000 -type f -exec chmod a-s {} \; || true
RUN uninstall_packages curl

COPY rootfs /
RUN /opt/bitnami/scripts/mongodb/postunpack.sh
ENV APP_VERSION="8.0.10" \
    BITNAMI_APP_NAME="mongodb" \
    PATH="/opt/bitnami/common/bin:/opt/bitnami/mongodb/bin:$PATH"

EXPOSE 27017

USER 1001
ENTRYPOINT [ "/opt/bitnami/scripts/mongodb/entrypoint.sh" ]
CMD [ "/opt/bitnami/scripts/mongodb/run.sh" ]
