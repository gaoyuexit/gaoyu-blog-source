---
title: 博客折腾记录
author: Gao先生
toc: false
top: false
mathjax: false
comments: true
copyright: true
categories: Blog
tags:
  - Blog
  - Hexo
  - Next
date: 2018-03-27 23:36:15
updated: 2018-03-27 23:36:15
---

\> *最近又折腾了一下, 更换了主题*
<!-- more -->

## 主题

### 配置主题文件

根据 Hexo 官方的推荐，不要直接修改主题的配置文件

> The file should be placed in your site folder, both yml and json are supported. theme inside  _config.yml must be configured for Hexo to read _config.[theme].yml

1. 配置主题为 Next 主题

   ```yaml
   # _config.yml
   theme: "next"
   ```

2. 在 site 根目录新建 `_config.next.yml`文件

   ```shell
   $ touch _config.next.yml
   ```

3. 打开 `theme/next/_config.yml`把需要修改的选项 copy 到 `_config.next.yml`中

   ```yaml
   # _config.next.yml
   # 设置网站图标
   favicon:
     small: /favicon.ico
     medium: /favicon.ico
     apple_touch_icon: /apple-touch-icon.png
     safari_pinned_tab: /apple-touch-icon.png
   ```

4. hexo启动时，会自动合并 `_config.next.yml` 与`theme/next/_config.yml` 的设置内容，从而达到配置主题的作用。

> 这样我们可以不直接修改theme/next中的内容, 每次拉取theme/next仓库中的更新时, 不会因修改了`_config.yml`而发生冲突。（每个主题都是一个独立的 `git`项目）



### 修改主题样式

>  Hexo的NexT主题采用njk来作为HTML预处理器，使用styl来扩展css，所以可以简单的理解成 (html \ subset njk) ，(css \ subset styl)。它们扩充了相应的功能和语法支持来更加高效的架构网页，当然，我们也完全可以使用html和css的语法来美化我们的网页。

因为每个主题都是一个独立的 `git`项目, 每个主题的安装大都为

```shell
$ git clone https://github.com/theme-next/hexo-theme-next themes/next
```

一般我们都会更改自己的主题,不推荐直接在themes/next仓库内直接进行修改, 这样在拉取最新的主题更新时, 我们的修改会被覆盖/冲突。

#### 1. 指定custom_file_path来修改

更改主题的方式hexo可以使用指定custom_file_path的样式文件styles.styl来进行样式覆盖, 比如head.njk文件来进行预处理

```yaml
custom_file_path:
  head: source/_data/head.njk
  #header: source/_data/header.njk
  #sidebar: source/_data/sidebar.njk
  #postMeta: source/_data/post-meta.njk
  postBodyEnd: source/_data/post-body-end.njk
  #footer: source/_data/footer.njk
  bodyEnd: source/_data/body-end.njk
  variable: source/_data/variables.styl
  mixin: source/_data/mixins.styl
  style: source/_data/styles.styl
```

比如指定的source/_data/styles.styl来进行覆盖默认样式

```css
/* header-页面背景 */
body {
  background-image: url($body-bg-url);
  background-repeat: repeat;
  background-attachment: fixed;

  @media (prefers-color-scheme: dark) {
    background-image: none;
  }
}

/* sidebar-标题 */
.site-title {
  font-weight: bold;
}
```

#### 2. 使用submodule管理主题

1. 在GitHub上，把原作者的主题fork到我们自己的仓库中
2. 运行以下命令，把fork的NexT主题添加为我们博客的子模块

```shell
$ git submodule add https://github.com/<username>/hexo-theme-next themes/next
```

我们可以把自己修改的主题提交到我们自己fork的仓库中进行存储

submodule命令会在目录下多一个`.gitmodules`文件

```shell
$ cat .gitmodules
[submodule "themes/next"]
    path = themes/next
    url = git@github.com:gaoyuexit/hexo-theme-next
```

如何进行版本控制 ? 

1.推送至 GitHub

当我们对主题（子模块）的内容进行修改时，Hexo 站点（主模块）只知道我们进行了修改，但是并不会管理我们具体进行了什么修改。

> 因此，子模块和主模块需要分开 push，一般先推送子模块，后推送主模块。

