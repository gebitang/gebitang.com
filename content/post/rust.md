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

#### Hello World

- 单个无依赖的rust文件可以使用`rustc main.rs`进行编译，生成可执行文件
- 项目管理使用`cargo`(不同平台下，命令相同)：
  - `cargo build` 解析cargo.toml文件，管理下载，编译生成可执行文件，默认生成的为debug版本，位于`target/debug`目录下
  - `cargo check` 只做编译检查，不生成可执行文件，速度更快
  - `cargo run` 编译+运行
  - `cargo build --release`生成优化后的发布版本，位于`target/release`目录下，编译更耗时
  - `cargo doc --open`可以本地浏览当前项目中用到的所有crate的文档

[cargo 官方文档](https://doc.rust-lang.org/cargo/)

- 两类数据类型子集：标量（scalar）和复合（compound） 
  - 四种基本的标量类型：整型、浮点型、布尔类型和字符类型
  - 两个原生的复合类型：元组（tuple）和数组（array）

```rust

fn main() {

    //let x = 5;
    let mut x = 5;
    println!("the value of x is {}", x);

    x = 6;
    println!("the value of x is {}", x);

    const MAX_POINT: u32 = 100_000;

    println!("max point is {}", MAX_POINT);

    let y = 5;
    let y = y + 2;
    let y = y + 1;
    println!("now the value of y is {}", y);

    let space = "abc";
    let space = space.len();
    println!("the length of space is {}", space);

    // 加法
    let sum = 5 + 10;
    // 减法
    let difference = 95.5 - 4.3;
    // 乘法
    let product = 4 * 30;
    // 除法
    let quotient = 56.7 / 32.2;
    // 取余
    let remainder = 43 % 5;

    println!("sum:{} difference:{} product:{}, quatient:{} remainder:{}", sum, difference, product, quotient, remainder);

    // char 类型的大小为四个字节(four bytes)，并代表了一个 Unicode 标量值（Unicode Scalar Value)
    let c = 'z';
    let z = 'ℤ';
    let heart_eyed_cat = '😻';
    println!("c:{} z:{} hec:{}", c, z, heart_eyed_cat);

    //元组长度固定：一旦声明，其长度不会增大或缩小
    let tup: (i32, f64, u8) = (500, 6.4, 1);
    let (x, y, z) = tup; // ignore x and z

    println!("The value of x is: {} = {}", tup.0, x);
    println!("The value of y is: {} = {}", tup.1, y);
    println!("The value of z is: {} = {}", tup.2, z);

    let tup = ("abc", 1);
    println!("the first tuple value: {}", tup.0);

    // Rust 中的数组是固定长度的：一旦声明，它们的长度不能增长或缩小。
    let months = ["January", "February", "March", "April", "May", "June", "July",
              "August", "September", "October", "November", "December"];

    let a: [i32; 5] = [1, 2, 3, 4, 5];

    println!("print a: {}", a[2]);
    println!("print month {}", months[3]);

    // 包含 5 个元素，这些元素的值最初都将被设置为 3
    let a = [3; 5];
    println!("len: {}", a.len());

    // let a = [1, 2, 3, 4, 5];
    // let index = 10;
    // let element = a[index];
    // println!("The value of element is: {}", element);

    another_function(5);

    let x = 5;

    let y = {
        let x = 3;
        x + 1 //表达式的结尾加上分号，它就变成了语句，而语句不会返回值
    };

    println!("The value of y is: {} x:{}", y, x);

    println!("five func return {}", five());

    println!("plus one func return {}", plus_one(7));


    let condition = true;
    let number = if condition {
        5
    } else {
        6
    };

    println!("The value of number is: {}", number);

    let mut counter = 0;

    let result = loop {
        counter += 1;

        if counter == 10 {
            break counter * 2;
        }
    };

    println!("The result is {}", result);


    // for 循环
    let a = [10, 20, 30, 40, 50];

    for element in a.iter() {
        println!("the value is: {}", element);
    }

    for number in (1..4).rev() {
        println!("{}!", number);
    }
    println!("LIFTOFF!!!");
}

fn another_function(x: i32) {
    println!("this is another function. {}", x);
}

// 没有函数调用、宏、甚至没有 let 语句——只有数字 5。这在 Rust 中是一个完全有效的函数
fn five() ->i32 {
    5
}

// 包含 x + 1 的行尾加上一个分号，把它从表达式变成语句，将是一个错误。mismatched types
fn plus_one(x: i32) -> i32 {
    x +1 
}

```

### Ownership

所有权（系统）是 Rust 最为与众不同的特性，它让 Rust 无需垃圾回收（garbage collector）即可保障内存安全。相关功能包括：借用、slice 以及 Rust 如何在内存中布局数据。

程序管理使用计算机内存的方式：

- 垃圾回收机制。如Java 
- 手动分配分配和释放内存。如C，C++
- 所有权系统管理内存。如Rust。其优点：编译器在编译时会根据一系列的规则进行检查。在运行时，所有权系统的任何功能都不会减慢程序(也就没有STW问题)。

首先要了解“栈（Stack）与堆（Heap）”的四个主要区别——

- 栈：后进先出（last in, first out）；所有数据都必须占用已知且固定的大小；分配快，(因为（入栈时）操作系统无需为存储新数据去搜索内存空间；其位置总是在栈顶)；访问快(因为存储的连续性）
- 堆：指针访问；数据大小可能变化；分配慢(需要先搜索到足够大的空间；标记已用；返回指针——allocating on the heap)；访问慢（需要通过指针跳转）

>>当代码调用一个函数时，传递给函数的值（包括可能指向堆上数据的指针）和函数的局部变量被压入栈中。当函数结束时，这些值被移出栈。

所有权意义：(管理堆数据)

- 跟踪哪部分代码正在使用堆上的哪些数据
- 最大限度的减少堆上的重复数据的数量
- 清理堆上不再使用的数据确保不会耗尽空间

所有权规则：

1. Rust 中的每一个值都有一个被称为其 所有者（owner）的变量。Each value in Rust has a variable that’s called its *owner*.
2. 值在任一时刻有且只有一个所有者。There can only be one owner at a time.
3. 当所有者（变量）离开作用域，这个值将被丢弃。When the owner goes out of scope, the value will be dropped.

```
let s1 = String::from("hello");
let s2 = s1;

println!("{}, world!", s1); // 无效
println!("{}, world!", s2);
```