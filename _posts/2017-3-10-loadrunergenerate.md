---
 layout: post
 title:  LoadRunner 之调用Linux下loadgenerate 方案
 tags: [ Loadrunner, LoadGenerator, linux, 压力测试, 编码]
 comments: true
 image:
  background: triangular.png
---

最近突然想到试试用Linux安装Loadrunner 的 负载器来实现多集群并发压力测试，可是这个过程着实废了一些时间，现在把自己的步骤和问题总结出来，给大家以后做一个参考，使大家花费更多时间做一些有意义的事情。

1. 首先是安装负载器 其实，每一个安装包都是在loadrunner自带的安装文件中，只要大家自己好好的看看安装文档和安装文件的目录是可以找到的，现在为了节省时间，我把安装包地址发布给大家（我的redhat是64位的大家根据自己的位数选择，还有我提供的一下路径基本都是基于64为运行安装，如果冲突，希望大家自己去搜索解决办法）

> http://yunpan.cn/cVzuU4Ed4JRk7 （提取码：f235）

2. 安装过程是一个很重要的过程，在这里我推荐大家一个很好的安装步骤的链接，真的很全 

> http://blog.sina.com.cn/s/blog_9aa583cf0101bu4y.html

3. 安装过程中遇到的问题，有rpm包不存在 或者有依赖的问题

  1. 推荐大家下载rpm包之后直接使用暴力安装的方式，这样就可以忽略 包与包之间的依赖 
```shell
rpm -i --force --nodeps
```
  2. 在安装好负载器之后，会执行 ./verify_generator  来查看是否安装成功，其中我遇到的提示 

  libgcc_s.so.1 的问题，开始我以为是lib是x86_64的原因，后来直接下载一个libgcc_s.so.1的

安装包，这个问题就解决 ，运行测试就成功了 ，下载地址是

> http://yunpan.cn/cVzrb64tWzmEf （提取码：7b96）

  3. 如果实在存在依赖包的问题，还发现了一个不错的 包网站，下载很方便 ，网址是

> http://pkgs.org/

备注：解决包依赖问题的几种方法  http://blog.csdn.NET/kai27ks/article/details/7473683

4. 安装完成，调试成功，接下来就是用controller 调试试一下 ，在这里我是防火墙直接关闭的省去了添加           例外的过程，大家可能会问linux怎么关闭防火墙啊，亲，我刚好会 
```shell
  service iptables stop   //#的意思就是 root权限执行
```

5. 调试连接，连接过程中，我每个地方都是通过的但是，连接运行执行就是Fail  ，我真想说FK，可是后来我仔细一看 原来是 需要关闭一个配置的地方 

打开controller 下的loadgenerat下的 详细（Details设置 UNIX Environment中的 Don't use  RSH ，点击保存，再次运行场景OK ，点击connect ，状态变为 ready预示着我成功了



6. 既然成功了，那么接下来该运行脚本，小小的跑一下啊 ，可是这回我就真是有种骂人的冲动,ERROR提示  error  19797   19799，这两个错误意思就是符合编译规范，所以我就不断的去更改脚本，可是怎么都不成         功，我仔细检查脚本发现没有错啊。运行都是OK的 ，切换到windows下的loadgenerate运行一下，完全正           常。故此我疑惑了。我想也许并不是脚本本身的问题，而是linux可能不支持某种底层编码的问题 。所以。         我把脚本中含有的中文去除 ，变为英文，把脚本的字体格式也修改 。再次在linux下运行OK 成功了 。

解决步骤 ：

（1）vugen - tools  -general options -environment -select font 设置字体格式为 Fixedsys

（2）删除脚本中的中文，全部用英文代替 

 备注：当然还有一种方式 就是直接修改 export LANG=zh_CN.UTF-8    (locale  默认的环境编码是 en_US.UTF-8是不支持中文的)


7.脚本在场景中运行成功，本次的测试结束.偷笑