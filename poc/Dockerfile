FROM alpine:3.24.3

ARG OPENSSH_VERSION=10.3p1

# 安装编译依赖
RUN apk add --no-cache \
    build-base \
    openssl-dev \
    zlib-dev \
    linux-headers \
    wget \
    tar

# 下载并编译 OpenSSH 上游源码
RUN set -eux; \
    wget -O /tmp/openssh.tar.gz \
      "https://cdn.openbsd.org/pub/OpenBSD/OpenSSH/portable/openssh-${OPENSSH_VERSION}.tar.gz"; \
    mkdir -p /tmp/openssh-src; \
    tar -xzf /tmp/openssh.tar.gz -C /tmp/openssh-src --strip-components=1; \
    cd /tmp/openssh-src; \
    ./configure \
      --prefix=/usr \
      --sysconfdir=/etc/ssh \
      --with-privsep-path=/var/empty \
      --with-privsep-user=sshd \
      --without-pam; \
    make -j"$(getconf _NPROCESSORS_ONLN)"; \
    make install; \
    rm -rf /tmp/openssh-src /tmp/openssh.tar.gz

# 创建 SSH 运行环境和实验账户
RUN set -eux; \
    getent group sshd >/dev/null || addgroup -S sshd; \
    id sshd >/dev/null 2>&1 || adduser -S -D -H -G sshd -s /sbin/nologin sshd; \
    mkdir -p /var/empty /run/sshd; \
    chmod 755 /var/empty; \
    ssh-keygen -A; \
    adduser -D -s /bin/sh alice; \
    adduser -D -s /bin/sh bob; \
    echo 'bob:BobPass-123' | chpasswd; \
    sed -i 's/^alice:[^:]*:/alice:*:/' /etc/shadow

# SSH 服务配置
RUN set -eux; \
    cat > /etc/ssh/sshd_config <<'EOF'
Port 22
HostKey /etc/ssh/ssh_host_ed25519_key
UsePAM no
PasswordAuthentication yes
KbdInteractiveAuthentication no
PermitRootLogin no
PubkeyAuthentication yes
AllowUsers alice bob
PidFile /run/sshd.pid
EOF

# 检查编译结果和配置
RUN set -eux; \
    /usr/sbin/sshd -t; \
    /usr/sbin/sshd -T | grep -Ei 'usepam|passwordauthentication'

EXPOSE 22

CMD ["/usr/sbin/sshd", "-D", "-e"]
