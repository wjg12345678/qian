# redis-rate-limiter

Lightweight distributed rate limiting component built with `hiredis`, `C++17`, and `pybind11`.

## Features

- RAII-managed Redis connection pool with health checks
- Lua-backed sliding window limiter
- Lua-backed token bucket limiter for smoother throttling
- Python bindings for direct use from application code

## Build

```bash
cmake -S . -B build \
  -Dpybind11_DIR="$(python3 -c 'import pybind11; print(pybind11.get_cmake_dir())')"
cmake --build build
```

If `hiredis` is not installed in a standard location, pass:

```bash
-DHIREDIS_INCLUDE_DIR=/path/to/include \
-DHIREDIS_LIBRARY=/path/to/libhiredis.so
```

## Python usage

```python
import redis_limiter

cfg = redis_limiter.RedisConfig()
cfg.host = "127.0.0.1"
cfg.pool_size = 8

pool = redis_limiter.RedisPool(cfg)
limiter = redis_limiter.TokenBucketLimiter(pool, max_tokens=100, refill_rate=50.0)

result = limiter.allow("login:user:123")
print(result.allowed, result.remaining, result.retry_after_ms)
```

See [`examples/python_demo.py`](/Users/mac/Desktop/redis-rate-limiter/examples/python_demo.py) for a complete example.
