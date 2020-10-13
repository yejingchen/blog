+++
title = "YouCompleteMe 合理配置 Rust 和 ccls"
date = 2020-10-12 21:56:11+08:00
[taxonomies]
categories = ["计算机"]
tags = ["vim", "rust", "c", "c++"]
+++

使用 VIM 不仅可以积极参与编辑器圣战，还能和不同补全框架甚至编程语言的拥趸打成一
片，更能配环境配一整天，充分体会摸鱼的快乐。~~甚至还能水一篇博文~~

[YouCompleteMe][0] 作为 VIM 的老牌补全框架，在 [LSP][1] 横空出世后也加入了对其
的支持，极大地扩展了能补全 / lint 的语言数量。和我前段时间在用的注重移植 VSCODE
扩展的 [coc.nvim][3] 相比，YCM 用起来更有 VIM 味。但是 YCM 也以安装的困难程度
闻名，虽然最近听说情况变好了不少，但毕竟无脑一键脚本无法满足所有人的需求，因此
下面来看看更合理的使用方法吧。

<!-- more -->

目标
----
我们的目标是在 Arch Linux 和 Mac 上安装 YCM 并支持 C 系、Rust、Python 语言的补
全 / lint，并且避开安装过程里不合理的地方。

 - 首先是使用单独的 [ccls][4] 而不是默认的 clangd 作为 C 系语言的 LSP，至于
   原因只是个人喜好罢了，毕竟还能少编译点东西（在 Arch 官方仓库和 Homebrew 上都
   有预编译二进制）。

 - 其次是使用已有的 Rust 工具链和 [rust-analyzer][8]，原因同样是可以少点编译
   （同样在 Arch 官方仓库、Homebrew、rustup 或包管理中有），并且使用和自己的工
   作环境相同的工具链应该是比让补全工具单独用一套工具链要合理点。

 - 最后 Python 支持是 YCM 自带的，使用的是自己 vendor 的 Jedi。虽然另有 Python
   langauge server 可以用，不过那也是基于 Jedi 实现的，这里就直接让 YCM 自己处
   理吧，毕竟其实我不怎么写 Python ~~（真正的原因是安装脚本里没有不安装 Python
   支持的选项）~~

另外还有几个小目标：

 - 在 Lightline 里显示警告和错误的数量

 - 不要和 [ALE][2] 打架，让 ALE 负责其他格式文件的 linting，上面提到的语言则让
   YCM 完成

 - 给跳转设置按键

 - 改善生活的细节设置

下面就开始吧！

安装本体
--------
YCM 本体绝大部分功能都在 C++ 写的 `ycmd` 中，VIM 侧只是个客户端，另外还需要
Python 来联系两者。参照 YCM 自己的说明，安装前你需要下面的依赖（本文撰写时）：

 - VIM 8.1.2269+，带 Python 3.6+ 支持（Mac 上必须是 MacVim）
 - C++ 编译器
 - CMake
 - Python 3.6+，带动态链接支持和开发文件

最好需要：VIM 的插件管理器，我用的是 [vim-plug][5]，YCM 推荐 [Vundle][6]，基本
上只要是支持安装或更新完后运行命令的管理器都可。我不知道完全手动怎么装！

以 vim-plug 为例，在 vimrc 中加入:

```vim
Plug 'ycm-core/YouCompleteMe', { 'do': '/usr/bin/python3 install.py' }
```

> ⚠注意
>
> 这里我们并没有给安装脚本加任何参数，因为添加 `--clangd-completer` 和
> `--rust-completer` 会在安装过程中编译 `clangd`（需要 10GB 的 Xcode）和
> `rust-analyzer`（会全新安装 rustup、额外的老的 Rust nightly 工具链），而我们
> 要手动安装预编译的版本。

重新加载 vimrc 或重启 VIM 之后运行 `:PlugInstall`，等待克隆插件并等待安装脚本运行完毕。如果中途出了差错想要再次运行安装脚本可以用感叹号 `:PlugInstall!`。

