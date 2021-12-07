# https://github.com/hashicorp/vault/pull/12358
FROM golang:1.17
WORKDIR /digitalocean_exporter

deps:
  COPY go.mod go.sum ./
	RUN go mod download
    # Output these back in case go mod download changes them.
	SAVE ARTIFACT go.mod AS LOCAL go.mod
	SAVE ARTIFACT go.sum AS LOCAL go.sum

version-info:
    LOCALLY
    RUN git rev-parse --short HEAD > REVISION
    RUN git rev-parse --abbrev-ref HEAD > BRANCH

    SAVE ARTIFACT REVISION
    SAVE ARTIFACT BRANCH

build:
    FROM +deps
    COPY +version-info/REVISION .
    COPY +version-info/BRANCH .
    COPY VERSION .
    COPY collector ./collector/
    COPY *.go .
    RUN CGO_ENABLED=0 go build -ldflags="-extldflags '-static' -s -w -X main.Version=v$(cat VERSION) -X main.Revision=$(cat REVISION) -X main.Branch=$(cat BRANCH) -X main.BuildUser=$(id -u -n) -X main.BuildDate=$(date --iso-8601='seconds')" -o bin/digitalocean_exporter
    SAVE ARTIFACT bin/digitalocean_exporter /digitalocean_exporter AS LOCAL bin/digitalocean_exporter

test:
    FROM +deps
    COPY ./mock ./mock
    COPY main.go .
    COPY collector .
    RUN go test

docker:
  FROM alpine:latest
  RUN apk add --update ca-certificates
  COPY +build/digitalocean_exporter /usr/bin/digitalocean_exporter
  COPY github.com/bloominlabs/bloominlabs-otel-collector+build/bloominlabs-otel-collector /usr/bin/bloominlabs-otel-collector

  ENTRYPOINT ["/usr/bin/bloominlabs-otel-collector"]
  SAVE IMAGE digitalocean_exporter:latest
  SAVE IMAGE --push ghcr.io/bloominlabs/digitalocean_exporter:latest
