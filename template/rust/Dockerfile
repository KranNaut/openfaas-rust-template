# Build Stage
FROM rust:1.68.1-alpine AS builder

RUN rustup target add x86_64-unknown-linux-musl

RUN USER=root
WORKDIR /usr/src/openfaas
COPY function ./function
COPY main ./main


RUN apk add build-base
RUN cd main && cargo build --release

RUN cd main && cargo install --target x86_64-unknown-linux-musl --path .

# Runner stage
FROM alpine:3.12 as runner 

RUN apk --no-cache add curl ca-certificates \
    && addgroup -S app && adduser -S -g app app

ADD https://github.com/openfaas/faas/releases/download/0.18.10/fwatchdog /usr/bin
RUN chmod +x /usr/bin/fwatchdog

COPY --from=builder /usr/local/cargo/bin/main /usr/bin/main

HEALTHCHECK --interval=5s CMD [ -e /tmp/.lock ] || exit 1

ENV fprocess="main"

RUN ls -la /

CMD ["fwatchdog"]

