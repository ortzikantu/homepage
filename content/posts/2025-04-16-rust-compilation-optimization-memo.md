+++
title = '配置优化 Rust 编译速度备忘录'
date = 2025-04-16T19:26:00+08:00
update = ""
author = "Hao Wu"
description = ""
tags = []
license = "CC BY-NC-SA 4.0"
license_url = "https://creativecommons.org/licenses/by-nc-sa/4.0/"
toc = false
draft = false
cover = "/assets/photos/gtshow2024/IMG_20240331_141604_hu_dbadc47d29ebf683.webp"
+++

构建 Rust 项目最难受的点，编译速度慢，编译后的二进制文件体积也不小，这里作为备忘录记一些配置参数。

```toml
[profile.release]
opt-level = "3"
lto = true
codegen-units = 1
strip = "debuginfo"
panic = "abort"
debug = false
```

#### 1. opt-level
- `0` 无优化
- `1` 基本档
- `2` 平衡档
- `3` 全开档
- `s` 优化并稍微减小代码体积
- `z` 最小代码体积，但会降低运行性能

#### 2. lto
- `false` 禁用 LTO
- `true` 启用 LTO
- `thin` 启用 Thin LTO
- `fat` 启用最激进的 LTO

#### 3. codegen-units
- 有16个档位，数字越小优化强度越高,但编译速度越慢

#### 4. strip
- `none` 保留所有信息
- `debuginfo` 移除调试信息
- `symbols` 移除符号表但保留调试信息
- `all` 移除所有信息

#### 5. panic
- `unwind` 展开栈
- `abort` 直接中止进程

#### 6. debug
- `debug = false` 禁用 Debug 功能（加上也无妨）
