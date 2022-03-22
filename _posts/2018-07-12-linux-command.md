---
title: Linux命令
---

## 命令集合网站

* https://ss64.com/bash/
* https://www.runoob.com/linux/linux-command-manual.html
* https://tldr.ostera.io/
* https://cheat.sh/
* http://bropages.org/

## 简单笔记

* cal 月 年：日历
* setleds：设置键盘指示灯
* chroot：改变根目录
* date：http://www.runoob.com/linux/linux-comm-date.html %s为时间戳
* xclip：复制到剪切板上，不自带
* sudo update-grub：自动修复引导
* htpasswd -nb -B admin password | cut -d ":" -f 2
* fc：在editor中编辑上一条输入的命令，并在退出时执行
* factor：求一个数的所有因数
* logrotate：切割日志的程序
* nroff -man manpage.1：显示man格式的.1文件
* broot：新版tree

## xargs

* 把标准输入按空白字符分隔变成指定命令的参数，默认命令是echo
* 默认一次传所有参数，-n3一次传3个参数
* -d：指定分隔符
* -0：以NUL字符分隔读取，但需要源程序配合
* -t：先打印待执行的命令再执行，-p先询问再执行
* -P：多进程并行处理，设为0时尽可能多
* -i：按行处理，并把每一行替换到命令中的`{}`作为单个参数
* 对cd无效，因为会使用外置的命令

## 系统信息

* free -ht：内存信息
* vmstat：监控系统状况，但对人类很难读，不学
* ldd --version：能看到glibc版本
* top：显示进程内存/cpu占用信息，会自动更新；htop：多彩的界面，不自带；bashtop不自带；cat /proc/loadavg和uptime：显示负载，信息少，不会自动更新（可用watch运行）
* ps auxf：a显示其它用户的进程，u第一列显示用户，x显示后台进程，f显示父子进程关系但导致不完全按时间排序。直接写数字就是指定pid，-u/g/C分别指定user/group/CMD，不清楚前俩大小写的区别；pstree：以简单形式显示父子程序名关系；在`procps`包中
* lscpu：相比于`cat /proc/cpuinfo`不会每个核都显示一遍

## 文件处理

* cp . dest/：复制当前文件夹下的内容，而非一个整文件夹；-L/--dereference：如果src是软链接，可复制源文件；cp -a：相当于-dpR
* stat：查看文件的属性；file：以自然语言描述文件格式
* gcp：有进度条的复制工具，不自带；或者rsync -a --progress src dest
* basename，dirname：取得路径的文件名或目录。前者第二个参数传后缀还会去掉它。后者支持任意数量的参数，处理后每个一行
* mkdir -p：创建子目录时，如果父目录不存在，则自动创建；文件夹已存在也不会报错
* ls：-R递归，-r倒序，-t按日期降序，-S按文件大小降序，-d显示当前文件夹自己的信息，-A列出除.和..以外的所有文件
* cd -：切换到之前的目录
* tail -f：持续输出指定文件，如果有变化立即显示；与less -F相同
* which、whereis：找到程序的路径，其中which只在PATH中找可执行文件，whereis还在一些系统目录中找且可找大多数类型的文件
* rename：把所有.c的文件重命名为.cpp的：`rename 's/.c$/.cpp/' *`
* shasum/md5sum：指定文件时默认二进制模式，从stdin读取时默认文本模式，-c验证
* base64：默认加密，-d解密，-w0加密时输出不换行，支持跟文件或从stdin读取
* ln：参数意义与cp相同，-P硬链接（默认？），-s软链接，-f覆盖dest；src一般要写绝对路径，在-s下src写`./xxx`产生的是相对符号文件的链接而不指当前工作目录下的xxx，后者需用-rs；文件夹一般只能用软链接，root权限下才可用-d创建文件夹硬链接；cp也可创建链接：-l硬链接，-s软链接
* shred：粉碎文件
* FILE=$(mktemp)：在/tmp下创建临时文件并获得文件名
* iconv -f gbk -t utf-8 source-file -o target-file
* less：空格或f或z翻一页，d翻半页，回车或e翻一行，b或w上翻一页，u上翻半页，y上翻一行，可以在前面加数字，具体看h帮助；g移动到第一行，G移动到最后一行，/向下搜索，n搜索下一个，N搜索上一个，q退出，v调用editor编辑；-N显示行号，-s合并连续空行
* split -b 50m huge_file分隔文件，合并用cat

### find

