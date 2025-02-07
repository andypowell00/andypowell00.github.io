---
layout: post
title:  "Using Redis for a cache with a Next.JS app"
date:   2025-01-30 16:21:59 -0400
categories: nextjs redis cache vercel
tags: nextjs redis cache vercel
---

# Migrating from unstable_cache to Redis in Next.js

## Why Redis?

When deploying Next.js applications on Vercel, the built-in `unstable_cache` can be unreliable. Redis provides a more robust caching solution with several advantages:

- Persistent storage across server restarts and deployments
- Configurable Time-To-Live (TTL) for cache entries
- Reliable performance at scale
- Consistent behavior across all deployment environments

## Implementation Guide

### 1. Install Redis Client

```bash
npm install redis
```

### 2. Create Redis Connection Helper

Create a new file `lib/redis.ts`:

```typescript
import { createClient } from 'redis';

const redis = createClient({
  socket: {
    host: process.env.REDIS_HOST,
    port: Number(process.env.REDIS_PORT),
  },
  password: process.env.REDIS_PW,
});

redis.on('error', (err) => console.warn('Redis Error:', err));

// Initialize connection
(async () => {
  await redis.connect();
})();

export default redis;
```

### 3. Environment Configuration

Add these variables to your `.env` file:

```
REDIS_HOST=your-redis-host
REDIS_PORT=your-redis-port
REDIS_PW=your-redis-password
```

### 4. Migrating Cache Implementation

Before (with unstable_cache):
```typescript
import { unstable_cache } from 'next/cache';

export async function getData() {
  const cachedData = await unstable_cache(
    async () => {
      // fetch data
    },
    ['cache-key'],
    { revalidate: 3600 }
  )();
  
  return cachedData;
}
```

After (with Redis):
```typescript
import redis from '@/lib/redis';

export async function getData() {
  const cacheKey = 'your-cache-key';
  
  // Try to get from cache
  const cachedData = await redis.get(cacheKey);
  if (cachedData) {
    return JSON.parse(cachedData);
  }
  
  // If not in cache, fetch and store
  const data = await fetchData();
  await redis.set(cacheKey, JSON.stringify(data), {
    EX: 3600 // Expire in 1 hour
  });
  
  return data;
}
```

## Best Practices

1. **Error Handling**: Always implement fallbacks for Redis connection failures
2. **Data Serialization**: Remember to JSON.stringify() before storing and JSON.parse() when retrieving
3. **TTL Management**: Set appropriate expiration times based on data volatility
4. **Cache Keys**: Use consistent naming conventions for cache keys to avoid collisions

## Production Considerations

- Use a managed Redis service (like Upstash or Redis Labs) for production
- Implement connection pooling for high-traffic applications
- Monitor Redis memory usage and implement eviction policies
- Set up proper error tracking and alerting

Remember to update your application's documentation to reflect the new caching implementation and any new environment variables required.
