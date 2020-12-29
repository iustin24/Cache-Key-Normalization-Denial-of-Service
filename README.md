

<img style="float: left;padding: 10px;margin-top:-20px" src="https://raw.githubusercontent.com/iustin24/Cache-Key-Normalization-Denial-of-Service/main/test.jpg">

### youstin
<a style="line-height: 2em;" href="https://twitter.com/iustinBB" target="_blank"><img style="float: left;" src="https://raw.githubusercontent.com/iustin24/Cache-Key-Normalization-Denial-of-Service/main/twitter.png" alt="Foo"> @iustinBB </a><br />
Published: 29 December 2020
<br><br>

### Cache Poisoning DoS

In today’s web, websites are often built with large bundles of Javascript derived from complex stacks of Typescript, SCSS, Webpack and more. To help mitigate the looming increase in load times for the standard webpage, caching is leveraged to reduce the load on servers and decrease latency for users. While caching is often meant to help increase the reliability of the service, making it more accessible to users, some custom cache configurations can introduce denial-of-service vulnerabilities that bring your service to its knees.

### Cache Poisoning DoS Basics 

A Cache poisoning vulnerability arises when the cache is tricked by an attacker into serving an altered response to every other user requesting the resource.
This is what a cache poisoning denial of service attack would look like:


![](https://github.com/iustin24/Cache-Key-Normalization-Denial-of-Service/blob/main/diagram.png?raw=true)


### Background

As you can see, all it takes in order to achieve a DoS attack is an uncached header that will force the Origin server into sending a malformed request.

I decided to look for potential DoS vulnerabilties on a few private programs, by applying the following methodology:

1. Detect all subdomains that were using caching services by identifying cache specific headers (`X-Cache, cf-cache-status, etc`).

2. Use <a href="https://github.com/PortSwigger/param-miner" target="_blank">Param Miner</a> in order to bruteforce potential uncached headers.

It didn't take me too long to find Cache Poisoning DoS on `assets.redacted.com`, the subdomain hosting every js & css file used on one of the private programs. The vulnerability was caused by Fastify’s `Accept-Version` header, which allows the client to describe which version of a resource to send back. I was able to abuse the feature like so:


```ceylon
GET /assets/login.js?cb=1                  GET /assets/login.js?cb=1
Host: assets.redacted.com                  Host: assets.redacted.com
Accept-Version: iustin

HTTP/1.1 404 Not Found                     HTTP/1.1 404 Not Found
X-Cache: Miss                              X-Cache: Hit
```


Since the Accept-version header is not included in the cache key, any user requesting the js file will recieve the cached 404 response. This was rewarded 2000$ and to my surprise, because fastify had no option to disable the `Accept-Version` header, it was also asigned <a href="https://snyk.io/vuln/SNYK-JS-FINDMYWAY-1038269" target="_blank">CVE-2020-7764</a>. 

However, after testing further hosts, it was increasingly apparent that I was going to be unable to find further vulnerable targets with this technique. I decided to do some additional research on other possible Cache-Poisoning DoS gadgets.

Most of the research I read, discussed how unkeyed input, such as the example header above, can lead to DoS, but they mostly ignored the keyed input such as the Host Header or path. I was able to come up with the two new following attacks, and succesfully reproduce them on bug bounty programs. 

### \#1 Host Header case normalization

According to <a href="https://tools.ietf.org/html/rfc4343" target="_blank">RFC 4343</a>, FQDN (Fully qualified domain names) should always be case insensitive, however, for some reason, this is not always respected by frameworks. Interestingly enough, since the host value should be case insensitive, some developers assume it's safe to lowercase the host header value when introducing it into the cachekey, without altering the actual request sent to the backend server.

When pairing the two behaviors, I was able to achieve the following DoS attack on a host using a customly configured Varnish as a caching solution.

```ceylon
GET /images/posion.png?cb=1                GET /images/posion.png?cb=1  
Host: Cdn.redacted.com                     Host: cdn.redacted.com

HTTP/1.1 404 Not Found                     HTTP/1.1 404 Not Found
X-Cache: Miss                              X-Cache: Hit
``` 
Notice the capitalized host header value, causing a 404 error, which will then be cached by Varnish using the normalized value of the host header in the cache key. This report was fixed quite quickly, and I recieved a 800$ bounty. 

The program also informed me that their loadbalancer (HAProxy), is the one responding with the 404 error when provided a capitalized header.

Besides host headers, parameters and paths could also be lower-cased before being injected into the cache key, so it's always worth checking how the cache treats them. 

### \#2 Path Normalization 

While identifying subdomains using caches, I found a particular subdomain hosting images used to construct maps. Requesting an image would look something like this:

```ceylon
GET /maps/1.0.5/map/4/151/16.png
Host: maps.redacted.com
```
Just like before, Param Miner was not able to find any hidden headers, so I decided to take a deeper look. As far as I could tell, the last three numbers in the path were ranges meant to tell the server what part of the map it should return. I played around with those for a good amount of time, but I was not getting anywhere.

Initially, I thought `1.0.5`, was just the version, so I didn't give it much attention, but to my surprise, when I tried `1.0.4`, I noticed I got a cache HIT. Naturally, I thought some other APIs might be using an older version, so I tested `1.0.0`, which also returned a cache HIT. It didn't take me too long to realize that whatever directory I was replacing `1.0.5` with was returning `200 OK` and an `X-Cache Hit` repsonse header. I came up with the follwoing DoS POC: 

```ceylon
GET /maps/%2e%2e/map/4/77/16.png          GET /maps/1.0.5/map/4/77/16.png
Host: maps.redacted.com                   Host: maps.redacted.com

HTTP/1.1 404 Not Found                    HTTP/1.1 404 Not Found
X-Cache: Miss                             X-Cache: Hit
```

Yet again, while trying to increase the cache-hit ratio, developers did not take in consideration potential DoS attacks, which allowed me to inject `%2e%2e`(URL encoded `..`) and redirect requests to `/map/4/77/16.png`, which did not exist on the server, therefore leading to the 404. This was triaged, and the team is working on a fix.

### Conclusion

When looking for cache poisoning DoS vulnerabilities, it's trivial to identify if the cache might be running a custom configuration meant to increase the hit-ratio, by normalizing parts of the uri. I have yet to research how often lowercase normalization is implemented on paths / parameters, so there's potentially more to be played with regarding uri normalization and caches.

Lastly, I'd like to thank <a href="https://skeletonscribe.net" target="_blank">James Kettle</a>, <a href="https://twitter.com/atul_hax" target="_blank">0xatul</a> and <a href="https://twitter.com/d0nutptr" target="_blank">d0nut</a> who have inspired / helped me through out my research.

