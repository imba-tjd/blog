---
title: Linux命令
---

## 命令集合网站

* https://ss64.com/bash/
* https://man.linuxde.net/
* https://www.runoob.com/linux/linux-command-manual.html
* https://tldr.ostera.io/
* https://github.com/chubin/cheat.sh

## 简单笔记

* zless：查看压缩文件内容
* lsblk：列出块设备。除了RAM外，以标准的树状输出格式，整齐地显示块设备。-l参数为列表；lsusb：显示usb设备
* md5sum
* dd：刻录
* history：按住“CTRL + R”就可以搜索已经执行过的命令，它可以在你写命令时自动补全
* cal 月 年：日历
* stat：查看文件的属性；file：以自然语言描述文件是什么，但debian不自带
* source/.：解析配置文件
* setleds：设置键盘指示灯
* top：显示进程内存/cpu占用信息，会自动更新；htop：多彩的界面，不自带；bashtop不自带；cat /proc/loadavg和uptime：显示负载，信息少，不会自动更新（可用watch运行）
* screen：窗口管理器的命令行界面版本
* uname -a：查看内核版本
* chroot：改变根目录
* gcp：有进度条的复制工具，不自带；或者rsync -a --progress src dest
* fdisk -l：显示磁盘信息；cfdisk：命令行中的图形化的分区工具；mkfs.ext4：格式化分区
* basename，dirname：取得路径的文件名与目录名
* du -sh [filename]：显示目录的占用空间；df -h：显示挂载点的总大小、已用空间、剩余空间
* mount | column -t：显示挂载分区状态
* mkdir -p：创建子目录时，如果父目录不存在，则自动创建；文件夹已存在也不会报错
* unar：https://theunarchiver.com/command-line，可以正确解压非Unicode的zip
* ls：-r倒序，-R递归，-t按日期降序，-S按文件大小升序，-d显示当前文件夹自己的信息，-1每一行只显示一个文件名；ls -1A | wc -l或ls -A | wc -w：显示出有多少个文件，使用A就不会包含.和..；sl：显示火车
* date：http://www.runoob.com/linux/linux-comm-date.html
* fsck：检测文件系统错误
* cd -：切换到之前的目录，cd !$：切换到刚刚用mkdir新建的目录
* free -mt：显示内存容量，以MB为单位
* tail -f：跟踪指定文件，如果有变化立即显示，删除后停止；与less -F相同
* tasksel：在Debian中快速安装软件
* cat \<\<EOF\>out.txt：输入以后继续输入文字，当最后一行输入EOF文本的时候结束输入，用-EOF可以忽略空白字符
* which、whereis：找到程序的路径，其中which只在PATH中找可执行文件，whereis还在一些系统目录中找且可找大多数类型的文件；type可以区分内置还是外置命令，-a可以显示包含alias在内的命令查找顺序
* xclip：复制到剪切板上，不自带
* head -10：显示前10行信息
* rename：把所有.c的文件重命名为.cpp的：`rename 's/.c$/.cpp/' *`
* sha1sum/md5sum -c xxx.sha1：自动验证对应的文件是否符合；不加-c是验证，未指定文件时从stdin读入
* ln：参数意义与cp相同，-P硬链接（默认？），-s软连接，-f覆盖dest；src一般要写绝对路径，在-s下src写`./xxx`产生的是相对符号文件的链接而不指当前工作目录下的xxx，后者需用-rs；文件夹一般只能用软连接，root权限下才可用-d创建文件夹硬链接；cp也可创建链接：-l硬链接，-s软链接
* sudo update-grub：自动修复引导
* lscpu：相比于`cat /proc/cpuinfo`，不会每个核都显示一遍
* shred：粉碎文件
* ldd --version：查看glibc版本
* sshfs：把远程目录挂载到本地，不自带
* base64：默认加密，-c解密，-w0不换行；直接跟文件名就是处理文件，可以用管道给到输入流或者用\<\<\<
* exec：常用来替代当前 shell 并重新启动一个 shell，换句话说，并没有启动子 shell。使用这一命令时任何现有环境都将会被清除。在有些脚本中要用exec命令运行node应用。否则不能顺利地关闭容器，因为SIGTERM信号会被bash脚本进程吞没。exec命令启动的进程可以取代脚本进程，因此所有的信号都会正常工作
* htpasswd -nb -B admin password | cut -d ":" -f 2
* ps auxf：显示所有进程，且显示父子进程关系，但这样就不完全按时间排序了，想要后者就去掉f。直接写数字就是指定pid，-u/g/C分别指定user/group/CMD，不清楚前俩大小写的区别；pstree：以简单形式显示父子进程关系，不会有pid；在`procps`包中
* cp . dest/：会把当前文件夹下的内容复制过去，而不是只复制一整个文件夹；cp -L/--dereference：如果src是软连接，可以追踪到源文件；cp -a：相当于-dpR
* iconv -f gbk -t utf-8 source-file -o target-file
* eval
* vmstat：监控系统状况的程序。`vmstat 5 5`：在5秒时间内进行5次采样；-f显示从系统启动至今的fork数量，-s显示内存相关统计信息，-d查看磁盘的读写，-m查看slab信息
* cat > file：接下来输入内容，ctrl+d结束；可以快速地创建一个有内容的文件
* less：空格或f或z翻一页，d翻半页，回车或e翻一行，b或w上翻一页，u上翻半页，y上翻一行，可以在前面加数字，具体看h帮助；g移动到第一行，G移动到最后一行，/向下搜索，n搜索下一个，N搜索上一个，q退出，v调用editor编辑；-N显示行号，-s合并连续空行
* fc：在editor中编辑上一条输入的命令，并在退出时执行
* factor：求一个数的所有因数
* logrotate：切割日志的程序
* nroff -man manpage.1 | more：显示man格式的.1文件
* mktemp：在/tmp下创建一个空的临时文件，输出该文件路径，因此一般用法为FILE=$(mktemp) ...

