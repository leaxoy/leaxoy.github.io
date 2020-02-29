# Rust初探

&emsp;&emsp;看到这个标题，可能要有人问了，现在的编程语言已经那么多了，为什么还要在学一门，再学一门可以让我涨工资吗，这个语言比现有的语言好在哪里呢？这个问题我回答不了，但是时间会告诉你答案。

&emsp;&emsp;为什么突然想写这个了呢，一是最近把荒废了三年之久的博客有从新捡了起来，二是`Rust`最近发布了稳定的`Async/Await`功能(之前Block长达一年之久，我也等待了一年之久:joy:)，生态也在开始迁移了，二者缺一不可。
## 安装
&emsp;&emsp;推荐使用`rustup`来安装使用。
```shell
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh
```
安装过程中可能要选择版本，系统等信息，如果不晓得自己需要用啥，直接用默认的就好。经过一段时间的运行后，安装脚本会退出，使用下面的命令检查是否安装成功。
```shell
rustc -V
```
如果正确安装后，应该输出`rust`的版本号，就目前来说，输出的应该是`rustc 1.39.0 (4560ea788 2019-11-04)`，看到这个输出，意味着已经完成了rust的安装，恭喜你，你已经完成90%了，:smirk:。
## 起步
不同于Golang和Python，Rust自带了一个功能非常强大的管理工具`Cargo`，生成项目，管理项目/以来，交叉编译/打包。下面我们就开始吧：

生成一个新的项目:
```shell
cargo new my_first_rust_app
cd my_first_rust_app
cargo run
```
这条命令会生成一个简单的模版项目,并且运行相应的代码，可以看到命令行中已经出现`Hello, world!`,恭喜你，你已经掌握了Rust了。

我们看一下项目中的文件。

src/main.rs
```rust
fn main() {
    println!("Hello, world!");
}
```
项目的代码文件，声明了一个`main`函数，然后打印`Hello, world!`, 是不是非常的相似，只不过Go中是`func`， 而`Rust`中变成了`fn`，果然更加简洁，可能眼尖的同学已经发现了另外一处不同，`println!`怎么过了一个`!`呢，是不是写错了呢，

Cargo.toml
```toml
[package]
name = "my_first_rust_app"
version = "0.1.0"
authors = ["lixiaohui <lixiaohui0812@gmail.com>"]
edition = "2018"

# See more keys and their definitions at https://doc.rust-lang.org/cargo/reference/manifest.html

[dependencies]

```
项目描述的文件，说明了项目名称`my_first_rust_app`, 版本`0.1.0`, 作者信息`["lixiaohui <lixiaohui0812@gmail.com>"]`, rust的大版本`2018`, 下面还有一个模块`dependencies`声明了我们项目中使用到了依赖。接下来我们一一介绍各个部分的作用:
#### [package]
#### [(dev-/build-/target.*.)dependencies]
#### [profile.*]
#### [feature]
#### [workspace]

## 特色
### 无Runtime
### 零开销抽象
### 无空指针
### 无需手动释放内存/无GC
## 领域
### 数据库
### 图像
### 网关
### 游戏
### 音视频处理
## 和其他语言对比
### C++
### Go
### Java

