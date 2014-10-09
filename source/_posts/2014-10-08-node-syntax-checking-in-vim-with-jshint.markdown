---
layout: post
title: "Node syntax checking in vim with JSHint"
date: 2014-10-08 22:37:33 -0400
comments: true
categories: vim, nodejs 
---

I've been using syntastic for a while with vim, and recently I've been doing a lot work with a nodejs project, and the default syntax checking for javascript was annoying - it kept warning me about semi-colons and bad line breaks, which don't really matter much in a node environment. I discovered that if I ran the command :SyntasticInfo while working with a js file, it will tell you which plugin it uses for lint checking the code, which turned out to be [JSHint](http://jshint.com/). The nice thing about jshint is that you can configure it with a [bewildering number of options](http://jshint.com/docs/options/) to suit your needs in the ".jshintrc" file which lives in your home directory and is just json. I found the following options turned off the annoying warnings I was getting and made the lint check suitable for node development:

```javascript
{
  "asi": true,
  "laxcomma": true,
  "node": true
}
```
