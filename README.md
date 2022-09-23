### 通过GitHub Pages构建博客

> 引言

>> 构建自己的博客方式有很多种，网上也有很多开源的博客模板，但有的属于动态模板，还有自己的博客后台管理系统，需要关系数据库、缓存、服务器等环境支持，个人还是选择了GitHub Pages构建静态博客，写文章的话直接在md里写完推送，不需要数据库等环境支持。采用了[Hexo](https://hexo.io/zh-cn/)来构建博客。

#### 一、 环境准备

Git、GitGub Pages、NodeJS 


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



#### 四、参考

- [GitHub Pages + Hexo搭建个人博客网站，史上最全教程](https://blog.csdn.net/yaorongke/article/details/119089190)
- [Hexo文档](https://hexo.io/zh-cn/docs/)


