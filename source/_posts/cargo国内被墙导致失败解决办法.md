---
title: cargo国内被墙导致失败解决办法
top: false
cover: false
toc: true
mathjax: false
date: 2023-02-28 17:24:26
author:
img:
coverImg:
password:
summary:
tags:
- rust
- 编程环境
categories: [Programming Language,Rust language]
---
# 打开cargo 配置文件
```shell
vi ~/.cargo/config
```

# 替换文中内容

```toml
# 放到 `$HOME/.cargo/config` 文件中
[source.crates-io]
registry = "https://github.com/rust-lang/crates.io-index"

# 替换成你偏好的镜像源
replace-with = 'sjtu'
#replace-with = 'ustc'

# 清华大学
[source.tuna]
registry = "https://mirrors.tuna.tsinghua.edu.cn/git/crates.io-index.git"

# 中国科学技术大学
[source.ustc]
registry = "git://mirrors.ustc.edu.cn/crates.io-index"

# 上海交通大学
[source.sjtu]
registry = "https://mirrors.sjtug.sjtu.edu.cn/git/crates.io-index"

# rustcc社区
[source.rustcc]
registry = "git://crates.rustcc.cn/crates.io-index"

[net]
git-fetch-with-cli=true

```