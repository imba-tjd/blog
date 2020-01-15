---
title: Linux命令
---

> http://blogread.cn/it/article.php?id=3285

命令集合网站
------------

* https://ss64.com/bash/
* https://man.linuxde.net/
* https://www.runoob.com/linux/linux-command-manual.html
* https://tldr.ostera.io/

简单笔记
--------

* zless：查看压缩文件内容
* lsblk：列出块设备。除了RAM外，以标准的树状输出格式，整齐地显示块设备。-l参数为列表；lsusb：显示usb设备
* md5sum
* dd：刻录
* history：按住“CTRL + R”就可以搜索已经执行过的命令，它可以在你写命令时自动补全
* cal 月 年：日历
* stat：查看文件的属性；file：以自然语言描述文件是什么，但debian不自带
* source/.：解析配置文件
* setleds：设置键盘指示灯
* top：显示进程内存/cpu占用信息，会自动更新；htop：多彩的界面，不自带；cat /proc/loadavg和uptime：显示负载，信息少，不会自动更新（可用watch运行）
* screen：窗口管理器的命令行界面版本
* uname -a：查看内核版本
* fg：把ctrl+z挂起的程序恢复到前台；bg：让ctrl+z挂起的程序在后台运行（输命令时在最后加个&会默认在后台运行）；jobs：显示后台挂起的任务
* chroot：改变根目录
* gcp：有进度条的复制工具，不自带；或者rsync -a --progress src dest
* fdisk -l：显示磁盘信息；cfdisk：类似图形界面的分区工具；mkfs.ext4：格式化分区
* basename，dirname：取得路径的文件名与目录名
* du -sh \<目录名\>：显示目录的占用空间；df -h：显示挂载点的总大小、已用空间、剩余空间
* mount | column -t：显示挂载分区状态
* mkdir -p：创建子目录时，如果父目录不存在，则自动创建；文件夹已存在也不会报错
* unar：https://theunarchiver.com/command-line，可以正确解压非Unicode的zip
* ls：-r倒序，-R递归，-t按日期降序，-S按文件大小升序，-d显示当前文件夹自己的信息，-1每一行只显示一个文件名；ls -1A | wc -l或ls -A | wc -w：显示出有多少个文件，使用A就不会包含.和..；sl：显示火车
* ps ww -alef：查看进程，各种参数的具体用法例子：https://www.tecmint.com/ps-command-examples-for-linux-process-monitoring/
* date：http://www.runoob.com/linux/linux-comm-date.html
* fsck：检测文件系统错误
* cd -：切换到之前的目录，cd !\$：切换到刚刚用mkdir新建的目录
* free -mt：显示内存容量，以MB为单位
* tail -f：跟踪指定文件，如果有变化立即显示，删除后停止；与less -F相同
* tasksel：在Debian中快速安装软件
* cat \<\<EOF\>out.txt：输入以后继续输入文字，当最后一行输入EOF文本的时候结束输入，用-EOF可以忽略空白字符
* which、whereis：找到程序的路径，其中which只在PATH中找可执行文件，whereis还在一些系统目录中找且可找大多数类型的文件
* xclip：复制到剪切板上，不自带
* head -10：显示前10行信息
* nohup：`nohup ./test &`能在exit后继续任务，默认所有信息输出到\$HOME/nohup.out中；不自带
* rename：把所有.c的文件重命名为.cpp的：`rename 's/.c$/.cpp/' *`
* sha1sum/md5sum -c xxx.sha1：自动验证对应的文件是否符合；不加-c是验证，未指定文件时从stdin读入
* cp创建链接：-l为硬链接，-s为软链接；但参数顺序意义与ln相同，后者-P为硬链接，-s为软连接，-r为相对链接
* sudo update-grub：自动修复引导
* lscpu：相比于`cat /proc/cpuinfo`，不会每个核都显示一遍
* shred：粉碎文件
* ldd --version：查看glibc版本
* sshfs：把远程目录挂载到本地，不自带
* adduser可以交互式添加用户，useradd只有一大堆参数
* base64：默认加密，-c解密，-w0不换行；直接跟文件名就是处理文件，可以用管道给到输入流或者用\<\<\<
* exec：常用来替代当前 shell 并重新启动一个 shell，换句话说，并没有启动子 shell。使用这一命令时任何现有环境都将会被清除。在有些脚本中要用exec命令运行node应用。否则不能顺利地关闭容器，因为SIGTERM信号会被bash脚本进程吞没。exec命令启动的进程可以取代脚本进程，因此所有的信号都会正常工作
* htpasswd -nb -B admin password | cut -d ":" -f 2

