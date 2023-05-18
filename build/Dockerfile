FROM golang:1.19-alpine AS builder
WORKDIR ${GOPATH}/src/github.com/percona

RUN apk update && apk --no-cache add git \
    && git clone --depth 1 -b v1.14.0-dev https://github.com/gaozhenhai/percona-server-mongodb-operator.git \
    && cd percona-server-mongodb-operator \
    && go mod download \
    && GIT_COMMIT=$(git rev-parse HEAD) \
    && GIT_BRANCH=${VERSION:-$(git rev-parse --abbrev-ref HEAD | sed -e 's^/^-^g; s^[.]^-^g;' | sed -e 's/_/-/g' | tr '[:upper:]' '[:lower:]')} \
    && GO_LDFLAGS="-w -s -trimpath -race" \
    && mkdir -p build/_output/bin \
    && cp -ar build/*.sh /tmp \
    && CGO_ENABLED=0 GO_LDFLAGS=$GO_LDFLAGS \
       go build -ldflags "-w -s -X main.GitCommit=$GIT_COMMIT -X main.GitBranch=$GIT_BRANCH" \
           -o build/_output/bin/mongodb-operator cmd/manager/main.go \
    && cp -r build/_output/bin/mongodb-operator /usr/local/bin/mongodb-operator \
    && CGO_ENABLED=0 GO_LDFLAGS=$GO_LDFLAGS \
       go build -ldflags "-w -s -X main.GitCommit=$GIT_COMMIT -X main.GitBranch=$GIT_BRANCH" \
           -o build/_output/bin/mongodb-healthcheck cmd/mongodb-healthcheck/main.go \
    && cp -r build/_output/bin/mongodb-healthcheck /usr/local/bin/mongodb-healthcheck

FROM alpine:3.17
MAINTAINER Zhenhai Gao <gaozh1988@live.com>

RUN apk --no-cache add bash

COPY --from=builder /usr/local/bin/mongodb-operator /usr/local/bin/mongodb-operator
COPY --from=builder /usr/local/bin/mongodb-healthcheck /mongodb-healthcheck
COPY --from=builder /tmp/*.sh /

ENTRYPOINT [ "/usr/local/bin/mongodb-operator" ]
