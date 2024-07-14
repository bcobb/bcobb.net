---
layout: post
title: "Debugging edge caching behavior"
seo:
  title: "Debugging edge caching behavior"
  description: "Debugging edge caching behavior"
published: true
slug: debugging-edge-caching-behavior
category: tbd
date: 2020-05-30 11:00
modified: 2020-05-30 11:00
---

I really loved [this comic][comic] Julia Evans posted, and it made me want to share a little "spy tool" I made [at work][harrys] earlier this year to figure out if a change I wanted to make to our edge caching configuration would have the desired result.

### The spy

Here's a simplified version of the spy:

```ruby
class RequestSpyController < ApplicationController
  def show
    render json: {
      ips: {
        remote_ip: request.remote_ip,
        ip: request.ip
      },
      headers: request.env.select { |k, _| k.match(/^[A-Z]/) }.as_json,
    }
  end
end
```

There's not much to it!
It's an API endpoint that returns the request headers, and the Rails-interpreted IP addresses.
It's been a useful way to quickly test my assumptions about how our edge cache configuration and our Rails configuration affect request headers.

### Appendix: what is edge caching

Before I joined Harry's, I wasn't too familiar with the concept of edge caching.

The idea is that you can put a geographically-distributed caching proxy between users and a web application to reduce the time it takes for cached URLs to be served.
That is, by some mechanism, requests will be routed through the geographically closest proxy server.
Edge caches are highly configurable, but typically they work as follows.

If the server doesn't have a cached response, the request will be forwarded on to the origin server.
The origin's response might be cached before being forwarded to the client.

<figure class="full-width">
  <img src="{{ site.url }}{% asset_path edge-cache-miss.png %}" alt="Diagram illustrating a cache miss">
  <figcaption>Forwarding a request to the origin server when there's nothing cached for the request.</figcaption>
</figure>

But, if the edge server has a cached response for a URL, the response will be served without needing to forward the request through to the origin server.

<figure class="full-width">
  <img src="{{ site.url }}{% asset_path edge-cache-hit.png %}" alt="Diagram illustrating a cache hit">
  <figcaption>Returning a cached response without consulting the origin.</figcaption>
</figure>

Serving from the cache is especially nice if the origin server is thousands of miles away from the computer making the request!

Especially when using Varnish and VCL, it feels similar to configuring nginx as a front-end web server to a back-end like Wordpress or Rails.

### Appendix: Why would it set headers?

The big caveat with edge caching is that, by default, once a URL is cached, the same response will be served until the cache entry expires.
Without special configuration, this makes it unsafe to include any information that could vary per-request in the response, such as a navigation menu which includes the user's name.

A common way to work around this caveat is to program the edge cache to alter the request headers right after it receives a request -- in particular to add some custom headers with a normalized set of values.
For example, the edge cache could add an `X-Segment` header to the inbound request, before it forwards the request to the origin server.
Maybe it looks to see if a session cookie is present in the request, and sets the `X-Segment` header value to `logged-in` if it is, and `logged-out` if it's not.
If the origin server returns a [`Vary`][vary] response header with a value of `X-Segment`, the edge cache will store one response per combination of URL and the value of the `X-Segment` header, rather than one response per URL.

With this kind of cache segmentation, the origin server could do something like return a different navigation bar for logged-in users than for logged-out users, and still get the benefits of edge caching.
But this approach hinges on setting the `X-Segment` header correctly, which makes the `RequestSpyController` above very handy!

[comic]: https://twitter.com/b0rk/status/1265360282513281025
[harrys]: https://www.harrys.com
[vcl]: https://varnish-cache.org/docs/6.4/users-guide/vcl.html
[vary]: https://www.fastly.com/blog/best-practices-using-vary-header