at命令
------

```
echo 123 | at now + 1 minutes
at now + 1 minutes
warning: commands will be executed using /bin/sh
at> echo 123
at>
job 2 at Tue Jun 26 16:30:00 2018
Can't open /var/run/atd.pid to signal atd. No atd running
```

* 此命令Debian没有
* 可用-f指定文件来作为标准输入
* 如果是直接输入，最后要用ctrl + d结束
* atq或at -l查看at定时队列，atrm或at -d删除某个定时任务
* 任务都存放在/var/spool/at里，也可以用rm删
* /etc/at.allow和/etc/at.deny控制哪些用户可用使用at，前者优先级更高
* service atd status：查看at服务状态，要启用时才有效

tar
---

> https://www.cnblogs.com/manong--/p/8012324.html

* 压缩：tar czf jpg.tar.gz \*.jpg（c为压缩，z为tar.gz，f必须是最后一个参数，后跟压缩包名）
* 解压：tar xf abc.tar.gz（x为解压缩；类型现在都可以自动识别了；最后可跟路径来只解压指定的部分文件）
* 其他主选项：t查看内容，r追加，u更新；这仨和c和x只能选其中一个
* 其它参数：C指定解压目录，v显示详细信息，j代表tar.bz2，-J/--xz代表xz
* 默认即使用\*也不会打包以点开头的文件，可以加`.[!.]*`匹配上

显示登陆的用户
--------------

* w：显示所有登陆的用户名、终端、时间、cpu使用时间、在做什么。有标题（可关），信息最多；多个会话多条
* who：显示所有登陆的用户名、终端、时间；没有标题；多个会话多条
* whoami：显示自己的用户，相当于id -un；只有一条？
* who am i：显示自己登陆的用户名、终端、时间；是who的子集，只有一条
* users：只显示登陆的用户名；多个会话多条

find
----

