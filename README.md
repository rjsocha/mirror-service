# Docker registry mirror service

For gitlab runners with Docker in Docker pipelines.

Based on [wyga-site](https://github.com/rjsocha/wyga-site).

Check out [docker-shell](https://github.com/rjsocha/wyga-docker-shell) for Gitlab Runner configuration.

## How it's works

 1. wyga-site provides TLS certificates for e.wyga.site namespace.
 2. IP 100.100.100.100 is for host-only networking
 3. DinD service starts with registry-mirror option enabled

# Setup

## Install
```
mkdir -p /service && cd $_
git clone https://github.com/rjsocha/mirror-service.git mirror-service && cd $_
```
## Interface configuration (on Ubuntu)
```
install -m 600 netplan/docker-mirror.yaml /etc/netplan/docker-mirror.yaml
netplan generate && netplan apply
```
## Start service
```
docker compose pull --quiet
docker compose up -d
```
## Update (via cron)
```
  (crontab -l 2>/dev/null; \
  printf -- '30 0 * * 1'; \
  printf -- ' cd /service/mirror-service &&'; \
  printf -- ' docker compose pull --quiet &&'; \
  printf -- ' docker compose up -d &>/dev/null\n') | crontab -
```
## Test
```
curl -sf https://mirror.e.wyga.site/v2/_catalog
```
# Example Usage

With [Gitlab's Dependency Proxy](https://docs.gitlab.com/ee/user/packages/dependency_proxy/) enabled.

.gitlab-ci.yml:
```
default:
  image: ${GL_DPROXY}/wyga/docker-shell:latest
  services:
    - name: ${GL_DPROXY}/docker:dind
      alias: docker
      command: [ "--registry-mirror=https://mirror.e.wyga.site" ]
  tags:
    - docker-in-docker

variables:
  GL_DPROXY: "${CI_DEPENDENCY_PROXY_DIRECT_GROUP_IMAGE_PREFIX}"
  DOCKER_TLS_CERTDIR: "/certs"
  DOCKER_CREATE_CONTEXT: 1
```
