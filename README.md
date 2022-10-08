### 通过GitHub Pages构建博客

> 引言

>> 构建自己的博客方式有很多种，网上也有很多开源的博客模板，但有的属于动态模板，还有自己的博客后台管理系统，需要关系数据库、缓存、服务器等环境支持，个人还是选择了GitHub Pages构建静态博客，写文章的话直接在md里写完推送，不需要数据库等环境支持。采用了[Hexo](https://hexo.io/zh-cn/)来构建博客。

#### 一、 环境准备

Git、GitHub Pages、NodeJS 

#### 二、 安装Hexo

```
全局安装 npm install hexo-cli -g 或者单独安装 nmp install hexo // 安装hexo
npx hexo init <folder> // <folder>为自定义目录，如npx hexo init blog
cd <folder> // cd blog
npm install // 安装模板依赖
npx hexo generate // 简写为npx hexo g 生成静态文件。-d, --deploy文件生成后立即部署网站
npm hexo server // 简写为 npx hexo s 启动服务器。默认情况下，访问网址为:http://localhost:4000/
```

#### 三、 更换主题

下载[hexo主题](https://hexo.io/themes/)后放置在themes路径下,并修改 _config.yml 内的 theme 设定，即可切换主题

推荐两款主题[fluid](https://github.com/fluid-dev/hexo-theme-fluid)、[matery](https://blinkfox.github.io/)

##### fluid
- [Hexo-Fluid主题美化](https://blog.csdn.net/weixin_43471926/article/details/109798811)
- [fluid文档](https://hexo.fluid-dev.com/docs/start)

##### matery
- [hexo-theme-matery github](https://github.com/blinkfox/hexo-theme-matery)
- [Hexo-Matery主题自定义(共六篇)](https://blog.csdn.net/weixin_43662760/category_11551976.html)
- [基于Hexo的matery主题搭建博客自定义修改篇2](https://blog.17lai.site/posts/4d8a0b22/)
- [Hexo+Matery访问速度优化](https://blog.sky03.cn/posts/42790.html)


#### 四、部署

使用**npx hexo g**生成public目录,将public目录推送至GitHub仓库即可。

[其他方式](https://hexo.io/zh-cn/docs/github-pages)

#### 五、 发布文章

```
npx hexo new [layout] <title> // 例如npx hexo new post text相当于在source/_posts下生成text.md文件
您可以在命令中指定文章的布局（layout），默认为 post，可以通过修改 _config.yml 中的 default_layout 参数来指定默认布局。
Hexo 有三种默认布局：post、page 和 draft。在创建这三种不同类型的文件时，它们将会被保存到不同的路径；而您自定义的其他布局和 post 相同，都将储存到 source/_posts 文件夹。
布局	路径
post source/_posts
page source
draft source/_drafts

如修改_config.yml:post_asset_folder: true 打开这个配置是为了在生成文章的时候生成一个同名的资源目录用于存放图片等文件。
也可以直接在source/_posts下新建md文件里面内容写上
---
title: {title}
date: {date}
sticky: 
...
---
即可
或者直接复制/scaffolds下的模板文件
```

#### 参考

- [GitHub Pages + Hexo搭建个人博客网站](https://blog.csdn.net/yaorongke/article/details/119089190)
- [Hexo文档](https://hexo.io/zh-cn/docs/)
