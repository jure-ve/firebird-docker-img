#
# This file was auto-generated. Do not edit. See /src.
#

FROM ubuntu:noble

ARG ARCH_ARM64

# ARM64 arch
ENV FIREBIRD_RELEASE_URL=${ARCH_ARM64:+https://github.com/FirebirdSQL/firebird/releases/download/v5.0.0/Firebird-5.0.0.1306-0-linux-arm64.tar.gz}
ENV FIREBIRD_RELEASE_SHA256=${ARCH_ARM64:+bdb62abc91a4c26a86b9377256529622e3046f4ec4f66fbdcf46747688d45033}

# AMD64 arch
ENV FIREBIRD_RELEASE_URL=${FIREBIRD_RELEASE_URL:-https://github.com/FirebirdSQL/firebird/releases/download/v5.0.0/Firebird-5.0.0.1306-0-linux-x64.tar.gz}
ENV FIREBIRD_RELEASE_SHA256=${FIREBIRD_RELEASE_SHA256:-137b356ea464e224b6ed1241e6dba95b6b950ff1feef91696d4071d4da880768}

ENV FIREBIRD_VERSION=5.0.0
ENV FIREBIRD_MAJOR=5

# https://linuxcommand.org/lc3_man_pages/seth.html
#   -e  Exit immediately if a command exits with a non-zero status.
#   -u  Treat unset variables as an error when substituting
#   -x  Print commands and their arguments as they are executed.

# Prerequisites
RUN set -eux; \
    apt-get update; \
    apt-get install -y --no-install-recommends \
        libatomic1 \
        libicu74 \
        libncurses6 \
        libtomcrypt1 \
        libtommath1 \
        netbase \
        procps \
        tzdata; \
    rm -rf /var/lib/apt/lists/*

# Fix libtommath for FB 3.0 -- https://github.com/FirebirdSQL/firebird/issues/5716#issuecomment-826239174
RUN [ $FIREBIRD_MAJOR -eq 3 ] && ln -sf /usr/lib/x86_64-linux-gnu/libtommath.so.1 /usr/lib/x86_64-linux-gnu/libtommath.so.0 || true

# Download
ADD --checksum="sha256:$FIREBIRD_RELEASE_SHA256" \
    --chown=root:root \
    --chmod=777 \
    $FIREBIRD_RELEASE_URL \
    /tmp/firebird-bundle.tar.gz

# Extract, install, clean
RUN set -eux; \
    cd /tmp; \
    tar --extract --file=firebird-bundle.tar.gz --gunzip --verbose --strip-components=1; \
    ./install.sh -silent; \
    rm *.tar.gz *.sh *.txt \
    # Remove unnecessary files
    rm -rf /opt/firebird/doc \
           /opt/firebird/examples \
           /opt/firebird/help \
           /opt/firebird/include; \
    # Remove 'employee' sample database from 'databases.conf'
    sed -i '/^employee/d' /opt/firebird/databases.conf

# System path
ENV PATH=/opt/firebird/bin:$PATH

# Data directory
ENV FIREBIRD_DATA=/var/lib/firebird/data
RUN set -eux; \
    mkdir -p "$FIREBIRD_DATA"; \
    chown -R firebird:firebird "$FIREBIRD_DATA"; \
    chmod 644 "$FIREBIRD_DATA"
VOLUME $FIREBIRD_DATA

# Entrypoint
COPY entrypoint.sh /usr/local/bin/
RUN set -eux; \
    chmod +x /usr/local/bin/entrypoint.sh
ENTRYPOINT ["entrypoint.sh"]

EXPOSE 3050/tcp

# Fix terminfo location
ENV TERMINFO=/lib/terminfo/

CMD ["firebird"]

