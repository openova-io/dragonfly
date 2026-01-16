# ADR: Caching with Dragonfly

**Status:** Accepted
**Date:** 2024-05-01

## Context

Need in-memory cache for sessions, rate limiting, and ephemeral data.

## Decision

Use Dragonfly as Redis-compatible cache.

## Rationale

| Option | Memory Efficiency | Compatibility | Ops |
|--------|-------------------|---------------|-----|
| Redis OSS | Baseline | 100% | Simple |
| **Dragonfly** | 30-40% better | 100% | Simple | **Selected** |
| Redis Cluster | Baseline | 100% | Complex |
| KeyDB | Better | 99% | Moderate |

**Key Decision Factors:**
- 100% Redis compatibility
- 30-40% more memory efficient
- Multi-threaded performance
- Simple deployment (no cluster mode)

## Configuration

```yaml
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: dragonfly
  namespace: databases
spec:
  replicas: 3
  template:
    spec:
      containers:
        - name: dragonfly
          image: docker.dragonflydb.io/dragonflydb/dragonfly:latest
          args:
            - --requirepass=$(REDIS_PASSWORD)
            - --maxmemory=1gb
            - --cache_mode=true
```

## Use Cases

| Use Case | TTL | Eviction |
|----------|-----|----------|
| Session cache | 24h | LRU |
| Rate limiting | 1m | Fixed |
| API cache | 5m | LRU |
| Feature flags | 1m | LRU |

## Consequences

**Positive:** Memory efficiency, Redis compatibility, simple ops
**Negative:** Newer project, smaller community

## Related

- [SPEC-DRAGONFLY-CONFIGURATION](./SPEC-DRAGONFLY-CONFIGURATION.md)
