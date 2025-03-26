# Rust 入门

## Linux 安装 Rust

* `curl --proto '=https' --tlsv1.2 https://sh.rustup.rs -sSf | sh` 或者 `brew install rust rustup`
* `apt install gcc`
* 验证: `rustc --version`
* 升级: `rustup update`
* 卸载: `rustup self uninstall`

## Hello World
创建一个文件 hello.rs，定义函数：
```Rust
fn main() {
    println!("Hello, world!");
}
```
Rust 的缩进风格使用 4 个空格，而不是 1 个制表符（tab）。

`println!` 调用了一个 Rust 宏（macro）。如果是调用函数，则应输入 `println` （没有!）。当看到符号 ! 的时候，就意味着**调用的是宏而不是普通函数**，并且宏并不总是遵循与函数相同的规则。

语句以分号结尾（;），这代表一个表达式的结束和下一个表达式的开始。大部分 Rust 代码行以分号结尾。

编译：`rustc hello.rs`

## Cargo
+ 查看 cargo 版本：`cargo --version`

### 使用 Cargo 创建项目
`cargo new hello_cargo` ,会创建一个 hello_cargo 目录，里边有一个 Cargo.toml 文件和一个 src 目录：
```
└─$ tree hello_cargo/
hello_cargo/
├── Cargo.toml
└── src
	└── main.rs
```

这也会在 hello_cargo 目录初始化了一个 git 仓库，以及一个 .gitignore 文件。如果在一个已经存在的 git 仓库中运行 cargo new，则这些 git 相关文件则不会生成；可以通过运行 `cargo new --vcs=git` 来覆盖这些行为。

### 构建并运行 Cargo 项目
在项目根目录下运行：`cargo build`，这个命令会创建一个可执行文件 `target/debug/hello_cargo` ，而不是放在目前目录下。由于默认的构建方法是调试构建（debug build），Cargo 会将可执行文件放在名为 debug 的目录中:
```
└─$ tree
.
├── Cargo.lock
├── Cargo.toml
├── src
│   └── main.rs
└── target
    ├── CACHEDIR.TAG
    └── debug
        ├── build
        ├── deps
        │   ├── hello_cargo-deaad4c17ac37e0b
        │   └── hello_cargo-deaad4c17ac37e0b.d
        ├── examples
        ├── hello_cargo
        ├── hello_cargo.d
        └── incremental
            └── hello_cargo-2pueg9mozk8ji
                ├── s-gxicz4d46o-1bykc7u-11wguxc3jq96bydmcaozl5au
                │   ├── 1jcp40zb4zxwgzzu.o
                │   ├── 22kjk1sxdhcblxcw.o
                │   ├── 2fltho8o95x6j6qq.o
                │   ├── 4x9bxtcyjq62i9bx.o
                │   ├── 54s9b2m4at8mbuh4.o
                │   ├── dep-graph.bin
                │   ├── jqh9v2bjf4umt2x.o
                │   ├── query-cache.bin
                │   └── work-products.bin
                └── s-gxicz4d46o-1bykc7u.lock
```

首次运行 `cargo build` 时，也会使 Cargo 在项目根目录创建一个新文件：`Cargo.lock`。这个文件记录项目依赖的实际版本。这个项目并没有依赖，所以其内容比较少。你自己永远也不需要碰这个文件，让 Cargo 处理它就行了。

前面我们使用 `cargo build` 构建了项目，并使用 ./target/debug/hello_cargo 运行了程序，也可以使用 `cargo run` 在一个命令中同时编译并运行生成的可执行文件。

Cargo 如果发现文件并没有被改变，它不会有重新编译，而是直接运行了可执行文件。如果修改了源文件的话，Cargo 会在运行之前重新构建项目。

`cargo check` 该命令快速检查代码确保其可以编译，但并不产生可执行文件：
```
┌──(admin㉿DESKTOP-CODER)-[~/rust-workspace/hello_cargo]
└─$ cargo check
    Checking hello_cargo v0.1.0 (/home/admin/rust-workspace/hello_cargo)
    Finished `dev` profile [unoptimized + debuginfo] target(s) in 0.08s

┌──(admin㉿DESKTOP-CODER)-[~/rust-workspace/hello_cargo]
└─$ tree
.
├── Cargo.lock
├── Cargo.toml
├── src
│   └── main.rs
└── target
    ├── CACHEDIR.TAG
    └── debug
        ├── build
        ├── deps
        │   ├── hello_cargo-8834076585394559.d
        │   └── libhello_cargo-8834076585394559.rmeta
        ├── examples
        └── incremental
            └── hello_cargo-2e0bekeiibgkq
                ├── s-gxid6hpfkn-4cqa94-69k2wdgv6sk6mt1irzppt3rw9
                │   ├── dep-graph.bin
                │   ├── query-cache.bin
                │   └── work-products.bin
                └── s-gxid6hpfkn-4cqa94.lock

10 directories, 10 files
```

+ `cargo new`: 创建项目。
+ `cargo build`: 构建项目。
+ `cargo run`: 一步构建并运行项目。
+ `cargo check`: 在不生成二进制文件的情况下构建项目来检查错误。
+ `cargo doc --open`: 来构建所有本地依赖提供的文档，并在浏览器中打开crate 使用说明文档
+ 有别于将构建结果放在与源码相同的目录，Cargo 会将其放到 `target/debug` 目录。

### 发布（release）构建
当项目最终准备好发布时，可以使用 `cargo build --release` 来优化编译项目。这会在 `target/release` 而不是 `target/debug` 下生成可执行文件。这些优化可以让 Rust 代码运行的更快，不过启用这些优化也需要消耗更长的编译时间。这也就是为什么会有两种不同的配置：一种是为了开发，你需要经常快速重新构建；另一种是为用户构建最终程序，它们不会经常重新构建，并且希望程序运行得越快越好。如果你在测试代码的运行时间，请确保运行 `cargo build --release` 并使用 `target/release` 下的可执行文件进行测试。
