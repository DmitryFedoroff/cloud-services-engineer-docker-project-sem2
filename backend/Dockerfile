# syntax=docker/dockerfile:1.7

FROM --platform=$BUILDPLATFORM golang:1.22.7-alpine AS builder

WORKDIR /src

COPY go.mod go.sum ./
RUN --mount=type=cache,target=/go/pkg \
    go mod download

COPY . .
RUN --mount=type=cache,target=/go/pkg \
    CGO_ENABLED=0 GOOS=linux GOARCH=$TARGETARCH \
    go build -trimpath -ldflags="-s -w -buildid=" \
    -o /momo-store ./cmd/api

FROM alpine:3.21 AS runtime

LABEL org.opencontainers.image.authors="Dmitry Fedoroff <fedoroffx@gmail.com>" \
    org.opencontainers.image.source="https://github.com/DmitryFedoroff/cloud-services-engineer-docker-project-sem2" \
    org.opencontainers.image.description="Dumpling House No.2 backend - build stage" \
    org.opencontainers.image.version="1.0.0"

# hadolint ignore=DL3018
RUN apk --no-cache add ca-certificates wget \
    && addgroup -S appgroup \
    && adduser  -S -G appgroup -u 10001 appuser

WORKDIR /app
COPY --from=builder /momo-store ./momo-store

USER 10001:10001
EXPOSE 8081

HEALTHCHECK --interval=30s --timeout=5s --start-period=5s --retries=3 \
    CMD wget -qO- http://127.0.0.1:8081/health || exit 1

ENTRYPOINT ["/app/momo-store"]
