---
title: Linux软件
---

## APT

* 使用`grep " install " /var/log/apt/history.log`可查看最近安装的软件，**不包含因为依赖装上的**
* 清理已删除的软件包：`sudo apt purge $(dpkg -l | awk '/^rc/ { print $2 }')`
* ifconfig：在net-tools中；但现在可用if替代
* figlet：把文本转换为某些字符拼凑显示
* apt-transport-https、ca-certificates：使得APT支持https？
* curl、~~software-properties-common~~
* autoremove python(2)以后会被删除的包：sudo
* mtr： traceroute + ping
* locate：安装后要手动sudo updatedb更新一下数据库，之后 在/etc/cron.daily/locate这个脚本每天自动更新
* netcat

### Debian阿里源

```
# src是获取源代码时用的，不必要
# 如果只是从测试源中安装某一个软件，可用apt -t testing install xxx
deb https://mirrors.aliyun.com/debian/ stretch main non-free contrib
# deb-src https://mirrors.aliyun.com/debian/ stretch main non-free contrib
deb https://mirrors.aliyun.com/debian-security stretch/updates main
# deb-src https://mirrors.aliyun.com/debian-security stretch/updates main
deb https://mirrors.aliyun.com/debian/ stretch-updates main non-free contrib
# deb-src https://mirrors.aliyun.com/debian/ stretch-updates main non-free contrib
deb https://mirrors.aliyun.com/debian/ stretch-backports main non-free contrib
# deb-src https://mirrors.aliyun.com/debian/ stretch-backports main non-free contrib

# testing
deb https://mirrors.aliyun.com/debian testing main contrib non-free
deb https://mirrors.aliyun.com/debian testing-updates main contrib non-free
deb https://mirrors.aliyun.com/debian testing-backports main contrib non-free
deb https://mirrors.aliyun.com/debian-security testing-security main contrib non-free
```

### Ubuntu

发行版命名，现在最新版是eoan，测试版是focal。可以看出是根据首字母来的。

```
deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ disco main restricted universe multiverse
# deb-src https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ disco main restricted universe multiverse
deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ disco-updates main restricted universe multiverse
# deb-src https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ disco-updates main restricted universe multiverse
deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ disco-backports main restricted universe multiverse
# deb-src https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ disco-backports main restricted universe multiverse
deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ disco-security main restricted universe multiverse
# deb-src https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ disco-security main restricted universe multiverse
```

## PIP

### 安装和升级

```bash
python3 -m ensurepip --default-pip
python3 -m pip install --upgrade pip setuptools wheel

pip3 list [--outdated/-o]
pip3 install <package_name> [--upgrade/-U] [--pre] [--user]
pip3 install <local.whl/tar.gz> -or- -e <dir(/*.whl)> -or- setup.py install
pip3 show <package_name>
pip3 uninstall

# 更新所有：
import pip
from subprocess import call
for dist in pip.get_installed_distributions():
    call("pip install --upgrade " + dist.project_name, shell=True)

# error: Microsoft Visual C++ 14.0 is required.
https://www.lfd.uci.edu/~gohlke/pythonlibs/

# conda也可以管理，不过也许和pip有兼容性问题
```

#### 国内源

```conf
# %APPDATA%\pip\pip.ini；~/.config/pip/pip.conf；-i
[global]
#timeout = 6000
index-url = https://mirrors.aliyun.com/pypi/simple/
[install]
trusted-host = mirrors.aliyun.com
```

* 二进制安装位置：/usr/local/bin/；~/.local/bin/；%AppData%\\Python\\Python38\\Scripts。Linux用PATH=$PATH:...添加，但好像全局的是默认添加的
* 程序安装位置，用pip show能看到：/usr/local/lib/python3.7/site-packages；~/.local/lib/python3.7/site-packages
* 安装pip本身和一些库：python3-dev python3-venv python3-pip，Linux下安装后的名称只会是pip3
* 删除python2：apt purge python2.7-minimal libpython2.7-minimal，但可能造成已有的程序无法启动。如果想改python这个命令，可以用alternatives，不要直接删了然后ln
* pip升级：python3 -m pip install --upgrade pip setuptools wheel
* 缓存：`%LocalAppData%\pip\Cache`；~/.cache/pip

* thefuck
* mssql-cli
* [httpie](https://httpie.org/)：专注于http协议的curl的替代品；中文翻译文档：https://keelii.com/2018/09/03/HTTPie/
* qrcode：装好后用管道把东西传给它。貌似命令行也可用（只要字体支持）

## Ruby

* gem install lolcat：彩虹颜色的管道输出
* gem install travis

### 不在包管理器中的软件

* [chafa](https://github.com/hpjansson/chafa)：在终端中显示图像，支持gif，不过是像素化显示的
* [browsh](https://github.com/browsh-org/browsh)：基于文本的运行于终端的浏览器，图片是像素化显示的
* [deepin-wine-ubuntu](https://github.com/wszqkzqk/deepin-wine-ubuntu)：安装后可安装微信QQ
* tldr：https://github.com/tldr-pages/tldr，Debian没有，找bash版本

## 其他

* /usr/games/fortune
* cowsay

## Apache

* 配置文件：/etc/apache2/apache2.conf（Debian系）、/etc/httpd/conf/httpd.conf（RH系）；里面显示端口配置在/etc/apache2/ports.conf
* 网站目录：/var/www/html/
* 操作服务：service apache2/httpd/apachectl start/stop/restart
* 检查配置文件有没有语法错误：apachectl -t或apache2/httpd -t，但它们好像不同
* 配置https：https://mp.weixin.qq.com/s/Tw4UzX73Q7MSw3GJXnpN8A

### 虚拟主机

1. Apache1需要去掉`LoadModule vhost_alias_module modules/mod_vhost_alias.so`和adf`Include conf/extra/httpd-vhosts.conf`前的井号。Apache2不是这样。把sites-available下的文件软连接到sites-enable下即可启用，mods同理。ssl默认是不启用的。
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
* 需要用户名和密码认证才能访问？

## VMware Tools

### 安装

1. mkdir -p /media/cdrom0
2. sudo mount /dev/cdrom /media/cdrom0
3. cd /media/cdrom0
4. tar xvf VMwareTools\* -C /tmp
5. sudo perl /tmp/vmware-tools-distrib/vmware-install.pl-d default
6. ~~sudo umount /media/cdrom0~~

有个开源的版本叫做open-vm-toolbox，但是会和闭源的冲突，功能也不全。

### 使用

* 卸载：sudo perl  /usr/bin/vmware-uninstall-tools.pl
* 收缩硬盘大小：sudo vmware-toolbox-cmd disk shrink /

## LXDE

* 改变DPI：https://iamjagjeetubhi.wordpress.com/2017/07/01/change-dpi-in-lxde/
