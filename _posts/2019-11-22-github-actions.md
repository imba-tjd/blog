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
    paths-ignore:
      - '**.md'
  schedule:
  - cron: "0 0 * * *"

env:
  k: v

jobs:
  build:
    runs-on: ubuntu-latest
    if: "! contains(github.event.head_commit.message, '[ci skip]')"

    steps:
    - uses: actions/checkout@v2 # 官方文档很完善了

    - name: 每一步都可以有名称
      shell: bash # Win默认就是pwsh，且是最新稳定版；注意容器中一般为sh
      run: |
        echo Hello World!;
      env:
        key: val
      continue-on-error: true # 仅本步有效且最后仍为失败

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

* `action.yml`定义元数据
* 输入的变量在环境中用$INPUT_MY_NAME获取

## cache

* 各个语言缓存依赖的例子：https://github.com/actions/cache/blob/main/examples.md

```yaml
- uses: actions/cache@master
  id: cache
  with:
    path: |
      .cache
      <glob patterns>
    key: ${{ runner.os }}-${{ hashFiles('文件名') }}

- if: steps.cache.outputs.cache-hit != 'true'
  run: <创建缓存>
```

## 环境

* 软件环境：https://github.com/actions/runner-images/blob/main/images/linux/Ubuntu2204-Readme.md
* 环境变量：https://help.github.com/en/actions/configuring-and-managing-workflows/using-environment-variables 在某一步骤中写入以供后续使用 `echo k=v >> $GITHUB_ENV`
* 直接使用apt必须加sudo，但在容器里就必须不用
* win自带nuget，但不自带msbuild的可执行文件，要用microsoft/setup-msbuild；dotnet msbuild好像不能用于fx的
* 命令找不到时的报错：`/__w/_temp/xxx.sh: 7: /__w/_temp/xxx.sh: pushd: not found`
* 不要自己装golang-go，会报`compile: version 'xxx' does not match go tool version 'xxx'`

### Python环境

* 目前python3命令行默认3.10
* 需要PATH=$PATH:~/.local/bin，注意不同step应该是不共享的。或者可以试试python3 -m
* setup-python支持设置`cache: pip`缓存全局依赖，默认将requirements.txt作为缓存的key，但又能够解析它使得不锁版本时装最新的，感觉不锁时意义不大。不设置python-version时会用PATH里的

### /etc/docker/daemon.json

```json
{ "cgroup-parent": "/actions_job" }
```

## 收集

* fregante/setup-git-user
* mxschmitt/action-tmate
* nektos/act
* actions/github-script
* crazy-max/ghaction-github-pages
* lycheeverse/lychee-action 检查链接有效性