* -type指定文件类型，f为普通文件，l为链接文件（f不包含这一项），d为文件夹
* `find path -name '*.txt' -exec wc -l {}\;` ：统计txt文件有多少个，其中{}会被依次替换成找到的文件；也可以使用|xargs
* -not -path：排除目录；默认会搜索隐藏目录，但这个参数好像不能递归排除；另外还存在一个prune参数作用好像也是排除
* -size +1m：大于1M的文件
* -mtime -1：一天之内修改的；atime是访问时间，ctime是创建时间
* -print0：与xargs -0匹配使用
* -delete：直接删除找到的文件
* 切换到指定文件的目录：cd \$(find . -name \*\*\*)，[不能用xargs](https://www.zhihu.com/question/67430958)

awk
---

* 如果以文件作为参数，记得用xargs
* 如果分隔符是分号，要用单引号包裹

### 选择除第一列以外的列

* awk '{\$1="";print}'，但直接输出结果，会在最前面加个空格
* awk '{for(i=2;i\<=NF;i++) printf \$i" ";printf "\\n"}'
* 以上两种方法似乎都会对行进行一次排序？总之行序和本来的不同

grep
----

* 如果直接使用管道传递给grep，会被当作标准输入而不是参数，因此如果要把文件作为参数传给它，要用xargs；如果是选取文件名本身，才用find和非xargs的grep
* grep *option* *pattern* *filename ...*，其中文件名可以是\*；如果文件名有多个，会在每一行前面打印出匹配到内容的文件名，可用-h隐藏；如果文件名只有一个，可用-H强制显示出来（用find -exec就是这种情况）；如果只想显示出文件名，可用-l；显示的路径风格和提供的相同
* 使用正则表达式可用grep -E或egrep，但前者用某些元字符（比如大括号）需要转义，后者不需要；-o可以只显示匹配到的内容而不显示一整行
* 统计每个文件内匹配到了多少次用-c，多个文件会显示文件名；顺便输出行号用-n；递归搜索用-r；忽略大小写用-i；pattern之间-e匹配多个模式，相当于或；不输出结果用-q，可用于返回值判断
* 打印匹配文本之前、之后、两边的几行，分别用-A、-B、-C
* fgrep：只查找指定的表达式，没有通配符和正则，但速度快
* grep "aaa" file\* -lZ | xargs -0 rm：删除多个文件，Z为0字节后缀输出

xargs
-----

* 把由管道获得的标准输入变成指定命令的参数，默认命令是echo
* 不会从标准输入读取的程序：kill、rm
* 例如在1.txt这个文件的内容中找1，以下三条命令的效果相同：grep 1 1.txt、cat 1.txt|grep 1、echo 1.txt|xargs grep 1
* 把文件的内容变成单行输出：cat test.txt | xargs。-n3指定3列换行，实际就是一次传3个参数
* -d指定分隔符，默认是换行符；分隔符会去掉不要然后加上空格
* -P 并行处理；默认每次只获取一部分数据
* `ls *.jpg | xargs -n1 -I cp {} /data/images`：复制所有图片文件到 /data/images 目录下。-I指定一个替换字符串，默认为{}，这个字符串在xargs扩展时会被替换掉，每一个参数命令都会被执行一次

curl
----

* 其他人做的笔记：https://gist.github.com/subfuzion/08c5d85437d5d4f00e58
* 支持多种协议，默认限时内容；-I仅下载Header，-i也显示Header
* -o或者\>写入文件，-O使用网站提供的名字
* -A指定用户代理；-H可指定所有Header，用"key: value"，但每个要分开
* -c/--cookie-jar加文件名保存cookie，-b/-cookie加@文件名读取cookie，-b加"key1=val1;key2=val2"发送在命令行中指定的cookie
* -\#显示进度条，在-O或者重定向输出时默认会有
* -x使用proxy
* -C断点续传
* -e/--referer提供referer
* -s安静模式，不显示进度条；-sS安静模式下仍显示错误
* -L跟随30x跳转
* -d 'para1=val1&para2=val2'使用POST方式请求
* -T使用PUT方式上传文件，-X手动使用其它HTTP请求
* -k忽略证书错误
* url里用中括号加数字范围可以批量下载
* --http2允许用HTTP/2，如果服务器不支持仍可用1.1，需--version中有模块
* 访问httpbin/get可以看到服务器收到的信息

### dig

* sudo apt install dnsutils
* `dig @dns.googlehosts.org www.baidu.com`
* +vc：查询时使用tcp（不一定支持）
* +short：只显示ip（但可能有多个）
* +trace：显示解析过程。其实是自己主动迭代解析
* +additional：显示glue records
* +dnssec
* +cmd：默认开启，会显示dig的版本
* -p 5353：指定端口
* AAAA：查询ipv6的记录（有可能返回一个网址，原因不明，大概是不支持吧）；A：ipv4的记录；MX：邮件服务器记录；NS：该域名由哪个dns服务器负责解析；CNAME：查询别名；-x：查询PTR记录，只能这样，前面的方法不适用
* AUTHORITY SECTION显示最终解析指定域名的dns服务器，ADDITIONAL SECTION显示那些dns服务器的ip
* -\>\>HEADER\<\<-中的status: NXDOMAIN表示不存在，此时一般会返回SOA；SERVFAIL表示与DNS服务器响应超时；这两者ANSWER字段为0；成功是是NOERROR，ANSWER字段为2
* 另外还有kdig（knot-utils）和host命令

crontab
-------

* 分布式的：cronsun，国人开发的

iproute2
--------

* 替代net-tools(ifconfig, arp, route, netstat)

rsync
-----

* rsync from to，其中远端用user@host:/path，from是文件，to是目标文件夹；-r复制整个文件夹，如果是复制文件夹里的所有内容，from保留末尾的/
* -avPH：a保留文件所有属性且递归，v详细模式，P断点续传，H保留硬链接
* -u只更新变化了的，-z启用压缩，-n是dry run，--delete删除to中所有不在from中的文件
* -e 'ssh -p 22'可以通过ssh传输
* https://zhuanlan.zhihu.com/p/40022680
* https://zhuanlan.zhihu.com/p/85087767

sed
---

```
sed -i "15i Contents" Lab.txt # 在文件的第15行插入指定内容
find -type f | xargs sed -i '1s/^\xEF\xBB\xBF//' # 全部去掉BOM；注意隐藏文件夹
```

增加系统的熵
------------

* cat /proc/sys/kernel/random/entropy\_avail
* apt install rng-tools
* rngd -r /dev/urandom 这个用法是错的没边，相当于把/dev/urandom重新导入/dev/random，欺骗内核让他认为有足够的熵源
* 必须要有/dev/hwrng才行。如果是在虚拟环境中，rngd会报Cannot find a hardware RNG device to use。如果还想用，必须编辑虚拟机本身的环境，用/dev/hwrng
* 可以用apt install haveged，貌似是傻瓜式的
* 用cat /dev/random | rngtest -c 1000可以测试生成速度

tmux、screen、nohup、systemctl、supervisor(python)、PM2 (for node.js)

vmstat、lsof -i:\$PORT

nftables：https://zhuanlan.zhihu.com/p/88981486


