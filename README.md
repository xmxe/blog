## 通过GitHub Pages/Cloudflare Pages构建博客

> 构建自己博客的方式有很多种，网上也有很多开源的博客模板，但有的属于动态模板，需要搭建后台管理系统，需要数据库、缓存、服务器等环境支持，使用GitHub Pages构建静态博客则不需要上述环境，写文章的话直接在md里写完后编译成HTML即可。
> 构建静态博客的主流技术栈有[Hugo](https://gohugo.io/),[Hexo](https://hexo.io/zh-cn/)等。
> 此项目采用了[Hexo](https://hexo.io/zh-cn/)来搭建自己的个人博客。

### 环境准备

Git、GitHub Pages、NodeJS

### 安装Hexo

```shell
#全局安装或者单独安装nmp install hexo
npm install hexo-cli -g
# <folder>为自定义目录，如npx hexo init blog
npx hexo init <folder>
# cd blog
cd <folder>
# 安装模板依赖
npm install
# 简写为npx hexo g生成静态文件。-d,--deploy文件生成后立即部署网站，依赖hexo-deployer-git插件
npx hexo generate
# 简写为npx hexo s启动服务器。默认情况下，访问网址为:http://localhost:4000/
npx hexo server
```

### 更换主题

下载[hexo主题](https://hexo.io/themes/)后放置在themes路径下,并修改_config.yml内的theme设定，即可切换主题。推荐两款主题[fluid](https://github.com/fluid-dev/hexo-theme-fluid)、[matery](https://blinkfox.github.io/)

#### fluid
- [Hexo-Fluid主题美化](https://blog.csdn.net/weixin_43471926/article/details/109798811)
- [fluid文档](https://hexo.fluid-dev.com/docs/start)

#### matery

> 此项目对matery主题进行了大量的个性化更改

- [Hexo-Matery主题自定义(共六篇)](https://blog.csdn.net/weixin_43662760/category_11551976.html)
- [基于Hexo的matery主题搭建博客自定义修改篇2](https://blog.17lai.site/posts/4d8a0b22/)
- [Hexo+Matery访问速度优化](https://blog.sky03.cn/posts/42790.html)
- [hexo博客添加一级分类相册功能](https://liyangzone.com/article/2019-07-22-hexo-blog-add-gallery-tutorial/)
- [使用AES算法加密Hexo相册](https://liyangzone.com/article/2019-07-30-hexo-blog-encrypt-gallery/)

### 部署

使用**npx hexo g**生成public目录(可更改public_dir属性),将public目录推送至GitHub仓库即可。

> 此项目文件生成在了/docs目录下,将编译后的HTML及源码一块上传到了GitHub,在部署GitHub Pages时选择/docs目录即可

[其他方式](https://hexo.io/zh-cn/docs/github-pages)

### 发布文章

```shell
npx hexo new [layout] <title>
# 例如npx hexo new post text相当于在source/_posts下生成text.md文件(可以在命令中指定文章的布局（layout），默认为post，可以通过修改_config.yml中的default_layout参数来指定默认布局)

npx hexo new draft text
# 在source/_draft下生成text.md草稿文件,查看草稿的话使用npx hexo s --draft或者在_config.yml下将render_drafts修改成true

npx hexo page text
# 在source目录下生成文件

# hexo new [lauout]实际上是在scaffolds目录下寻找文件生成，例如在scaffolds模板文件下有abc.md,使用hexo new abc text即依据abc.md模板生成文件
```

> 更多命令查看[Hexo官方文档](https://hexo.io/zh-cn/docs/)
