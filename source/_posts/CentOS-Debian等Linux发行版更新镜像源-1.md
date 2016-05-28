---
title: 'CentOS,Debian等Linux发行版更新镜像源'
tags: []
date: 2015-07-25 11:31:00
---

安装后CentOS镜像源,默认使用国外的,很慢,你知道的, 国内更换163、aliyun (看了很多人的博客, 今天有空,自己写一篇)

*   找到想要更换的源的官方网站, 比如

    *   [网易163](http://mirrors.163.com/)&nbsp;页脚找到帮助中心 &nbsp;&nbsp;

        *   [阿里云](chrome-extension://pghodfjepegmciihfhdipmimghiakcjf/sandbox/http;//mirrors.aliyun.com/)&nbsp;找到对应的发行版本,后面有个**help**可以点击进去看帮助

*   比如CentOS6使用163镜像

    *   备份系统的配置文件
<pre>    mv /etc/yum.repos.d/CentOS-Base.repo /etc/yum.repos.d/CentOS-Base.repo.backup
    cd /etc/yum.repos.d/
    wget http://mirrors.163.com/.help/CentOS6-Base-163.repo
    yum clean all
    yum makecache</pre>