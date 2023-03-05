---
title : Eventmachine 1.2.7  error during bundle with ruby 3.0.0
date : 2023-01-30 00:03:00 +0900
categories : [jekyll]
tags : [jekyll, ruby] #소문자만 가능
pinned : 0
---

```
4 warnings generated.
linking shared-object rubyeventmachine.bundle
ld: warning: directory not found for option '-L/usr/local/opt/openssl/lib/'
Undefined symbols for architecture arm64:
  "_SSL_get1_peer_certificate", referenced from:
      SslBox_t::GetPeerCert() in ssl.o
ld: symbol(s) not found for architecture arm64
clang: error: linker command failed with exit code 1 (use -v to see invocation)
make: *** [rubyeventmachine.bundle] Error 1
```
![eventmachine](/assets/img/posts/eventmachine.jpeg)
Error during bundle.

This needs help in locating the ssl path.

Solved by
```
arch -arm64 gem install eventmachine -v '1.2.7' -- --with-openssl-dir=$(brew --prefix openssl)
```
For further [reference](https://github.com/eventmachine/eventmachine/issues/932)
![eventmachine_solved](/assets/img/posts/eventmachine_solved.jpeg)