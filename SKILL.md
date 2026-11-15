---
name: docker-optimization
description: Dockerfile build optimization — slim vs alpine (glibc vs musl), layer ordering for cache hits, .dockerignore build-context trimming, multi-stage builds, and digest pinning. Use whenever you write, edit, or review a Dockerfile, docker-compose file, or container build.
---

# Dockerfile Optimization Guide

Five critical misconceptions in building Docker images (source: DevOps Toolbox). Apply these to every project to optimize build speed, size, and stability.

## 1. Alpine vs. Slim Tags
* **Issue:** Defaulting to *Alpine* for small size — but Alpine uses `musl libc` instead of the standard `glibc` used by most Linux distributions.
* **Fix:** Prefer **Slim** variants. Slim is Debian/Ubuntu-based and uses `glibc`, ensuring compatibility with pre-built binary dependencies and preventing unnecessary recompilation from source.

## 2. Abusing the Cache
* **Issue:** Placing `COPY . .` too early invalidates the cache for all subsequent layers whenever a single file changes.
* **Fix:** Layer the build:
  1. Copy only dependency files (e.g., `package.json`).
  2. Run install commands (e.g., `npm ci`).
  3. Copy the rest of the source code.

## 3. Build Context Bloat
* **Issue:** Sending unnecessary files (`node_modules`, `.git`, local logs) into the Docker daemon context increases build times.
* **Fix:** Use a `.dockerignore` file to exclude files not required for the build, like `.gitignore`.

## 4. Multi-Stage Builds
* **Issue:** Including build tools, source code, and runtimes in the final image creates massive, insecure production images.
* **Fix:** Use **multi-stage builds**. Compile in a "builder" stage and copy only the final, minimal binary into a lean production image (e.g., `scratch` or `distroless`).

## 5. Digests vs. Tags
* **Issue:** Mutable tags (`:latest`, `:26`) can change behavior if the upstream author pushes an update.
* **Fix:** Pin every `FROM` and CI tool image by content **digest** (`image@sha256:...`), never `latest`. Get the digest: `docker buildx imagetools inspect node:22-slim --format '{{json .Manifest.Digest}}'`. See the `supply-chain-security` skill for the full pinning rules.
* **Exception:** Kubernetes deploy manifests are NOT pinned. Pinning breaks the auto-deploy jobs. Pin `FROM` lines and CI images only.
