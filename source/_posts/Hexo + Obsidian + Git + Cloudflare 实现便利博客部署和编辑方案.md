---
title: Hexo + Obsidian + Git + Cloudflare 实现便利博客部署和编辑方案
date: 2023-05-31 12:20:39
tags: []
abstract: 快速利用github pages搭建博客的一套完整解决方案
---
# 使用Hexo搭建博客
参考这篇大佬写的文章：[Hexo 快速搭建指南](https://blog.esunr.xyz/2022/06/64163235c30f.html)
注意我们是使用github搭建博客，所以直接跳过那篇文章的第二部分
主题不用担心，后面改的话是很方便的
Github Action 工作流真的巨赞
# 使用Obsidian管理博客
大体上依旧参考那位大佬的文章：[Hexo + Obsidian + Git 完美的博客部署与编辑方案](https://blog.esunr.xyz/2022/07/e9b42b453d9f.html)
但是注意在使用obsidian的时候，可以不用选择hexo根文件夹，使用source也是非常好的。因为文章中的obsidian插件 `File Tree Alternative Plugin` 已经改版了，`Focuse on Folder` 变的极其奇怪，所以选用在source目录下使用obsidian
![image.png](https://s2.loli.net/2023/05/31/vJi1yCHjk752KFN.webp)
iCloud同步个人觉得没必要，直接用Github解决
# 自定义域名和开启HTTPS
参考这位大佬的文章：[GitHub Pages 启用 Cloudflare 加速及 HTTPS](https://siriusq.top/github-pages-%E5%90%AF%E7%94%A8-cloudflare-%E5%8A%A0%E9%80%9F%E5%8F%8A-https.html)
需要注意的是当你填写完域名的时候
![image.png](https://s2.loli.net/2023/05/31/qKkc5amt6pA9f7l.webp)会自动在release分支里新增一个CNAME文件，这个文件的作用是标识你的域名。但是这个文件在你上传网页的时候自动给覆盖掉，这样在你新上传的时候github就没法知道你的域名，你的网页很可能展示404。为了每一次到release分支都有CNAME文件，你需要把CNAME文件添加到source里面，这样当你新上传的时候github就可以感知的到。
source文件夹下的最终样子
![image.png](https://s2.loli.net/2023/05/31/LKlIWop5Brsanez.webp)
