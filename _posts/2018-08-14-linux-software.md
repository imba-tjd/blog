---
title: Linux软件
---

## APT

* 使用`grep " install " /var/log/apt/history.log`可查看最近安装的软件，**不包含因为依赖装上的**
* 清理已删除但保留配置的软件包：`sudo apt purge $(dpkg -l | awk '/^rc/ { print $2 }')`
* 可以使用apt install ./xxx.deb直接安装本地的deb包
* 查看更新记录：`cat /var/log/apt/history.log`，而`/var/lib/apt/periodic`中什么也没有
* apt list -i只能用dpkg -l替代，前者无法直接输出到`code -`里
* apt-cache rdepends -i：查询反向依赖
* apt-mark hold/unhold <pkgname>：锁定/解锁版本，可一次指定多个，showhold显示哪些锁定了；还可以编辑`/etc/apt/preferences[.d]`，注意apt-mark不是它的前端

### 软件列表

* ifconfig：在net-tools中；但现在可用if替代
* figlet：把文本转换为某些字符拼凑显示
* apt-transport-https、~~ca-certificates~~：使得APT支持https？后者装curl的时候会自动装上
* curl：还会顺带装openssl
* software-properties-common：含有add-apt-repository
* autoremove python(2)以后会被删除的包：sudo
* mtr： traceroute + ping
* locate：安装后要手动sudo updatedb更新一下数据库，之后 在/etc/cron.daily/locate这个脚本每天自动更新
* netcat
* ag/rg：比grep、ack更快地递归搜索文件内容；https://einverne.github.io/post/2019/09/ripgrep-recursively-searches-directories-using-regex-pattern.html
* jq：json文件处理以及格式化显示，支持高亮；json_pp是perl自带
* fpp：用管道传递给它可以自动把文件染色
* axel：多线程下载工具
* cloc：代码统计工具，能够统计代码的空行数、注释行、编程语言
* https://github.com/sharkdp/fd 现代版的find；https://github.com/sharkdp/bat：现代版的cat
* https://github.com/fail2ban/fail2ban 自动禁止登陆失败次数过多的IP
* authbind：允许普通用户绑定1024以下的端口
* tldr
* gparted：图形化的管理磁盘分区的工具
* network-manager、network-manager-gnome：为了使网络配置尽可能简单而开发的网络管理软件包
* axel：多线程下载工具，-n指定线程数，其他的基本没有要设置的
* pv：用于显示进度，放在两个管道之间或放到最前面起cat的作用
* checkinstall：在make后运行，可能是替代make install的，用于生成deb，方便出问题时卸载
* ripgrep：快速的正则搜索程序，安装后使用rg命令
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
deb https://mirrors.aliyun.com/debian/ stretch main non-free contrib
deb https://mirrors.aliyun.com/debian-security stretch/updates main
deb https://mirrors.aliyun.com/debian/ stretch-updates main non-free contrib
deb https://mirrors.aliyun.com/debian/ stretch-backports main non-free contrib
# deb-src https://mirrors.aliyun.com/debian/ stretch main non-free contrib
# deb-src https://mirrors.aliyun.com/debian-security stretch/updates main
# deb-src https://mirrors.aliyun.com/debian/ stretch-updates main non-free contrib
# deb-src https://mirrors.aliyun.com/debian/ stretch-backports main non-free contrib

# testing
deb https://mirrors.aliyun.com/debian testing main contrib non-free
deb https://mirrors.aliyun.com/debian testing-updates main contrib non-free
deb https://mirrors.aliyun.com/debian-security testing-security main contrib non-free
#deb https://mirrors.aliyun.com/debian testing-backports main contrib non-free
```

### Ubuntu

发行版命名，测试版是focal。可以看出是根据首字母来的。

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
* 依赖安装位置，用pip show能看到：/usr/local/lib/python3.7/site-packages；~/.local/lib/python3.7/site-packages
* 缓存：`%LocalAppData%\pip\Cache`；~/.cache/pip；现在可以使用pip cache purge清除wheel，但是还是有http缓存
* 许多包也能从apt获得，以`python3-`加包名获得；若用pip卸载时提示：`Not uninstalling xxx at /usr/lib/python3/dist-packages, outside environment /usr`，这表明此包是用apt装的

```bash
python3 -m ensurepip --upgrade --default-pip # 一般最终用户不需要调用它，除非安装python时没有装pip
python3 -m pip install -U pip setuptools wheel

