<iustin http-equiv="Refresh" content="0; url=https://youst.in/posts/cache-key-normalization-denial-of-service/" />

```ceylon
GET /index.html HTTP/1.1                      HTTP/2 301 Moved Permanently
Host: redacted.com                            Location: https://iustin.com/index.html
x-forwarded-scheme: https                     X-Cache: Hit From Cloudfront
x-forwarded-host: iustin.com
```


```ceylon
HTTP/2 301 Moved Permanently
Location: https://iustin.com/index.html
X-Cache: Hit From Cloudfront
```
