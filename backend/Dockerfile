FROM golang:1.22-alpine AS build

WORKDIR /app

COPY go.mod go.sum ./
RUN go mod download

COPY . .

RUN CGO_ENABLED=0 GOOS=linux go build -o app ./cmd/api

FROM alpine:3.21

RUN apk add --no-cache ca-certificates wget \
    && adduser -D -H appuser

WORKDIR /app

COPY --from=build /app/app .

USER appuser

EXPOSE 8081

HEALTHCHECK --interval=30s --timeout=3s --start-period=5s --retries=3 \
    CMD wget -q -O /dev/null http://localhost:8081/health || exit 1

CMD ["./app"]
