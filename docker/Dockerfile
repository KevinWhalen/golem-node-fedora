ARG FEDORA_BASE=fedora
ARG FEDORA_VERSION=36
ARG YA_CORE_VERSION=0.12.3
ARG YA_WASI_VERSION=0.2.2
ARG YA_VM_VERSION=0.3.0
ARG YA_DIR_BIN=/usr/bin/
ARG YA_DIR_BIN_TMP=/golem-provider-bin/
ARG YA_DIR_PLUGINS=/lib/yagna/plugins/

FROM ${FEDORA_BASE}:${FEDORA_VERSION} as installer
ARG YA_CORE_VERSION
ARG YA_WASI_VERSION
ARG YA_VM_VERSION
ARG YA_DIR_BIN_TMP
ARG YA_DIR_PLUGINS
RUN \
dnf install -y \
    wget \
    ca-certificates \
    findutils
RUN \
mkdir -p ${YA_DIR_BIN_TMP} \
&& mkdir -p ${YA_DIR_PLUGINS} \
&& wget -q "https://github.com/golemfactory/yagna/releases/download/v${YA_CORE_VERSION}/golem-provider-linux-v${YA_CORE_VERSION}.tar.gz" \
&& wget -q "https://github.com/golemfactory/ya-runtime-wasi/releases/download/v${YA_WASI_VERSION}/ya-runtime-wasi-linux-v${YA_WASI_VERSION}.tar.gz" \
&& wget -q "https://github.com/golemfactory/ya-runtime-vm/releases/download/v${YA_VM_VERSION}/ya-runtime-vm-linux-v${YA_VM_VERSION}.tar.gz" \
&& tar -zxvf golem-provider-linux-v${YA_CORE_VERSION}.tar.gz \
&& tar -zxvf ya-runtime-wasi-linux-v${YA_WASI_VERSION}.tar.gz \
&& tar -zxvf ya-runtime-vm-linux-v${YA_VM_VERSION}.tar.gz \
&& find golem-provider-linux-v${YA_CORE_VERSION} -executable -type f -exec cp -t ${YA_DIR_BIN_TMP} {} + \
&& cp -R golem-provider-linux-v${YA_CORE_VERSION}/plugins/* ${YA_DIR_PLUGINS} \
&& cp -R ya-runtime-wasi-linux-v${YA_WASI_VERSION}/* ${YA_DIR_PLUGINS} \
&& cp -R ya-runtime-vm-linux-v${YA_VM_VERSION}/* ${YA_DIR_PLUGINS}

FROM ${FEDORA_BASE}:${FEDORA_VERSION}
ARG YA_DIR_BIN
ARG YA_DIR_BIN_TMP
ARG YA_DIR_PLUGINS
RUN dnf install -y ca-certificates \
&& dnf clean all \
&& rm -Rf /var/cache/dnf
ENV \
SSL_CERT_DIR=/etc/pki/tls/certs/ \
SSL_CERT_FILE=/etc/pki/tls/certs/ca-bundle.crt
COPY --from=installer ${YA_DIR_PLUGINS} ${YA_DIR_PLUGINS}
COPY --from=installer ${YA_DIR_BIN_TMP} ${YA_DIR_BIN}

CMD ["golemsp", "run"]
VOLUME ["/root/.local/share/ya-provider/", "/root/.local/share/yagna/"]

# FIXME: is not being restarted after failed healthchecks.
#HEALTHCHECK \
#    --interval=1m \
#    --timeout=10s \
#    CMD golemsp status || exit 1
