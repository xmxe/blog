---
title: MacOS相关
tags: 随笔
img: https://picx.zhimg.com/70/v2-2f28f965ea1e7a2cd668d5a7f6d38fbb_1440w.avis

---

## mac软件推荐

### [Typora](https://typora.io/)

找到/Applications/Typora.app/Contents/Resources/TypeMark/page-dist/static/js/LicenseIndex.180dd4c7.5dc16d09.chunk.js这一行代码`e.hasActivated = "true" == e.hasActivated`改为`e.hasActivated = "true" == "true"`保存即可。

### [Navicat](https://www.navicat.com.cn/download/navicat-premium)

脚本重置试用日期，项目地址[点这里](https://gitee.com/ProgHub/unlimited_trial_navicat_premium)

```bash
#!/bin/bash
defaults delete com.navicat.NavicatPremium 91F6C435D172C8163E0689D3DAD3F3E9
defaults delete com.navicat.NavicatPremium B966DBD409B87EF577C9BBF3363E9614
rm -rf ~/Library/Application\ Support/PremiumSoft\ CyberTech/Navicat\ CC/Navicat\ Premium/
```

### [Git](https://git-scm.com/download/mac)

源码构建：准备Git依赖库autotools、curl、zlib、openssl、expat和libiconv

> [起步-安装-Git](https://git-scm.com/book/zh/v2/起步-安装-Git)
> 如果报错缺少环境可以使用`brew install autoconf`安装

```bash
tar -zxf git-2.8.0.tar.gz
cd git-2.8.0
make configure
./configure --prefix=/Users/ventura/Downloads/git
make
make prefix=/Users/ventura/Downloads/git install
```

### [CleanMyMac X](https://www.cleanmymac.cn/)

清理垃圾时一直让输密码解决方案：`sudo launchctl load -w /Library/LaunchDaemons/com.macpaw.CleanMyMac.Agent.plist`

### [Command Line Tools](https://developer.apple.com/download/all/)

安装：在终端使用`xcode-select --install`命令

卸载：`sudo rm -rf /Library/Developer/CommandLineTools`

### [Homebrew](https://brew.sh/zh-cn/)

- 安装Homebrew需要先安装[CommandLineTools](https://developer.apple.com/download/all/)
- 通过`brew install`安装软件的路径为`/usr/local/Cellar`然后将下载的可执行二进制程序软连接到`/usr/local/bin`

### [Parallels Desktop](https://www.parallels.cn/products/desktop/)

- 构建arm版windows ISO

1. 访问[uupdump](https://uupdump.net/)，选择要构建的版本后下载压缩文件
2. 阅读read.unix.md文件安装依赖
> 注意：安装依赖可能不成功，解决方案见:https://github.com/sidneys/homebrew-homebrew/issues/2
> 省流版：
>
> ```shell
> # brew tap sidneys/homebrew
> # brew install cabextract wimlib cdrtools sidneys/homebrew/chntpw
> # 使用下面的命令替换上面的命令
> brew install cabextract wimlib cdrtools
> brew tap minacle/chntpw
> brew install minacle/chntpw/chntpw
> brew install aria2
> ```
3. `sh uup_download_macos.sh`或`./uup_download_macos.sh`,如果提示权限不足的话授权命令：`chmod +x uup_download_macos.sh`

- 通过[itellyou](https://next.itellyou.cn/)下载镜像，可通过文件哈希值对比校验
> 查看文件MD5值
>> Windows：`certutil -hashfile 文件路径 MD5`或者`Get-FileHash -Path 文件路径 -Algorithm MD5`
>> Linux：`md5sum 文件路径`
>> MacOS：`md5 文件路径`
>
> 查看文件SHA-1或SHA-256值
>> Windows：`Get-FileHash -Path 文件路径 -Algorithm SHA1` 或者`Get-FileHash -Path 文件路径 -Algorithm SHA256`
>> Linux：`shasum -a 1 文件名`或者`shasum -a 256 文件名`或者`sha1sum 文件名`或者`sha256sum 文件名`
>> MacOS：`shasum -a 1 文件名`或者`shasum -a 256 文件名`或者`openssl dgst -sha1 文件名`或者`openssl dgst -sha256 文件名`

### [Peek](https://www.quicklookplugins.com/)

> quicklook插件安装将下载的.qlgenerator文件移动到~/Library/QuickLook后运行`qlmanage -r`
> `qlmanage -caches`：列出Quick Look缓存中的文件和其相关信息。
> `qlmanage -m plugins`：查看系统中已安装的Quick Look插件，并显示它们支持的文件类型。
> `qlmanage -p 文件路径`：会在Quick Look中打开指定文件的预览。

### [Office](https://macwk.cn/2141.html)

|                             /_ \                             |                            (T_T)                             |                             X﹏X                             |
| :----------------------------------------------------------: | :----------------------------------------------------------: | :----------------------------------------------------------: |
| [Microsoft_Word_Installer.pkg](https://go.microsoft.com/fwlink/?linkid=525134) | [Microsoft_Excel_Installer.pkg](https://go.microsoft.com/fwlink/?linkid=525135) | [Microsoft_PowerPoint_Installer.pkg](https://go.microsoft.com/fwlink/?linkid=525136) |
| [Microsoft_Outlook_Installer.pkg](https://go.microsoft.com/fwlink/?linkid=525137) | [Microsoft_Remote_Desktop_installer.pkg](https://go.microsoft.com/fwlink/?linkid=868963) | [Microsoft_AutoUpdate_Updater.pkg](https://go.microsoft.com/fwlink/?linkid=830196) |
| [Microsoft_Office_License_Removal.pkg](https://go.microsoft.com/fwlink/?linkid=849815) | [Microsoft 365 and Office.pkg](https://go.microsoft.com/fwlink/?linkid=525133) | [Microsoft 365 and Office_BusinessPro.pkg](https://go.microsoft.com/fwlink/?linkid=2009112) |


### 其他

|                           (●'◡'●)                           |                           (●ˇ∀ˇ●)                            |                        (✿◡‿◡)                         |                          ( $ _ $ )                           |
| :---------------------------------------------------------: | :----------------------------------------------------------: | :---------------------------------------------------: | :----------------------------------------------------------: |
|     [Bandizip](https://www.bandisoft.com/bandizip.mac/)     |       [Betterzip](https://macitbetter.com/downloads/)        |          [Termius](https://www.termius.com/)          | [Chrome](https://google.cn/chrome?standalone=1&platform=mac) |
|  [Clash X](https://github.com/yichengchen/clashX/releases)  |         [iShot](https://www.better365.cn/apps.html)          |    [超级右键](https://www.better365.cn/apps.html)     | [右键助手专业版](https://apps.apple.com/cn/app/%E5%8F%B3%E9%94%AE-%E4%B8%93%E4%B8%9A%E7%89%88/id6473515689) |
|                [Motrix](https://motrix.app/)                |               [Downie](https://www.downie.cn/)               | [App Cleaner & Uninstaller](https://app-cleaner.com/) |          [NodeJS](https://nodejs.org/en/download/)           |
| [Java](https://www.oracle.com/java/technologies/downloads/) |      [Python](https://www.python.org/downloads/macos/)       |   [VS Code](https://code.visualstudio.com/Download)   |     [Sublime Text](https://www.sublimetext.com/download)     |
|      [Intellij IDEA](https://www.jetbrains.com/idea/)       | [微信开发者工具](https://developers.weixin.qq.com/miniprogram/dev/devtools/download.html) |    [MySQL](https://dev.mysql.com/downloads/mysql/)    |                   [IINA](https://iina.io/)                   |
|                [Fliqlo](https://fliqlo.com/)                |    [Mac EveryThing](https://tool.atomtech.top/index.html)    |   [ProFind](https://www.zeroonetwenty.com/profind/)   |            [NDM](https://neatdownloadmanager.com)            |


## mac安装软件

- 打开任何来源命令: `sudo spctl --master-disable`
> 关闭任何来源命令: `sudo spctl --master-enable`

- 移除应用的安全隔离属性: `sudo xattr -rd com.apple.quarantine /Applications/*.app`
> 命令用于递归删除指定目录中的全部文件的“quarantine”扩展属性。在macOS系统中，当你从网络或其他未知来源下载并打开文件时，系统会将该文件标记为“quarantine”，以防止其潜在的安全风险。然而有时候，这种标记可能会对某些文件操作造成限制，例如某些脚本文件不能被执行。因此，当使用`xattr -r -d com.apple.quarantine`命令时，系统会删除指定目录中所有文件的“quarantine”扩展属性，以便您可以自由地使用这些文件。该命令的具体含义如下：
> - xattr：用于查看、设置和删除扩展属性的命令。
> - r：表示以递归方式处理目录及其子目录中的文件。
> - d：表示删除指定文件的扩展属性。com.apple.quarantine：是指要删除的扩展属性的名称，即“quarantine”。

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

> 注意：检查mac的shell是zsh还是bash，新系统默认是zsh。zsh读取的配置文件是`~/.zshrc`，bash读取的配置文件是`~/.bash_profile`。所以当使用`source ~/.bash_profile`时退出终端再打开会导致环境变量失效，此时要么在`~/.zshrc`文件编辑同样的内容后执行`source ~/.zshrc`，要么在`~/.zshrc`文件里面添加`source ~/.bash_profile`内容

## 更换app图标

在[这里](https://macosicons.com/)选取图标，拖动网页中的图标到“显示简介”界面左上角旧图标处即可

## 允许远程登陆

- 通用-共享-打开远程登录

- 授权sftp软件访问目录：隐私-完全磁盘访问权限-sshd

## .fseventsd，.Spotlight-V100，.Trashes

这三个文件夹（.fseventsd，.Spotlight-V100，.Trashes）是Mac OS X系统生成的隐藏文件，用于存储系统的临时数据和缓存。通常，它们不应该被手动删除，因为它们对系统的正常运行非常重要。但是，如果你确实需要删除它们，可以尝试使用命令行或者第三方软件来进行操作。

1. .fseventsd文件夹：这个文件夹是Mac OS X的File System Events框架使用的，它用于跟踪文件和目录的更改。这个框架是Mac OS X的核心部分，很多应用程序都依赖于它，因此，手动删除这个文件夹可能会导致系统不稳定或者应用程序出错。

2. .Spotlight-V100文件夹：Spotlight是Mac OS X的内置搜索引擎，.Spotlight-V100文件夹是Spotlight用来存储索引数据的。如果你删除了这个文件夹，Spotlight将无法正常工作，你可能会在搜索文件时遇到问题。

3. .Trashes文件夹：这个文件夹用于存储每个用户在废纸篓中删除的文件。每个用户都有自己的.Trashes子文件夹，这些子文件夹被链接到用户的废纸篓。手动删除这个文件夹可能会导致你无法正常使用废纸篓，或者丢失已删除的文件。

## mac制作U盘启动盘

**Windows**

1. 准备U盘和ISO文件
> 如果U盘无法正常读取且提示无法格式化，windows上的解决办法
> ```bash
> list disk
> select disk n
> clean
> create partition primary
> # 如果要格式化为NTFS，将fat32替换为ntfs或exFAT
> format fs=ntfs quick
> ```

2. 插入U盘后打开终端，输入`diskutil list`查看是否有U盘相关信息
3. 格式化U盘并重命名：`diskutil eraseDisk NTFS "WIN7" GPT /dev/disk2`,其中”WIN7“为U盘重命名，GPT为磁盘格式，可选为MBR, /dev/disk2为U盘路径，根据实际情况填写
4. `cp -rp /Volumes/IR5_CCSA_X86FRE_ZH-CN_DV9/* /Volumes/WIN7/`,将ISO文件拷贝到U盘。其中IR5_CCSA_X86FRE_ZH-CN_DV9为打开ISO后的磁盘映像。
> `rsync -avh --progress --exclude=sources/install.wim /Volumes/CCCOMA_X64FRE_EN-CN_DV9/ /Volumes/WINDOWS11`以归档模式同步文件，并显示详细进度信息，但排除名为install.wim的文件。
>> rsync和cp都是用于文件复制的命令，但它们之间有几个关键的区别：
>> **功能**：cp命令用于简单的文件复制，它将源文件复制到目标位置，但不会保留额外的元数据或权限信息。rsync命令不仅可以复制文件，还可以实现文件同步。它可以在本地文件系统之间，或者通过网络，同步文件和目录，并保留文件的属性、权限等。
>> **同步能力**：cp命令复制文件时，如果目标文件已经存在，它会直接覆盖目标文件。rsync命令能够进行增量同步，只复制发生变化的部分，因此效率更高。它还可以在复制过程中实时显示进度，方便监控和管理。
>> **网络传输**：cp命令只能在本地文件系统内复制文件，不能在网络上进行文件同步。rsync命令可以在本地和远程系统之间同步文件，因此适用于在不同主机之间同步文件或备份数据。

**Ubuntu**
1. 用hdiutil将ISO转dmg
```bash
hdiutil convert -format UDRW -o /Users/Downloads/ubuntu /Users/Downloads/ubuntu-22.04.1-live-server.iso
```
2. 查看U盘名称
```bash
diskutil list
```
3. 解除U盘占用
```bash
diskutil unmountDisk /dev/disk2
```
4. 将镜像写入U盘(带r加快速度)
```bash
sudo dd if=/Users/Downloads/ubuntu.dmg of=/dev/rdisk2 bs=1m 
```
5. 弹出U盘
```bash
diskutil eject /dev/disk2
```

**Ubuntu系统制作Windows启动盘**

1. 打开磁盘工具，格式化U盘(NTFS)，并记录U盘设备名称`/dev/sdb`
1. 下载[woeusb.bash](https://github.com/WoeUSB/WoeUSB/releases/)并授权`chmod +x woeusb-5.2.4.bash`
2. `apt install wimtools`
3. `bash woeusb-5.2.4.bash --device win7.iso /dev/sdb --target-filesystem NTFS`


## 其他

- [Mac-list](https://github.com/qianguyihao/Mac-list)
- [awesome-mac](https://github.com/jaywcjlove/awesome-mac)