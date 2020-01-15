---
title: GitHub Actions
tags:
    - GitHub
    - CI
---

## Python环境

* 自带python 2.7和3.6，pip9，但需要pip3 install wheel。而pip升级会失败：ImportError: cannot import name main
* 安装不需要加--user，好像是用了虚拟环境，但安装包又不是在当前目录下，不懂
* 需要PATH=\$PATH:\~/.local/bin，注意不同step应该是不共享的。但也可以直接用python3 -m来调用

### 自带的包

```
asn1crypto (0.24.0)
attrs (17.4.0)
Automat (0.6.0)
blinker (1.4)
certifi (2018.1.18)
chardet (3.0.4)
click (6.7)
cloud-init (19.2)
colorama (0.3.7)
command-not-found (0.3)
configobj (5.0.6)
constantly (15.1.0)
cryptography (2.1.4)
distro-info (0.18ubuntu0.18.04.1)
httplib2 (0.9.2)
hyperlink (17.3.1)
idna (2.6)
incremental (16.10.1)
Jinja2 (2.10)
jsonpatch (1.16)
jsonpointer (1.10)
jsonschema (2.6.0)
language-selector (0.1)
MarkupSafe (1.0)
netifaces (0.10.4)
oauthlib (2.0.6)
PAM (0.4.2)
pip (9.0.1)
pyasn1 (0.4.2)
pyasn1-modules (0.2.1)
pygobject (3.26.1)
PyJWT (1.5.3)
pyOpenSSL (17.5.0)
pyserial (3.4)
python-apt (1.6.4)
python-debian (0.1.32)
PyYAML (3.12)
requests (2.18.4)
requests-unixsocket (0.1.5)
service-identity (16.0.0)
six (1.11.0)
ssh-import-id (5.7)
systemd-python (234)
Twisted (17.9.0)
ufw (0.36)
unattended-upgrades (0.1)
urllib3 (1.22)
WALinuxAgent (2.2.40)
zope.interface (4.3.2)
```

## TODO

* 需要学一下如何保留artifacts
