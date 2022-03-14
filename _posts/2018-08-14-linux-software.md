---
title: Linux软件
category: linux
---

## APT

* 使用`grep " install " /var/log/apt/history.log`可查看最近安装的软件，**不包含因为依赖装上的**
* 清理已删除但保留配置的软件包：`sudo apt purge $(dpkg -l | awk '/^rc/ { print $2 }')`
* 可以使用apt install ./xxx.deb直接安装本地的deb包
* 查看更新记录：`cat /var/log/apt/history.log`，而`/var/lib/apt/periodic`中什么也没有
* apt list -i只能用dpkg -l替代，前者无法直接输出到`code -`里
* apt-cache rdepends -i：查询反向依赖；-i理论上是只显示已安装的，但实际好像有些未安装的也显示了？
* apt-mark hold/unhold <pkgname>：锁定/解锁版本，可一次指定多个，showhold显示哪些锁定了；还可以编辑`/etc/apt/preferences[.d]`，注意apt-mark不是它的前端
* 使用前最好安装一下gnupg2（apt-key需要）、apt-transport-https、ca-certificates
* apt edit-sources

### 软件列表

* ifconfig：在net-tools中；但现在可用if替代
* figlet：把文本转换为某些字符拼凑显示
* software-properties-common：含有add-apt-repository
* locate：安装后要手动sudo updatedb更新一下数据库，之后 在/etc/cron.daily/locate这个脚本每天自动更新
* netcat
* jq：json文件处理以及格式化显示，支持高亮 https://github.com/stedolan/jq；还有个yq是py的；json_pp是perl自带
* fpp：用管道传递给它可以自动把文件染色
* cloc：代码统计工具，能够统计代码的空行数、注释行、编程语言
* https://github.com/sharkdp/fd 现代版的find；https://github.com/sharkdp/bat：现代版的cat；https://github.com/dundee/gdu 快速的du
* https://github.com/fail2ban/fail2ban 自动禁止登陆失败次数过多的IP，活着
* authbind：允许普通用户绑定1024以下的端口
* gparted：图形化的管理磁盘分区的工具
* network-manager、network-manager-gnome：为了使网络配置尽可能简单而开发的网络管理软件包
* axel：多线程下载工具，-n指定线程数，其他的基本没有要设置的
* pv：用于显示进度，放在两个管道之间或放到最前面起cat的作用
* checkinstall：在make后运行，可能是替代make install的，用于生成deb，方便出问题时卸载
* ssl-cert：方便地自签证书
* nmap
* neofetch：显示一些基本信息，不过需要安装较多依赖。linuxlogo可替代一小部分
* iotop
* sudo strace -p 17187 2>&1：记录指定PID进程进行的系统调用
* virt-what：查看VPS使用了哪种虚拟化技术，如kvm
* ncdu：带有进度条的du

### Debian阿里源

```
# src是获取源代码时用的，不必要
# 如果只是从测试源中安装某一个软件，可用apt -t testing install xxx；好像backports默认就是只有这样才能装的
deb https://mirrors.aliyun.com/debian bullseye main non-free contrib
deb https://mirrors.aliyun.com/debian-security bullseye-security main non-free contrib
deb https://mirrors.aliyun.com/debian bullseye-updates main non-free contrib
deb https://mirrors.aliyun.com/debian bullseye-backports main non-free contrib
# deb-src https://mirrors.aliyun.com/debian stretch main non-free contrib
# deb-src https://mirrors.aliyun.com/debian-security stretch/updates main
# deb-src https://mirrors.aliyun.com/debian stretch-updates main non-free contrib
# deb-src https://mirrors.aliyun.com/debian stretch-backports main non-free contrib

# testing
deb https://mirrors.aliyun.com/debian testing main contrib non-free
deb https://mirrors.aliyun.com/debian testing-updates main contrib non-free
deb https://mirrors.aliyun.com/debian-security testing-security main contrib non-free
#deb https://mirrors.aliyun.com/debian testing-backports main contrib non-free
```

### Ubuntu

发行版命名根据首字母，目前测试版是hirsute

