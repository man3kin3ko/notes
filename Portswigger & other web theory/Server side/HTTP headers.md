## Custom headers reconnaissance

You can find any custom HTTP-headers that is used by the server using a preflight request. Then, the server must include all expected headers in `Access-Control-Request-Headers`.

Sometimes custom headers can appear in 400-code responses.

Pay attention to those headers that is mentioned in `Vary` response header. Usually it contains headers that will create a cache key.
## Hop-by-hop headers

You can use `Connection: close, HEADER` to instruct proxy to not cache and remove those header. As a result of transmitting `HEADER` through the proxy, you can perform cache poisoning and bypass some access controls.

Also try different values to double check there weren't any false negatives:
```
Forwarded: for=example.com;host=example.com;proto=https, for=23.45.67.89
```

### See also
- https://www.yeswehack.com/learn-bug-bounty/http-header-exploitation