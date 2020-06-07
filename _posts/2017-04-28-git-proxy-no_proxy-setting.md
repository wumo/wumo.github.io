+++
title= "Git proxy and no_proxy setting (both working on Ubuntu/Windows)"
date= "2017-04-28"
tags= ["git"]
+++
<!--more-->
ref [Only use a proxy for certain git urls/domains?](http://stackoverflow.com/a/16069911)

set the environment variables:
```
http_proxy=http://username:password@proxydomain:port
https_proxy=http://username:password@proxydomain:port
no_proxy=.domain.com,localhost,127.0.0.1,::1
```
git uses these environment variables.