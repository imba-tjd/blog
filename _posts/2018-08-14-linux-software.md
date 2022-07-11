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
# 使用stable可避免手动更新代号
deb https://mirrors.aliyun.com/debian testing main contrib non-free
deb https://mirrors.aliyun.com/debian testing-updates main contrib non-free
deb https://mirrors.aliyun.com/debian-security testing-security main contrib non-free
#deb https://mirrors.aliyun.com/debian testing-backports main contrib non-free
# 如果只是从测试源中安装某一个软件，可用apt -t testing install xxx；好像backports默认就是只有这样才能装的
# deb-src https://mirrors.aliyun.com/debian testing main non-free contrib # 获取源代码时用的，不必要
```

### Ubuntu

发行版命名根据首字母。

```
deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ jammy main restricted universe multiverse
deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ jammy-updates main restricted universe multiverse
deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ jammy-security main restricted universe multiverse
deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ jammy-backports main restricted universe multiverse
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

## Apache2

* 配置文件
  * Debian /etc/apache2/apache2.conf，端口配置 /etc/apache2/ports.conf
  * RH和Win /etc/httpd/conf/httpd.conf
* 网站目录：/var/www/html/
* 操作服务：service apache2/httpd/apachectl start/stop/restart。httpd.exe -k install
* 检查配置文件有没有语法错误：apachectl -t或apache2/httpd -t，但它们好像不同
* Win版：https://www.apachelounge.com/download/ https://www.apachehaus.com/cgi-bin/download.plx
* `make_sock: could not bind to address`：端口已被占用

### 虚拟主机

1. 取消注释`LoadModule vhost_alias_module modules/mod_vhost_alias.so`和`Include conf/extra/httpd-vhosts.conf`。Debian把sites-available下的文件软链接到sites-enable下即可启用，mods同理。ssl默认是不启用的
2. 修改conf/extra/httpd-vhosts.conf。Debian改sites-available里的文件
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

## 下载和传输文件

### MEGAcmd

* dpkg -i https://mega.nz/linux/MEGAsync/Debian_10.0/amd64/megacmd-Debian_10.0_amd64.deb
* 支持交互式命令mega-cmd以及一大堆脚本式命令mega-*，都会自动启动mega-cmd-server；login登录，会话信息保存在home中；默认不开启HTTPS，用https on开启
* get下载且可直接从分享链接下载，put上传，quit退出交互式shell
* 类shell命令：pwd，cd，ls，find，du/df -h，rm；lcd和lpdw为交互式时的local路径
* help列出所有命令，--help和-vvv略；sync本地和云端同步，backup本地定时备份到云端且可保存历史版本，webdav支持文件夹或stream文件，whoami -l显示所有的会话
* Windows版不会自动添加PATH，且经过测试创建软链接无法使用，可以添加PS的alias，$env:LOCALAPPDATA\MEGAcmd\MEGAcmdShell.exe，不需要用脚本式的bat

### rclone

* curl https://rclone.org/install.sh | sudo bash
* rclone config, n, 22(onedrive)
* (mkdir;) rclone mount onedrive: /www/wwwroot/your_ip/onedrive --allow-other --allow-non-empty --vfs-cache-mode writes &
* https://github.com/rclone/rclone https://zhuanlan.zhihu.com/p/104480400

### axel多线程下载

* -n指定线程数，默认4
* -o指定输出文件名，不指定时若目标已存在会自动重命名加.0
* 自动断点续传，根据.st状态文件
* -q静默模式
* 支持HTTP_PROXY，支持设置UA和其它头
* 可同时指定多个url，但不是同时下载多个文件；只会依次使用，连接失败时才换下一个，只要有一个下载成功就结束

### [aria2](https://aria2.github.io/manual/en/html/aria2c.html)

* 支持BT和磁力，不支持ed2k，不支持HTTP2，不支持UPnP
* 支持HTTP_PROXY
* 支持断点续传，只要原本未完成的文件是用aria2下的，再次下相同的URI就自动断点续传，因为会创建同名的.aria2控制文件；若要续传其他软件未完成的顺序下载，加-c

