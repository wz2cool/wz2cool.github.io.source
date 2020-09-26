---
title: 解决老版本virual studio 不能使用 nuget
date: 2020-09-26 18:11:53
tags:
  - C#
  - virual studio
---

# 前言

很久不写 C# 了，最新想写一个 WPF 小程序，需要下载第三方包，自己用的是 vs2013, 发现各种下载不了，去网上一看原来是 nuget 不再支持 v2 版，而且最要命的是 visual studio 2013 还没有一个地方专门升级 nuget packager 这个插件，现在要用的话需要升级到 vs2015 以上，真的是很坑，经过多方研究发现其实 nuget 还可以单独下一个 nuget.exe 来完成，现在看一下我们如何绕过这个坑吧。

## 下载nuget.exe
去nuget官网直接下载最新即可， 这里我给上一个连接：https://dist.nuget.org/win-x86-commandline/latest/nuget.exe

## 代理配置

国内容易被墙，还有下载慢的问题，所以这里我们可以用 cnblogs 的代理： https://api.nuget.org/v3/index.json ，可以在 tools -> optoins 里面搜索 nuget 修改 Package Sources, 下面还有个 local 的，那个是本地仓库, 设置结果如图：
![nuget-repros](https://i.niupic.com/images/2020/09/26/8JlN.png)

## 添加本地仓库

这里我们还要加个本地源，一般为 nuget 下载的地址，可以在以后 nuget 命令下载中可以看到保存的地址，加的方法同上， 其实以后 visual stuido 用的就是这个本地仓库
![nuget-cmd](https://i.niupic.com/images/2020/09/26/8JlL.png)

## 测试

现在我们就可以做测试了

1. 先用 nuget.exe 把文件下下来，比如上图，我们下载了一个 MvvmLight

```bash
$ nuget.exe install MvvmLight
```

2. 打开 visual studio 中 nuget packager 选择 local, 我们就可以看到有东西了  
![nuget-vs](https://i.niupic.com/images/2020/09/26/8JlO.png)

# 结束

其实我们拿 nuget.exe 做了一个下载包的中间步骤来绕过这个问题
