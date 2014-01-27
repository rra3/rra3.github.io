---
layout: post
title: "Frozen Strings"
date: 2014-01-25 16:34:44 -0500
comments: true
categories: [ruby,rails]
---
### Frozen String spookiness
Last week I was working on a rails project for work when I encountered something funky and slightly maddening. I was attempting to slice selected values from the HTTP request hash:

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
...when the controller evaluated this code, it would simply stop..."freeze" if you will, emitting no errors. After some searching I discovered that Rails freezes the hash keys of core extensions. At first blush, it seemed that there was no way around the issue - no matter how I sliced or diced it, if I attempted to use the same strings in another hash, the app would choke on them. Even if you attempt a ".dup" on the string, the object's frozen flag is copied to the new string object. You can check if an object is frozen by sending the #frozen? message (and an object can be frozen with #freeze!):
```
2.0.0-p247 :002 > foo = ["one","two","three"]
 => ["one", "two", "three"]
2.0.0-p247 :003 > foo.collect! {|x| x.freeze}
 => ["one", "two", "three"]
2.0.0-p247 :004 > foo.each {|v| puts v.frozen?}
true
true
true
 => ["one", "two", "three"]
```

### What about symbols?
A Ruby symbol is a distinctly different object type from String in Ruby, but in Rails they are interchangable with strings for hash keys - at least for the HTTP Request object.  For this and other core extensions, Rails creates any associated hashes with [hash_with_indifferent_access](https://github.com/rails/rails/blob/master/activesupport/lib/active_support/hash_with_indifferent_access.rb?source=cc). This makes the fix for the above code very straight forward:


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

...to give your own hashes this handy behavior, just instantiate the hash using the ActiveSupport hash constructor: 

``` ruby Endow your own hashes with "indifferent access"
foo_hash = ActiveSupport::HashWithIndifferentAccess.new
foo_hash["bar"] = "baz"
foo_hash[:bar] # => returns "baz"
```