```conf
# 注释支持单行，但不支持行尾，此处为笔记就不管了
# 配置文件放到~/.config/aria2/aria2.conf中
# 其他人的配置：https://github.com/P3TERX/aria2.conf

# 下载
# 默认超时60秒，重试5次，缓存16MB，存在同名文件自动重命名加数字
max-connection-per-server=5 # 可理解为多线程，默认1，命令行中用-x5。还有个max-concurrent-downloads是同时下载多任务，默认5不用改
file-allocation=falloc # 当使用ext4 btrfs xfs NTFS时此项最好，但需要管理员权限
min-split-size=20M # 【默】进行多线程的最小块，此处只有文件大于40M才会启用两个线程，命令行中用-k
# BT
# 当下载的文件是.torrent时，自动开始BT任务
bt-enable-lpd=true
enable-peer-exchange=true
bt-save-metadata=true # 保存磁力链接元数据为种子文件
bt-tracker=https://cdn.staticaly.com/gh/XIU2/TrackersListCollection/master/best_aria2.txt

# 运行为服务
daemon=true # Win无效，WSL有效
dir=xxx # 默认下载目录，不设置时为CWD，命令行中用-d
log=aria2.log # 默认不记录日志
log-level=notice # debug【默】, info, notice【console-log-level默】, warn, error
save-session=aria2.session # 未完成及出错的任务记录，支持.gz后缀自动压缩
save-session-interval=60 # 默认只在退出时保存一次记录
input-file=aria2.session # 原本用于从文件中读取多个下载链接，命令行中用-i且支持-表示stdin，此处用于运行后自动读取恢复记录
# RPC，默认监听双栈localhost:6800
# UI：https://github.com/ziahamza/webui-aria2 https://github.com/mayswind/AriaNg https://aria2c.com/
enable-rpc=true
rpc-allow-origin-all=true # RPC的响应头添加Access-Control-Allow-Origin:*
rpc-listen-all=true # 默认只允许本地回环访问
```

### wget2

* 自动多线程
* -O指定要保存的文件名，不加时默认用URL最后的部分。支持`-q -O-`输出到stdou
* -c断点续传
* -nv只显示错误信息和最基本的信息。默认就启用了verbose，会显示进度条
* -i下载文件中列出的url
* --spider：只检测目标是否存在，可与-i配合批量检测书签
* -b：转入后台下载，日志输出到wget-log文件中
* -m -p -k -P ./local url：镜像一个网页及其依赖文件放到./local里
* Win版：现在作者已经在Release里发Win的二进制了 https://www.lumito.net/2020/12/05/released-wget2-1-99-2-for-windows/ https://github.com/ibook86/wget2-windows。MSYS2的包直接下二进制无法运行

### youtube-dl

* -F：列出可用格式；-f使用指定格式，可指定数字或类型，一般手动指定-f best
* --download-archive archive.txt：下载列表时保存已下过的，恢复更快，可用于会更新的播放列表
* 字幕：--write-sub --sub-lang zh --all-subs，--write-auto-sub --embed-subs，--skip-download仅下载字幕
* 把视频转成音频：-x；--audio-quality 9，默认5
* 保存的文件名，支持元信息：-o `%(title)s.%(ext)s`，默认含有id
* --playlist-start、end
* 缓存大小是自动调整的
* -s：dry-run
* --add-metadata
* -i：下载列表时跳过出错的；-w强制不覆盖文件
* -a links.txt/-
* 多线程下载单个视频：--external-downloader aria2c --external-downloader-args -x5
* 只下载播放列表里所有的链接：`youtube-dl -j --flat-playlist '播放列表链接' | jq -r '.id' | sed 's_^_https://youtu.be/_' > links.txt`

### rsync

* rsync -avzP src dest：a保留文件所有属性且递归，z启用压缩，P断点续传且显示每个文件的进度，v详细模式但在P下只会再多显示一点总结信息
* -H保留硬链接，-u只更新变化了的（文件存在于dest且mtime更新），-n是dry run，--delete删除dest中所有不在src中的文件，-c用校验和而不是时间和大小判断是否不同会大量消耗资源，-vvvv显示debug级别的信息
* 设置--inplace和--append后好像是增量同步；有限速功能避免把服务器带宽占满（scp也有）；Host::/path用的是rsync协议，运行daemon时可以类似ftp提供文件出去，可以设置只读和IP黑白名单；不提供dest等价于运行ll，此时-h才有用；-R的作用：`-rR /var/./log/nginx /tmp`将会创建/tmp/log/nginx；-S发送稀疏文件时使用
* 维护一个local copy：rsync -rlptzv --progress --delete --exclude=.git "user@hostname:/remote/source/code/path" .
* 多线程的管理脚本：https://github.com/pigsboss/toolbox/blob/master/pfetch.py
* TODO：https://www.digitalocean.com/community/tutorials/how-to-use-rsync-to-sync-local-and-remote-directories-on-a-vps https://zhuanlan.zhihu.com/p/331838860

### httpie

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

## VNC和RDP

* https://uvnc.com/
* https://github.com/TigerVNC/tigervnc
* https://www.tightvnc.com/
* https://remmina.org/

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
* croc GO，传输文件，需要服务端
* https://github.com/Code-Hex/pget 类wget，目前还不够完善