## 压缩/解压

### tar

* 压缩：tar czf jpg.tar.gz *.jpg（c为压缩，z为tar.gz，f必须是最后一个参数，后跟压缩包名）
* 解压：tar xf abc.tar.gz（x为解压缩；类型现在都可以自动识别了；最后可跟路径来只解压指定的部分文件）
* 其他主选项：t查看内容，r追加，u更新；这仨和c和x只能选其中一个
* 其它参数：C指定解压目录，v显示详细信息，j代表tar.bz2，-J/--xz代表xz
* 默认即使用*也不会打包以点开头的文件，可以加`.[!.]*`匹配上

### xz

* 不做把多个文件打包成一个文件的工作
* 流压缩：cat a.txt | xz -9e > a.txt.xz；注意源文件名不会保留，解压后的名字就是去掉.xz的部分；不加文件名或文件名是-就是这种模式
* 分别压缩多个文件：xz -9e files；默认压缩完了就把源文件删了，-k可以保留
* 解压：xz -d file.xz；加-c可以写到stdout中；也可以使用unxz，则不用加-d，其实就是xz的软连接

### unzip

* 不自带；可以考虑使用`python -m zipfile`替代
* -p写到stdout中
* tldr的文档是错的
* gunzip是用来解压gzip(gz)的，不是用来解压zip的
* 对于分卷压缩包，先`cat test.zip* > ~/test.zip`合并起来再解压就好了

## find

* `find path -name '*.txt' -exec wc -l {}\;` ：统计txt文件有多少个，其中{}会被依次替换成找到的文件；也可以使用|xargs
* -type指定文件类型，f为普通文件，l为链接文件（f不包含这一项），d为文件夹
* -not -path：排除目录；默认会搜索隐藏目录，但这个参数好像不能递归排除；另外还存在一个prune参数作用好像也是排除
* -size +1m：大于1M的文件
* -mtime -1：一天之内修改的；atime是访问时间，ctime是创建时间
* -print0：与xargs -0匹配使用
* -delete：直接删除找到的文件
* -maxdepth：最大搜索深度
* 进入当前目录下所有子文件夹分别执行命令：`find . -maxdepth 1 -type d ! \( -name ".*" -o -name "System Volume Information" -o -name '$RECYCLE.BIN' \) -exec ./script.sh {} \;`

