FROM debian:bullseye as unbound
LABEL maintainer="Matthew Vance"

ENV NAME=unbound \
    UNBOUND_VERSION=1.16.3 \
    UNBOUND_SHA256=ea0c6665e2c3325b769eac1dfccd60fe1828d5fcf662650039eccb3f67edb28e
ENV UNBOUND_DOWNLOAD_URL="https://nlnetlabs.nl/downloads/unbound/unbound-${UNBOUND_VERSION}.tar.gz"

RUN build_deps="curl gcc libc-dev libevent-dev libexpat1-dev libnghttp2-dev libssl-dev make" && \
    set -x && \
    DEBIAN_FRONTEND=noninteractive apt-get update && apt-get install -y --no-install-recommends \
      $build_deps \
      bsdmainutils \
      ca-certificates \
      ldnsutils \
      libevent-2.1-7 \
      libexpat1 \
      libnghttp2-14 \
      libprotobuf-c-dev \
      protobuf-c-compiler && \
    curl -sSL $UNBOUND_DOWNLOAD_URL -o unbound.tar.gz && \
    echo "${UNBOUND_SHA256} *unbound.tar.gz" | sha256sum -c - && \
    tar xzf unbound.tar.gz && \
    rm -f unbound.tar.gz && \
    cd "unbound-${UNBOUND_VERSION}" && \
    groupadd _unbound && \
    useradd -g _unbound -s /dev/null -d /etc _unbound && \
    ./configure \
        --disable-dependency-tracking \
        --with-pthreads \
        --with-username=_unbound \
        --with-libevent \
        --with-libnghttp2 \
        --enable-dnstap \
        --enable-tfo-server \
        --enable-tfo-client \
        --enable-event-api \
        --enable-subnet && \
    make DESTDIR=/ -j install && \
    apt-get purge -y --auto-remove \
      $build_deps && \
    rm -rf \
        /usr/unbound/share/man \
        /tmp/* \
        /var/tmp/* \
        /var/lib/apt/lists/*

ENV NAME=unbound \
    VERSION=1.0 \
    SUMMARY="${NAME} is a validating, recursive, and caching DNS resolver." \
    DESCRIPTION="${NAME} is a validating, recursive, and caching DNS resolver."

LABEL org.opencontainers.image.version=${UNBOUND_VERSION} \
      org.opencontainers.image.title="mvance/unbound" \
      org.opencontainers.image.description="a validating, recursive, and caching DNS resolver" \
      org.opencontainers.image.url="https://github.com/MatthewVance/unbound-docker" \
      org.opencontainers.image.vendor="Matthew Vance" \
      org.opencontainers.image.licenses="MIT" \
      org.opencontainers.image.source="https://github.com/MatthewVance/unbound-docker"

EXPOSE 53/tcp
EXPOSE 53/udp

HEALTHCHECK --interval=30s --timeout=30s --start-period=10s --retries=3 CMD drill @127.0.0.1 cloudflare.com || exit 1

CMD ["unbound", "-d", "-c", "/etc/unbound/unbound.conf"]
