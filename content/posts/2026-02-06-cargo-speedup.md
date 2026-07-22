+++
title = 'Rust 在 Windows 上下载极慢的解决方案'
date = 2026-02-06T23:28:00+08:00
update = ""
author = "Hao Wu"
description = ""
tags = []
license = "CC BY-NC-SA 4.0"
license_url = "https://creativecommons.org/licenses/by-nc-sa/4.0/"
toc = false
draft = false
cover = "/assets/photos/2025/IMG_20251029_164918_hu_878f2c059ca215e0.webp"
+++

将下面字节跳动提供的镜像配置复制到`C:\Users\name\.cargo\config.toml`文件中即可。

```toml
[source.crates-io]
replace-with = 'rsproxy'

[source.rsproxy]
registry = "https://rsproxy.cn/crates.io-index"

# 稀疏索引，要求 cargo >= 1.68
[source.rsproxy-sparse]
registry = "sparse+https://rsproxy.cn/index/"

[registries.rsproxy]
index = "https://rsproxy.cn/crates.io-index"

[net]
git-fetch-with-cli = true
```
