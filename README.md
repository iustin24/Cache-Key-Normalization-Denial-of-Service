```ceylon
GET /images/logo.png?size=32x32&siz%65=0 HTTP/1.1    HTTP/1.1 400 Bad Request
Host: img.redacted.com                               X-Cache: MISS
                                                     x-cache-key: /images/logo.png?size=32x32

GET /images/logo.png?size=32x32 HTTP/1.1             HTTP/1.1 400 Bad Request
Host: img.redacted.com                               X-Cache: HIT 
                                                     x-cache-key: /images/logo.png?size=32x32
```