## awk

* 如果以文件作为参数，记得用xargs
* 如果分隔符是分号，要用单引号包裹

### 选择除第一列以外的列

* awk '{$1="";print}'，但直接输出结果，会在最前面加个空格
* awk '{for(i=2;i\<=NF;i++) printf $i" ";printf "\n"}'
* 以上两种方法似乎都会对行进行一次排序？总之行序和本来的不同

## grep

* 如果直接使用管道传递给grep，会被当作标准输入而不是参数，因此如果要把文件作为参数传给它，要用xargs；如果是选取文件名本身，才用find和非xargs的grep
* grep *option* *pattern* *filenames*，其中文件名可以是*；如果文件名有多个，会在每一行前面打印出匹配到内容的文件名，可用-h隐藏；如果文件名只有一个，可用-H强制显示出来（用find -exec就是这种情况）；如果只想显示出文件名，可用-l；显示的路径风格和提供的相同
* 使用正则表达式可用grep -E或egrep，但前者用某些元字符（比如大括号）需要转义，后者不需要；-o可以只显示匹配到的内容而不显示一整行
* 统计每个文件内匹配到了多少次用-c，多个文件会显示文件名；顺便输出行号用-n；递归搜索用-r；忽略大小写用-i；pattern之间-e匹配多个模式，相当于或；不输出结果用-q，可用于返回值判断；-v反向查找
* 打印匹配文本之前、之后、两边的几行，分别用-A、-B、-C
* fgrep：只查找指定的表达式，没有通配符和正则，但速度快
* grep "aaa" file* -lZ | xargs -0 rm：删除多个文件，Z为0字节后缀输出
* grep -- -a：`-a`不会被认为是grep的参数

## xargs

* 把由管道获得的标准输入变成指定命令的参数，默认命令是echo
* 不会从标准输入读取的程序：kill、rm
* 例如在1.txt这个文件的内容中找1，以下三条命令的效果相同：grep 1 1.txt、cat 1.txt|grep 1、echo 1.txt|xargs grep 1
* 把文件的内容变成单行输出：cat test.txt | xargs。-n3指定3列换行，实际就是一次传3个参数
* -d指定分隔符，默认是换行符；分隔符会去掉不要然后加上空格
* -P 并行处理；默认每次只获取一部分数据
* `ls *.jpg | xargs -n1 -I cp {} /data/images`：复制所有图片文件到 /data/images 目录下。-I指定一个替换字符串，默认为{}，这个字符串在xargs扩展时会被替换掉，每一个参数命令都会被执行一次
* 无法用`... | xargs cd`，因为后者会变成/bin/cd，但只用bash的内置cd命令才能改变当前目录；可用`cd $(...)`

## dig

* sudo apt install dnsutils
* `dig @dns.googlehosts.org www.baidu.com`
* +vc：查询时使用tcp（不一定支持）
* +short：只显示ip（但可能有多个）
* +trace：显示解析过程。其实是自己主动迭代解析
* +additional：显示glue records
* +dnssec：返回RRSIG记录，不清楚是否会进行校验
* +cmd：默认开启，会显示dig的版本
* +subnet=your_ip/24：EDNS，9.11之后默认开启；但DNSPod程序员表示他们只支持/32，不满足会就把请求丢弃，防止随机IP的攻击，此时可加+nocookie/+noedns不发送EDNS请求
* -p 5353：指定端口
* AAAA：查询ipv6的记录；A：ipv4的记录；MX：邮件服务器记录；NS：该域名由哪个dns服务器负责解析；CNAME：查询别名；-x：查询PTR记录，只能用此参数，前面的方法不适用
* AUTHORITY SECTION显示最终解析指定域名的dns服务器，ADDITIONAL SECTION显示那些dns服务器的ip
* ->\>HEADER\<\<-中的status: NXDOMAIN表示不存在，此时一般会返回SOA；SERVFAIL表示上游DNS服务器响应超时（用于递归DNS服务器）；flags:QR表示为响应报文，AA表示是权威DNS回应的，RD表示DNS服务器必须递归处理该报文，RA表示该DNS支持递归查询
* 查询EDNS状态：`dig edns-client-sub.net -t TXT @8.8.8.8`
* 另外还有kdig（knot-utils）和host命令

