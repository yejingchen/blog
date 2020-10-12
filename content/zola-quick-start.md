+++
title = "Zola 静态博客快速指南"
date = 2020-02-24 01:36:56+08:00
[taxonomies]
categories = ["正文"]
tags = ["blog"]
+++

仅有一篇文章 WordPress 博客收到了几十份 spam 评论，于是想直接关了换静态网站算了
。说干就干。

<!-- more -->

> 这个博客的源码在 <https://github.com/yejingchen/blog>，可以一起看。

## 安装

首先安装 Zola。Mac 用 brew 装 `zola`，Arch Linux 从 AUR 上装 `zola`。装完了建个
文件夹 `zola init` 一下，高兴的话也 `git init` 一下。

## 初见

`zola init` 完会创建配置文件 `config.toml` 和几个文件夹：

- `content/` 放所有的文章
- `sass/` 如字面意义放 sass
- `templates/` 下面放 Tera 模板，即 Zola 使用的 HTML 模板引擎。
- `content/` 放我们的文章
- `static/` 图片 / js 等静态资源
- `themes/` 主题

这时候直接运行 `zola serve` 可以看到默认页面在告诉你你没有主页和文章的模板。

先在 `config.toml` 里加个

```toml
title = "网站名称"
```

页面应该会自动刷新，网页标题已经变成修改后的内容了。

由于现在没有配置模板或者主题，这里先不写东西，等下面装好 even 主题再说。

## 安装 even 主题

从[官网](https://www.getzola.org/themes/)挑一个看起来顺眼的主题，就决定是 even
了。按说明需要如下操作。

在博客目录下克隆主题到 `themes/even`：

```bash
$ git clone https://github.com/getzola/even.git themes/even
```

克隆好后先编辑博客的 `config.toml`，目前里面只有 `zola init` 过程中回答的以及上面加
的选项。先在`[extra]` 上方加几个选项：

```toml
theme = "even" # 等会要配置的主题
taxonomies = [ # even 主题要求的配置
    # 按自己需要开关 RSS
    {name = "categories", rss = true},
    {name = "tags", rss = true},
]
```

`[extra]` 下方也需要一点配置。

```toml
even_title = "CAIYE 的窝" # even 主题页面左上角
author = "caiye"
```

然后需要指定首页的配置 `content/_index.md`：

```
+++
paginate_by = 5
sort_by = "date"
+++
```

到这里主题就好了！`zola serve` 能看到空空荡荡的 even 主题的页面。

## 一点规则

我们目前只会在`content/` 和 `static/` 下放东西，其他的都靠主题，放在
`themes/` 下。

### 文章层级

`content/` 下的每个 `.md` 文件都是一篇文章，文章的 URL 根据文件在 `content/` 下
的路径决定，如 `content/one/two/three.md` 的路径是 `example.com/one/two/three`。
不过 even 主题要求所有的文章都直接放在 `content` 下，所以这里就不建立额外的目录
层级了。

值得一提的是如果你的某一篇文章需要附带别的文件（比如图片），也可以用把文件换成同
层级的目录，如原本是 `foo.md`：

```
content/
  |- _index.md
  `- foo.md
```

要换成目录方便附带文件的话，需要改成同前缀名、且包含一个 `index.md` 文件的目录：

```
content/
  |- _index.md
  `- foo/
       |- index.md
       `- image.png
```

`index.md` 包含着原来 `foo.md` 的内容。这点和 Rust 自己的模块文件命名规则是一致
的，毕竟 zola 是用 Rust 写的。

### 文章格式

每篇文章都是直接放在 `content` 目录下的 `.md` 文件，文件头部要有由一对 `+++` 包
起来的元信息（叫做 front matter），记录如文章标题、日期、关键字、套用的模板等，
后面就是普通的Markdown，比如这篇文章的开头：

```
+++
title = "Zola 静态博客快速指南"
date = 2020-02-24 01:36:56+08:00
[taxonomies]
categories = ["正文"]
tags = ["blog"]
+++

仅有一篇文章 WordPress 博客收到了几十份 spam 评论，于是想直接关了换静态网站算了
。说干就干。

<!-- more -->

> 这个博客的源码在 <https://github.com/yejingchen/blog>，可以一起看。
```

知道了这些就可以写文章塞进 `content` 里了！

### About 页面

这是介绍自己的页面。照着 even 主题自己的做法，`content/pages/about.md` 的源信息
是这样的：

```
+++
title = "About"
path = "about"
template = "about.html"
+++
```

后面就写点关于自己的内容吧。

### 静态资源

`static/` 下的文件在其他地方以根开头的 URL 访问，比如 `static/favicon.ico` 用
`/favicon.ico`。

## 打包发布

如果上面没出什么错，`zola serve` 自动刷新的页面就能看到目前的效果了。如果打算发
布了，在博客目录顶层运行 `zola build`，生成的 `public` 文件夹就可以放到如 NGINX
下面公开了。大功告成！
