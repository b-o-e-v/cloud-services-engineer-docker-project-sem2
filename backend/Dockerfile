# Этап 1: Сборка приложения
FROM golang:1.24.3-alpine AS builder

RUN apk add --no-cache git

WORKDIR /app

COPY go.mod go.sum ./
RUN go mod download

COPY . .

RUN CGO_ENABLED=0 GOOS=linux go build -ldflags="-w -s" -o server ./cmd/api

# Этап 2: Финальный образ
FROM alpine:3.21

RUN apk add --no-cache ca-certificates tzdata && update-ca-certificates
RUN addgroup -g 1001 -S server-group && adduser -u 1001 -S server-user -G server-group

WORKDIR /app

COPY --from=builder /app/server .

USER server-user
EXPOSE 8081

# Добавляем healthcheck
HEALTHCHECK \
  --interval=30s \
  --timeout=5s \
  --start-period=10s \
  --retries=3 \
  CMD wget -q --spider http://localhost:8081/health || exit 1

# Запуск приложения
CMD ["./server"]
