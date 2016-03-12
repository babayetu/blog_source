---
title: vscode_c++
date: 2016-03-12 15:57:55
tags: [vscode,cpp,gnu,global]
categories: [cpp,IDE]
---

**把Visual Studio Code配置成C++编辑器**

* 下载获取最新的Visual Studio Code

* ⌘P打开中央命令行

* ext install在提示中输入[C++ Intellisense](https://code.visualstudio.com/docs/editor/extension-gallery?pub=austin&ext=code-gnu-global)会有提示
![](http://7xrspw.com1.z0.glb.clouddn.com/c%2B%2BIntellisense.png)

* 安装boost
```sh
$brew install boost
```

* 安装[GNU global](https://code.visualstudio.com/docs/editor/extension-gallery?pub=austin&ext=code-gnu-global)并构建tag数据库，包括boost和标准C++库

```sh
$cd /Users/liujingjing/c++_repo/MiniDB
$export GTAGSFORCECPP=
$brew install global
$more /usr/local/share/gtags/FAQ
$cd 
$ln -s /usr/local/include/boost .
$ln -s /usr/local/include .
$gtags
```

* 如果不确定mac下面C++需要include的目录
```sh
$echo "" | gcc -xc - -v -E
```

* Visual Code打开文件[MiniDB](https://github.com/halfvim/MiniDB)
在VSCode里面Open ~/c++_repo/MiniDB/src,
点击某个文件，可以看到右键有Go to Definition和Go to Reference
![](http://7xrspw.com1.z0.glb.clouddn.com/vscode2.png)

* 特殊符号存起来备用, [来源](https://www.v2ex.com/t/43873)
```sh
↵ ⇥ → ← ↓ ↑ ⎋ ⇧ ® © ⌫ ⌃ ⌥ ⌘ 
```