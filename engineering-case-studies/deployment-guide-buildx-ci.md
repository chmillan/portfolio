# Deployment Guide — Docker Buildx + CI/CD for Analytical Applications

**Last updated:** 11/02/2026  

This guide presents a reproducible deployment strategy for containerized analytical applications using Docker Buildx, multi-stage builds, and CI pipelines.

All infrastructure identifiers have been generalized for public reference.

---

## Goal

### On development pushes (non-default branches)

- Build and push one image with tags:
  - `:dev`
  - `:dev-<SHORT_SHA>`
- Reuse registry cache layers for faster builds.

### On release commits (default branch)

- Build and push image with tags:
  - `:prod`
  - `:<VERSION>`
  - `:<VERSION>-<SHORT_SHA>`
- Reuse the same registry cache.

---

## Why Docker Buildx?

- Enables `--secret` mounts (BuildKit)
- Supports registry-based caching (`--cache-to`, `--cache-from`)
- Works well with multi-stage builds
- Improves reproducibility in analytical stacks

---

# Dockerfile Strategy

## Multi-Stage Structure

- `builder` stage: installs system dependencies and restores packages
- `test` stage: optional unit test execution inside container
- `runtime` stage: minimal production image

This reduces final image size and avoids shipping build tools to production.

---

## A) Secure Secrets Handling

Use BuildKit secrets — never bake tokens into image layers.

```dockerfile
# Example: using a BuildKit secret to authenticate to a private source
# (the secret is available at /run/secrets/<id> only during this RUN step)
RUN --mount=type=secret,id=private_repo_token \
    set -eu; \
    TOKEN="$(cat /run/secrets/private_repo_token)"; \
    # Use the token only in this layer (do not print it)
    git config --global url."https://oauth2:${TOKEN}@gitlab.example.com/".insteadOf \
      "https://gitlab.example.com/"; \
    R -q -e 'renv::restore(prompt=FALSE)'; \
    # Clean up temporary git configuration
    git config --global --unset-all url."https://oauth2:${TOKEN}@gitlab.example.com/".insteadOf; \
    unset TOKEN

```
## B) Dependency isolation


```dockerfile
# Example of env configuration
ENV RENV_PATHS_LIBRARY=/renv/library \
    RENV_PATHS_CACHE=/renv/cache \
    RENV_CONFIG_PAK_ENABLED=FALSE \
    PAK_CACHE_DIR=/pak/cache \
    RENV_CONFIG_AUTOLOADER_ENABLED=FALSE \
    RENV_CONFIG_CACHE_SYMLINKS=FALSE \
    RENV_CONFIG_REPOS_OVERRIDE="https://packagemanager.posit.co/cran/latest"

```


## C) Cache Optimization

```dockerfile
# Use BuildKit cache mounts to speed up dependency installation
RUN --mount=type=cache,target=/renv/cache \
    --mount=type=cache,target=/pak/cache \
    set -eu; \
    install2.r --error renv pak; \
    R -q -e 'Sys.setenv(RENV_PATHS_LIBRARY="/renv/library", \
                        RENV_PATHS_CACHE="/renv/cache"); \
             renv::consent(provided=TRUE); \
             renv::restore(prompt=FALSE)'

```

## D) Runtime library detection

```dockerfile
# Ensure the runtime image detects the correct renv path
RUN mkdir -p /usr/local/lib/R/etc && \
    PLAT="$(Rscript -e 'cat(R.version$platform)')" && \
    RVER="$(Rscript -e 'cat(paste0("R-", R.Version()$major, ".", strsplit(R.Version()$minor, "\\.")[[1]][1]))')" && \
    RENV_ROOT="${RENV_PATHS_LIBRARY:-/renv/library}" && \
    RENV_LIB="$(find "$RENV_ROOT" -maxdepth 4 -type d -path "$RENV_ROOT/*/$RVER/$PLAT" | head -n 1)" && \
    printf '%s\n' "R_LIBS_SITE=$RENV_LIB" > /usr/local/lib/R/etc/Renviron.site

```


# Makefile Strategy (Buildx + Registry Cache)

This section provides a practical Makefile structure to standardize local and CI builds with Docker Buildx, registry cache, and consistent tagging.

---

## Defaults and Image Coordinates

```makefile
# Platform (adjust if needed)
PLATFORM ?= linux/amd64

# Tag defaults
DOCKER_BUILD_TAG ?= dev
CACHE_TAG ?= buildcache

# Buildx builder names
BUILDX_BUILDER_LOCAL ?= desktop-linux
BUILDX_BUILDER_CI ?= ci_builder

# Registry coordinates (public-safe placeholders)
REGISTRY_URL ?= registry.example.com
REGISTRY_PROJECT ?= analytics
IMAGE_NAME ?= monitoring-app

REGISTRY_IMAGE := $(REGISTRY_URL)/$(REGISTRY_PROJECT)/$(IMAGE_NAME)

# Versioning
VERSION := $(shell cat VERSION 2>/dev/null || echo "0.0.0")
GIT_COMMIT := $(shell git rev-parse --short HEAD)
```