## crontab

* crontab -l [-u username]：列出当前/某个用户的任务；列出所有用户的任务：`cat /etc/passwd | cut -f 1 -d : |xargs -I {} crontab -l -u {}`
* crontab -e：编辑；-r：删除
* 默认开机会自动启动crond。cron的调度文件：crontab、cron.d、cron.daily、cron.hourly、cron.monthly、cron.weekly
* systemctl list-timers

每次有计划任务运行都会往`/var/log/auth.log`里写一条`pam_unix(cron:session)...`。解决方法：打开`/etc/pam.d/common-session-noninteractive`，往`session required pam_unix.so`前加`session [success=1 default=ignore] pam_succeed_if.so service in cron quiet use_uid`

分布式的：cronsun，国人开发的。

## iproute2

替代net-tools(ifconfig, arp, route, netstat)。
https://www.cnblogs.com/sparkdev/p/9253409.html
https://www.cnblogs.com/sparkdev/p/9262825.html

* ip

### ss(Socket Statistics)

* 替代netstat
* ss -t为TCP，-u为UDP，-w为raw，-x为Unix Socket，不加就都有
* 默认只显示establish了的，-l显示listen状态的，-a同时显示所有状态，-4/-6略
* -p显示使用端口的进程和uid；默认会把常用端口号解析成服务名，-n阻止这种解析
* -s显示summary，-e显示额外信息，-o显示keepalive时间信息
* filter过滤连接状态：`ss state/exclude all/connected/各种TCP状态`，help里有
* expression过滤ip和端口：`ss src :22`，冒号前填IP，Peer地址用dst，其它一些过时记法略；可以用and和or连接多个条件，则需要再在外面加个单引号；IP可以用`192.168.0/24`这种形式；运算符可用==、=、!=、ge、le、gt、lt
* Local Address指的就是本机，但它既可以是Listen的，也可以是发出的；Perr Address就是“另一端”
* 监听地址为*的就是双栈，`[::]`就只是V6

## 传输文件

aria2、axel、httpie放到软件的文章里去了。

### curl

* 安装时会装上openssl；支持http2、ftp等多种协议；一次可请求多个URL且会复用，某些选项要多次指定，--next可把接下来的选项都指定为下一个URL的
* 默认显示body，-I用HEAD请求，-i顺便显示Header，`-D –`不限方法只显示头，-X手动指定HTTP请求类型
* -o或者>写入文件，-O使用网站提供的名字；多URL时要多次指定，或者后者可改用--remote-name-all
* -A指定用户代理；-H可指定所有Header，用"key: value"，但每个要分开指定
* -x使用proxy（正代）
* `-C -`断点续传
* -e/--referer提供referer
* -s安静模式，不显示进度条；-sS安静模式下仍显示错误
* -L跟随30x跳转
* -d 'para1=val1&para2=val2'使用POST方式请求，类型默认是`x-www-form-urlencoded`，也可多次使用-d；`-d @file`是从文件中读取，一行一个；`--data-raw @file`不会识别成文件，就是真正的`@`；`--data-urlencode`会帮你做一次URL编码，像值中有空格时可用；-F的类型是`multipart/form-data`
* -k忽略证书错误
* --compressed：自动添加Accept-Encoding: deflate, gzip, br并自动解码；如果头里手动指定了AE，也必须加此项；Win不支持
* -c/--cookie-jar加文件名保存cookie；-b/-cookie加@文件名读取cookie，-b加"key1=val1;key2=val2"发送在命令行中指定的cookie；文件格式见https://github.com/curl/curl/blob/master/docs/HTTP-COOKIES.md
* url通配：`[1-10]`、`[01-10]`、`[1-10:2]`、`[a-z]`、`{asdf,zxcv}`，-g禁用这一行为；在-o的文件名中可用`#1`对应通配变量
* -K opt.txt：从文件中读取命令行选项，一行一个可不带横杠，井号注释，也想指定url必须用`url`；默认会寻找`~/.curlrc`，Win下是~及exe所在目录下的`_curlrc`
* -#显示进度条，在-O或者重定向输出时默认会有
* -J：与-O同时使用时会从Content-Disposition中读取文件名，小心覆盖，不会url解码
* -m指定超时时间，15--speed-limit 1000指定15秒内至少传输1000字节
* -ssl：自动升级到https，如果无法建立仍用http
* --libcurl xxx.c：生成对应的c语言代码
* 访问httpbin.org/get可以看到服务器收到的请求信息
* 只显示各个阶段消耗的时间，需要请求完毕才会输出：`curl -o /dev/null -s -w %{time_namelookup}::%{time_connect}::%{time_starttransfer}::%{time_total}::%{speed_download}"\n" <url>`
* 其他人做的笔记：https://gist.github.com/subfuzion/08c5d85437d5d4f00e58