这步完成之后就得到了只有 Python 支持的 YCM。可以随便打开个 Python 脚本，运行
`:YcmDebugInfo` 看 YCM 能否正常启动，以及日志里有没有报错。

安装和配置 ccls
---------------
ccls 调用 LLVM 来提供 C 系语言的支持，所以很不幸，安装它也必须安装完整的 LLVM
（而不是 macOS 自带的 Apple 牌阉割版 LLVM）；但幸运的是 Arch 和 Homebrew 上都有
预编译的 ccls 和 LLVM。

Arch 仓库里和 Homebrew 里的包名都叫做 `ccls`。

安装好后需要设置 YCM，在 vimrc 中设置 LSP：

```vim
let g:ycm_language_server =
            \[
            \   {
            \       'name': 'ccls',
            \       'cmdline': ['ccls'],
            \       'filetypes': ['c', 'cpp', 'objc', 'objcpp'],
            \       'project_root_files': ['.ccls-root', 'compile_commands.json']
            \   },
            \]
```

ccls 完成！

开玩笑，ccls 和 C 系项目的配合参见 [ccls wiki][7]，主要内容就是如何填写或让你的
项目产生上面写到的 `.ccls-root`、`compile_commands.json` 文件。不过此时直接打开
单个 c 文件已经有基本的补全和提示了，说完成也八九不离十了吧。

另外如果你的 ccls 不在 `PATH` 里，比如你直接从 GitHub 上下载了二进制，上面的
`cmdline` 配置里需要写上完整的路径。当然最好还是直接放进 `PATH` 里啦。

安装和配置 rust-analyer
-----------------------
Rust 的配置分为两部分，分别是工具链和 rust-analyzer 自己。

工具链的安装方法就不再赘述了，用系统包管理也好、`rustup` 也好、自己手动编译安装
也好，总之要求就是：**必须同时安装了工具链对应的源码**，比如在 Arch 上安装了
`rust-docs` 包，或者在`rustup` 上安装了 `rust-src` 组件。当然我相信会用到 YCM
补全 Rust 的人早就已经有装好的工具链了。

rust-analyzer 的安装就没什么复杂的，直接用 `pacman` 或者 Homebrew 装即可，报
名就叫 `rust-analyzer`。如果你用的是 `rustup` 安装的 nightly 工具链，也可以直接
用 `rustup` 装 rust-analyer。

> ⚠注意
>
> 下面的步骤手动设置了 Rust LSP，适用于上面提到的任何安装情况。但如果你的工具链
> 和 rust-analyzer是用同一种方式装的（如**都是** `rustup` 或**都是**系统包管理
> 器），那么实际上工具链和 analyzer 拥有相同的 sysroot，此时可以不必设置下面的
> LSP 配置，而只需要设置 `g:ycm_rust_toolchain_root` 即可，因为 YCM 内置的 Rust
> 支持就是将它们安装到一起的。

装好之后设置 LSP，在上面设置的 `g:ycm_language_server` 列表中添加一项：

```vim
" ...
            \   {
            \       'name': 'rust',
            \       'cmdline': ['rust-analyzer'],
            \       'filetypes': ['rust'],
            \       'project_root_files': ['Cargo.toml'],
            \   },
" ...
let g:ycm_rust_toolchain_root = '/path/to/rust/toolchain/sysroot'
```

上面的 Rust 工具链根目录可以用下面的命令找到：

```sh
rustc --print sysroot
```

将输出原样写进变量里即可。

这样 rust-analyzer 就装好并让 YCM 能认出来了，可以用 VIM 打开个 Cargo 项目并运
行 `:YcmDebugInfo` 查看加载情况。

小目标
------
待续……


[0]: https://github.com/ycm-core/YouCompleteMe
[1]: https://microsoft.github.io/language-server-protocol/
[2]: https://github.com/dense-analysis/ale
[3]: https://github.com/neoclide/coc.nvim
[4]: https://github.com/MaskRay/ccls/
[5]: https://github.com/junegunn/vim-plug/
[6]: https://github.com/VundleVim/Vundle.vim
[7]: https://github.com/MaskRay/ccls/wiki/Project-Setup
[8]: https://rust-analyzer.github.io
