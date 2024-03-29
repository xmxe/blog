---
title: MacOS相关
tags: 随笔
img: https://picx.zhimg.com/70/v2-2f28f965ea1e7a2cd668d5a7f6d38fbb_1440w.avis
---

## mac安装软件

- 打开任何来源命令: `sudo spctl --master-disable`
> 关闭任何来源命令: `sudo spctl --master-enable`

- 移除应用的安全隔离属性: `sudo xattr -rd com.apple.quarantine /Applications/*.app`
> 命令用于递归删除指定目录中的全部文件的“quarantine”扩展属性。在macOS系统中，当你从网络或其他未知来源下载并打开文件时，系统会将该文件标记为“quarantine”，以防止其潜在的安全风险。然而有时候，这种标记可能会对某些文件操作造成限制，例如某些脚本文件不能被执行。因此，当使用`xattr -r -d com.apple.quarantine`命令时，系统会删除指定目录中所有文件的“quarantine”扩展属性，以便您可以自由地使用这些文件。该命令的具体含义如下：
> - xattr：用于查看、设置和删除扩展属性的命令。
> - -r：表示以递归方式处理目录及其子目录中的文件。
> - -d：表示删除指定文件的扩展属性。com.apple.quarantine：是指要删除的扩展属性的名称，即“quarantine”。

- 关闭SIP
1. 关机，然后重新启动你的Mac电脑，在开机时一直按住`Command+R`迸入Recovery模式（m1改为长按电源键，点击选项，选择一个用户进去）
2. 进入Recovery模式后在顶部菜单栏点击:实用工具->终端
3. 在终端上输入以下命令然后回车：`csrutil disable`
> 打开SIP:`csrutil enable`，查看SIP:`csrutil status`。或系统信息-软件-系统完整性保护
4. 点击左上角苹果图标，点击重新启动

## mac显示隐藏文件

快捷键`command+shift+.`。或者在终端输入

```bash
# Mac显示“隐藏文件”命令
defaults write com.apple.finder AppleShowAllFiles -bool true
# Mac隐藏“隐藏文件”命令
defaults write com.apple.finder AppleShowAllFiles -bool false
```
之后`killall Finder`重启访达进程

## mac设置环境变量

创建`~/.bash_profile`文件。编辑java环境变量

```bash
export JAVA_HOME=/Library/Java/JavaVirtualMachines/jdk1.8.0_131.jdk/Contents/Home
export PATH=$PATH:$JAVA_HOME/bin
# 终端执行命令使环境变量生效
source ~/.bash_profile
```

> 注意: 检查mac的shell是zsh还是bash，新系统默认是zsh。zsh读取的配置文件是`~/.zshrc`，bash读取的配置文件是`~/.bash_profile`。所以当使用`source ~/.bash_profile`时退出终端再打开会导致环境变量失效，此时要么在`~/.zshrc`文件编辑同样的内容后执行`source ~/.zshrc`，要么在`~/.zshrc`文件里面添加`source ~/.bash_profile`内容

## typora

找到`/Applications/Typora.app/Contents/Resources/TypeMark/page-dist/static/js/LicenseIndex.180dd4c7.5dc16d09.chunk.js`这一行代码`e.hasActivated = "true" == e.hasActivated`改为`e.hasActivated = "true" == "true"`保存即可。

## Navicat

脚本重置试用日期，项目地址[点这里](https://gitee.com/ProgHub/unlimited_trial_navicat_premium)

```bash
#!/bin/bash
defaults delete com.navicat.NavicatPremium 91F6C435D172C8163E0689D3DAD3F3E9
defaults delete com.navicat.NavicatPremium B966DBD409B87EF577C9BBF3363E9614
rm -rf ~/Library/Application\ Support/PremiumSoft\ CyberTech/Navicat\ CC/Navicat\ Premium/
```

## 源码构建git

准备git依赖库autotools、curl、zlib、openssl、expat和libiconv

> [1.起步-安装-Git](https://git-scm.com/book/zh/v2/起步-安装-Git) [2.download/mac](https://git-scm.com/download/mac)
> 如果报错缺少环境可以使用`brew install autoconf`安装


```bash
tar -zxf git-2.8.0.tar.gz
cd git-2.8.0
make configure
./configure --prefix=/Users/ventura/Downloads/git
make
make prefix=/Users/ventura/Downloads/git install
```

## clean my mac x

清理垃圾时一直让输密码解决方案: `sudo launchctl load -w /Library/LaunchDaemons/com.macpaw.CleanMyMac.Agent.plist`

## 更换app图标

在[这里](https://macosicons.com/)选取图标，拖动网页中的图标到“显示简介”界面左上角旧图标处即可

## 允许远程登陆

通用-共享-打开远程登录,授权sftp软件访问目录：隐私-完全磁盘访问权限-sshd

## command line tools

访问[这里](https://developer.apple.com/download/all/)下载

## 盘里多了.fseventsd，.Spotlight-V100，.Trashes三个文件夹

这三个文件夹（.fseventsd，.Spotlight-V100，.Trashes）是Mac OS X系统生成的隐藏文件，用于存储系统的临时数据和缓存。通常，它们不应该被手动删除，因为它们对系统的正常运行非常重要。但是，如果你确实需要删除它们，可以尝试使用命令行或者第三方软件来进行操作。

1. .fseventsd文件夹：这个文件夹是Mac OS X的File System Events框架使用的，它用于跟踪文件和目录的更改。这个框架是Mac OS X的核心部分，很多应用程序都依赖于它，因此，手动删除这个文件夹可能会导致系统不稳定或者应用程序出错。

2. .Spotlight-V100文件夹：Spotlight是Mac OS X的内置搜索引擎，.Spotlight-V100文件夹是Spotlight用来存储索引数据的。如果你删除了这个文件夹，Spotlight将无法正常工作，你可能会在搜索文件时遇到问题。

3. .Trashes文件夹：这个文件夹用于存储每个用户在废纸篓中删除的文件。每个用户都有自己的.Trashes子文件夹，这些子文件夹被链接到用户的废纸篓。手动删除这个文件夹可能会导致你无法正常使用废纸篓，或者丢失已删除的文件。

## 其他

- [离线chrome](https://google.cn/chrome?standalone=1&platform=mac)
- [Mac-list](https://github.com/qianguyihao/Mac-list)
- [awesome-mac](https://github.com/jaywcjlove/awesome-mac)