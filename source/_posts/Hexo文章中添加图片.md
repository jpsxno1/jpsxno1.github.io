---
title: Hexo文章中添加图片

date: 2017-05-23 15:12:47

categories: "Hexo教程"
tags:

       - Hexo
description: 如何在文章中插入图片
---


## 解决方案

 1. 首先将站点配置文件_config.yml中 post_asset_folder: 这个选项设置为true。
 2. 在Git Bush 中 hexo目录下，下载插件，执行

```bash
$ npm install hexo-asset-image --save
``` 

 3. 执行成功后，再运行hexo new "xxxx"生成md博文时， /source/_posts文件夹内除了生成xxxx.md文件外还生成了一个同名的文件夹

 4. 若想在xxxx.md中引入图片，则先把图片赋值到xxxx文件夹中，然后只需要在xxxx.md中按照markdown的格式引入图片

![替代文字](xxxx/图片名.jpg)

 5. 最后检查一下， hexo  g 生成页面后，进入public\2017\05\23\index.html文件中查看相关字段，可以发现，html标签内的语句是<img src="2017/05/23/xxxx/图片名.jpg">， 而不是<img src="xxxx/图片名.jpg">。这很重要，关乎你的网页是否可以真正加载你想插入的图片。