---
title: 年后开工
author: Gao先生
toc: false
top: false
mathjax: false
comments: true
copyright: true
categories: Live
tags:
  - Live
date: 2018-2-25 15:36:41
updated: 2018-2-25 15:36:41
---

## 新博客
以后在这里更新博客了, 在这里记录一些开发经验和一些生活点滴, 以前的文章点击[这里](https://www.jianshu.com/u/c4a4505bef4f), 恩, 就酱...

<!-- more -->

## 新Mac
年假马上过去了, 年后开工, 买台新电脑犒劳一下自己. `2w`大洋, 肉疼~~ 
不多说, 赶紧配置起来 ^_^

## 安装必备软件
1. 输入法: 搜狗输入法
2. Shadowsocks-NG

    > 这个不多说, 必备
3. Chrome浏览器 
    > 安装`JSON Viewer`插件, json格式化工具
4. Alfred3 
    > 效率神器 配置所在的路径`~/Library/Application Support/Alfred 2/Alfred.alfredpreferences`, 从老电脑拷贝到新电脑[Alfred在不同电脑上同步自定义设置](https://www.zhihu.com/question/39098799)
5. iTerm 
    > 配合`ZSH` 个性化设置
6. CleanMyMac 3
    > 清理MAC
7. Teamviewer
    > 远程协助
8. Charles
    > 抓包工具
9. Sketch
    > UI设计必备
10. Xcode
    > iOS开发必备
11. 微信开发者工具 
12. Sourcetree
    > git可视化工具 
13. Phpstorm [激活](http://blog.csdn.net/voke_/article/details/78794567)
14. Reveal
    > iPhone app 层级查看
15. Mweb 
    > 心仪的`Markdown`编辑器
16. Sequel Pro 
    > 数据库工具
17. LICEcap /  Snip
18. ThemeEngine 
    > 提取iOS assets.car中的图片
19. PaintCode 
20. XMind
21. (Be Focused Pro)番茄闹钟
22. WWDC
24. 印象笔记
25. 播放器 Blu-ray Player Pro
26. Dropbox 配置代理服务器

## 配置环境

1. 配置SSH [多台电脑公用一个SSH](http://www.cnblogs.com/ayseeing/p/4646292.html)

    > 拷贝旧电脑的 .ssh 到新电脑中

2. 安装`Homebrew`和`wget` [步骤](https://brew.sh)
3. 安装`npm` [步骤](https://www.npmjs.com.cn/getting-started/installing-node/)
4. 安装`mysql` [dmg安装](https://dev.mysql.com/downloads/mysql/) / [brew安装](https://aaaaaashu.gitbooks.io/mac-dev-setup/content/MySql/index.html)
5. 安装`Composer` [步骤](http://docs.phpcomposer.com/00-intro.html)
6. 安装`laravel` [步骤](https://d.laravel-china.org/docs/5.5/installation)
7. 安装`cocopods` [步骤](https://guides.cocoapods.org/using/getting-started.html)  意外发现根目录下的.cocopods仓库中的目录和以前版本不同了
8. 导出钥匙串中的证书/p12到新电脑中
9. 同步备忘录中的文章信息

## 配置iterm2/ zsh 参考
https://segmentfault.com/a/1190000009919714
https://segmentfault.com/a/1190000010518195
https://laoshuterry.gitbooks.io/mac_os_setup_guide/content/4_ZshConfig.html
ZSH的主题参考 https://github.com/robbyrussell/oh-my-zsh/wiki/Themes

> bash和zsh互相切换

- 切换bash

```
chsh -s /bin/bash
```

- 切换zsh

```
chsh -s /bin/zsh
```


## 关于环境变量
[环境变量](http://blog.csdn.net/u010416101/article/details/54618621)
[MAC设置环境变量path的几种方法](https://www.cnblogs.com/shineqiujuan/p/4693404.html)


1）/etc/paths （全局建议修改这个文件 ）
编辑 paths，将环境变量添加到 paths文件中 ，一行一个路径
Hint：输入环境变量时，不用一个一个地输入，只要拖动文件夹到 Terminal 里就可以了。

2）/etc/profile （建议不修改这个文件 ）
全局（公有）配置，不管是哪个用户，登录时都会读取该文件。

3）/etc/bashrc （一般在这个文件中添加系统级环境变量）
全局（公有）配置，bash shell执行时，不管是何种方式，都会读取此文件。

4）
1.创建一个文件：
sudo touch /etc/paths.d/mysql
2.用 vim 打开这个文件（如果是以 open -t 的方式打开，则不允许编辑）：
sudo vim /etc/paths.d/mysql
3.编辑该文件，键入路径并保存（关闭该 Terminal 窗口并重新打开一个，就能使用 mysql 命令了）
/usr/local/mysql/bin
这样可以自己生成新的文件，不用把变量全都放到 paths 一个文件里，方便管理。

> .d文件不常见，有可能是某个应用自己特有的文件，.d也可能是default的意思,表示默认（配置）文件，还有可能是Dynamic的意思，表示动态意义的文件。
带.d的文件夹比较常见，比如/etc/rc3.d、/etc/dev.d表示文件加下有系统缺省的配置文件。
    

profile、bash_profile、bashrc文件的作用与区别参考
https://itbilu.com/linux/management/NyI9cjipl.html
https://jingyan.baidu.com/article/f25ef254a8f4a6482d1b8261.html
