---
title: Hello Hexo
subtitle: "使用Github Pages+Hexo+Anisina搭建博客"
date: 2019-04-06 16:53:00
header-img: /images/bg1.png
author: "BerniLin 21"
# cdn: 'header-on'
tags:
  - Hexo
---

{% blockquote  %}
博客 / 教程
{% endblockquote %}

搭建属于自己的博客，想必是所有的开发者都曾想过且有为数不少的人在做的事情吧！
我也不例外，最近有接触到一些关于搭建博客的开源项目，就开始搭建自己的博客。
我的博客采用的解决方案是Github Pages+Hexo+Anisina，话不多说，开始吧！


## 先决环境

1、git
2、node
关于git和node不是本文的重点不进行声明


## GitHub创建个人仓库

创建一个和你用户名相同的仓库，后面加.github.io，只有这样，将来要部署到GitHub page的时候，才会被识别，也就是xxxx.github.io，其中xxx就是你注册GitHub的用户名。

## 然后就开始我们的主角 Hexo

Hexo是一款基于Node.js的静态博客框架。

### 安装依赖

安装hexo： 
``` bash
$ install -g hexo-cli
```
安装hexo部署到git page的deployer：
``` bash
$ npm install hexo-deployer-git –save
```
可以通过
``` bash
$ hexo -v
```
来查看hexo是否安装完成以及版本号


### 搭建框架

```bash
$ hexo init <folder>
$ cd <folder>
$ npm install
```
搭建后的目录结果是这样的：
```bash
├── _config.yml
├── package.json
├── scaffolds
├── source
|   ├── _drafts
|   └── _posts
└── themes
```
这时候，其实hexo项目已经搭建完成，也就是我们常说本地开发版。

## 运行
```
hexo g    //生成静态文件。
hexo s    //启动服务器，默认接口4000
```
这时候访问http://localhost:4000/ 就可以看见如下界面
{% img http://pp80ekjth.bkt.clouddn.com/blog1-1.png  %}

## Anisina
我使用的主题是Anisina，更多主题可在hexo的官网挑选（{% link 主题 https://hexo.io/themes %}）

### Anisina下载及其配置
将主题文件clone到项目\themes中
```
git clone https://github.com/Haojen/hexo-theme-Anisina.git themes/Anisina
```
修改项目根目录下的_config.yml文件，
```
themes: Anisina
```
并且拷贝 Anisina 文件夹下的 _config.yml文件 替换 hexo 默认的 _config.yml
更多关于Anisina的配置可参见 {% link Anisina 中文使用教程 https://haojen.github.io/2017/05/09/Anisina-%E4%B8%AD%E6%96%87%E4%BD%BF%E7%94%A8%E6%95%99%E7%A8%8B/#undefined %}

### cdn-url

Anisina是对七牛有支持的，注册一个你自己的七牛CDN ，然后拷贝你自己的CDN URL ：
```
cdn-url: http://you-cdn.com/
```
在七牛里上传图片
在博文的 front-matter 上指定图片名称
```
 header-img: some-images.png
```
当然，也可以在根目录下中的source文件夹中建立一个文件夹创建个image文件夹用来存放图片，取图片的时候使用项目的绝对路径

### 配置侧边栏

``` code

# Sidebar settings
sidebar: true
sidebar-about-description: "醒来，读书，写长长的代码" # 个人描述
sidebar-avatar: http://pp80ekjth.bkt.clouddn.com/header.jpg    # use absolute URL

# Featured Tags
featured-tags: true                     # whether or not using Feature-Tags
featured-condition-size: 0              # A tag will be featured if the size of it is more than this condition value

# friends                               #友链
friends: [
  {
    title: "vuejs",
    href: "https://vuejs.org"
  },
  {
    title: "juejin im",
    href: "https://juejin.im/user/59a2ef36f265da24843e677c"
  },
  {
    title: "Ruan Yifeng's",
    href: "http://www.ruanyifeng.com"
  },
  {
    title: "liao xue feng",
    href: "hhttp://https://www.liaoxuefeng.com"
  },
]
```

### 创建 Tags导航页面

页面模板有tags和works
```
hexo new page "Tags"
```

页面布局配置
```
---
title: Tags
date: 2019-03-31 16:05:07
layout: tags    //在这里
---
```
创建后记得 hexo clean && hexo g , 然后可以运行 hexo s在本地查看效果


### works的配置

进入yourblog/source，建立一个文件夹，名字是_data，注意下划线哦
然后在里面建立一个文件，名为：project.json

``` code
    {
        "一段话": {
            "title": "这是一个标题",
            "subTitle": "这是一个副标题",
            "img_link": " ",
            "use" : ["java"],
            "link": "",
            "data":"2019.01-01",
            "direction": "关于work的一个测试"
        },
        "一段话2": {
            "title": "这是一个标题2",
            "subTitle": "基于 这是一个副标题2",
            "img_link": "",
            "use" : ["java","mysql","spring"],
            "link": "",
            "data": "2018.01-01",
            "direction":
                "关于work的一个测试"
        }
    }
```


## 创建文章

```
hexo new "your-post-name"
```
文章内部配置:

``` code
--
layout: post
title: 中文测试
subtitle: "is a test"
date: 2019-03-24 17:17:17
author: "BerniLin 21"
header-img: bg10.jpg
cdn: 'header-on'
tags:
  - test
  - 古文
---

博文内容。。。

```
更多关于博文页面的配置以及写在可参见 {% link hexo官网 https://hexo.io/zh-cn/ %}  

## 部署

我们要把搭建好的博客项目放到先前创建好的github仓库中，copy你的仓库路径，然后在_config.yml文件下修改：
```
deploy:
  type: git
  repository: https://github.com/linsizao/BerniLin21.github.io.git
  branch: master
```
之后再hexo clean && hexo g && hexo d，发布完成，起到仓库查看你会发现仓库内容不再是空了，在这时候再访问xxx.github.io，这样能看到你在本地看到的博客项目了