```
deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ eoan main restricted universe multiverse
deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ eoan-updates main restricted universe multiverse
deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ eoan-security main restricted universe multiverse
deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ eoan-backports main restricted universe multiverse
# deb-src https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ eoan main restricted universe multiverse
# deb-src https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ eoan-updates main restricted universe multiverse
# deb-src https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ eoan-backports main restricted universe multiverse
# deb-src https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ eoan-security main restricted universe multiverse
```

### 其它源

* ftp.cn.debian.org
* ftp2.cn.debian.org
* mirrors.tuna.tsinghua.edu.cn
* mirros.ustc.edu.cn
* mirrors.163.com
* mirrors.huaweicloud.com

### 问题

* `N: Download is performed unsandboxed as root as file '/root/xxx.deb' couldn't be accessed by user '_apt'. - pkgAcquire::Run (13: Permission denied)`：https://askubuntu.com/questions/908800 但是好像可以安装成功

## PIP

* 安装pip本身和一些库：python3-dev python3-venv python3-pip python-pip-whl python3-setuptools python3-wheel，Linux下安装后的名称只会是pip3
* 删除python2：apt autoremove python2.7-minimal libpython2.7-minimal，但可能造成已有的程序无法启动。如果想改python这个命令，可以用alternatives，不要直接删了然后ln
* 二进制安装位置：/usr/local/bin/；~/.local/bin/；%AppData%\Python\Python38\Scripts；%LocalAppData%\Programs\Python\Python39\DLLs\Scripts
* 依赖安装位置，用pip show xxx或者python -m site能看到：/usr/local/lib/python3.7/site-packages；~/.local/lib/python3.7/site-packages
* pip cache purge清除缓存；dir显示缓存文件夹，info显示占用大小，list显示缓存了哪些包
* 许多包也能从apt获得，以`python3-`加包名获得；若用pip卸载时提示：`Not uninstalling xxx at /usr/lib/python3/dist-packages, outside environment /usr`，这表明此包是用apt装的
* 对于已经装好的包，只要依赖仍然满足，-U只会更新本体。可用--upgrade-strategy eager全部更新，或--force-reinstall

```bash
python3 -m ensurepip --upgrade --default-pip # 一般用不到，除非安装python时没有装pip
python3 -m pip install -U pip setuptools wheel

pip3 list [--outdated/-o]
pip3 install <package_name> [--upgrade/-U] [--pre预览版] [--user]
pip3 install local.whl/tar.gz -or- 包名 -f <含有whl的文件夹> -or- setup.py install
pip3 install git+https://github.com/user/repo.git@branch # 会下到所有历史且pip维护者拒绝加depth，可改用下zip代替，下不到submodule
pip3 show <package_name>
pip3 uninstall：不会卸载依赖，可用pip-autoremove代替
pip3 check：能显示出某个模块的依赖冲突和缺失

# 更新所有包（能自动修复依赖缺失，但因为pip自己的问题，可能产生依赖冲突）
pip list -o --format=freeze | % {pip install -U $_.split("=")[0]}
# 或pip3 install -U `pip3 list -o | awk 'NR>2 {print $1}'`
# pipupgrade -ly 在win下有各种各样的问题；千万不要装成pip-upgrade了；还有个pipdate只能在Linux上用，pip-review不积极开发了，pip-upgrader好像还能用

pipdeptree [-p package1,p2]：显示依赖哪些包，也有check的效果；-r：显示某个包是被那些依赖的
```

### 国内源

```conf
# %APPDATA%\pip\pip.ini；~/.config/pip/pip.conf；pip -i；pip config set global.index-url xxx
[global]
#timeout = 6000
index-url = https://mirrors.aliyun.com/pypi/simple/
# https://pypi.doubanio.com/simple/ https://mirrors.163.com/pypi/simple/ https://pypi.tuna.tsinghua.edu.cn/simple/
[install]
trusted-host = mirrors.aliyun.com
```

### pipx

* 用于安装全局命令行工具，会把每个工具分别放在虚拟空间中防止依赖冲突
* pipx install xxx
* pipx ensurepath：自动往PATH中添加`~/.local/bin`
* pipx run：相当于npx，还支持.py文件的网页链接
* list、upgrade、upgrade-all（对每个包用only-if-needed策略运行upgrade）、reinstall-all（更新python版本后使用）
* inject pkg dep：往指定包的虚拟环境里装包；runpip pkg pip_commands：在指定包的虚拟环境里运行pip

### 软件列表

