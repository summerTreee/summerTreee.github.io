---
layout: post
title: "一台电脑设置多个github账号"
date: 2017-10-16 22:57:23 +0800
comments: true
categories: 
---
#####前言：
很多时候我们需要在一台电脑上配置多个git账号用于不同的项目，例如公司代码管理的git账户以及自己的github账户，下面就简单介绍下具体步骤。

<!-- more -->
##### 账户信息配置
账户信息的配置可以体现在使用git进行项目管理时的个人信息。对于`global`范围的账户信息，可以使用以下操作进行设置或者取消：

```
# 通过以下语句查看设置的账户信息
$ git config --global user.name
$ git config --global user.email

# 取消设置的全局账户
$ git config --global --unset user.name
$ git config --global --unset user.email

# 通过以下语句设置的账户信息
$ git config --global user.name "your name"
$ git config --global user.email "your email"

```
对于不同的项目，如果你需要设置不同的用户信息，可以`cd`到该目录下，进行以上的设置，只是在设置的时候去掉`--global`参数。

##### SSH配置
在个人电脑上使用如下命令可生产`SSH KEY`：

```
$ ssh-keygen -t rsa -C "youremail@email.com" -f ~/.ssh/id_rsa_one

```
这时会在用户根目录下生成一个**.ssh**文件夹，一个私钥：id_rsa_one，一个公钥：id_rsa_one.pub，该公钥和私钥包含了你邮箱的信息，具有随机不可复现性。

有多个`github`账户时，需要进行多次生产SSH-KEY的操作，例如你还有另一个账户：

```
$ ssh-keygen -t rsa -C "youremail_two@email_two.com" -f ~/.ssh/id_rsa_two

```

这时在**.ssh**文件夹下就会存在如下文件：

> id\_rsa\_one  
> id\_rsa\_one.pub  
> id\_rsa\_two  
> id\_rsa\_two.pub  
> known_hosts

然后需要把刚才生产的`.pub`后缀的公钥添加到远程主机中去。

本地需要识别刚才生产的**SSH-KEY**，就需要使用一下命令进行添加：

```
# 使用-K可以将私钥添加到钥匙串，不用每次开机后还要再次输入这条命令
$ ssh-add -K ~/.ssh/id_rsa_one        
$ ssh-add -K ~/.ssh/id_rsa_two

```

可以在添加前使用下面命令删除所有的key:

```
$ ssh-add -D

```

最后可以通过下面命令，查看key的设置：

```
$ ssh-add -l

```
当我们作为以上的操作后就需要修改`.ssh`文件夹下的`config`文件信息，以确保我们不同的项目使用不同的私钥进行操作校验。


```
$ cd ~/.ssh/
# 没有config文件时，用以下命令生成
$ touch config

```
打开`.ssh`文件夹下的`config`文件，进行配置:

```
#githun one
Host one.github.com #这里名称可以随意取，和下面的不一样就行
HostName github.com
User git
IdentityFile ~/.ssh/id_rsa_one

#github two
Host two.github.com
HostName github.com
#Port 7999
User git
IdentityFile ~/.ssh/id_rsa_two

```
需要注意的是User需要设置为`git`。

设置好以上信息后就可以通过以下命令来验证设置是否正确：

```
$ ssh -vT git@one.github.com
# 执行后如果出现信息，就表示验证通过了：
Hi yourname! You've successfully authenticated, but GitHub does not provide shell access.

```

