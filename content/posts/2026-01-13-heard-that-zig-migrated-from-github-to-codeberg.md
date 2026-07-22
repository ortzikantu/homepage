+++
title = '听闻 Zig 从 Github 迁移到 Codeberg'
date = 2026-01-13T20:07:00+08:00
update = ""
author = "Hao Wu"
description = ""
tags = []
license = "CC BY-NC-SA 4.0"
license_url = "https://creativecommons.org/licenses/by-nc-sa/4.0/"
toc = false
draft = false
cover = "/assets/photos/2025/IMG_20250627_155439_hu_80cd94a4b7f95d9f.webp"
+++

最近刷到去年的几条讨论，是 Zig 把代码仓库迁移到 Codeberg 的事。

Andrew Kelly 吐槽 Github Actions 的陈年 Bug 和平台 All in AI（ Copilot ）的事情在开发社区引起了不小的讨论。看不惯 GitHub 操作的不止他一个，像 Dillo 、Scrobbles4j、Bit101 等均已另寻平台，其中 Dillo 甚至直接用 Cgit 自己搭了一个代码托管服务。

在不少人眼里，GitHub 堪称代码世界的理想家园、开源领域的文明灯塔，甚至被赋予了某种 “开源乌托邦” 式的崇拜色彩。但我想说，时至今日，仍有部分国家无法正常使用 GitHub 服务 —— 这并非国内常见的 DNS 污染，而是源于美国贸易限制与制裁政策的单方面封杀。

早年间伊核问题发酵之际，就曾发生过 GitHub 无预警、无通知封禁伊朗开发者账号的事件。尽管账号后来得以解封，但这份毫无章法的操作，早已让它在部分开发者心中失去了信任的根基。

这正是我当年着手寻找 GitHub 替代品的契机。人人都喊 “开源无国界”，可这些商业公司背后，分明刻着清晰的国家边界，EAR 条例、各类制裁，就是最好的佐证。依赖这种中心化、商业化的第三方平台，终究难逃潜在的隐患。

这些年在服务器上搭建并体验过 Cgit、Gogs、Gitea、Onedev、Gitbucket（ 用 Scala 开发的开源项目，不是 Bitbucket ）甚至还有 Fossil，表现都相当亮眼，35块一个月的双核1G的轻量云服务器都能丝滑运行。

前文提到的 Codeberg 算是当下风头正劲的新兴代码托管平台，背靠位于德国柏林的非营利组织 Codeberg e.V.，其所运行的基石 Forgejo 是 Codeberg 基于 Gitea 分支二次开发而来的，哪怕不想在Codeberg上使用服务，也可以自己租服务器来运行一个 Forgejo 实例。除此之外，还有一个颇具 “原教旨主义” 色彩的平台 Sourcehut 也不错，保持了 KISS 原则，按功能拆分成 git.sr.ht、hg.sr.ht、todo.sr.ht、lists.sr.ht、builds.sr.ht 这几个模块，而且它不使用常见的PR模型，只用邮件列表和提交补丁的方式协作开发，有点类似Linux内核开发流程，我倒是很喜欢这个方案。

需要说明的是，我并非主张所有人逃离GitHub。它的生态优势毋庸置疑，仍是多数开源项目的优选协作载体。我真正希望的，是每个开发者都能跳出“平台依赖”的惯性，多一份居安思危的警惕。