* `find path -name '*.txt' -exec wc -l {}\;` ：统计txt文件有多少个，其中{}会被依次替换成找到的文件
* -type指定文件类型，f为普通文件，l为链接文件，d为文件夹
* -not -path：排除目录；默认会搜索隐藏目录，但好像不能递归排除；还有个-prune更快
* -size +1m：大于1M的文件
* -mtime -1：一天之内修改的；atime 访问时间，ctime 创建时间
* -print0：与xargs -0配合用
* -delete：直接删除找到的文件
* -maxdepth：最大搜索深度
* 进入当前目录下所有子文件夹分别执行命令：`find . -maxdepth 1 -type d ! \( -name ".*" -o -name "System Volume Information" -o -name '$RECYCLE.BIN' \) -exec ./script.sh {} \;`
* https://www.zhihu.com/question/487213837

#### fd

* fd abc相当于`find -iname '*abc*'`，第二个参数指定开始搜索的根目录
* 模式默认为正则，-g改为glob
* 模式只存在小写字母时大小写不敏感，有大写字母时自动变为大小写敏感
* 默认忽略点开头文件和gitignore中的文件，用-HI分别禁用前后者，小心禁用前者后能搜到.git中的文件
* 无参使用会递归列出所有文件，模式用`.`仅列出当前文件夹
* -t/--type指定type
* -e指定后缀，可多次使用
* 默认只匹配文件名(基本名)，指定-p匹配路径
* -x并行执行外部命令，每项都执行一次；-X执行一次外部命令，把所有项当作那一次的参数；-i交互模式每次执行前询问。占位符支持{.}路径去扩展，{/}文件名带扩展，{//}父目录，{/.}文件名无扩展；不存在时默认在最后加一个{}
* -E排除路径，为glob
* 原生不具有-delete功能
* Debian在fd-find包中，安装后的二进制名为fdfind

### 压缩/解压

* unar：自动正确解压非Unicode的zip，可惜最后更新时间2015年
* unzip：不自带，最后更新时间2009年
* 分卷zip，先`cat test.zip* > ~/test.zip`合并起来再解压就好了
* gunzip是用来解压gzip(gz)的，不是用来解压zip的
* unrar：是rar官方的，但在non-free中
* upx --lzma
* zless：查看压缩文件内容
* lzip：处理 .lz，仅使用LZMA非2
* lz4：压缩率比zip低，默认级别为-1，解压速度非常快，用--best会大幅减慢压缩速度且压缩率一般但解压速度不变
* zstd：速度不如lz4但也不错，最高压缩为--ultra -22，根据IO状态动态调整级别用--adapt，多线程用-T0
* brotli：默认已使用最高压缩级别，根据测试，平均不如zstd，仅在单线程下br好一点

#### tar

* 压缩：tar caf jpg.tar.gz *.jpg。c为压缩，a为自动按后缀选择压缩格式，f指定压缩包名，必须是最后一个参数
* 解压：tar xf abc.tar.gz。x为解压，类型现在都可自动根据后缀识别；最后可跟路径来只解压部分文件
* 其他主选项：t查看内容，r追加，u更新；它们与c和x只能选其中一个
* 其它参数：C指定解压目录，v显示详细信息，z代表tar.gz，j代表tar.bz2，-J/--xz代表xz，-I手动指定压缩或解压程序且能设定参数
* 现在Windows也自带了，但只支持gzip

#### xz

* 压缩：xz -9e data.txt，会生成data.txt.gz
* 流压缩：cat data.txt | xz -9e > data.txt.xz
* 解压：xz -d file，用-c输出到stdout；自带软连接unxz相当于-d，xzcat相当于-dc，其他压缩程序一般也支持这类
* 默认压缩完了会删除源文件，指定-k保留
* 源文件名不会保留，解压文件后的名字就是去掉.xz的部分
* 不做把多个文件打包成一个文件的工作，可指定多个文件但只是分别压缩
* 实际算法为LZMA2和未压缩混合

## 网络

### dig

* 在dnsutils包中
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
* ->>HEADER<<-中的status: NXDOMAIN表示不存在，此时一般会返回SOA；SERVFAIL表示上游DNS服务器响应超时（用于递归DNS服务器）；flags:QR表示为响应报文，AA表示是权威DNS回应的，RD表示DNS服务器必须递归处理该报文，RA表示该DNS支持递归查询
* 查询EDNS状态：`dig edns-client-sub.net -t TXT @8.8.8.8`
* 另外还有kdig（knot-utils）和host命令

### nmap

* nmap -s[scan method] -p[portrange] targetip
* ip范围可以用-和/和逗号；端口范围可以用-和逗号，且可在前面加U:和T:表示UDP和TCP
* 默认扫描TOP1000的端口，-F扫TOP100的；-r不打乱端口顺序
* -T0-6：扫描时序，越高速度越快，一般用T4；-A为进攻性(Aggressive)扫描，会完整全面地扫描
* -sL：仅列出解析指定范围后的IP
* 主机发现(-sn)：默认会发ICMP和80和443，只要有一个回复就说明在线，局域网内是ARP；-Pn跳过主机发现，将所有指定的主机视作开启的
* 端口扫描：-sS/sT/sA/sW/sM/sN/sF/sX分别为TCP SYN/Connect()/ACK/Window/Maimon/Null/FIN/Xmas，后三种比较隐蔽；-sU为UDP
* 服务与版本侦测(-sV)：用于确定目标主机开放端口上运行的具体的应用程序及版本信息，--version-light使用轻量侦测，--version-all尝试使用所有的probes进行侦测
* 操作系统扫描：-O
* 指定后面的项目会把前面的也都进行一遍；扫描方式可以同时使用多个，如果什么都不加应该就是SYN，也有一种说法是会用四种方式
* 规避技巧：-S伪造源IP，--spoof-mac伪造mac，--data-length随机填充数据到指定长度，--badsum: 使用错误的checksum来发送数据包，正常情况下应被丢弃，如果收到回复，说明回复来自防火墙
* TODO: https://nmap.org/man/zh/   nmap -sP -PR：ARP搜索？
* nping --tcp -p 443 addr：V6要用`--tcp-connect -6`，--tcp效果很差，连回显都没有，注意addr无需也不能用中括号因为端口单独指定；这样V6虽然有回显了但少了很多信息，如果想看seq win mss只能用--tcp -e interface --source-ip --source-mac --dest-mac
* https://securitytrails.com/blog/nmap-cheat-sheet

### mtr

* 命令行在mtr-tiny包中，图形界面在mtr包中
* mtr [-6] -rw addr，-rw是终端输出总结结果，-p是输出每条结果，都不用就会进入另一个界面
* -c是次数
* 本来-T是TCP，-P指定端口，但不懂为什么未监听的端口也不丢包
* WSL1需要管理员权限运行控制台，无需sudo，但是速度超级慢，现在好像完全用不了了

### iproute2

替代net-tools(ifconfig, arp, route, netstat)。
https://www.cnblogs.com/sparkdev/p/9253409.html
https://www.cnblogs.com/sparkdev/p/9262825.html

#### ip

ip route get xxx 可以直接查找其对应的路由
ip route show cache
ip route flush cache
ip route list/show
ip neigh
ip addr
ip link

#### ss(Socket Statistics)

* 替代netstat
* ss -t为TCP，-u为UDP，-w为raw，-x为Unix Socket，不加就都有
* 默认只显示establish了的，-l显示listen状态的，-a同时显示所有状态，-4/-6略
* -p显示使用端口的进程和uid；默认会把常用端口号解析成服务名，-n阻止这种解析
* -s显示summary，-e显示额外信息，-o显示keepalive时间信息
* filter过滤连接状态：`ss state/exclude all/connected/各种TCP状态`，help里有
* expression过滤ip和端口：`ss src :22`，冒号前填IP，Peer地址用dst，其它一些过时记法略；可以用and和or连接多个条件，则需要再在外面加个单引号；IP可以用`192.168.0/24`这种形式；运算符可用==、=、!=、ge、le、gt、lt
* Local Address指的就是本机，但它既可以是Listen的，也可以是发出的；Perr Address就是“另一端”
* 监听地址为*的就是双栈，`[::]`就只是V6

## 下载和传输文件

aria2、axel、httpie放到软件的文章里去了。TODO: croc GO，传输文件，需要服务端

### curl

* 安装时会装上openssl；支持http2、ftp等多种协议；一次可请求多个URL且会复用，某些选项要多次指定，--next可把接下来的选项都指定为下一个URL的
* 默认显示body，-I用HEAD请求，-i顺便显示Header，`-D –`不限方法只显示头，-X手动指定HTTP请求类型
* -o或者>写入文件，-O使用网站提供的名字；多URL时要多次指定，或者后者可改用--remote-name-all
* -A指定用户代理；-H可指定所有Header，用"key: value"，但每个要分开指定
* -x使用proxy（正代），默认端口1080，例外是https时是443
* `-C -`断点续传
* -e/--referer提供referer
* -s安静模式，不显示进度条；-sS安静模式下仍显示错误
* -L跟随30x跳转
* -d 'para1=val1&para2=val2'使用POST方式请求，类型默认是`x-www-form-urlencoded`，也可多次使用-d；`-d @file`是从文件中读取，一行一个；`--data-raw @file`避免识别成文件，就是真正的`@`；`--data-urlencode`自动对值做一次URL编码，像有空格时可用，但key没有编码，如果想要，再在key前面加一个等号；-F的类型是`multipart/form-data`
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
* --cert-status：检查OCSP
* 访问httpbin.org/get可以看到服务器收到的请求信息；reqbin.com/echo/get/json简单的返回请求的body
* URL不会做UrlEncode，而是直接发送。比如get /测试.txt，Linux下为`/\u00e6\u00b5\u008b\u00e8\u00af\u0095`，Win下甚至为GBK编码，正确的是`/%E6%B5%8B%E8%AF%95`
* 只显示各个阶段消耗的时间，需要请求完毕才会输出：`curl -o /dev/null -s -w %{time_namelookup}::%{time_connect}::%{time_starttransfer}::%{time_total}::%{speed_download}"\n" <url>`
* 不自带“下载文件中的所有链接”的功能，可用`xargs -n 1 curl -O < urls.txt`，不要按每一行手动运行因为那样无法利用keepalive
* 其他人做的笔记：https://gist.github.com/subfuzion/08c5d85437d5d4f00e58
* Win版：https://curl.se/windows/

### scp

* scp -rpC src dest，user@host_or_ip:/path/filename
* r为递归，p为保留日期等，C为压缩
* -P指定端口，默认22
* src可有多个文件，如果dest为相对路径则相对于~
* win下的好像无法识别中文路径
* -3可以（通过本机）在两个服务器之间传文件
* src如果以`/`结尾，就是传输文件夹里的内容，不是传输一个文件夹；不以斜杠结尾再加`-r`就能传输一个文件夹

### rsync

* rsync -avzP src dest：a保留文件所有属性且递归，z启用压缩，P断点续传且显示每个文件的进度，v详细模式但在P下只会再多显示一点总结信息
* -H保留硬链接，-u只更新变化了的（文件存在于dest且mtime更新），-n是dry run，--delete删除dest中所有不在src中的文件，-c用校验和而不是时间和大小判断是否不同会大量消耗资源，-vvvv显示debug级别的信息
* 设置--inplace和--append后好像是增量同步；有限速功能避免把服务器带宽占满（scp也有）；Host::/path用的是rsync协议，运行daemon时可以类似ftp提供文件出去，可以设置只读和IP黑白名单；不提供dest等价于运行ll，此时-h才有用；-R的作用：`-rR /var/./log/nginx /tmp`将会创建/tmp/log/nginx；-S发送稀疏文件时使用
* 维护一个local copy：rsync -rlptzv --progress --delete --exclude=.git "user@hostname:/remote/source/code/path" .
* 多线程的管理脚本：https://github.com/pigsboss/toolbox/blob/master/pfetch.py
* TODO：https://www.digitalocean.com/community/tutorials/how-to-use-rsync-to-sync-local-and-remote-directories-on-a-vps https://zhuanlan.zhihu.com/p/331838860

### wget2

* 自动多线程
* -i下载文件中列出的url
* Win版：https://github.com/rockdaboot/wget2/issues/234#issuecomment-968329681 https://www.lumito.net/2020/12/05/released-wget2-1-99-2-for-windows/

### youtube-dl

* -F：列出可用格式；-f使用指定格式，可指定数字或类型，一般手动指定-f best
* --download-archive archive.txt：下载列表时保存已下过的，恢复更快，可用于会更新的列表
* --write-sub --sub-lang zh；--write-auto-sub(仅u2b)，--skip-download仅下载字幕
* -x：把视频转成音频；--audio-quality 9，默认5
* -o：支持以元信息命名文件，如`%(title)s.%(ext)s`
* --playlist-start、end
* 缓存大小是自动调整的
* -s：dry-run
* --add-metadata
* -i：下载列表时跳过出错的；-w强制不覆盖文件

## 文本处理

* wc：-l按行统计，-w按空白分隔的单词统计
* tr old new：替换stdin的内容但功能更多，如`tr 'a-z' 'A-Z'`能把小写的都换成大写的
* cut：-c2,5-8取第2和5到8个字符的内容；-d默认为制表符，-f指定取第几个区域

### awk

* awk 'BEGIN {commands} condition /pattern/ {commands} END {commands}' file，其中模式支持正则
* condition支持变量与常见运算符，等于用=，正则匹配用`~`，正则不匹配用`!~`。pattern为正则匹配过滤$0
* 变量
  * 使用时必不能放在双引号内否则不会扩展
  * `$0`为当期行，无参时也为它，`$1`为第一项
  * NF：本行字段个数
  * 自定义变量可在大括号内直接赋值，支持C风格数组
  * -v从命令行传递参数
* 流程控制（大括号中）
  * if、while、break、continue：与C相同
  * for(变量 in 数组)，也支持与C相同的但不同声明变量类型
* print
  * 输出字符串字面量必须放在双引号内，相邻的自动合并
  * 逗号会扩展为OFS变量的值，默认为空格
  * 会自动换行
* `-F:`指定行内分隔符为冒号；如果为分号，要加引号包裹。相当于设定FS变量
* 函数
  * gsub(x,y,z)：在z中以x为模式正则替换为y，z默认用$0
  * length：无参调用时默认用$0
  * index(str1, str2)：str2在str1中的位置，从1开始，不存在时为0
  * substr(str, from , len)：from从1开始
  * split(str, A, p)：把str按正则模式p分隔，结果保存在A数组中，返回项数
  * 自定义函数：function关键字。一般在 .awk 文件中，用-f执行脚本，从BEGIN{}为入口

### grep

* grep option pattern filename，也支持从stdin读取
* 没必要用普通的grep，egrep对正则的支持更标准，引号中的大括号无需再转义，fgrep完全按本身匹配，-P使用Perl的规则
* 当文件有多个时会在每一行前面打印出匹配到内容的文件名，用-h隐藏；当文件只有一个时，用-H强制显示
* -l：仅输出匹配到的所在的文件名，与xargs配合时一般加-Z以NUL分隔输出
* -o：只显示匹配到的内容而非那一整行；但仅仅是第零组，似乎没有办法输出捕获组
* -c：统计每个文件内匹配到了多少次
* -n：顺便输出行号
* -r：递归搜索，适用于filename是个文件夹
* -i：忽略大小写
* -C2：还输出匹配到了的那行的前后两行
* -q：静默，返回值0为找到了，1为无匹配，2为文件不存在
* -v：反向(invert)，输出不匹配的
* -w：单词，相当于前后加了\b
* --：使得之后以横杠开头的参数不看作选项
* 还有个ugrep貌似速度很快，但是社区不够好，也不稳定

#### ripgrep(rg)

* 模式默认为正则
* 默认递归搜索CWD下所有文件，但排除点开头文件和gitignore，-uu不忽略
* -t指定后缀，-z搜索压缩包
* -E指定编码，默认搜GBK的会乱码
* -r/--replace替换内容，-or '$1'能得到取第一个捕获组的效果
* -c -i -l -w -v -C：与grep相同

### sed

* 选项
  * -i：指定了文件名时，直接修改文件内容，而非输出
  * -e：指示后一个参数为指令而非文件，同时用多个指令时使用。如`sed -e '2,5d' -e '8d'`删除2至5行和第8行，其中那个第8行是删除2-5行前的
  * -r：使正则支持扩展字符，否则用+等字符时需要加反斜杠
* 命令
  * 命令前可加行号，从1开始。如果不加则对每一行都应用
  * a：新增，如 5axxx 在第5行后新增xxx，即xxx在第6行
  * i：插入，如 5ixxx 将xxx插入为第5行，原第5行变为第6行
  * c：替换
  * d：删除，如 /aaa/d 删除所有匹配到了aaa的行，/aaa/,+5d 从匹配到了aaa的行开始再删除后面的5行，1~2d删除奇数行
  * s：替换，如 s/aaa/bbb/g 把所有的aaa都替换为bbb，用`\1`引用匹配组
  * p：通常加-n使得不默认每一行都输出，如 /aaa/p 输出aaa的那一行
  * n：获取下一行，如 /aaa/{n;s/bbb/ccc} 把aaa下一行的bbb换成ccc
  * w：保存，如 /aaa/w data.txt
* 还支持“保持空间”用于储存数据

## 磁盘管理

* smartctl：查看硬盘smart信息，在smartmontools中
* fdisk -l：显示磁盘信息；cfdisk：命令行中的图形化的分区工具
* mkfs.ext4 /dev/xxx：格式化分区。若用xfs需先安装xfsprogs。fstransform：转换文件系统
* du -sh [filename]：显示目录的占用空间；dust：新版du
* df -h：显示挂载点的总大小、已用空间、剩余空间
* dd if=源iso of=/dev/xxx bs=4M status=progress：刻录，需sudo
* fsck：检测文件系统错误
* mount | column -t：显示挂载分区状态
* lsblk：列出块设备。除了RAM外，以标准的树状输出格式，整齐地显示块设备。-l参数为列表；lsusb：显示usb设备
* sshfs：基于ssh的文件系统，能挂载远程文件夹，不自带

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

### crontab

* crontab -l [-u username]：列出当前/某个用户的任务；列出所有用户的任务：`cat /etc/passwd | cut -f 1 -d : |xargs -I {} crontab -l -u {}`
* crontab -e：编辑；-r：删除
* 默认开机会自动启动crond。cron的调度文件：crontab、cron.d、cron.daily、cron.hourly、cron.monthly、cron.weekly
* systemctl list-timers（systemd-timer）
* https://crontab.guru/
* https://zhuanlan.zhihu.com/p/58719487

每次有计划任务运行都会往`/var/log/auth.log`里写一条`pam_unix(cron:session)...`。解决方法：打开`/etc/pam.d/common-session-noninteractive`，往`session required pam_unix.so`前加`session [success=1 default=ignore] pam_succeed_if.so service in cron quiet use_uid`

分布式的：cronsun，国人开发的。

### tmux

* 新建会话：tmux new -s <sessio_name>
* ctrl b + c：新窗口，ctrl b + p：切换到上一个窗口，ctrl b + n：下一个窗口；ctrl b + s：列出并切换窗口
* ctrl b + %：竖分屏，ctrl b + "：横分屏，ctrl b + 方向键：切换panel
* https://github.com/skywind3000/awesome-cheatsheets/blob/master/tools/tmux.txt
* https://zhuanlan.zhihu.com/p/27915505
* https://github.com/gpakosz/.tmux
* https://github.com/tmux-python/tmuxp
* https://pityonline.gitbooks.io/tmux-productive-mouse-free-development_zh/content/index.html
* https://louiszhai.github.io/2017/09/30/tmux/

## systemd

/lib/systemd/system/

### systemctl

* systemctl start（当前启动一次）、stop、enable（开机自启）、disable（禁止自启）、status、restart、try-restart（已启动才重启否则不做操作）、reload-or-restart、reload-or-try-restart、mask（禁止自动和手动启动）、unmask name.service
* 查看已激活的服务：systemctl list-units --type=service；加-a显示所有的；查看所有服务的自启状态：systemctl list-unit-files -t service
* systemctl show --property=Environment docker
* systemctl daemon-reload
* systemctl hibernate（休眠）、hybrid-sleep（交互式休眠）、rescue（进入单用户救援状态）
* chkservice：交互式管理服务的程序 https://zhuanlan.zhihu.com/p/35264613
* machinectl：取代su，polkit取代sudo

### 其它

* systemd-analyze：查看启动耗时；blame指令：查看每个服务的启动耗时；critical-chain [name.service]指令：显示瀑布状的启动过程流
* journalctl：管理日志，取代syslog
* crond 也被 systemd 的 timer 单元取代

## 参考

* https://www.cnblogs.com/manong--/p/8012324.html
* https://blog.csdn.net/aspirationflow/article/details/7694274
* https://www.ruanyifeng.com/blog/2016/03/systemd-tutorial-commands.html
* https://zhuanlan.zhihu.com/p/43687973

### TODO

* killall、pkill、kill -9、pgrep
* nftables：https://zhuanlan.zhihu.com/p/88981486 https://zhuanlan.zhihu.com/p/139678395
* https://www.oschina.net/translate/useful-linux-commands-for-newbies
* https://einverne.github.io/categories.html#每天学习一个命令
* https://github.com/trimstray/the-book-of-secret-knowledge
* IPTraf-ng：监控网络流量
* https://github.com/denisidoro/navi
* fzf https://zhuanlan.zhihu.com/p/91873965 https://github.com/junegunn/fzf https://einverne.github.io/post/2019/08/fzf-usage.html
* nc/ncat
* iface：查看网卡信息
