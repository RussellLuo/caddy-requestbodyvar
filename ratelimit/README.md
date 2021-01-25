# ratelimit

A Caddy v2 extension to apply rate-limiting for HTTP requests.


## Installation

```bash
$ xcaddy build --with github.com/RussellLuo/caddy-ext/ratelimit
```

## Caddyfile Syntax

```
rate_limit <key> <rate> [<zone_size> [<reject_status>]]
```

Parameters:

- `<key>`: The variable used to differentiate one client from another. Currently supported variables ([Caddy shorthand placeholders][1]):
    + `{path.<var>}`
    + `{query.<var>}`
    + `{header.<VAR>}`
    + `{cookie.<var>}`
    + `{remote.host}`
    + `{remote.port}`
    + `{remote.ip}`
- `<rate>`: The request rate limit (per key value) specified in requests per second (r/s) or requests per minute (r/m).
- `<zone_size>`: The size (i.e. the number of key values) of the LRU zone that keeps states of these key values. Defaults to 10,000.
- `<reject_status>`: The HTTP status code of the response when a client exceeds the rate. Defaults to 429 (Too Many Requests).


## Example

With the following Caddyfile:

```
localhost:8080 {
    route /foo {
        rate_limit {query.id} 2r/m

        respond 200
    }
}
```

You can apply the rate of `2 requests per minute` to the path `/foo`, and Caddy will respond with status code 429 when a client exceeds:

```bash
$ curl -w "%{http_code}" 'https://localhost:8080/foo?id=1'
200
$ curl -w "%{http_code}" 'https://localhost:8080/foo?id=1'
200
$ curl -w "%{http_code}" 'https://localhost:8080/foo?id=1'
429
```

An extra request with another value for the request parameter `id` will not be limited:

```bash
$ curl -w "%{http_code}" 'https://localhost:8080/foo?id=2'
200
```


[1]: https://caddyserver.com/docs/caddyfile/concepts#placeholders