### scp

* scp -rpC src dest
* user@Host或IP:/path/filename ./
* r为递归，p为保留日期等，C为压缩
* -P指定端口
* src可有多个文件
* win下的好像无法识别中文路径
* -3可以（通过本机）在两个服务器之间传文件
* src如果以`/`结尾，就是传输文件夹里的内容，不是传输一个文件夹；如果不以斜杠结尾再加`-r`就能传输一个文件夹

### rsync

* rsync -avzP src dest：a保留文件所有属性且递归，z启用压缩，P断点续传且显示每个文件的进度，v详细模式但在P下只会再多显示一点总结信息
* -H保留硬链接，-u只更新变化了的（文件存在于dest且mtime更新），-n是dry run，--delete删除dest中所有不在src中的文件，-c用校验和而不是时间和大小判断是否不同会大量消耗资源，-vvvv显示debug级别的信息
* 设置--inplace和--append后好像是增量同步；有限速功能避免把服务器带宽占满（scp也有）；Host::/path用的是rsync协议，运行daemon时可以类似ftp提供文件出去，可以设置只读和IP黑白名单；不提供dest等价于运行ll，此时-h才有用；-R的作用：`-rR /var/./log/nginx /tmp`将会创建/tmp/log/nginx；-S发送稀疏文件时使用
* 维护一个local copy：rsync -rlptzv --progress --delete --exclude=.git "user@hostname:/remote/source/code/path" .
* 多线程的管理脚本：https://github.com/pigsboss/toolbox/blob/master/pfetch.py
* TODO：https://www.digitalocean.com/community/tutorials/how-to-use-rsync-to-sync-local-and-remote-directories-on-a-vps

## sed

* -i为直接修改文件内容
* -e指定后一个参数为sed的指令，当需要一次性指定多个时有用

```
find -type f | xargs sed -i '1s/^\xEF\xBB\xBF//' # 全部去掉BOM；注意隐藏文件夹
'6i Contents' # 在文件的第6行插入指定内容
's/aaa/bbb/' # 替换文本
sed -e '2,5d' -e '8d' file.txt # 删除2至5行和第8行，关键是那个第8行是删除2-5行前的第8行，而不是变化后的
```

## 持续运行

