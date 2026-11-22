# Stage 1: Build patched Caddy binary on Wolfi
FROM cgr.dev/chainguard/wolfi-base:latest AS builder

ENV CADDY_VERSION=v2.11.9

RUN apk add --no-cache go git ca-certificates

WORKDIR /src
RUN git clone --depth 1 --branch "${CADDY_VERSION}" https://github.com/caddyserver/caddy.git . && \
    go get -u \
        github.com/go-jose/go-jose/v3 \
        github.com/klauspost/compress \
        golang.org/x/net \
        go.opentelemetry.io/otel \
        go.opentelemetry.io/otel/sdk \
        go.opentelemetry.io/otel/trace && \
    go mod tidy && \
    CGO_ENABLED=0 go build -trimpath -ldflags="-s -w" -o /usr/bin/caddy ./cmd/caddy

# Stage 2: Minimal Wolfi runtime image
FROM cgr.dev/chainguard/wolfi-base:latest

RUN set -eux; \
    apk add --no-cache \
        ca-certificates \
        curl \
        libcap-utils \
        mailcap; \
    mkdir -p /config/caddy /data/caddy /etc/caddy /usr/share/caddy; \
    chmod 1777 /config/caddy /data/caddy; \
    curl -fsSL -o /etc/caddy/Caddyfile "https://github.com/caddyserver/dist/raw/33ae08ff08d168572df2956ed14fbc4949880d94/config/Caddyfile"; \
    curl -fsSL -o /usr/share/caddy/index.html "https://github.com/caddyserver/dist/raw/33ae08ff08d168572df2956ed14fbc4949880d94/welcome/index.html"; \
    apk del --no-network curl

COPY --from=builder /usr/bin/caddy /usr/bin/caddy

RUN setcap cap_net_bind_service=+p /usr/bin/caddy && \
    chmod +x /usr/bin/caddy && \
    apk del --no-network libcap-utils && \
    caddy version

ENV XDG_CONFIG_HOME=/config \
    XDG_DATA_HOME=/data

EXPOSE 80 443 443/udp 2019
WORKDIR /srv
USER nonroot

CMD ["caddy", "run", "--config", "/etc/caddy/Caddyfile", "--adapter", "caddyfile"]