##  Buildx Initialization

```makefile
.PHONY: buildx-init
buildx-init:
	@set -eu; \
	BUILDER="$(BUILDX_BUILDER_CI)"; \
	echo "Initializing buildx builder: $$BUILDER"; \
	docker buildx create --name "$$BUILDER" --use --driver docker-container >/dev/null 2>&1 || \
	  docker buildx use "$$BUILDER"; \
	docker buildx inspect --bootstrap >/dev/null

```

# Dev build 
Build and push images for non-default branches (or local dev if desired), reusing registry cache.

```makefile
.PHONY: build-ci-dev
build-ci-dev: buildx-init
	@set -eu; \
	CACHE_REF="$(REGISTRY_IMAGE):$(CACHE_TAG)"; \
	echo "Building dev image with cache: $$CACHE_REF"; \
	docker buildx build \
	  --builder "$(BUILDX_BUILDER_CI)" \
	  --platform "$(PLATFORM)" \
	  --cache-from type=registry,ref="$$CACHE_REF" \
	  --cache-to type=registry,ref="$$CACHE_REF",mode=max \
	  -t "$(REGISTRY_IMAGE):dev" \
	  -t "$(REGISTRY_IMAGE):dev-$(GIT_COMMIT)" \
	  --push \
	  .

```

# Production build


```makefile
.PHONY: build-ci-prod
build-ci-prod: buildx-init
	@set -eu; \
	CACHE_REF="$(REGISTRY_IMAGE):$(CACHE_TAG)"; \
	echo "Building prod image with cache: $$CACHE_REF"; \
	docker buildx build \
	  --builder "$(BUILDX_BUILDER_CI)" \
	  --platform "$(PLATFORM)" \
	  --cache-from type=registry,ref="$$CACHE_REF" \
	  --cache-to type=registry,ref="$$CACHE_REF",mode=max \
	  -t "$(REGISTRY_IMAGE):prod" \
	  -t "$(REGISTRY_IMAGE):$(VERSION)" \
	  -t "$(REGISTRY_IMAGE):$(VERSION)-$(GIT_COMMIT)" \
	  --push \
	  .



```

## Build CI test

### build a test image

```makefile
.PHONY: build-ci-test
build-ci-test: buildx-init
	@set -eu; \
	CACHE_REF="$(REGISTRY_IMAGE):$(CACHE_TAG)"; \
	IMAGE_TEST="$(REGISTRY_IMAGE):test-$(GIT_COMMIT)"; \
	echo "Building test image: $$IMAGE_TEST"; \
	docker buildx build \
	  --builder "$(BUILDX_BUILDER_CI)" \
	  --platform "$(PLATFORM)" \
	  --target test \
	  --cache-from type=registry,ref="$$CACHE_REF" \
	  --cache-to type=registry,ref="$$CACHE_REF",mode=max \
	  -t "$$IMAGE_TEST" \
	  --push \
	  .

```

### run tests from the test image

```makefile
.PHONY: test-ci
test-ci:
	@set -eu; \
	IMAGE_TEST="$(REGISTRY_IMAGE):test-$(GIT_COMMIT)"; \
	echo "Running tests from: $$IMAGE_TEST"; \
	docker run --rm "$$IMAGE_TEST"
```

## Minimal gitlab-ci.yml

```yaml
image: docker:24

services:
  - name: docker:24-dind

variables:
  DOCKER_DRIVER: overlay2
  DOCKER_TLS_CERTDIR: "/certs"
  DOCKER_HOST: "tcp://docker:2376"
  DOCKER_TLS_VERIFY: "1"
  DOCKER_CERT_PATH: "$DOCKER_TLS_CERTDIR/client"
  DOCKER_DEFAULT_PLATFORM: "linux/amd64"

stages:
  - test
  - development
  - production

before_script:
  - echo "$REGISTRY_PASSWORD" | docker login "$REGISTRY_URL" -u "$REGISTRY_USERNAME" --password-stdin
  - export GIT_COMMIT="$CI_COMMIT_SHORT_SHA"
  - export VERSION="$(cat VERSION 2>/dev/null || echo 0.0.0)"

test_image:
  stage: test
  rules:
    - if: '$CI_PIPELINE_SOURCE == "merge_request_event"'
    - if: '$CI_PIPELINE_SOURCE == "push"'
  script:
    - make build-ci-test
    - make test-ci

build_development:
  stage: development
  rules:
    - if: '$CI_COMMIT_BRANCH != $CI_DEFAULT_BRANCH'
  script:
    - make build-ci-dev

build_production:
  stage: production
  rules:
    - if: '$CI_COMMIT_BRANCH == $CI_DEFAULT_BRANCH'
  script:
    - make build-ci-prod
```