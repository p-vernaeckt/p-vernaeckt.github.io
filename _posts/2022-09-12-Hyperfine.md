---
title: Hyperfine, a better version of time
date: 2022-09-12 18:30:00 +0200
categories: [Sysadmin, Better CLI tools]
tags: [time, cli, bash, zsh]     # TAG names should always be lowercase
---

Sick of boring `time` command? Sure, there is nothing very exciting about this one, even if it does its job quite well:

![time](/assets/img/posts/2022-09-12-Hyperfine/time.png)

Here is [Hyperfine](https://github.com/sharkdp/hyperfine), a drop-in replacement for `time` written in Rust that offers way, way more features than plain ol' `time`.

![hyperfine demo](https://camo.githubusercontent.com/88a0cb35f42e02e28b0433d4b5e0029e52e723d8feb8df753e1ed06a5161db56/68747470733a2f2f692e696d6775722e636f6d2f7a31394f5978452e676966)

Hyperfine can handle program warm-up, multiple runs, parameters for your benchmarks and many more. To me, the best alternative for `time`.

Have fun!