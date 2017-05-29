---
title: 搭建hexo过程中遇到的问题及解决办法
description: 搭建hexo过程中遇到的问题及解决办法
---
#搭建hexo过程中遇到的问题及解决办法
---
**问题一：**
 ` ERROR Deployer not found : git ` 
 <br>
**解决办法：**
 <br>
输入下述命令：`$ npm install hexo-deployer-git --save`
 <br><br>
**问题二：**
hexo已部署到github但是出现404
 <br>
**解决办法：**
在github搭建repository时，注意教程**yourname**.github.io，这里的**yourname**一定务必是自己github的用户名 
<br><br>
**问题三：**
如何在hexo中使用本地图片上传?
<br>
(**放在根目录：**早期大部分的方案是把图片放在`source/img`下，然后markdown里写`![img](source.img/img.png)`)。显然这样在本地的编辑器里完全不能正确识别图片的位置。)
<br>
**解决办法：**
*asser-image*一个新的语法来插入本地图片
[CodeFalling/hexo-asset-image](https://github.com/CodeFalling/hexo-asset-image)
<br>
首先确认`_config.yml`中有`post_asset_folder:true`。
在hexo目录，执行
`npm install https://github.com/CodeFalling/hexo-asset-image --save`
假设在<br>
<pre>MacGesture2-Publish
├── apppicker.jpg
├── logo.jpg
└── rules.jpg
MacGesture2-Publish.md</pre>
这样的目录结构（目录名和文章名一致），只要使用 `![logo](MacGesture2-Publish/logo.jpg) `就可以插入图片。