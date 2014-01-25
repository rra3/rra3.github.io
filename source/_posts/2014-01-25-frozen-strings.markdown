---
layout: post
title: "Frozen Strings"
date: 2014-01-25 16:34:44 -0500
comments: true
categories: ruby
---
### Frozen String spookiness
Last week I was working on a rails project for work when I encountered something funky and slightly maddening: frozen strings. I was attempting to slice selected values from the HTTP request hash:

``` ruby plucking values from the Rails Request object
    allowed_keys = [
        "REMOTE_HOST",
        "REMOTE_ADDR",
        "HTTP_REFERER",
        "ORIGINAL_FULLPATH",
        "REQUEST_METHOD",
        "REQUEST_URI",
        "HTTP_HOST",
        "HTTP_ORIGIN",
        "HTTP_USER_AGENT",
        "REQUEST_PATH",
        "QUERY_STRING",
        "CONTENT_LENGTH"
      ]
 self.visitor_track = self.env.slice(*allowed_keys)
```
...when the controller evaluated this code, it would simply stop..."freeze" if you will, emitting no errors. After some searching I discovered that Rails freezes the hash keys of core extensions. At first blush, it seemed that there was no way around the issue - no matter how I sliced or diced it, if I attempted to use the same strings in another hash, the app would choke on them. Even if you attempt a ".dup" on the string, the object's frozen flag is copied to the new string object.

### What about symbols?
Symbols are distinctly different object types in Ruby. Since rails offers "with indifferent access" on hashes for core extensions, this works perfectly:

``` ruby use symbols to access data on the request instead of strings. 
    allowed_keys = [
        :REMOTE_HOST,
        :REMOTE_ADDR,
        :HTTP_REFERER,
        :ORIGINAL_FULLPATH,
        :REQUEST_METHOD,
        :REQUEST_URI,
        :HTTP_HOST,
        :HTTP_ORIGIN,
        :HTTP_USER_AGENT,
        :REQUEST_PATH,
        :QUERY_STRING,
        :CONTENT_LENGTH
      ]
 self.visitor_track = self.env.slice(*allowed_keys)
```

