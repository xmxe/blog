### 通过GitHub Pages构建博客

> 引言

>> 构建自己的博客方式有很多种，网上也有很多开源的博客模板，但有的属于动态模板，还有自己的博客后台管理系统，需要关系数据库、缓存、服务器等环境支持，个人还是选择了GitHub Pages构建静态博客，写文章的话直接在md里写完推送，不需要数据库等环境支持。采用了[Hexo](https://hexo.io/zh-cn/)来构建博客。

#### 一、 环境准备

Git、GitHub Pages、NodeJS 


#### 二、 安装Hexo

```
mkdir hexo
cd hexo
nmp install hexo // 安装hexo
npx hexo init blog // blog为自定义目录
cd blog
npm install // 安装模板依赖
npx hexo g // 生成静态文件。-d, --deploy文件生成后立即部署网站
npm hexo server // 启动服务器。默认情况下，访问网址为： http://localhost:4000/
```

#### 三、 更换主题

- [hexo主题](https://hexo.io/themes/)

下载主题后放置在themes路径下,并修改 _config.yml 内的 theme 设定，即可切换主题


#### 四、部署

将 public 目录中的文件和目录推送至 GitHub 仓库

#### 五、 发布文章

```
npx hexo new post 测试文章 // 在source/_posts下生成文章 如修改_config.yml:post_asset_folder: true 打开这个配置是为了在生成文章的时候生成一个同名的资源目录用于存放图片文件。
也可以直接在source/_posts下新建md文件 里面内容写上
---
title: {title}
date: {date}
tags:
---
即可

Hexo 有三种默认布局：post、page 和 draft。在创建这三种不同类型的文件时，它们将会被保存到不同的路径；而您自定义的其他布局和 post 相同，都将储存到 source/_posts 文件夹。
布局	路径
post	source/_posts
page	source
draft	source/_drafts


```


#### 参考

- [GitHub Pages + Hexo搭建个人博客网站，史上最全教程](https://blog.csdn.net/yaorongke/article/details/119089190)
- [Hexo文档](https://hexo.io/zh-cn/docs/)


