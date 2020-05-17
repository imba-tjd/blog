---
title: GitHub Actions
tags:
    - GitHub
    - CI
---

## Event

### push

因为push触发的CI里面如果又push了，不会循环触发；不知其它事件中push会不会触发；或者也许只是不触发自己这个workflow，其它有可能会触发。

### cron

从左到右依次是分、时、日、月、星期。`0 0 * * *`就是每天8点（UTC0点）；星期天是0。

## upload-artifact

* path为要打包的文件，含义是Linux下的文件，即也可以指定文件夹，但只能有一个，星号是无效的；两者都会自动用zip打包一遍
* name是Actions中的artifact中可下载的文件名，不应以zip结尾否则会出现后缀

## Docker

```
# 直接在容器中运行
jobs:
  my_job:
    #runs-on仍要
    container: golang # 若要详细配置，参见 workflow-syntax-for-github-actions#jobsjob_idcontainer

# 快速从hub拉取和使用容器
uses: docker://golang
with:
  args: go version
```

## 设置值

```
- name: get version
  id: get_version
  run: echo "::set-output name=version_tag::${GITHUB_REF/refs\/tags\//}"
- name: release
  env:
    TAG: ${{ steps.get_version.outputs.version_tag }}
```

## 环境

* 软件环境：https://github.com/actions/virtual-environments/blob/master/images/linux/Ubuntu1804-README.md
* 环境变量：https://help.github.com/en/actions/configuring-and-managing-workflows/using-environment-variables https://help.github.com/en/actions/reference/context-and-expression-syntax-for-github-actions
* 直接使用apt必须加sudo，但在容器里就必须不用
* clang默认就是9

### Python环境

* 自带python 2.7和3.6，pip9，但需要pip3 install wheel。而pip升级会失败：ImportError: cannot import name main
* 安装不需要加--user，和Win不一样
* 需要PATH=$PATH:~/.local/bin，注意不同step应该是不共享的。但也可以直接用python3 -m来调用

#### 自带的包

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

### /etc/docker/daemon.json

```json
{ "cgroup-parent": "/actions_job" }
```