* thefuck
* mssql-cli
* qrcode：装好后用管道把东西传给它。貌似命令行也可用（只要字体支持）
* `sublist3r.py -d example.com -e Baidu,Yahoo,Google,Bing,Ask,Netcraft,Virustotal,SSL`：搜寻子域名
* [trash-cli](https://github.com/andreafrancia/trash-cli)：把文件移动到`.Trash`中
* [bleachbit](https://github.com/bleachbit/bleachbit)：清理垃圾的
* ps_mem：显示各个进程的内存占用情况
* Glances：监控所有系统信息（类似于vmstat）
* userpath：添加和验证PATH的程序，也能作为库使用
* fierce：扫描域名，基本上是取附近IP的反查PTR；aiodnsbrute爆破查找域名
* csvkit：一系列命令行工具，包括能把xlsx和json转换成csv。但依赖太多了，还需要装libicu-dev
* harelba/q：在csv和sqlite上运行SQL语句，但没发布PyPI包
* xsv：rust的csv处理工具

## APK

* apk info：列出安装了的包，加包名显示指定包的描述信息，再加-a显示依赖 被依赖 二进制 大小
* apk update
* apk upgrade：升级全局包
* apk add [-u] 包名：安装包
* apk del
* apk search -v
* sed -i -r -e 's/dl-cdn.alpinelinux.org/mirrors.aliyun.com/g' -e 's/v[.0-9]+/latest-stable/g' /etc/apk/repositories
* 缓存：/var/cache/apk

## Ruby

```bash
apt install ruby-dev # 千万不要安装gem这个包
gem sources --add https://gems.ruby-china.com/ --remove https://rubygems.org/
gem install bundler # 也能用apt装，但是会装一大堆依赖，包括gcc和g++
```

* gem install lolcat：彩虹颜色的管道输出

## 不在包管理器中的软件

* [chafa](https://github.com/hpjansson/chafa)：在终端中显示图像，支持gif，不过是像素化显示的
* [browsh](https://github.com/browsh-org/browsh)：基于文本的运行于终端的浏览器，图片是像素化显示的
* [deepin-wine-ubuntu](https://github.com/wszqkzqk/deepin-wine-ubuntu)：安装后可安装微信QQ
* VSC 32bit：https://go.microsoft.com/fwlink/?LinkID=760680 官方最后的版本是1.35.1
* [uGet](https://ugetdm.com/)：图形化下载工具，开源但不在GitHub上
* [hfish](https://hfish.io/)：各种蜜罐
* https://github.com/chaitin/xray ：扫描常见的Web安全问题，不开源
* [Teleconsole](https://www.teleconsole.com/)：分享当前Shell，也能支持端口转发

## 其他

* /usr/games/fortune
* cowsay
* gnome-tweak
* 桌面版debian删除不用的程序：apt autoremove libreoffice* thunderbird smplayer smtube (不知有无mpv和vlc*和mplayer，有可能是smtube的依赖)

## Apache

* 配置文件：/etc/apache2/apache2.conf（Debian系）、/etc/httpd/conf/httpd.conf（RH系）；里面显示端口配置在/etc/apache2/ports.conf
* 网站目录：/var/www/html/
* 操作服务：service apache2/httpd/apachectl start/stop/restart
* 检查配置文件有没有语法错误：apachectl -t或apache2/httpd -t，但它们好像不同

### 虚拟主机

1. Apache1需要去掉`LoadModule vhost_alias_module modules/mod_vhost_alias.so`和`Include conf/extra/httpd-vhosts.conf`前的井号。Apache2不是这样。把sites-available下的文件软链接到sites-enable下即可启用，mods同理。ssl默认是不启用的。
2. Apache1修改conf/extra/httpd-vhosts.conf，Apache2直接改sites-available里的文件。
3. 修改虚拟主机文件

```
<VirtualHost *:80>
    ServerAdmin webmaster@dummy-host2.example.com
    DocumentRoot "/var/www/html" # 网站的根路径
    ServerName www.mydomain.com # hosts中设置的域名
    #ServerAlias *.mydomain.com # 泛域名解析
    ErrorLog "${APACHE_LOG_DIR}/error.log"
    CustomLog "${APACHE_LOG_DIR}/access.log" combined
</VirtualHost>
```

```
<Directory /var/www/>
    Options Indexes FollowSymLinks MultiViews
    AllowOverride None
    Order deny,allow
    Allow from all
    Require all granted
</Directory>
```

* 如果要使用别的端口，需要在ports.conf里Listen
* Indexes前加-或者删掉可以禁止目录浏览
* MultiViews可以让访问时不输入后缀(index)，它会以一个默认的列表进行匹配（比如先html再php）
* 关于黑白名单（allow和deny）：http://www.nowamagic.net/academy/detail/1225512
* deny 与allow之间没有空格只有一个逗号隔开
* 如何设置只允许域名访问，不允许直接ip访问？

### .htaccess

```
<Files ~ "^.(htaccess|htpasswd)$">
deny from all
</Files>
RewriteEngine on
RewriteCond %{HTTP_HOST} ^(www\.ttps:/)(:80)? [NC]
RewriteRule ^(.*) https://$1 [R=301,L]
Redirect permanent /123 https://target
order deny,allow
```

## VMware Tools

### 安装

1. ~~mkdir -p /media/cdrom0~~
2. ~~sudo mount /dev/cdrom /media/cdrom0~~
3. tar xf /media/cdrom0/VMwareTools* -C /tmp
4. sudo perl /tmp/vmware-tools-distrib/vmware-install.pl
5. 一路回车。上一条命令最后好像能跟default使得所有询问都相当于回车，但除了第一个询问，如果开源版本的tools可用，默认值是no，导致退出
6. ~~sudo umount /media/cdrom0~~

那个`run_upgrader.sh`好像无法使用，反正至少无法用来安装。

有个开源的版本叫做open-vm-toolbox，但是会和闭源的冲突，功能也不全。

### 使用

* 卸载：sudo perl  /usr/bin/vmware-uninstall-tools.pl
* 收缩硬盘大小：sudo vmware-toolbox-cmd disk shrink /

## MEGAcmd

* dpkg -i https://mega.nz/linux/MEGAsync/Debian_10.0/amd64/megacmd-Debian_10.0_amd64.deb
* 支持交互式命令mega-cmd以及一大堆脚本式命令mega-*，都会自动启动mega-cmd-server；login登录，会话信息保存在home中；默认不开启HTTPS，用https on开启
* get下载且可直接从分享链接下载，put上传，quit退出交互式shell
* 类shell命令：pwd，cd，ls，find，du/df -h，rm；lcd和lpdw为交互式时的local路径
* help列出所有命令，--help和-vvv略；sync本地和云端同步，backup本地定时备份到云端且可保存历史版本，webdav支持文件夹或stream文件，whoami -l显示所有的会话
* Windows版不会自动添加PATH，且经过测试创建软链接无法使用，可以添加PS的alias，$env:LOCALAPPDATA\MEGAcmd\MEGAcmdShell.exe，不需要用脚本式的bat

## rclone

* curl https://rclone.org/install.sh | sudo bash
* rclone config, n, 22(onedrive)
* (mkdir;) rclone mount onedrive: /www/wwwroot/your_ip/onedrive --allow-other --allow-non-empty --vfs-cache-mode writes &
* https://github.com/rclone/rclone https://zhuanlan.zhihu.com/p/104480400

## axel多线程下载

* -n指定线程数，默认4
* -o指定输出文件名，如果目标已经存在，会检查是否存在.st状态文件，如果存在就断点续传，不存在就报错
* 在不指定-o时会自动选择文件名，自动断点续传；如果存在同名文件会自动添加.0后缀，不会覆盖，-c指定此种情况时跳过
* -q静默模式
* 支持环境变量设置代理，支持设置UA和其它头
* 可同时指定多个url，但不是同时下载多个文件；只会依次使用，连接失败时才换下一个，只要有一个下载成功就结束

## [aria2](https://aria2.github.io/manual/en/html/aria2c.html)

* 支持BT和磁力，不支持ed2k，不支持HTTP2，不支持UPnP。感觉唯一的用处就是离线下载，或者放在路由器上
* UI：https://github.com/ziahamza/webui-aria2 https://github.com/mayswind/AriaNg https://aria2c.com/
* 命令行：-o 保存的文件名，相对路径为相对dir；--all-proxy=xxx设置代理，无简写方式
* 开启RPC：--enable-rpc，默认监听双栈localhost:6800

```conf
# 注释支持单行，但不支持行尾，此处为笔记就不管了
# 配置文件放到~/.config/aria2/aria2.conf中
# 其他人的配置：https://github.com/P3TERX/aria2.conf http://aria2c.com/usage.html
daemon=true # Win下无效，WSL有效
enable-rpc=true
#rpc-allow-origin-all=true # 设置Access-Control-Allow-Origin:*，用于跨域

dir=xxx # 下载目录，命令行中用-d
log=xxx
log-level=notice # 默认debug
console-log-level=warn # 默认notice
max-connection-per-server=4 # 默认1；命令行中用-x4
#continue=true # 用于没有状态文件时的断点续传，例如其它程序下了一半；有状态文件时是自动的
input-file=xxx/aria2.session # 原本是用于从文件中读取多个下载链接
save-session=xxx/aria2.session # 任务记录
save-session-interval=60
force-save=true # 好像是保留已完成的任务，用于做种
bt-enable-lpd=true
bt-tracker=xxx,xxx
```

## httpie

```cmd
http :8080 # 相当于localhost:8080，只用一个单独的冒号相当于80
http POST httpbin.org/post header:123 q=="你 好" name=John field=@file.txt # 协议、头、查询字符串（内容自动转义）、JSON数据（自动设置Content-Type，默认把value用字符串括起来，需要为数字等时用:=，=@读取文件内容），-f设为form-encoded，file域略
http PUT httpbin.org/put @files/data.xml # 会自动设置Content-Type；重定向标准输入也为原始数据
```

* 依赖requests
* 网卡的时候无法响应Ctrl+C
* 自带json染色，但导致必须完全接收响应才输出，-S关闭缓冲但仍能单行染色；重定向时默认关染色
* 默认请求gzip，UA为HTTPie/xxx
* 默认显示header和body，相当于--print hb，H和B代表请求的；单独的-h或-b是--print h或b，不支持-hb；-h不会使用HEAD，但在接收完头之后就会关闭；-v相当于--all --print=bHBh
* -a user:pass设置basic验证
* -F跟随跳转，--all显示中间响应，--history-print h只显示中间响应的响应头
* --session=./session.json明文保存和使用cookie等信息；如果不含斜杠，只有名字和“后缀”，就会保存到一个特定的httpie/sessions目录下；另有只读会话
* -d下载模式，隐含-F，会在终端显示header但把body保存到文件中，且会显示用时和大小，-o另外指定文件名；单独-o也是只保存body，但不会显示头；-c断点续传；可连起来用-dco加文件名；PS上不要用重定向
* --check-status如果请求失败，在命令行中返回错误代码；--ignore-stdin用户非交互式脚本使用
* --timeout超时时间，默认为0即无限
* http-prompt：同组织的另一个库，交互式的客户端
* https://pie.dev/ 自建的httpbin

## FFmpeg

* ffmpeg 全局参数 输入文件产生 -i 输入文件/文件列表.txt 输出文件参数 输出文件
* -hide_banner隐藏编译参数
* 可不加输出文件参数，则只会查看元数据
* 转换编码：-c:v libx265
* 转换容器会根据后缀自动处理，可用-c copy不重新编码加快速度
* 调整码率：-minrate 964K -maxrate 3856K -bufsize 2000K
* 调整分辨率：-vf scale=480:-1
* 视频转音频：-vn -c:a copy；去除音频流：-an
* 裁剪一段：-ss [start] -to [end]
* 合并：-f concat
* 为Web优化，将元数据放在开头：-movflags faststart

## iperf3

* 服务端：-s [-u]
* 客户端：-c <serverip> -P 5多线程 -b 100M -t 60

## TODO

* unattended-upgrades自动更新：https://www.cnblogs.com/sparkdev/p/11376560.html https://zhuanlan.zhihu.com/p/79215691
* https://github.com/iovisor/bcc
* https://github.com/robertdavidgraham/masscan
* airflow 任务调度
* pyload 离线下载
* supervisor(python)、PM2 (for node.js)
* tasksel：用于安装一组软件
* arping：能检查ip是否重复。arp-scan。ntopng：网络的top，web界面。addrwatch
* rustdesk：远程桌面
* https://github.com/Cyan4973/xxHash
* cloc、boyter/scc：分析repo由哪些语言组成
