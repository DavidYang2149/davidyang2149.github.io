---
layout  : wiki
title   : localhost 공유하기 - localtunnel
date    : 2023-01-10 19:15:00 +0900
updated : 2023-01-10 19:15:00 +0900
tag     : 
toc     : true
public  : true
parent  : tip
latex   : false
---
* TOC
{:toc}

## localhost 공유하기 - localtunnel

[localtunnel](https://github.com/localtunnel/localtunnel)는 localhost를 타인에게 공유할 수 있도록 돕는 툴이다.

```sh
# npm
npm install -g localtunnel

# brew
brew install localtunnel
```

해당 도구를 설치한 뒤에 원하는 포트를 지정하면 원격으로 접속할 수 있는 URL을 제공한다.

```sh
# Basic 
lt --port 3000

# Custom domain
lt --port 3000  --subdomain david
```