2.从原作者处更新主题

如果使用的主题仍在被维护，那么我们就能从远程获取更新:

```shell
$ git submodule update --remote
```

3.在另一台电脑上工作

```shell
$ git clone --recursive <your repo address>
```

或者 (👆 和 👇 的命令功能是一样的)

```shell
$ git clone <your repo address>
$ git submodule update --init
```

4.删除submodule

```shell
$ git submodule deinit <submodule-name>
```



## 看板娘

### 简易版**[hexo-helper-live2d](https://github.com/EYHN/hexo-helper-live2d)**

### 功能版**[live2d-widget](https://github.com/stevenjoezhang/live2d-widget)**

以[live2d-widget](https://github.com/stevenjoezhang/live2d-widget)为例, 先fork项目到自己的github仓库, 方便后续自己定制

在/themes/next/layout/_layout.swig的`head`标签内添加如下代码

```html
<head>
  ...
  <script src="https://cdn.jsdelivr.net/gh/gaoyuexit/live2d-widget@latest/autoload.js"></script>
  <link rel="stylesheet" href="https://cdn.jsdelivr.net/npm/font-awesome/css/font-awesome.min.css">
</head>
```



## *Pjax*

**单页应用**（英语：single-page application，缩写**SPA**）是一种网络应用程序或网站的模型，它通过动态重写当前页面来与用户交互，而非传统的从服务器重新加载整个新页面。这种方法避免了页面之间切换打断用户体验，使应用程序更像一个桌面应用程序。**与单页应用的交互通常涉及到与网页服务器后端的动态通信**。说白了也就是通过 `pushState() + XHR` 技术实现的页面切换。

而 Hexo 站点本身是个静态页面，无法发出动态请求，所以这里便引出了本文的主角 Pjax 框架了。其思路是通过拦截 `a` 链接，发送 XHR 请求，获取下级页面内容，接着替换指定区域完成整个过程。由于不是动态网站，Pjax 在请求过程中获取的是整个站点的 Html 内容，所以请求本身是无法达到加速的，但是可以减少页面中 JS 文件的重复请求，此外还可以利用一些预加载技术（预读缓存）和磁盘缓存进一步提升访问速度，实际体验效果是极佳的。

我们的页面存在着大量的事件监听处理。不巧的是在替换内容时，会发生部分事件监听丢失了，异常、错误、功能失效等等。

Javascript 部分自然是需要重新绑定注册的，但也不能无脑全部重新绑定, 否则会导致**重复事件监听**，它或许不会立即爆发危害，但是随着浏览页面的增加，事件绑定可能会越来越累加，影响效率，潜在造成错误等等。所以这里就需要利用 `send` 和 `complete` 这对事件了，合理利用，正确解决问题。

比如, 开启我的博客开启*Pjax*属性, 博客会局部刷新了, 加载速度也变快了, 我的音乐播放器可以全局播放了,  但是每个页面的评论消息错乱了或消失了, 只有`command + r`刷新整个页面, 才能加载出来。

解决, 在`/themes/next/layout/_scripts/pjax.swig`目录下, 根据这些函数重新调用加载**拉取评论**的方法

```javascript
document.addEventListener('pjax:send', function () {});
document.addEventListener('pjax:complete', function () {});
document.addEventListener('pjax:error', function () {});
```

因为以后博客的插件越来越多(搜索、评论、懒加载、图片弹出层、代码复制、背景幻灯片、音乐播放器、按钮涟漪动画、页面平滑滚动、公式、不蒜子计数、网址预加载),  这其中，有些无需处理，有些简单的封装成函数重新调用一下即可，而有些则就需要特意分析处理了。

因此暂时将该功能先关闭掉了



## 参考

[Hexo优化汇总](https://qianling.pw/hexo-optimization/)

[深度美化和定制Hexo和NexT方法](https://www.geminilight.cn/2020/08/16/ST%20-%20%E8%BD%AF%E4%BB%B6%E5%B7%A5%E5%85%B7/st-hexo-next-custom/)

[为 Hexo 博客更换主题](https://zhuanlan.zhihu.com/p/149710191)

[Hexo博客部署Pjax局部刷新](https://inkss.cn/blog/80b5f235/)

