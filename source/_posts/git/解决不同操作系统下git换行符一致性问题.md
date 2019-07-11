---
title: 解决不同操作系统下git换行符一致性问题
categories:
- 好好学习
tags:
  - GIT
date: 2018-10-24 20:19:46
---
# 解决不同操作系统下git换行符一致性问题

新的电脑配置环境时，有可能会出现提交代码不能正确增量显示的问题。
那么如何一次性解决不同操作系统下git换行符不一致的问题呢？

<!-- more -->

## 一、不同操系统下的换行符CR回车 LF换行

- Windows/Dos CRLF \r\n
- Linux/Unix LF \n
- MacOS CR \r

## 二、解决方法

打卡git bash，设置core.autocrlf和core.safecrlf（可不设置），建议设置autocrlf为input，safecrlf为true，同时设置你的Eclipse、IDEA等IDE的换行符为LF\n。
下面为参数说明，--global表示全局设置

### 2.1、autocrlf

#提交时转换为LF，检出时转换为CRLF
`git config --global core.autocrlf true`

#**提交时转换为LF，检出时不转换**
`git config --global core.autocrlf input`

#提交检出均不转换
`git config --global core.autocrlf false`

### 2.2、safecrlf

#**拒绝提交包含混合换行符的文件**
`git config --global core.safecrlf true`

#允许提交包含混合换行符的文件
`git config --global core.safecrlf false`

#提交包含混合换行符的文件时给出警告
`git config --global core.safecrlf warn`