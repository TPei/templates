# FROM golang:1.9.7-alpine3.8 as builder
FROM crystallang/crystal:0.26.1 as builder

RUN apt update \
    && apt install -y curl \
    && echo "Pulling watchdog binary from Github." \
    && curl -sSL https://github.com/openfaas/faas/releases/download/0.9.6/fwatchdog > /usr/bin/fwatchdog \
    && chmod +x /usr/bin/fwatchdog

WORKDIR /home/app
COPY . .

# Run a gofmt and exclude all vendored code.
# RUN test -z "$(gofmt -l $(find . -type f -name '*.go' -not -path "./vendor/*" -not -path "./function/vendor/*"))" || { echo "Run \"gofmt -s -w\" on your Golang code"; exit 1; }

#RUN CGO_ENABLED=0 GOOS=linux \
    #go build --ldflags "-s -w" -a -installsuffix cgo -o handler . && \
    #go test $(go list ./... | grep -v /vendor/) -cover

COPY function/shard.yml shard.yml
RUN shards install
RUN crystal build main.cr -o handler

FROM alpine:3.8
RUN apk --no-cache add ca-certificates

# Add non root user
RUN addgroup -S app && adduser -S -g app app
RUN mkdir -p /home/app

WORKDIR /home/app

COPY --from=builder /usr/bin/fwatchdog   .

#COPY --from=builder /go/src/handler/function/  .
#COPY --from=builder /go/src/handler/handler    .

COPY --from=builder /home/app/function/  .
COPY --from=builder /home/app/handler    .

RUN chown -R app /home/app

USER app

ENV fprocess="./handler"
EXPOSE 8080

HEALTHCHECK --interval=2s CMD [ -e /tmp/.lock ] || exit 1

CMD ["./fwatchdog"]
