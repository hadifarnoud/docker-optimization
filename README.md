# docker-optimization

A Claude Code skill: Dockerfile build optimization.

Covers slim vs alpine (glibc vs musl), layer ordering for cache hits, `.dockerignore` build-context trimming, multi-stage builds, and digest pinning.

## Install

```bash
npx skills add hadifarnoud/docker-optimization -g -a claude-code -y
```

Update later with `npx skills update`.

Pairs with [supply-chain-security](https://github.com/hadifarnoud/supply-chain-security) for the full image-pinning rules.
