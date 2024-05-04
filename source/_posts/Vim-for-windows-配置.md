---
title: Windows 下使用 Vim
date: 
tags: 
- vim
- git bash
---

## 简要说明

主要针对 ACM/ICPC 竞赛选手在 Windows 10 系统下使用 vim 编写 C/C++ 代码。

功能配置：

1. 编译和运行 *.cpp 文件；
2. 一键复制代码;
3. 记事本打开代码。

{% asset_img 1539090544618.png 效果图 %}

git bash 和 gvim 都配置了一遍。gvim 使用 Windows 自带的 cmd 运行的话，鼠标是没办法移动光标的，并且配置相对 git bash 较麻烦，所以推荐使用 git bash。

## Vimrc 配置

- 编辑安装路径下的 vimrc 文件，例如 "D:\Git\etc\vimrc"，配置快捷键。

  ```shell
  set nu ai ci si mouse=a ts=4 sts=4 sw=4
  nmap<F2> :vs %<.in <CR>
  nmap<F3> :w !clip.exe <CR> <CR>
  nmap<F4> :!write % <CR>
  nmap<F8> :!./%< < %<.in <CR>
  nmap<F9> :!g++ % -o %< -O2 -g -Wall -std=c++11 <CR>
  
  ```


## 额外配置

- vimrc 文件默认有一些配置，可根据需要修改。

  ```shell
  set vb             "这个不关的话，触发某些条件会闪屏
  set laststatus=1   "窗口底部状态栏的行数（默认是2），这里设置成1。
  au FileType c,cpp setlocal comments-=:// comments+=f:// "取消换行自动注释
  ```
  
- 设置 Ctrl + Alt + T 快捷键启动 Git Bash 终端。

  1. 创建 "git-bash.exe" 的快捷方式，打开快捷方式的属性窗口，修改**起始位置**，并设置快捷键。
  2. 打开**控制面板**，找到 “管理工具”， 将快捷方式复制到该文件夹中。

