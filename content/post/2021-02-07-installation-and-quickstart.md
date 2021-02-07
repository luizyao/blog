---
title: "Rust 安装和快速入门"
date: 2021-02-07
summary: "本文介绍如何安装 Rust，以及简单的 Demo。"
featured: false
draft: false
toc: true
featureImage: ""
thumbnail: ""
shareImage: ""
categories:
  - "Rust"
tags:
  - "rust"
---

## 安装

本文是在  MacOS 系统上实践的。

### rustup：Rust 安装和版本管理工具

 由于 Rust 更新的非常频繁，并且支持多个平台，[rustup](https://github.com/rust-lang/rustup) 工具能够帮助我们安装对应平台的版本，以及版本的更新。

我们可以通过以下命令，获取 rustup 并安装最新的 Rust：

```bash
❯ curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh
info: downloading installer

Welcome to Rust!

This will download and install the official compiler for the Rust
programming language, and its package manager, Cargo.

Rustup metadata and toolchains will be installed into the Rustup
home directory, located at:

  /Users/luizyao/.rustup

This can be modified with the RUSTUP_HOME environment variable.

The Cargo home directory located at:

  /Users/luizyao/.cargo

This can be modified with the CARGO_HOME environment variable.

The cargo, rustc, rustup and other commands will be added to
Cargo's bin directory, located at:

  /Users/luizyao/.cargo/bin

This path will then be added to your PATH environment variable by
modifying the profile files located at:

  /Users/luizyao/.profile
  /Users/luizyao/.zshenv

You can uninstall at any time with rustup self uninstall and
these changes will be reverted.

Current installation options:


   default host triple: x86_64-apple-darwin
     default toolchain: stable (default)
               profile: default
  modify PATH variable: yes

1) Proceed with installation (default)
2) Customize installation
3) Cancel installation
>2

I'm going to ask you the value of each of these installation options.
You may simply press the Enter key to leave unchanged.

Default host triple?


Default toolchain? (stable/beta/nightly/none)


Profile (which tools and data to install)? (minimal/default/complete)


Modify PATH variable? (y/n)
y


Current installation options:


   default host triple: x86_64-apple-darwin
     default toolchain: stable
               profile: default
  modify PATH variable: yes

1) Proceed with installation (default)
2) Customize installation
3) Cancel installation
>1

info: profile set to 'default'
info: setting default host triple to x86_64-apple-darwin
info: syncing channel updates for 'stable-x86_64-apple-darwin'
678.9 KiB / 678.9 KiB (100 %) 449.4 KiB/s in  1s ETA:  0s
info: latest update on 2020-12-31, rust version 1.49.0 (e1884a8e3 2020-12-29)
info: downloading component 'cargo'
  4.1 MiB /   4.1 MiB (100 %)   1.9 MiB/s in  2s ETA:  0s
info: downloading component 'clippy'
info: downloading component 'rust-docs'
 13.8 MiB /  13.8 MiB (100 %)   2.9 MiB/s in  4s ETA:  0s
info: downloading component 'rust-std'
 21.1 MiB /  21.1 MiB (100 %)   3.8 MiB/s in  5s ETA:  0s
info: downloading component 'rustc'
 50.8 MiB /  50.8 MiB (100 %)   7.4 MiB/s in  8s ETA:  0s
info: downloading component 'rustfmt'
info: installing component 'cargo'
info: using up to 500.0 MiB of RAM to unpack components
info: installing component 'clippy'
info: installing component 'rust-docs'
 13.8 MiB /  13.8 MiB (100 %)   3.5 MiB/s in  3s ETA:  0s
info: installing component 'rust-std'
 21.1 MiB /  21.1 MiB (100 %)   6.4 MiB/s in  3s ETA:  0s
info: installing component 'rustc'
 50.8 MiB /  50.8 MiB (100 %)   9.1 MiB/s in  5s ETA:  0s
info: installing component 'rustfmt'
info: default toolchain set to 'stable-x86_64-apple-darwin'

  stable-x86_64-apple-darwin installed - rustc 1.49.0 (e1884a8e3 2020-12-29)


Rust is installed now. Great!

To get started you need Cargo's bin directory ($HOME/.cargo/bin) in your PATH
environment variable. Next time you log in this will be done
automatically.

To configure your current shell, run:
source $HOME/.cargo/env
```

rustup 还会帮我们直接安装 [cargo](https://github.com/rust-lang/cargo)，Rust 的包管理工具，类似 Node.js 中的 npm。

安装完成后，在终端执行：

```bash
❯ rustc --version
rustc 1.49.0 (e1884a8e3 2020-12-29)
```

如果正常显示版本号，则表示安装成功。

> 针对于当前终端，需要先执行：`source $HOME/.cargo/env`，或者重新打开终端。

如果失败，多半是 `PATH` 中没有包含  `$HOME/.cargo/bin` 路径。

### 更新 Rust

更新 Rust 只需要执行：

```bash
❯ rustup update
info: syncing channel updates for 'stable-x86_64-apple-darwin'
info: checking for self-updates

  stable-x86_64-apple-darwin unchanged - rustc 1.49.0 (e1884a8e3 2020-12-29)

info: cleaning up downloads & tmp directories
```

> 这里我刚安装的，自然没有更新信息。

### 卸载 Rust

卸载 Rust 只需要执行：

```bash
❯ rustup self uninstall
```

## 快速开始

cargo 是 Rust 的构建工具和包管理器，使用 rustup 安装 Rust 时，会自动安装 cargo。可以通过以下命令查看 cargo 的版本信息：

```bash
❯ cargo --version
cargo 1.49.0 (d00d64df9 2020-12-05)
```

下面我们就使用 cargo 快速开始一个新项目。

### Hello World

使用 cargo 新建一个经典的 **Hello World** 项目：

```bash
❯ cargo new hello-rust
     Created binary (application) `hello-rust` package
```

这会新建一个 hello-rust 的文件夹， 它的目录结构如下：

```bash
hello-rust
├── Cargo.toml
└── src
    └── main.rs

1 directory, 2 files
```

- `Cargo.toml`：记录项目的配置项和依赖关系。所有的配置项可以参考：<https://doc.rust-lang.org/cargo/reference/manifest.html>

- `src/main.rs`：源代码。使用 `cargo new` 创建的项目，默认已包含以下内容：

  ```rust
  fn main() {
      println!("Hello, world!");
  }
  ```

  这是一个简单的打印 `Hello, world!` 的代码。

现在，我们可以直接使用 `cargo run` 运行这个项目：

```bash
❯ cargo run
   Compiling hello-rust v0.1.0 (/Users/luizyao/Private/projects/hello-rust)
    Finished dev [unoptimized + debuginfo] target(s) in 4.14s
     Running `target/debug/hello-rust`
Hello, world!
```

### 添加依赖

我们可以为这个项目添加第三方依赖，所有的依赖包都可以在这个网站 [crates.io](https://crates.io/) 上找到。

这里，我们尝试添加一个 [ferris-says](https://crates.io/crates/ferris-says) 依赖。

> Ferris 是 Rust 社区的吉祥物，它是一只“生锈”的螃蟹🦀️。ferris-says 能够在文本旁边打印出 Ferris。

我们需要在 `Cargo.toml` 文件中添加如下内容：

```toml
[dependencies]
ferris-says = "0.2.0"
```

然后，执行：

```bash
❯ cargo build
    Updating crates.io index
  Downloaded textwrap v0.11.0
  Downloaded unicode-width v0.1.8
  Downloaded vec_map v0.8.2
  Downloaded cfg-if v1.0.0
  Downloaded autocfg v1.0.1
  Downloaded ferris-says v0.2.0
  Downloaded clap v2.33.3
  Downloaded miniz_oxide v0.4.3
  Downloaded strsim v0.8.0
  Downloaded backtrace v0.3.56
  Downloaded gimli v0.23.0
  Downloaded rustc-demangle v0.1.18
  Downloaded ansi_term v0.11.0
  Downloaded bitflags v1.2.1
  Downloaded addr2line v0.14.1
  Downloaded object v0.23.0
  Downloaded smallvec v0.4.5
  Downloaded error-chain v0.10.0
  Downloaded atty v0.2.14
  Downloaded adler v0.2.3
  Downloaded libc v0.2.85
  Downloaded 21 crates (2.0 MB) in 4.70s
   Compiling libc v0.2.85
   Compiling autocfg v1.0.1
   Compiling bitflags v1.2.1
   Compiling gimli v0.23.0
   Compiling adler v0.2.3
   Compiling rustc-demangle v0.1.18
   Compiling cfg-if v1.0.0
   Compiling unicode-width v0.1.8
   Compiling object v0.23.0
   Compiling strsim v0.8.0
   Compiling ansi_term v0.11.0
   Compiling vec_map v0.8.2
   Compiling smallvec v0.4.5
   Compiling miniz_oxide v0.4.3
   Compiling textwrap v0.11.0
   Compiling atty v0.2.14
   Compiling clap v2.33.3
   Compiling addr2line v0.14.1
   Compiling backtrace v0.3.56
   Compiling error-chain v0.10.0
   Compiling ferris-says v0.2.0
   Compiling hello-rust v0.1.0 (/Users/luizyao/Private/projects/hello-rust)
    Finished dev [unoptimized + debuginfo] target(s) in 1m 01s
```

这会帮我们安装所有的依赖。

`Cargo.lock` 文件中锁定了所有依赖的版本信息。

如果想要使用上述安装的 `ferris-says`，我们需要修改 `main.rs` 文件：

```rust
use ferris_says::say;
```

> 这里我们删除 `main.rs` 文件之前所有的内容，因为这将是一个新的例子。

### 一个 Rust 小程序

这里我们使用上述安装的 ferris-says 依赖写一个 Rust 的小程序。代码如下：

```rust
use ferris_says::say;
use std::io::{stdout, BufWriter};

fn main() {
    let stdout = stdout();
    let message = String::from("Hello fellow Rustaceans!");
    let width = message.chars().count();
    let mut writer = BufWriter::new(stdout.lock());
    say(message.as_bytes(), width, &mut writer).unwrap();
}
```

执行：

```bash
❯ cargo run                           
   Compiling hello-rust v0.1.0 (/Users/luizyao/Private/projects/hello-rust)
    Finished dev [unoptimized + debuginfo] target(s) in 1.66s
     Running `target/debug/hello-rust`
 __________________________
< Hello fellow Rustaceans! >
 --------------------------
        \
         \
            _~^~^~_
        \) /  o o  \ (/
          '_   -   _'
          / '-----' \
```



