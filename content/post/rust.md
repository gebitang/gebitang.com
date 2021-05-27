+++
title = "Rust"
description = "Lean Rust"
tags = [
    "rust"
]
date = "2021-05-27"
topics = [
    "rust"
]
toc = true
+++

受[#本原 系列](https://mp.weixin.qq.com/s/jaKjzc_1rkDe67rfpnFTgg)吸引，安装了rust的windows开发环境，然后就没有然后了。现在有机会跟作者[学习一下](https://mp.weixin.qq.com/s/tkH6zZt5vsDL2gld6ufk2w)

先看看[slides](https://tyrchen.github.io/rust-training/rust-training-all-in-one.html)，补补基础课——好些简称我不知道啥含义:) 

- GIL (Global interpreter lock)
- CSP (Communicating Sequential Processes) [原始论文](https://www.cs.cmu.edu/~crary/819-f09/Hoare78.pdf)
- Duck Typing (动态类型)
- RAII (Resource Acquisition Is Initialization)
- FFI (Foreign Function Interface) 
- MRC (manual reference counting)
- ARC (automatic reference counting)

[RUST语言的编程范式](https://coolshell.cn/articles/20845.html)这篇之前看的时候早早就被劝退了，今天重读至少看完没晕。学习一段时间的Rust之后再重新去读这一篇:) 


学习Rust会“面对编译器编程”，需要一开始就面对所有的基础知识。[The True Story of Hello World](https://lisha.ufsc.br/teaching/os/exercise/hello.html) 介绍“hello world”的一生。涉及到[BSS, Stack, Heap, Data, Code/Text - Where each of these start in memory?](https://stackoverflow.com/questions/2223261/bss-stack-heap-data-code-text-where-each-of-these-start-in-memory)，[图片说明](https://imgur.com/a/JEObT)

### 安装

[rust 安装](https://www.rust-lang.org/tools/install)——

- Windows: 下载`rusti-init.ext`进行安装，默认选项，安装完毕就可以使用
- 类Unix平台：执行`curl https://sh.rustup.rs -sSf | sh` 

安装完成后，添加环境变量。其中`.cargo/bin`目录包含所有的工具——

- `rustup`管理rust工具本身：[rustup](https://github.com/rust-lang/rustup)是一个`the rust toolchain installer` 用来安装rust以及对于的工具组件。更详细内容参考[rustup book](https://rust-lang.github.io/rustup/installation/index.html)
- `rustc`相当于类似gcc的编译器，单个文件编译
- `Cargo` 是 rust 的构建系统和包管理器，相当于工程抓手





