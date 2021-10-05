# build the server binary
FROM golang:1.12 AS builder
LABEL stage=server-intermediate
WORKDIR /go/src/github.com/infobloxopen/dapr
COPY . .
RUN CGO_ENABLED=0 GOOS=linux go build -o bin/server ./cmd/server

# copy the server binary from builder stage; run the server binary
FROM alpine:latest AS runner
RUN apk --no-cache add ca-certificates
WORKDIR /bin
COPY --from=builder /go/src/github.com/infobloxopen/dapr/bin/server .

ENTRYPOINT ["server"]