pip3 list [--outdated/-o]
pip3 install <package_name> [--upgrade/-U] [--pre预览版] [--user]
pip3 install <local.whl/tar.gz> -or- -e <dir(/*.whl)> -or- setup.py install
pip3 install git+https://github.com/user/repo.git@branch
pip3 show <package_name>
pip3 uninstall：不会卸载依赖，可用pip-autoremove代替
pip3 check：能显示出某个模块的依赖冲突和缺失

# 对于已经装好的包，只要依赖仍然满足，-U只会更新本体。可用--upgrade-strategy eager解决，或直接--force-reinstall
# 更新所有包（自动修复依赖缺失，但因为pip自己的问题，可能产生依赖冲突）：pipupgrade --latest；或pip install -U `pip list -o|awk 'NR>2 {print $1}'`；或python -c "import pkg_resources, subprocess; subprocess.call('pip install --upgrade ' + ' '.join(dist.project_name for dist in pkg_resources.working_set), shell=True)"
#for dist in pkg_resources.working_set:
#    print("pip install --upgrade " + dist.project_name)
#    subprocess.call("pip install --upgrade " + dist.project_name, shell=True)

pipdeptree [-p package1,p2]：显示依赖哪些包，还有check的效果；-r：显示某个包是被那些依赖的
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

* 是用来安装全局的命令行工具的，对标apt和brew，会分别把每个工具放在虚拟空间中防止依赖冲突
* pipx install [--system-site-packages -f]
* pipx ensurepath：自动往PATH中添加`~/.local/bin`；pipx completions：安装自动完成
* run命令只运行不安装，下载的缓存持续几天，支持直接用链接的.py文件；如果命令行和包名不同，用`--spec`
* list、upgrade、upgrade-all（对每个包用only-if-needed策略运行upgrade）、uninstall、uninstall-all、reinstall-all（更新python版本后使用）
* inject pkg dep：往指定包的虚拟环境里装包；runpip pkg pip_commands：在指定包的虚拟环境里运行pip

### 软件列表

* thefuck
* mssql-cli
* [httpie](https://httpie.org/)：专注于http协议的curl的替代品；中文翻译文档：https://keelii.com/2018/09/03/HTTPie/
* qrcode：装好后用管道把东西传给它。貌似命令行也可用（只要字体支持）
* `sublist3r.py -d example.com -e Baidu,Yahoo,Google,Bing,Ask,Netcraft,Virustotal,SSL`：搜寻子域名
* [trash-cli](https://github.com/andreafrancia/trash-cli)：把文件移动到`.Trash`中
* [bleachbit](https://github.com/bleachbit/bleachbit)：清理垃圾的
* ps_mem：显示各个进程的内存占用情况
* Glances：监控所有系统信息（类似于vmstat）
* userpath：添加和验证PATH的程序，也能作为库使用
* fierce：扫描域名，基本上是取附近IP的反查PTR

## Ruby

* gem install lolcat：彩虹颜色的管道输出

### 不在包管理器中的软件

* [chafa](https://github.com/hpjansson/chafa)：在终端中显示图像，支持gif，不过是像素化显示的
* [browsh](https://github.com/browsh-org/browsh)：基于文本的运行于终端的浏览器，图片是像素化显示的
* [deepin-wine-ubuntu](https://github.com/wszqkzqk/deepin-wine-ubuntu)：安装后可安装微信QQ
* VSC 32bit：https://go.microsoft.com/fwlink/?LinkID=760680 官方最后的版本是1.35.1
* [uGet](https://ugetdm.com/)：下载工具，开源但不在GitHub上
* [hfish](https://hfish.io/)：各种蜜罐
* https://github.com/chaitin/xray ：扫描常见的Web安全问题，不开源

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

1. Apache1需要去掉`LoadModule vhost_alias_module modules/mod_vhost_alias.so`和`Include conf/extra/httpd-vhosts.conf`前的井号。Apache2不是这样。把sites-available下的文件软连接到sites-enable下即可启用，mods同理。ssl默认是不启用的。
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

* 卸载：sudo perl  /usr/bin/vmware-uninstall-tools.pl
* 收缩硬盘大小：sudo vmware-toolbox-cmd disk shrink /

## MEGAcmd

* https://mega.nz/linux/MEGAsync/Debian_10.0/amd64/megacmd-Debian_10.0_amd64.deb
* mega-login、mega-put

## rclone

* curl https://rclone.org/install.sh | sudo bash
* rclone config, n, 22(onedrive)
* (mkdir;) rclone mount onedrive: /www/wwwroot/your_ip/onedrive --allow-other --allow-non-empty --vfs-cache-mode writes &

## 参考

* https://zhuanlan.zhihu.com/p/101601709

### TODO

* rclone: https://zhuanlan.zhihu.com/p/104480400
* unattended-upgrades自动更新：https://www.cnblogs.com/sparkdev/p/11376560.html https://zhuanlan.zhihu.com/p/79215691
