---
layout: post
title:  "第一个blog"
date: 2017-01-01
categories: others
tags: blog,Jekyll
---
* 目录
{:toc}

## 一、 为什么要写博客
> 自知没什么自制力，所以一直以来也没有打算弄这么个东西实在是麻烦的很。但是没有总结就没有思考，没有思考就没有进步，于是，还是从头开始吧。也不算太晚。

## 二、 建立博客的过程

### 2.1 找文档
只要google一下github.io 和 Jekyll等关键词，各种wiki层出不穷也看不完，见参考

### 2.2 从零开始
一开始参考的是Jekyllcn上面的文档，但是文字太多，自知前段功底垃圾，怕一时半会儿也看不完，所以只是简单地在本地jekyll server跑起来观看一边效果之后就放弃了，效果实在是太朴素了，还不如从fork一个模版开始来得容易。

### 2.3 从模版开始
参考了一下这个，找一篇简单的地开始吧*[使用jeklly搭建自己的blog](http://jolestar.com/use-jekyll-as-blog/)*，年代久远了些，找个近点的开始，[一步步在GitHub上创建博客主页](http://www.pchou.info/ssgithubPage/2013-01-03-build-github-blog-page-01.html)，折腾了半个晚上，repository删了建，建了删，实在是折腾，总是不如任意，[jekyll sites](https://github.com/jekyll/jekyll/wiki/Sites)看了多个主题，总结了下，需要修改的东西太多，一时半会也该不好，还不如从头开始吧。

### 2.4 回归，还是从零开始
[从零开始折腾Jekyll](http://bluebiu.com/blog/learn-to-use-jekyll.html#section-9)


## 三、解决的问题

### 3.1 生成目录
kramdown生成目录的方式参考[Jekyll 目录树](https://www.zfanw.com/blog/jekyll-table-of-content.html)
样式如下，其中*号后面得有空格哦

```plain
* 目录
{:toc}
```

### 3.2 嵌入图片
插入图片的链接可以用Atom的插件生成，对话框中定位到具体的图片之后即可在光标所在位置插入图片的引用，但是有个问题，图片在本地可以引用相对路径，但是在服务器上只能从site.url下面的assets目录引用，注意名字也不能改变，原来尝试用images目录尝试怎么都不行，参考[撰写博客](http://jekyllcn.com/docs/posts/)

![image]({{site.url}}/assets/insert-image.png)


## 四、没有解决的问题

### 4.1 flowchart的支持
原本想把所有的元素包括图片都用代码生成，包括流程图、公式、序列图生成SVG嵌入页面，但是涉及到js库安装的问题，实在搞不定，后续再说吧。[flowchart](http://flowchart.js.org/)

## 五、后续
  真tm累啊，还没折腾出啥模样来，
  缺乏的东西还很多，折腾不动了
- [ ] 添加了对标签的全套支持（原主题不支持 Tags ）
- [ ] 去掉引起挂起的 Web Font，添加适合中文世界的 font-family
- 大量 CSS 优化：包括适合中文阅读的字号字重改进，移动端交互优化。
- Font-awesome (iconfont) 使用国内的七牛 CDN
- 使用 Github Flavored Markdown
- 使用 腾讯统计、百度统计
- 加入“多说” 评论支持
- 。。。
  后续估计也坚持不了多久，😄


## 六、参考
- [setting-up-your-github-pages-site-locally-with-jekyll](https://help.github.com/articles/setting-up-your-github-pages-site-locally-with-jekyll/)

- [jekyll themes](https://jekyllthemes.io/)

- [Jekyll&Github Pages博客模板挑选和配置](http://cenalulu.github.io/jekyll/choose-a-template-for-your-blog/#toc3)
