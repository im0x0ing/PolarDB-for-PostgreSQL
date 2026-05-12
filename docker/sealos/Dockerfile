FROM openanolis/anolisos:8

# install runtime helpers used by the entrypoint and initdb scripts
RUN dnf -y --setopt=install_weak_deps=False install ca-certificates gzip nss_wrapper wget xz zstd \
    && dnf clean all \
    && rm -rf /var/cache/dnf

# grab gosu for easy step-down from root (gosu execs, preserving signals; runuser/su do not)
ENV GOSU_VERSION 1.19
RUN set -eux; \
    arch="$(uname -m)"; \
    case "$arch" in \
        x86_64) gosu_arch=amd64 ;; \
        aarch64) gosu_arch=arm64 ;; \
        *) echo >&2 "unsupported arch: $arch"; exit 1 ;; \
    esac; \
    wget -O /usr/local/bin/gosu "https://github.com/tianon/gosu/releases/download/$GOSU_VERSION/gosu-$gosu_arch"; \
    chmod +x /usr/local/bin/gosu; \
    gosu --version; \
    gosu nobody true

ENV LANG en_US.UTF-8
ENV PATH=/u01/polardb_pg/bin:$PATH
ENV PG_MAJOR 15
ENV PGDATA /var/polardb/primary
ENV POLARDB_DATA_ROOT=/var/polardb
ENV POLARDB_SHARED_DIR=/var/polardb/shared
ENV POLARDB_PORT=5432

RUN mkdir /docker-entrypoint-initdb.d

COPY PolarDB-*.rpm /tmp/
COPY polardb-entrypoint.sh polardb-ensure-initdb.sh /usr/local/bin/

RUN set -eux; \
    rpm -ivh /tmp/PolarDB-*.rpm; \
    (id postgres >/dev/null 2>&1 || useradd -m postgres); \
    POLARDB_INSTALL_DIR="$(find /u01 -maxdepth 3 -type f -path '*/bin/postgres' -print -quit | sed 's#/bin/postgres##')"; \
    test -n "$POLARDB_INSTALL_DIR"; \
    if [ "$POLARDB_INSTALL_DIR" != "/u01/polardb_pg" ]; then ln -sfn "$POLARDB_INSTALL_DIR" /u01/polardb_pg; fi; \
    mkdir -p "$PGDATA" "$POLARDB_SHARED_DIR" /var/run/postgresql; \
    chown -R postgres:postgres "$POLARDB_DATA_ROOT" "$POLARDB_INSTALL_DIR" /var/run/postgresql; \
    chown -h postgres:postgres /u01/polardb_pg; \
    chmod 1777 "$POLARDB_DATA_ROOT"; \
    chmod 03775 /var/run/postgresql; \
    chmod +x /usr/local/bin/polardb-entrypoint.sh /usr/local/bin/polardb-ensure-initdb.sh; \
    ln -sT polardb-entrypoint.sh /usr/local/bin/docker-entrypoint.sh; \
    ln -sT polardb-ensure-initdb.sh /usr/local/bin/docker-ensure-initdb.sh; \
    ln -sT polardb-ensure-initdb.sh /usr/local/bin/docker-enforce-initdb.sh; \
    ln -sT polardb-ensure-initdb.sh /usr/local/bin/polardb-enforce-initdb.sh; \
    rm -f /tmp/PolarDB-*.rpm; \
    rm -rf /var/cache/dnf

EXPOSE 5432

STOPSIGNAL SIGINT

ENTRYPOINT ["polardb-entrypoint.sh"]
CMD ["postgres"]
