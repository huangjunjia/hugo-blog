---
title: "Rust 的安装与环境准备"
date: 2023-05-12T15:47:27+08:00
draft: false
---

# 环境准备

在 mac 下安装 Rust 是一件很容易的事情，步骤如下：

1. 安装 C 编译器

    ```shell 
    $ xcode-select --install
    ```

2. 打开终端输入

    ```shell
    $ curl --proto '=https' --tlsv1.2 https://sh.rustup.rs -sSf | sh
    ```

   如果安装成功，你将会在终端输出中看到

    ```shell
    Rust is installed now. Great!
    ```

## 其他基本操作

### 更新 `rustup`

```shell
$ rustup update
```

### 卸载 `rustup`

```shell
$ rustup self uninstall
```

### 查看 `rustup` 版本

```shell
$ rustc --version
```

你将会看到

```shell
rustc x.y.z (abcabcabc yyyy-mm-dd)
```

### 本地文档

```shell
$ rustup doc
```

## 替换源

在 `~/.cargo` 目录下创建 `config` 文件（如果没有的话，一般情况下没有），然后写入以下内容：

```toml
[source.crates-io]
registry = "https://github.com/rust-lang/crates.io-index"

# 替换成你偏好的镜像源
replace-with = 'rsproxy'

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

# 字节
[source.rsproxy]
registry = "https://rsproxy.cn/crates.io-index"
[source.rsproxy-sparse]
registry = "sparse+https://rsproxy.cn/index/"
[registries.rsproxy]
index = "https://rsproxy.cn/crates.io-index"

[net]
git-fetch-with-cli = true
```

保存即可

同时修改本地环境变量，一般是 `~/.zshrc` 或者 `~/.bash_profile`

```dotenv
export RUSTUP_DIST_SERVER="https://rsproxy.cn"
export RUSTUP_UPDATE_ROOT="https://rsproxy.cn/rustup"
```