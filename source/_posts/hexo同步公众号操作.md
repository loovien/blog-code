---
title: 同步hexo博客内容到公众号
date: 2020-01-09 21:49:56
desc: 再hexo中编写好的博客文件，实现自动同步到公众号中。
tags: blog, WeChat
image: source/imgs/hexo.jpg
---

最近想把hexo写的博客内容也同步到微信公众号中， 萌生了一个简单的项目，那就来搞它吧。

<!-- more -->

## 大致的思路

hexo 编写好的文档后， 推送到远程仓库， 仓库使用webhook通知服务器， 服务器脚本拉取最新代码后，

判断有没有新写的博客(难点， 怎么获取最近编辑的博客文件的markdown文件呢)， 有的话， 使用 `hexo deploy` 

发布部署后，再读取博客的标题，详情摘要, 有图片的话，再把图片对应生成微信的图文素材，调用微信开放接口,

同步到微信中。再调用群发接口，通知关注的用户(哈哈，好像很完美)

#### 项目地址

```bash
    git clone https://github.com/loovien/vxgo.git
```

