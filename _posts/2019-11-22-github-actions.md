---
title: GitHub Actions
tags:
    - GitHub
    - CI
---

https://github.github.io/actions-cheat-sheet/actions-cheat-sheet.pdf

## Workflow

`.github/workflow/main.yml`

```yaml
name: CI
# on: push
on:
  push:
    branches:
      - master
    paths:
      - _site/**
      - .github/workflows/deploy.yml
  schedule:
  - cron: "0 0 * * *"

env:
  k: v

jobs:
  build:
    runs-on: ubuntu-latest

    steps:
    - uses: actions/checkout@v2 # 官方文档很完善了

    - name: 每一步都可以有名称
      shell: bash # Win默认就是pwsh，且是最新稳定版；注意容器中一般为sh
      run: |
        echo Hello World!;
      env:
        key: val

    - uses: ./action-a # 这个点是本仓库，而不是workflow文件夹

    - uses: actions/upload-artifact@v2
      with:
        name: nginx # artifact中可下载的对象的文件名，无需以zip后缀结尾，否则会有双重后缀
        path: /sbin/nginx # 要打包的文件/文件夹的绝对或相对路径，可用|指定多个路径；单个路径只保留basename，多个保留最短公共前缀；支持通配，此时保留通配及之后的路径
        # 对于同一name，若两个文件的basename相同则会覆盖，不同则会都添加进压缩包中


  docker: # 直接在容器中运行整个job
    runs-on: ubuntu-latest # 不可省
    container: golang # 详细配置见 workflow-syntax-for-github-actions#jobsjob_idcontainer

    uses: docker://golang # 快速从hub拉取和使用容器
    with:
      args: go version

if: success()/falure()/caceled() # 官方文档用${{...}}，这个是upload-artifact的实例，不清楚哪个对
```

## Event

### push

因为push触发的CI里面如果又push了，不会循环触发；不知其它事件中push会不会触发；或者也许只是不触发自己这个workflow，其它有可能会触发。

### cron

从左到右依次是分、时、日、月、星期。`0 0 * * *`就是每天8点（UTC0点）；星期天是0。

## [在YAML中的变量](https://help.github.com/en/actions/reference/context-and-expression-syntax-for-github-actions)

```yaml
- name: get version
  id: get_version
  run: echo "::set-output name=version_tag::${GITHUB_REF/refs\/tags\//}"
- name: release
  env:
    TAG: ${{ steps.get_version.outputs.version_tag }}

${{ env.name }}
${{ github.workspace }}
```

## 创建Actions

有两种方式：Docker和JS。对于前者，创建一个Dockerfile，运行entrypoint.sh即可。

`action.yml`定义元数据：

```yaml
name: "Hello Actions"
description: "Greet someone"
author: "octocat@github.com"

inputs:
  MY_NAME: # 在映像中用$INPUT_MY_NAME获取
    description: "Who to greet"
    required: true
    default: "World"

runs:
  using: "docker"
  image: "Dockerfile"

branding:
  icon: "mic"
  color: "purple"
```

## 环境

* 软件环境：https://github.com/actions/virtual-environments/blob/main/images/linux/Ubuntu2004-README.md gcc是9.3，clang是10
* 环境变量：https://help.github.com/en/actions/configuring-and-managing-workflows/using-environment-variables
* 直接使用apt必须加sudo，但在容器里就必须不用
* win自带nuget，但不自带msbuild的可执行文件，要用microsoft/setup-msbuild；dotnet msbuild好像不能用于fx的
* 命令找不到时的报错：`/__w/_temp/xxx.sh: 7: /__w/_temp/xxx.sh: pushd: not found`
* 不要自己装golang-go，会报`compile: version 'xxx' does not match go tool version 'xxx'`

### Python环境

* 默认3.8但有3.9和2.7
* 需要pip3 install wheel；曾经有不要给pip升级的经验
* 需要PATH=$PATH:~/.local/bin，注意不同step应该是不共享的。或者可以试试python3 -m

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
