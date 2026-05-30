This mechanism of storing often accessed data is typically provided by CDNs and often incorporated into load balancers and reverse proxies. A web browser also has it's own cache, which is configured by the headers that were sent by a server with the content, like `Cache-Control`. `Cache-Control` set to `public` allows CDNs to cache data, while `private` restricts scope only to the browser.

![](https://www.youtube.com/watch?v=Cy2ZJOBgk84)

Content is recieved from the cache by a *cache key*. In the simplest case, it can be just an `ETag` header with SHA-1 of the content, but mostly it is a combination of `Host` header, requested URI and some other arbitary parameters. A *hit* occurs when data for a cache key is stored in the cache and it's not expired, and the cache serves it to the client, otherwise it's a *miss*. 

While a cached resource is recieved via a cache key, caches decide whether to store server response or not by *cache rules*. Cache rules determine what can be cached and for how long. Cache rules are often set up to store static resources, which generally don't change frequently and are reused across multiple pages. They often ignore HTTP methods other than `GET`, `HEAD` and `OPTIONS`, because they alter the state of the server.

```nginx
# Example Nginx cache rule based on file extension
location ~* \.(css|js|png)$ {
    proxy_cache my_cache;
}
```

If a web application uses an input, such as path, parameters, or headers, which change its behaviour, but they aren't included in the cache, or parsed inconsistently by cache and server side, it may introduce a vulnerability. A web browser's cache is not a practical target for these attacks, since it is isolated to a single user. However, its existence is still worth keeping in mind when designing an attack vector.

## Web cache poisoning

When an attacker tricks cache into serving a dangerous payload for many clients without saving it on server side, it's called *web cache poisoning*. It can be a powerful delivery technique, because we can cache a response that cannot be sent normally, for example, with an XSS injection in a custom header or request smuggling.

## Web cache deception

Sometimes confidential information can be obtained via a *web cache deception*. If an attacker sends victim a link, for example, `/account.php/nonexistent.png`, he would obtain personal data that is located at `/account.php` by tricking the caching system to belive that the page is a static content. To do so, an attacker needs first to find a discrepancy in how the cache and origin server parse the URL path.