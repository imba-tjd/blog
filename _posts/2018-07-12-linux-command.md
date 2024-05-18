---
title: Linux命令
---

## 命令集合网站

* https://ss64.com/bash/
* https://www.runoob.com/linux/linux-command-manual.html
* https://tldr.inbrowser.app/
* https://cheat.sh/
* http://bropages.org/
* https://command-not-found.com/
* https://cheatography.com/
* https://explainshell.com/
* https://devhints.io/

## 简单笔记

* cal 月 年：日历
* setleds：设置键盘指示灯
* chroot：改变根目录
* date：http://www.runoob.com/linux/linux-comm-date.html %s为时间戳
* xclip：复制到剪切板上，不自带
* htpasswd -nb -B admin password | cut -d ":" -f 2
* fc：在editor中编辑上一条输入的命令，并在退出时执行
* factor：求一个数的所有因数
* logrotate：切割日志的程序
* nroff -man manpage.1：显示man格式的.1文件
* broot：新版tree
* time xxx：计算运行xxx的用时

## xargs

* 把标准输入按空白字符分隔变成指定命令的参数，默认命令是echo
* 默认一次传所有参数，常用-n1一次传1个参数，多次执行
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
* top：显示进程内存/cpu占用信息，会自动更新，-c显示详细的命令；cat /proc/loadavg和uptime：显示负载，信息少，不会自动更新（可用watch运行）
  * htop
    * 支持用鼠标操作
    * 默认以线程而显示，导致出现很多重复的程序，按Shift+H隐藏
    * F5以树状显示，但就无法排序了，只按pid排序
    * RES表示占用的物理内存，无单位时是KB
    * F2设置里可以调整上面的状态和筛选列。可以把宽度改为2:1，CPU改成Average
    * 显示过长的Command：w
    * 选择内容：按住Shift
    * 编译：看Readme，不难。不需要静态编译，ncurses是内核的依赖。编译后要strip
  * btop(c++)：代替bashtop。glances(py)：可选webui和支持容器。nmon、njmon(c)：社区为sourceforge较差，后者适合做二次开发。gtop(node)、bottom(rust)：比较类似于overview面板，能显示网速，有折线图显示CPU和内存历史。iotop
* ps auxf：a显示其它用户的进程，u第一列显示用户，x显示后台进程，f显示父子进程关系但导致不完全按时间排序。直接写数字就是指定pid，-u/g/C分别指定user/group/CMD，不清楚前俩大小写的区别；pstree：以简单形式显示父子程序名关系；在`procps`包中
  * pkill：根据ps的一些Filter来kill，一般就是pkill -fe 进程名，f表示完全匹配防止误杀，e显示杀了的程序
  * pgrep：同理，另外查找进程时不会显示grep自身
  * dalance/procs：rust的重写
* lscpu：相比于`cat /proc/cpuinfo`不会每个核都显示一遍。能显示NUMA信息
* nethogs、nload：显示网速

## 文件处理

* cp . dest/：复制当前文件夹下的内容，而非一个整文件夹；-L/--dereference：如果src是软链接，可复制源文件；cp -a：相当于-dpR
* stat：查看文件的属性；file：以自然语言描述文件格式
* gcp：有进度条的复制工具，不自带；或者rsync -a --progress src dest
* basename，dirname：取得路径的文件名或目录。前者第二个参数传后缀还会去掉它。后者支持任意数量的参数，处理后每个一行
* mkdir -p：创建子目录时，如果父目录不存在，则自动创建；文件夹已存在也不会报错
* ls：-R递归，-r倒序，-t按日期降序，-S按文件大小降序，-d显示当前文件夹自己的信息，-A列出除.和..以外的所有文件
  * eza、lsd：Rust的重写
  * SUID和GUID权限：chmod u+s file，文件运行时会以owner的身份运行，是很大的安全问题，ll时以红底显示，代替x的位置
