---
title: go get through socks5 proxy
tags: 技术 go
---

go get 撞墙了，在`go get github.com/mattn/go-sqlite3` 的时候，绕过一道墙，然后go get 里面还有`git clone https://go.googlesource.com/net /Users/topcaver/code/go/src/golang.org/x/net` 这样的操作……

<!--more-->

### Step 1: set git through socks5 proxy:
    git config --global http.proxy socks5://127.0.0.1:7070

### Step 2: go get via socks5 proxy:
    http_proxy=socks5://127.0.0.1:7070 go get github.com/mattn/go-sqlite3

### Recover git global config:
    git config --global --unset http.proxy

### ref:
https://stackoverflow.com/questions/10383299/how-do-i-configure-go-to-use-a-proxy 
https://stackoverflow.com/questions/128035/how-do-i-pull-from-a-git-repository-through-an-http-proxy/3406766#3406766

