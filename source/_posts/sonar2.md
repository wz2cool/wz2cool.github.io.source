---
title: SonarQube入门（二）：创建项目
date: 2019-03-18 22:33:36
tags:
  - sonarqube
  - DevOps
---

# 前言

上一章我们我们建立好 [SonarQube 入门（一）：Docker 版本安装](https://wz2cool.github.io/2019/02/16/sonar1/), 这里我们简单介绍一下如何创建一个项目吧，这个每次我找都要找半天，方便自己记录一下啊。

# 步骤

1. 到右上角点击我的账号  
   ![sonarMyAccount](https://raw.githubusercontent.com/wz2cool/markdownPhotos/master/res/sonarMyAccount.png)
2. 点击安全选项卡  
   ![sonarMyAccountSecurity](https://raw.githubusercontent.com/wz2cool/markdownPhotos/master/res/sonarMyAccountSecurity.png)
3. 填项目名字，并生成 token， 这里我填写的是 test-project  
   ![sonarGenerateProject](https://raw.githubusercontent.com/wz2cool/markdownPhotos/master/res/sonarGenerateProject.png)
4. 复制 token 去跑一下 maven 项目,login 参数就是刚才的 token
   ```java
    mvn --batch-mode verify sonar:sonar -Dsonar.host.url=http://192.168.8.135:9000 -Dsonar.login=952c8b7b7c09a8fa9a24c3474420b598e370dff9
   ```
