VERSION 0.8
FROM golang:1.22-bookworm
WORKDIR /workspace

all:
  COPY +package/*.deb /workspace/dist/
  RUN cd dist && find . -type f | sort | xargs sha256sum >> ../sha256sums.txt
  SAVE ARTIFACT ./dist/*.deb AS LOCAL dist/
  SAVE ARTIFACT ./sha256sums.txt AS LOCAL dist/sha256sums.txt

tidy:
  LOCALLY
  ENV GOTOOLCHAIN=go1.22.1
  RUN go mod tidy
  RUN go fmt ./...

lint:
  FROM golangci/golangci-lint:v1.59.1
  WORKDIR /workspace
  COPY . .
  RUN golangci-lint run --timeout 5m ./...

test:
  COPY go.mod go.sum .
  RUN go mod download
  COPY . .
  RUN go test -coverprofile=coverage.out -v ./...
  SAVE ARTIFACT coverage.out AS LOCAL coverage.out

package:
  FROM debian:bookworm
  # Use bookworm-backports for newer golang versions
  RUN echo "deb http://deb.debian.org/debian bookworm-backports main" > /etc/apt/sources.list.d/backports.list
  RUN apt update
  # Tooling
  RUN apt install -y git devscripts dpkg-dev debhelper-compat dh-sequence-golang \
    golang-any=2:1.22~3~bpo12+1 golang-go=2:1.22~3~bpo12+1 golang-src=2:1.22~3~bpo12+1
  # Build Dependencies
  RUN apt install -y \
    gogoprotobuf \
    golang-github-containerd-continuity-dev \
    golang-github-docker-docker-dev \
    golang-github-moby-patternmatcher-dev \
    golang-github-opencontainers-go-digest-dev \
    golang-github-pkg-errors-dev \
    golang-github-stretchr-testify-dev \
    golang-github-gogo-protobuf-dev \
    golang-golang-x-sync-dev \
    golang-golang-x-sys-dev
  RUN mkdir -p /workspace/golang-github-tonistiigi-fsutil
  WORKDIR /workspace/golang-github-tonistiigi-fsutil
  COPY . .
  ENV EMAIL=damian@pecke.tt
  RUN export DEBEMAIL="damian@pecke.tt" \
    && export DEBFULLNAME="Damian Peckett" \
    && export VERSION="0.0~git$(git log -1 --date=format:%Y%m%d --pretty=format:%cd).$(git rev-parse --short HEAD)" \
    && dch --create --package golang-github-tonistiigi-fsutil --newversion "${VERSION}-1" \
      --distribution "UNRELEASED" --force-distribution  --controlmaint "Last Commit: $(git log -1 --pretty=format:'(%ai) %H %cn <%ce>')" \
    && tar -czf ../golang-github-tonistiigi-fsutil_${VERSION}.orig.tar.gz .
  RUN dpkg-buildpackage -us -uc
  SAVE ARTIFACT /workspace/*.deb AS LOCAL dist/