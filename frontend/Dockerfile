# syntax=docker/dockerfile:1.7

FROM node:22-alpine AS builder

LABEL org.opencontainers.image.authors="Dmitry Fedoroff <fedoroffx@gmail.com>" \
    org.opencontainers.image.source="https://github.com/DmitryFedoroff/cloud-services-engineer-docker-project-sem2" \
    org.opencontainers.image.description="Dumpling House No.2 frontend - build stage" \
    org.opencontainers.image.version="1.0.0"

WORKDIR /app

COPY package.json package-lock.json ./

ARG NPM_TOKEN
RUN if [ -n "$NPM_TOKEN" ]; then \
    echo "//registry.npmjs.org/:_authToken=${NPM_TOKEN}" > ~/.npmrc ; \
    fi \
    && npm ci --no-audit --no-fund \
    && rm -f ~/.npmrc \
    && npm cache clean --force

COPY . .

ENV NODE_ENV=production \
    VUE_APP_API_URL=/momo-store/api \
    NODE_OPTIONS=--openssl-legacy-provider

RUN npm run build

FROM nginx:1.27-alpine AS runtime

LABEL org.opencontainers.image.authors="Dmitry Fedoroff <fedoroffx@gmail.com>" \
    org.opencontainers.image.source="https://github.com/DmitryFedoroff/cloud-services-engineer-docker-project-sem2" \
    org.opencontainers.image.description="Dumpling House No.2 frontend - build stage" \
    org.opencontainers.image.version="1.0.0"

RUN apk --no-cache upgrade libxml2 \
    && mkdir -p /usr/share/nginx/html/momo-store \
    /var/cache/nginx /var/run /var/log/nginx \
    && chown -R nginx:nginx /usr/share/nginx /var/cache/nginx /var/run /var/log/nginx \
    && chmod -R 555 /usr/share/nginx/html

COPY --from=builder /app/dist /usr/share/nginx/html/momo-store
COPY nginx.conf /etc/nginx/conf.d/default.conf

RUN sed -i '/^user[[:space:]]\+nginx/d' /etc/nginx/nginx.conf

USER nginx
EXPOSE 80

HEALTHCHECK --interval=30s --timeout=5s --start-period=5s --retries=3 \
    CMD wget -qO- http://127.0.0.1/momo-store/ || exit 1

ENTRYPOINT []
CMD ["nginx", "-g", "daemon off;"]
