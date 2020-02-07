# 迁离 WordPress

## 为什么不选择WP

首先，我用的是`WordPress.com`这个托管的WP的免费版，而不是`.org`自建的。

1. 博客后台打开速度慢。虽然前台打开速度还可以，且毕竟是国外的网站，慢很正常。但就是慢。
2. 我本来也只有用MD写文章的需求，图文混排用得极少，更不需要自动嵌入视频等功能。WP试图兼容两种，但实际会造成混乱，例如一句话中不能有多个乘号或者下划线，否则会被识别成MD语法的效果，导致内容被删掉变成样式；解决方法是用行内代码，但如果是粘贴上去的，即使用纯文本粘贴也会生效，必须手打。
3. 自动删掉带有尖括号的内容且没有任何提示。因为那会被识别成HTML标记，而免费版能用的标记只有少量白名单，不在名单里，“为了安全”就删掉了。
4. 撤销效果令人窒息。经常把我前面写的一大段撤销回去，而且重做还有几率失败。
5. 评论系统基本没法用。虽然阻挡了上千条垃圾评论，但也许里面有正常评论，因为我没去看就被删了。
6. 没处理任何并发更新冲突。我经常开几十个网页，有时会打开两个相同的文章。如果同时改了同一篇文章并更新，那就会丢掉一部分内容，只有后更新的留下来。

本文即介绍如何把WP的数据变成Markdown。但需要具备一定的程序员的知识（否则也不会选择转成MD）。

## [jekyll-import](https://import.jekyllrb.com/)

此项目为jekyll官方的项目，支持许多来源。

1. 到WP的博客后台导出数据，`WordPress.com`的会发送到邮箱里。
2. 解压，就一个xml。重命名为`wordpress.xml`，传到服务器或虚拟机的Linux上；可用scp，Win10自带。
3. （灵活）执行以下命令。此处使用到了docker。如果不会就只能先装好ruby环境、装jekyll咯；操作完之后还要卸载，要不就在虚拟机里操作。反正不用docker麻烦得多。

```bash
# 假文件设传到了~/migrate中
cd ~
docker run -it -v migrate:/migrate jekyll/jekyll bash
cd /migrate
gem install jekyll-import hpricot open_uri_redirections
ruby -r rubygems -e 'require "jekyll-import"; JekyllImport::Importers::WordpressDotCom.run()'
exit
docker rm $(docker ps -ql)
docker rmi jekyll/jekyll
tar caf WP.tar.xz migrate/* # 可能因为那个image里的tar是busybox的，不支持此命令
rm -r migrate
# 复制回本地
```

转换结果是HTML，无法进一步编辑。不过转换效果真的极佳，直接加上Jekyll主题就已经好了。应该一部分原因是我本来就是当作MD写的，没用各种高级功能。文件名是URL转义后的，问题不大。

检查内容时发现metadata里有一堆的奇怪的以`_oembed_`开头后面跟hash值的key，而它们的值全都是`{{unknown}}`。我猜这些都是被WP吞掉的内容（即第一节第三点说的事）。

## HTML转MD

上一步博文的转换结果是一个个的HTML，所以需要再去找HTML转MD的程序。

我用的是`https://cloudconvert.com/html-to-md`，感觉效果还可以。不过此网站不开源，如果有必要可以看看隐私策略。而且免费版每天转换有数量限制，第二天才会重置，可以使用隐身模式加多proxy解决。

当然无论用哪种转换方式，内容肯定都是要人工过一遍的。推荐等到需要编辑那篇文章的时候再改，否则一次性全改完要累死。

### 此网站的坑

* 下划线、美元符号、波浪号、反斜线会转义一下。其它一些md符号也会严格转义
* h1用的是横杠的风格，而不是井号
* 代码块留下了`{.wp-block-code}`，不过这个批量删掉很容易，而且这一点应该算作上一段的工具的锅
* 好像会把过长的行自动换行
* 有序列表和无序列表用的不是一个空格

基本上都是一些风格问题，改起来虽然多，但不难。

## 后序步骤

上面只是转移了博客数据，接下来还要构建新网站。如果不用Jekyll，反正转到MD了，接下来想用什么就用什么。

如果用Jekyll，MD都放到`_posts`目录下。可以使用GitHub Page自带的`minima`主题，或者用`jekyll-theme`话题下star数最高的`minimal-mistakes`，后者有比较详细的文档，基本上调一调`_config.yml`就好。

如果想进一步定制，就需要学习如何使用Jekyll的模板引擎Liquid，以及学习一些前端知识。

幸好GitHub能自动构建，还是能省一些事的。如果想手动构建，那就还要学Gem和Bundler甚至Ruby了。

## [exitwp](https://github.com/thomasf/exitwp)

一个python2的工具，能直接把WP转换成md。

然而最终效果很一般，只能转换一部分，有的还是会留下HTML标签。而且空行很多，还出现了无法识别`wp-block`类型的错误。我就不进一步测试了。此处仅留一下最开始尝试用到的命令。

```bash
cd ~
git clone https://github.com/thomasf/exitwp.git --depth=1
# 上传文件略
docker run -it -v exitwp:/exitwp python:2-slim bash
cd /exitwp
pip install -r pip_requirements.txt
python exitwp.py
```
