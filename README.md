# lula-hub

Microservice to sync Redis message streams via bearer session token.

Stack includes:

- Redis
- Fastify

## Design

The proposed solution enables async communications with a central hub via HTTPS by leveraging Redis streams.

Remote clients have a unique client ID.

### Client publish to hub

A client publishes messages by adding these into a local Redis stream

```javascript
redis-cli XADD publish:x MAXLEN 1000 * topic test payload '{ "type": "hello" }'
```

A process on the client syncs this stream up to the hub as follows.

TBC

### Authentication

We authenticate bearer tokens managed by https://github.com/evanx/lula-auth,
which provides `/register` and `/login` endpoints. If the bearer token is valid, then the Redis
hashes key `session:${token}:h` will exist.

```javascript
fastify.register(require('fastify-bearer-auth'), {
  auth: async (token, request) => {
    const client = await fastify.redis.hget(`session:${token}:h`, 'client')
    if (client) {
      request.client = client
      return true
    }
    return false
  },
  errorResponse: err => {
    return { code: 401, error: err.message }
  },
})
```

## Related

- https://github.com/evanx/lula-auth
- https://github.com/evanx/lula-remote