* cd -：切换到之前的目录
* tail -f：持续输出指定文件，如果有变化立即显示；与less -F相同
* which、whereis：找到程序的路径，其中which只在PATH中找可执行文件，默认只显示直接使用的那一个，用-a显示全部；whereis还在一些系统目录中找且可找二进制、源文件、文档
* rename：把所有.c的文件重命名为.cpp的：`rename 's/.c$/.cpp/' *`
* shasum/md5sum：指定文件时默认二进制模式，从stdin读取时默认文本模式，-c验证
* base64：默认加密，-d解密，-w0加密时输出不换行，支持跟文件或从stdin读取
* ln：参数意义与cp相同，-P硬链接（默认？），-s软链接，-f覆盖dest；src一般要写绝对路径，在-s下src写`./xxx`产生的是相对符号文件的链接而不指当前工作目录下的xxx，后者需用-rs；文件夹一般只能用软链接，root权限下才可用-d创建文件夹硬链接；cp也可创建链接：-l硬链接，-s软链接
* shred：粉碎文件
* FILE=$(mktemp)：在/tmp下创建临时文件并获得文件名
* iconv -f gbk -t utf-8 source-file或省略表示stdin -o target-file
* less：空格或f或z翻一页，d翻半页，回车或e翻一行，b或w上翻一页，u上翻半页，y上翻一行，可以在前面加数字，具体看h帮助；g移动到第一行，G移动到最后一行，/向下搜索，n搜索下一个，N搜索上一个，q退出，v调用editor编辑；-N显示行号，-s合并连续空行
* split -b 50m huge_file分隔文件，合并用cat
* lsof path：查看哪些进程在占用文件。-i显示端口信息

### find

* `find path -name '*.txt' -exec wc -l {}\;` ：统计txt文件有多少个，其中{}会被依次替换成找到的文件
* -name仅匹配基本名，不匹配路径
* -type指定文件类型，f为普通文件，l为链接文件，d为文件夹
* -not -path：排除目录；默认会搜索隐藏目录，但好像不能递归排除；还有个-prune更快
* -size +1m：大于1M的文件
* -mtime -1：一天之内修改的；atime 访问时间，ctime 创建时间
* -print0：与xargs -0配合用
* -delete：直接删除找到的文件
* -maxdepth：最大搜索深度
* exec的结尾必须有\;或';'，这样设计是因为后面可以再加给find的参数，如可以有多个exec。fd保留了此行为，只不过如果确实只有一个-x且在最后则可省
* 进入当前目录下所有子文件夹分别执行命令：`find . -maxdepth 1 -type d ! \( -name ".*" -o -name "System Volume Information" -o -name '$RECYCLE.BIN' \) -exec ./script.sh {} \;`
* https://www.zhihu.com/question/487213837

#### fd