* ctrl+z：相当于运行了suspend
* fg：把ctrl+z挂起的程序恢复到前台
* bg：让ctrl+z挂起的程序在后台运行
* jobs：显示后台挂起的任务
* 使用`%1`指定目标任务，可以用kill杀掉
* 输命令时在最后加个&会默认在后台运行，一般会用`>log.txt 2>&1 &`不让输出到屏幕上；如果整个用小括号括起来，就不在当前终端中，jobs里看不到，应该和disown效果一样
* 再在最前面加个nohup，则正常退出会话时命令不会结束（异常退出还是会结束）；~~但这样的程序无法用fg恢复~~好像可以；默认会自动把输出重定向到$HOME/nohup.out中
* 如果没有用nohup和&就运行了程序，想要退出会话时不结束，用`ctrl+z; bg; disown`，之后那个进程就变成了独立的
* wait命令可以等待后台任务执行完
* 还有一种setsid，格式和nohup类似，不过原理不同，且必须加重定向输出

## systemd

/lib/systemd/system/

### systemctl

* systemctl start（当前启动一次）、stop、enable（开机自启）、disable（禁止自启）、status、restart、try-restart（已启动才重启否则不做操作）、reload-or-restart、reload-or-try-restart、mask（禁止自动和手动启动）、unmask name.service
* 查看已激活的服务：systemctl list-units --type=service；加-a显示所有的；查看所有服务的自启状态：systemctl list-unit-files -t service
* systemctl show --property=Environment docker
* systemctl daemon-reload
* systemctl hibernate（休眠）、hybrid-sleep（交互式休眠）、rescue（进入单用户救援状态）
* chkservice：交互式管理服务的程序 https://zhuanlan.zhihu.com/p/35264613

### 其它

* systemd-analyze：查看启动耗时；blame指令：查看每个服务的启动耗时；critical-chain [name.service]指令：显示瀑布状的启动过程流
* journalctl：管理日志，取代syslog
* crond 也被 systemd 的 timer 单元取代

## nmap

* nmap -T4 -s[scan method] -p[portrange] targetip
* ip范围可以用-和/和逗号；端口范围可以用-和逗号，且可在前面加U:和T:表示UDP和TCP
* 默认扫描TOP1000的端口，-F扫TOP100的；-r不打乱端口顺序
* -T0-5为扫描时序，越高速度越快约容易被封；-A为进攻性(Aggressive)扫描，会完整全面地扫描
* 列表扫描(-sL)：仅将指定的目标的IP列举出来
* 主机发现(-sn)：默认会发ICMP和80和443，只要有一个回复就说明在线，局域网内是ARP；-Pn跳过主机发现，将所有指定的主机视作开启的
* 端口扫描：，-sS/sT/sA/sW/sM/sN/sF/sX分别为TCP SYN/Connect()/ACK/Window/Maimon/Null/FIN/Xmas，后三种比较隐蔽，为秘密扫描方式；-sU使用UDP
* 服务与版本侦测(-sV)：用于确定目标主机开放端口上运行的具体的应用程序及版本信息，--version-light使用轻量侦测，--version-all尝试使用所有的probes进行侦测；
* 操作系统扫描(-O)：--osscan-guess大胆猜测系统类型，准确性会下降不少，结果变多
* 指定后面的项目会把前面的也都进行一遍；扫描方式可以同时使用多个，如果什么都不加应该就是SYN，也有一种说法是会用四种方式
* 规避技巧：-S伪造源IP，--spoof-mac伪造mac，--data-length随机填充数据到指定长度，--badsum: 使用错误的checksum来发送数据包，正常情况下应被丢弃，如果收到回复，说明回复来自防火墙

## 参考

* https://www.cnblogs.com/manong--/p/8012324.html
* https://blog.csdn.net/aspirationflow/article/details/7694274
* https://www.ruanyifeng.com/blog/2016/03/systemd-tutorial-commands.html

### TODO

* tmux（https://github.com/skywind3000/awesome-cheatsheets/blob/master/tools/tmux.txt https://zhuanlan.zhihu.com/p/27915505）、supervisor(python)、PM2 (for node.js)
* killall、pkill、kill -9
* nftables：https://zhuanlan.zhihu.com/p/88981486 https://zhuanlan.zhihu.com/p/139678395
* https://www.oschina.net/translate/useful-linux-commands-for-newbies
* https://einverne.github.io/categories.html#每天学习一个命令
* https://github.com/trimstray/the-book-of-secret-knowledge
