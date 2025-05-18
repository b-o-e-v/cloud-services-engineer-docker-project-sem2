# Этап 1: Сборка приложения
FROM node:16-alpine AS builder

WORKDIR /app

COPY package*.json ./
RUN npm ci

COPY . .

ENV NODE_ENV=production VUE_APP_API_URL=/momo-api
RUN npm run build

# Этап 2: Финальный образ
FROM nginx:1.28.0-alpine

RUN apk update && apk upgrade && apk add --no-cache curl
RUN addgroup -g 1001 -S app-group && adduser -u 1001 -S app-user -G app-group

COPY --chown=app-user:app-group nginx.conf /etc/nginx/conf.d/default.conf
COPY --from=builder /app/dist /usr/share/nginx/html

RUN \
  chown -R app-user:app-group /var/cache/nginx /var/log/nginx /etc/nginx/conf.d && \
  touch /var/run/nginx.pid && chown app-user:app-group /var/run/nginx.pid

USER app-user
EXPOSE 8080

# Добавляем healthcheck
HEALTHCHECK \
  --interval=30s \
  --timeout=5s \
  --start-period=10s \
  --retries=3 \
  CMD curl -f http://localhost:8080 || exit 1

# Запуск приложения
CMD ["nginx", "-g", "daemon off;"]