* fd abc相当于`find -iname '*abc*'`，第二个参数指定开始搜索的根目录
* 模式默认为正则，-g改为glob
* 模式只存在小写字母时大小写不敏感，有大写字母时自动变为大小写敏感
* 默认忽略点开头文件和gitignore中的文件，用-HI分别禁用前后者，其中禁用前者会搜到.git里的文件
* 无参使用会递归列出所有文件，相当于模式用`.`
* -t/--type
* -e指定后缀，可多次使用
* 默认只匹配文件名(基本名)，指定-p匹配路径
* -x并行执行外部命令，每项都执行一次；-X执行一次外部命令，把所有项当作那一次的参数；-i交互模式每次执行前询问，-j指定并行数。占位符支持{.}路径去扩展，{/}文件名带扩展，{//}父目录，{/.}文件名无扩展；不存在时默认在最后加一个{}
* -E排除路径，为glob
* 原生不具有-delete功能
* Debian在fd-find包中，安装后的二进制名为fdfind

### 压缩/解压

* unar：自动正确解压非Unicode的zip，可惜最后更新时间2015年
* unzip：不自带，最后更新时间2009年，不支持读取stdin，但支持-p表示输出到stdout
* 分卷zip：先`cat test.zip* > ~/test.zip`合并起来再解压就好了。加密：zipcloak
* gunzip是用来解压gzip(gz)的，不是用来解压zip的
* unrar：是rar官方的，但在non-free中。不支持解压其它任何格式
* upx --lzma
* zless：查看压缩文件内容
* lzip：后缀 .lz，仅使用LZMA非2
* lz4
  * 压缩率很低，比zip低，默认级别为-1，解压速度非常快，用--best会大幅减慢压缩速度且压缩率一般但解压速度不变，-BD增大压缩率
  * 官方CLI是单核的。一般内存压缩用lz4，磁盘文件压缩用zstd，因为IO慢限制lz4发挥不出优势
  * lzo：很老，比lz4压缩率高一点点，压缩速度类似，解压速度大幅落后，不考虑；lzo-rle：增加了一定的速度
  * lz4hc：其实就是高压缩级别的lz4，压缩速度非常慢，但解压速度不变
  * snappy：完全不如lz4
* zstd：后缀 .zst，速度不如lz4但也不错，最高压缩为--ultra -22，根据IO状态动态调整级别用--adapt，多线程用-T0。命令行程序也能处理gz xz lz4
* brotli：后缀 .br，默认已使用最高压缩级别。根据测试，各项都不如zstd，仅在单线程下br好一点
* 比lzma更高压缩率且速度差不多：lzham_codec_devel
* gzip -9

#### tar

* 压缩：tar caf jpg.tar.gz *.jpg。c为压缩，a为自动按后缀选择压缩格式，f指定压缩包名，必须是最后一个参数
* 解压：tar xf abc.tar.gz。x为解压，类型现在都可自动根据后缀识别；最后可跟路径来只解压部分文件
* 其他主选项：t查看内容，r追加，u更新；它们与c和x只能选其中一个
* 其它参数：C指定解压目录必须已存在，v显示详细信息，z代表tar.gz，j代表tar.bz2，-J/--xz代表xz，-I手动指定压缩或解压程序且能设定参数
* 现在Windows也自带了，但只支持gzip
* tgz等价于tar.gz

#### xz

* 压缩：xz -9e data.txt
  * 它会生成data.txt.gz且自动删除源文件，-k保留。只有gzip和它会删
  * 默认-6
* 流压缩：cat data.txt | xz -9e > data.txt.xz
* 解压：xz -d file，用-c输出到stdout
  * 自带软连接unxz相当于-d，xzcat相当于-dc，其他压缩程序一般也支持这类
* 源文件名不会保留，解压文件后的名字就是去掉.xz的部分
* 不做把多个文件打包成一个文件的工作，可指定多个文件但只是分别压缩
* 实际算法为LZMA2和未压缩混合。相比于7z对管道更友好
* 默认单线程，多线程有第三方的pxz

#### 7z

* https://sourceforge.net/projects/sevenzip/files/7-Zip/ 其中extra为精简版的7za，运行不需要附带的dll，差不多只支持7z xz zip gz tar；7za.dll只支持7z，7zxa.dll只支持解压7z，RAR就是使用的它。https://www.7-zip.org/a/7zr.exe 只支持7z
* 第三方版，支持更多算法：https://github.com/mcmilk/7-Zip-zstd
* 压缩：7z a 压缩包名.7z 文件名
  * 文件名以`./`开头时，不保留路径前缀直接把目标添加到压缩包根下
  * 文件夹名后用`/*`会添加内容而非单独一个文件夹
  * 递归处理所有txt：-r *.txt
* 解压：7z x 压缩包名 -o输出目录默认CWD必须不加空格
  * 遇到同名文件如何处理：-ao[a覆盖/s跳过/u自动重命名被释放的文件]
* 其他命令：l列出所有内容，t测试是否出错，d删除压缩包内的文件，u更新文件，e不保留指定前缀解压
* 通配符都由7z自己处理，查看内容时也能用
* 能根据后缀识别压缩包算法
* -p指定密码 -mhe=on加密文件名
* 排除文件：-x!*.7z
* 分卷压缩：-v30m
* -slp：申请大内存页，加快压缩速度，但刚开始时会停一下，需要管理员模式，推荐内存3GB+，压缩大量内容时启用
* -si：从stdin读取，-so：输出到stdout
* 图形化自解压：-sfx7z.sfx
* LZMA2
  * 极限压缩-mx=9，默认=5，=1的压缩率就大于zip
  * -ms=e -mqs 分文件类型的固实压缩，能秒开txt
  * 默认启用多线程，360压缩默认单线程，机械硬盘考虑`-mmt=-`关闭多线程
  * 是LZMA的修改版本，对于不可压缩数据能更好地处理，但是一定要开固实压缩
* ZIP：-mcu使用U8储存文件名，-mcp=xxx解压时使用指定代码页

## 网络

* 防火墙：ufw易用，有gufw图形界面，但yum里没有。firewalld较复杂。其它开源有GUI的：opensnitch portmaster SafeLine(国产WAF)

### dig

* 在dnsutils包中，实际是bind9的客户端
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
* host命令（deprecated）：默认只显示answer，第二个参数可指定服务器。-t指定请求的记录类型，-a效果和dig差不多
* kdig(knot-utils)：功能更多一点，支持DoT(+tls)但不支持DoH。zdns：快速查询大量记录。dnsenum：爆破子域名。doggo：Go写的多彩的解析器，支持Win
* delv：dnsutils 9.10中dig的继任者，对DNSSEC的支持更好

### nmap

* nmap -s[scan method] -p[port range] target
* 目标范围
  * IP范围可以用-和/和逗号和星号，其中-和,在每一分段都能用，单纯的-与星号一样
  * 端口范围可以用-和逗号，且可在前面加U:和T:表示UDP和TCP
  * 默认扫描TOP1000的端口，-F扫TOP100的
  * 从文件中读取目标：-iL
  * 启用IPV6扫描：-6
* 模式
  * 主机发现
    * 在正式扫描前会发ICMP和80和443，只要有一个回复就说明在线。如果不在线，就不进行后续高强度扫描
    * 不进行后续的端口扫描，相当于仅ping：-sn。不进行主机发现，都视为在线：-Pn。其它发现方式略
    * 如果在线则显示：Nmap scan report for xxx; Host is up (xxxs latency)
    * ARP：文档里提到了-PR，但实测局域网内默认强制使用ARP（需root），非局域网IP强制无法使用
  * 端口扫描
    * -sS/sT/sA/sW/sM/sN/sF/sX分别为TCP SYN/Connect()/ACK/Window/Maimon/Null/FIN/Xmas。后三种比较隐蔽，但不支持Win因为某些系统不完全遵循某个RFC
    * 默认是SYN，但非root使用默认Connect()，且好像指定其它模式会静默无效
    * -sU为UDP。如果没有响应，状态为open|filtered。如果端口关闭了，目标默认会返回ICMP端口不可达，但Linux限制了此消息对一个客户端一秒只能发一次，导致扫描非常慢
  * 服务与版本侦测-sV：用于确定端口上运行的具体的应用程序及版本信息，--version-all尝试使用所有的探测手段进行侦测
  * 操作系统扫描：-O
  * -A：完整扫描，相当于几种模式综合，非常慢
  * -sL：仅列出待扫描的IP目标，查看指定的IP范围的解析结果
* -T0-6：扫描时序，越高速度越快，但需要更好的网络环境；默认T3，一般用T4
* --packet-trace：显示发的包，一般配合-n不进行dns解析否则有一些无关信息
* script
  * 保存在安装文件夹的scripts目录下
  * --script-updatedb
  * --script-help xxx
  * --script xxx --script-args xxx
  * -sC 使用默认的脚本
  * whois-ip
  * dns-brute 枚举子域名
  * auth 检测空密码。brute 爆破某些账户
* --badsum: 使用错误的checksum来发送数据包，正常情况下应被丢弃，如果收到回复，说明回复来自防火墙。-f使用较小的ip分段，某些时候可使防火墙检测更困难
* Win版需要安装npcap才能用，一般就用安装包了，不能用绿色版。现在还自带zenmap GUI
* nping
  * 仅用于主机发现时更方便，还能即时显示主机的回应。或者能精细化控制Flag
  * --tcp -p 443。无特权时用--tcp-connect
  * --arp 好像真的可以发广播
  * -c 指定次数
  * echo模式：支持作为客户端和服务端监听某些端口，支持加密
  * 一定程度上支持IPV6，根据过去的经验，要用`--tcp-connect -6`，addr无需也不能用中括号因为端口单独指定。--tcp没有回显
* 有空的时候更新一下中文文档：https://github.com/nmap/nmap/blob/master/docs/man-xlate/nmap-man-zh.xml
* https://nmap.org/book/toc.html 读到3

### netcat/nc

* 现在版本安装的一般是openbsd版的，比gnu版的好
* 端口扫描：nc -vz ip 起始端口-结束端口。会输出每个端口是否打开
* 传输内容或文件：echo xxx | nc ip port -N 遇到EOF时结束否则会一直开着
* 监听：-l 端口。但客户端关闭后服务端也会关闭，无法持续使用；加-k不会关闭，但只允许单客户端连接
* 因为TCP是收发都可以，因此也可以写入listen端的stdin，以及重定向客户端的stdout
* -ku UDP模式
* -c/-e bash_cmd：客户端指定此参数，连接成功后服务端将会执行命令。openbsd版删除了
* ncat是nmap的，但完全兼容nc。debian装ncat就会把nc指向它

### socat

* socat addr1 addr2 将两者的输入输出相互传递
* 两个地址的顺序不重要因为是双向的。加-u表示从左到右
* 用-表示标准输入输出
* 端口转发：socat TCP-LISTEN:8080,fork,reuseaddr TCP:targetip:port
* 其它功能：服务端指定一个命令，当客户端连上后执行，并处理标准输入输出。文件传输。使用Socks或HTTP代理

### mtr

* 命令行在mtr-tiny包中，图形界面在mtr包中
* 默认发ICMP，进入交互式页面，显示每一跳。如果中间丢包多但最后丢包少，那实际就是丢包少
* -T指定TCP，-u指定UDP，-P指定端口。但感觉不太好用，目标没有监听那个端口时不算作丢包，而是说等待回应
* 不进入交互式页面：-rw显示最后汇总结果（但不知为何很慢），-p显示每条结果，-c指定次数
* Win下有Cygwin的版本，不大，但官方不release，而且只支持ICMP。WSL1下需要管理员权限运行控制台，无需sudo
* NextTrace：https://zhuanlan.zhihu.com/p/548617816 https://github.com/nxtrace/NTrace-core

### iproute2

替代net-tools(ifconfig, arp, route, netstat, iptunnel, nameif)。不是替代ipupdown的，它被NetworkManager(Cent)和systemd-networkd(Deb)替代。
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
* -p显示使用端口的进程和uid
* -n阻止把常用端口号解析成服务名，如8080显示成http-alt
* -s显示summary，-e显示额外信息，-o显示keepalive时间信息
* filter过滤连接状态：`ss state/exclude all/connected/各种TCP状态`，help里有
* expression过滤ip和端口：`ss src :22`，冒号前填IP，Peer地址用dst，其它一些过时记法略；可以用and和or连接多个条件，则需要再在外面加个单引号；IP可以用`192.168.0/24`这种形式；运算符可用==、=、!=、ge、le、gt、lt
* Local Address指的是本机，既可以是Listen的，也可以是发出的；Peer Address是“另一端”
* 监听地址为*的是双栈，`[::]`只是V6
* 统计数据（打开的TCP数、状态处于tw的数）：cat /proc/net/sockstat

### curl

* 安装时会装上openssl；支持http2、ftp等多种协议；一次可请求多个URL且会复用，某些选项要多次指定，--next可把接下来的选项都指定为下一个URL的
* 默认显示body，-I用HEAD请求，-i顺便显示Header，`-D –`不限方法只显示头，-X手动指定HTTP请求类型
* -o或>写入文件，-O使用URL的最后一部分作为名字，多URL时用--remote-name-all
* -A指定UA；-H可指定所有Header，用"key: value"，但每个要分开指定
* -x使用proxy（正代），默认端口1080，例外是https时是443。支持http_proxy（必须小写）和HTTPS_PROXY（小写也可）
* `-C -`断点续传
* -e指定referer
* -s安静模式，不显示进度条；-sS安静模式下仍显示错误
* -L跟随30x跳转
* body
  * -d 'k1=v1&k2=v2' -d p3=v3 使用POST方式请求，类型为 x-www-form-urlencoded
  * -d @file 从file文件中读取，一行一个。支持-表示stdin
  * --data-raw @符号也作为普通字符串，不作为读取文件名的标志
  * --data-urlencode 自动对v做URL编码，但key不变。k=@f从文件中读取v并编码
  * -F k=@f 类型指定为 multipart/form-data
  * --json '{"tool": "curl"}' 只是设置一些头，不更改或验证内容，官方推荐配合jo -p k=v | curl --json @- | jq。需7.82
  * -G配合-d拼接url参数
* -k忽略证书错误。--ssl-no-revoke不进行ocsp检查，此检查好像不受--proxy的影响，也可能是WinAPI的缘故
* --compressed：自动添加Accept-Encoding: deflate, gzip, br并自动解码；如果头里手动指定了AE，也必须加此项；Win不支持
* -c/--cookie-jar加文件名保存cookie；-b/-cookie加@文件名读取cookie，-b加"key1=val1;key2=val2"发送在命令行中指定的cookie；文件格式见https://github.com/curl/curl/blob/master/docs/HTTP-COOKIES.md
* url通配：`[1-10]`、`[01-10]`、`[1-10:2]`、`[a-z]`、`{asdf,zxcv}`，-g禁用这一行为；别用bash的展开，因为某些选项如-O只针对随后的一个；在-o的文件名中可用`#1`对应通配变量
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
* Win版：https://curl.se/windows/ Win10最初自带的7.55.1往控制台输出U8网页时会乱码。至少要7.85才支持TLS1.3

### scp

* scp -rpC src dest，user@host_or_ip:/path/filename
* r为递归，p为保留日期等，C为压缩
* -P指定端口，默认22
* src可有多个文件，如果dest为相对路径则相对于~
* win下的好像无法识别中文路径
* -3可以（通过本机）在两个服务器之间传文件
* src如果以`/`结尾，就是传输文件夹里的内容，不是传输一个文件夹；不以斜杠结尾再加`-r`就能传输一个文件夹
* 基于sftp协议

### [nftables](https://wiki.nftables.org)

* 代替iptables。ipset是与iptables配合使用的，而不是封装。bpfilter：作为daemon，将iptables或nftables作为前端，编译成eBPF
* 先创建table
  * family支持ip arp ip6 bridge inet netdev，其中ip仅指v4，也是省略时的默认值；不同family可以有同名table，因此一般不省
  * nft add 族类型 table filter(表名)
  * nft list tables、nft -nn list table filter 其中-nn阻止将ip解析为域名和将端口转换为服务名
* 再创建chain
  * nft add chain 族类型 表名 INPUT(链名) '{ type filter hook input priority 0; policy 默认accept或drop; }' 如果不加单引号就要转义分号或用交互模式
  * type根据table的family不同而不同，一般就是filter
  * hook对于ip有prerouting input forward output postrouting，就是对应iptables预定义的几个
  * 优先级越小越优先，相同优先级的处理顺序未定义
  * policy是rule未匹配时的行为
* 再创建rule
  * nft add rule 族类型 表名 链名 matches statements
  * matches是对三四层协议属性的匹配
  * statements：Verdict语句控制包的control flow，还预定义了一些其它功能如计数、限流
* set：类似于ipset，定义好后再在chain里复用
* nft -i进入交互模式。-f读取配置文件，里面的内容可以是交互模式的内容，或一种避免重复写table和chain的大括号方式。nft list ruleset将现有规则输出成配置
* https://zhuanlan.zhihu.com/p/139678395

## 文本处理

* wc：-l按行统计，-w按空白分隔的单词统计
* tr old new：替换stdin的内容但功能更多，如`tr 'a-z' 'A-Z'`能把小写的都换成大写的，-d删除匹配的。但字符集用的是POSIX风格，不支持正则字符集
* cut：类似于split，-d默认为制表符，-f指定取第几个区域。-c2,5-8取第2和5到8个字符的内容
* sort | uniq -c | sort -nk1,1：排序，统计重复行数，再按第一列的数字从小到大排序（实际此处-n也行）
* paste -s：相当于编程语言的join，把多行输入内容整合成一行。-d,指定分隔符为逗号
* join：TODO:

### awk

* awk 'BEGIN {commands} condition /pattern/ {commands} END {commands}' file。其中模式为正则，BEGIN后的{}只在第一行处理
* condition支持变量与常见运算符，等于用=，正则匹配用`变量 ~ /pattern/`，正则不匹配用`!~`。pattern为正则匹配过滤$0
* 变量
  * 使用时必不能放在双引号内否则不会扩展
  * `$0`为当期行，无参时也为它，`$1`为第一项
  * NF：本行字段个数
  * 自定义变量：大括号内var = xxx，使用时也无需加$。支持C风格数组
  * -v从命令行传递参数
* 流程控制（大括号中）
  * if、while、break、continue：与C相同
  * for(变量 in 数组)，也支持与C相同的但不同声明变量类型
* print
  * 输出字符串字面量必须放在双引号内，相邻的自动合并
  * 逗号会扩展为OFS变量的值，默认为空格
  * 会自动换行
* `-F:`指定行内分隔符为冒号。如果设为分号，要加引号包裹。相当于设定FS变量。默认为空格
* 函数
  * gsub(x,y,z)：在z中以x为模式正则替换为y，z默认用$0
  * length：无参调用时默认用$0
  * index(str1, str2)：str2在str1中的位置，从1开始，不存在时为0
  * substr(str, from , len)：from从1开始
  * split(str, A, p)：把str按正则模式p分隔，结果保存在A数组中，返回项数
  * 自定义函数：function关键字。一般在 .awk 文件中，用-f执行脚本，从BEGIN{}为入口

### grep

* grep option pattern filename，不提供filename时从stdin读取，无法指定文件夹
* 没必要用普通的grep，egrep对正则的支持更标准`? + {} | ()`，引号中的大括号无需再转义，fgrep完全按本身匹配，-P使用Perl的规则
* 当文件有多个时会在每一行前面打印出匹配到内容的文件名，用-h隐藏；当文件只有一个时，用-H强制显示
* -l：仅输出匹配到的所在的文件名，与xargs配合时一般加-Z以NUL分隔输出
* -o：只显示匹配到的内容而非那一整行；但仅仅是第零组，似乎没有办法输出捕获组
* -c：统计每个文件内匹配到了多少次
* -n：顺便输出行号
* -r：递归搜索，适用于filename是个文件夹
* -i：忽略大小写。默认大小写敏感
* -C2：还输出匹配到了的那行的前后两行
* -q：静默，返回值0为找到了，1为无匹配，2为文件不存在
* -v：反向(invert)，输出不匹配的
* -w：单词，相当于前后加了\b
* --：使得之后以横杠开头的参数不看作选项
* 输出文件夹中的每个文件：fgrep '' dir
* zgrep
* ugrep(c++)：社区不够好，不稳定，但号称是普通grep的drop-in替代，而rg没有这个目标

#### ripgrep(rg)

* 模式默认为正则
* 文件名可指定目录，不指定时默认递归搜索CWD（grep默认等待用户输入stdin），且默认排除点开头文件和gitignore，-uu不忽略
* -t指定后缀，-z搜索压缩包
* -E指定编码，默认搜GBK的会乱码
* -P使用PCRE2，支持lookaround和反向引用，但速度稍慢
* -r/--replace替换内容，-or '$1'能得到取第一个捕获组的效果
* -c -i -l -w -v -C：与grep相同

#### fzf

* 开启tui对stdin进行模糊匹配，按行划分，选中回车或鼠标双击，输出结果
  * 搜索开头、结尾、排除：`^music .mp3$ !fire`
  * Ctrl+JK或鼠标滚轮移动条目
  * 多选模式：-m，然后Tab或鼠标右键选择多个
* Bash热键：Alt+C运行cd，Ctrl+R匹配历史，Ctrl+T或两个星号加tab触发在当前位置补全而不会运行。对于kill、ssh等做了额外适配。还能配合bat预览文件内容
  * 通关包管理器安装的默认没有激活热键，要看不同包的说明
  * 普通地按TAB补全（不必输两个星号），但好像无法递归搜索，且无法用fd：https://github.com/lincheney/fzf-tab-completion
* 不用热键，按脚本的方式使用：xxx | fzf | xargs -n1 ...、xxx $(fzf)
* 无参使用时用的是原版find，不会忽略.git，有配置可以调

### sed

* 选项
  * -i：指定了文件名时，直接修改文件内容，而非输出
  * -e：指示后一个参数为指令而非文件，同时用多个指令时使用。如`sed -e '2,5d' -e '8d'`删除2至5行和第8行，其中那个第8行是删除2-5行前的
  * -r(-E)：使正则支持扩展字符，否则用+、括号等字符时需要加反斜杠
* 命令
  * 命令前可加行号，从1开始。如果不加则对每一行都应用
  * 以下的斜杠也可对应换为其他符号，如内容中存在 `http://` 时一般可用下划线
  * a：新增，如 5axxx 在第5行后新增xxx，即xxx在第6行。/aaa/a bbb 找到某一行含有aaa后在下一行插入bbb
  * i：插入，如 5ixxx 将xxx插入为第5行，原第5行变为第6行
  * c：替换
  * d：删除，如 /aaa/d 删除所有匹配到了aaa的行，/aaa/,+5d 从匹配到了aaa的行开始再删除后面的5行，1~2d删除奇数行
  * s：替换，如 s/aaa/bbb/g 把所有的aaa都替换为bbb。用`\1`引用匹配组。不加g则每行最多处理一次
  * p：通常加-n使得不默认每一行都输出，如 /aaa/p 输出aaa的那一行
  * n：获取下一行，如 /aaa/{n;s/bbb/ccc} 把aaa下一行的bbb换成ccc
  * w：保存，如 /aaa/w data.txt
* 还支持“保持空间”用于储存数据

## 持续运行

* ctrl+z：相当于运行了suspend
* fg：把ctrl+z挂起的程序恢复到前台。bg：让ctrl+z挂起的程序在后台运行
* jobs：显示后台挂起的任务
* 使用`%1`指定目标任务，可以用kill杀掉
* 输命令时就指定在后台运行：在最后加个&，一般用`>log.txt 2>&1 &`不让输出到屏幕上。如果整个用小括号括起来，就不在当前终端中，jobs里看不到，应该和disown效果一样。推荐再加nice降低优先级
* 再在前面（但在小括号里）加nohup，则正常退出会话时命令不会结束（异常退出还是会结束）。如果不手动重定向，默认会自动把输出都重定向到nohup.out中，但必须按一下回车交互
* 如果没有用nohup和&就运行了程序，想要退出会话时不结束，用`ctrl+z; bg; disown`，之后那个进程就变成了独立的
* wait命令可以等待后台任务执行完
* 还有一种setsid，格式和nohup类似，不过原理不同，且必须加重定向输出
* 上一个后台运行的程序的PID：`$!`
* https://github.com/Nukesor/pueue 用于交互式运行后台任务。分布式任务框架：SchedMD/slurm

### crontab

* crontab -l [-u username]：列出当前/某个用户的任务；列出所有用户的任务：`cat /etc/passwd | cut -d: -f1 | xargs -I {} crontab -l -u {}`
* crontab -e：编辑；-r：删除
* 默认开机会自动启动crond。cron的调度文件：crontab、cron.d、cron.daily、cron.hourly、cron.monthly、cron.weekly
* systemctl list-timers
* https://crontab.guru/ https://cron-ai.vercel.app https://crontab.cronhub.io/ http://www.cronmaker.com
* https://zhuanlan.zhihu.com/p/58719487

## 参考

* https://www.cnblogs.com/manong--/p/8012324.html
* https://blog.csdn.net/aspirationflow/article/details/7694274
* https://zhuanlan.zhihu.com/p/43687973

### TODO

* killall、kill -9
* https://www.oschina.net/translate/useful-linux-commands-for-newbies
* https://einverne.github.io/categories.html#每天学习一个命令
* https://github.com/trimstray/the-book-of-secret-knowledge
* IPTraf-ng：监控网络流量
* https://github.com/denisidoro/navi
* iface：查看网卡信息
* https://github.com/google/cdc-file-transfer
* open view see edit compose print：都是run-mailcap的alias，根据mime类型调用对应程序
* parallel
