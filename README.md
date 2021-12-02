<iustin http-equiv="Refresh" content="0; url=https://youst.in/posts/cache-key-normalization-denial-of-service/" />

```ceylon
GET / HTTP/2                                  HTTP/2 301 Moved Permanently
Host: www.shopify.com                         Location: https://www.shopify.com/
x-forwarded-scheme: http                      Cf-Cache-Status: HIT
                                              Age: 2
```


```ceylon
HTTP/2 301 Moved Permanently
Location: https://iustin.com/index.html
X-Cache: Hit From Cloudfront
```
