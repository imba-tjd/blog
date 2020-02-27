---
title: Git/GitHub笔记
---

## OverView

* git init/git clone username@host:/path/to/repository
* git pull [--rebase] [upstream master] = git fetch + git merge（到当前分支）；rebase会先stash当前更改，更新后再pop
* git add -A（所有修改）/.或*（包括修改和新建，不包括删除）/-u（更新，包括修改和删除，不包括新建）
* git status；加-uno会不检测未stage的文件，大大加快执行速度
* git commit -[a]m "message"
* git push origin master

## Branch

* git checkout [-f] patch：切换分支，检出到远端分支时HEAD会变成分离状态。文件发送冲突且不想要当前时可用-f直接切换
* git checkout [-b/B] new [from]：[新建并]切换分支；第二个分支参数可以填要跟踪的远端分支，则push时会更新那个分支。-B相当于先git branch -f再checkout
* git branch [-avv]：不加显示本地，r仅显示remote，a显示所有，v额外显示最后一次commit信息，vv显示对应的远端分支
* git branch -d/D patch：删除分支
* git branch [-f] new [hash]：新建分支，或者如果分支已存在，用-f可以把指定分支强制移动到当前[/指定]位置
* git branch -m [old] new：重命名当前[/指定]分支；重命名远端的只能先删除
* git fetch -p或用remote：更新在远端删除了的分支；但本地分支只会显示未发布，不会主动删除
* 删除远端分支：git push origin --delete [patch]或git push origin :patch
* 把分支推送到远端并设定联系： git push -u origin patch
* 把当前内容建立为一个没有历史的分支：git checkout --orphan new；但注意原来commit了的内容会自动stage，注意gitignore
* git checkout -：切换到上一个分支

### 合并分支

* git merge patch [master]：把patch分支合并到**当前分支**/master里，patch分支指针不变；`--no-ff`可以强制不快速前进
* git cherry-pick [hash1 ...]：选择某几次改动复制到当前branch/HEAD；可用`A..B`表示范围，A应该更老，但此范围不包括A，如果要包括就用A^
* git rebase master [patch]：把**patch**/当前分支以依次复制提交的方式合并到master里，并移动**patch**到master的前面（或之后的节点），master分支指针不变；之后需换一下参数顺序或用下下点快速前进
* git rebase master [patch] [-i]：选择当前[/patch]相对于master（可以为HEAD~n）需要哪些提交或重新排序，**只会更改当前分支**；如果路径上有其他指针，它们会保留；squash最前面必须是pick
* 注意快速前进是切换到落后的分支，git merge/rebase 先进的分支；两者一样是因为rebase没有要复制的，只利用它移动当前到指定之后；可用 git merge -：the '-' is shorthand for the previous branch
* git的rebase和GitHub的rebaes不同：都是在目标分支上再现，但git会移动当前分支到目标分支前面，而目标分支不动；GitHub则是目标分支移动，当前分支不变，相当于rebase后ff master且又把“当前”分支切换回原来的
* 如果要继续在子分支上开发，最好选择merge，这样才能有公共的父结点；否则下一次合并的时候会再把之前的比较一遍，一旦master有提交，就会产生冲突。另一种方法是squash前把master merge进dev，这一步可能产生冲突，是正常现象，否则合并到master本来也会冲突；这样就会产生一个公共结点，再把dev squash进master；如果担心污染dev此处也可以用squash
* 删除已经合并到 master 的分支：`git branch --merged master | grep -v '^\*\|  master' | xargs -n 1 git branch -d`

### 冲突

* 解决冲突后需要add，然后用git merge --continue，不用加分支
* git和GitHub不同：后者是把base往source合并一次，然后再无冲突合并回base；前者直接在merge commit中解决了
* checkout分支时如果工作区内有新文件，不会有问题。如果修改了已存在的文件，就无法直接切换了

## Remote

* git remote add/remove/set-url origin server_url
* git branch [-u] origin/master [foo]：让foo[/当前分支]跟踪**远端**的master分支，如果不用-b新建时指定而是之后修改就要这样
* git remote update origin --prune：与git fetch -p效果一样，但不加分支则默认处理所有分支
* git config branch.master.remote gitee：把master分支的默认push和比较设为指定的远端仓库

## 撤销更改

* git reset --soft：把分支往回移，做的改变留在暂存区（stage/index）里，即已跟踪未提交；git reset --mixed：默认选项，做的改变留在本地（working copy），即未跟踪未提交；git reset --hard 完全与回移到的地点一致，会丢弃本地更改，但不会丢弃未跟踪（从来没有被添加过）的文件
* git reset --soft HEAD^：把东西从commit还原到暂存区里；git reset HEAD：去掉暂存区里的东西；git reset --hard HEAD：丢弃在本地的所有改动；git reset --hard origin/master：丢弃在本地的所有改动与提交
* 如果reset的不是当前分支，则会进入分离模式
* git revert pushed：在**当前分支**上创建一个撤销pushed分支最后一次更改的更改
* git commit --amend：修补最后一次的提交（但hash会变），可以用-m参数只修改信息，或--no-edit只修改提交内容；可以先git rebase -i HEAD~n把之前需要修改的放到最后（用edit），修改后再放回去
* git commit --fixup hash：把stage了的自动写提交信息作为指定hash的修正，之后用rebase --autosquash可以自动合并到一个提交里
* git checkout [hash] -- filename：此命令会使用HEAD中的最新内容/指定commit中的内容替换掉你的工作目录中的文件，已添加到暂存区的改动以及新文件都不会受到影响；可先用git rev-list -n 1 HEAD -- file_path找到删除那个文件的commit，再用hash^即可找回文件；另有git restore做相似的事
* git reset --hard upstream/master：这个命令好像会重新释放一遍指定分支，可能会很耗费资源
* git checkout --merge br：相当于stash, checkout, stash pop

## Config

* global的设置在~/.gitconfig里，system的设置在/etc/gitconfig里，local的设置在git仓库的.git/config里；-e打开文本编辑器进行操作
* color.ui true：彩色的 git 输出；但默认为auto，不需要更改
* --global credential.helper store：储存密码，输入一次以后就会明文放在~/.git-credential里；如果安装了GitGUI则默认为manager，要求输入账户密码时是打开一个网页，Win下可设为wincred，这俩都会储存到控制面板的账户里的Credential Manager中，后者可用git credential-wincred管理
* --global user.email、--global user.name
* gc.pruneexpire "30 days"：不在branch上的30天后清理；gc.auto 0：关闭gc
* core.quotePath false：当路径出现中文时，不会进行转义，即能显示中文
* core.ignorecase false：默认情况下，已经push到远端的文件夹，在本地只修改文件名大小写是不会被检测的；但启用后仅仅只是会push一个另一个大小写的文件过去？可以考虑在Linux端改名，Win端直接删除文件，然后pull
* --global core.editor "code --wait"：编辑信息时要使用的编辑器；不过普通WSL下会有问题，要Remote扩展才行
* --global https.https://github.com.proxy socks5://127.0.0.1:1080：这样可以只访问github时代理。但对ssh无效，ssh要修改`~/.ssh/config`
* --global http.postBuffer 524288000：有人说能加速传输，有人说无效
* --get-all remote.origin.url：获取对应section的值，效果与--list中看到的一样。主要是有的无法被设置，只能用这个看
* feature.manyFiles/experimental true：启用实验性功能
* core.fileMode false：不再将权限变化视为改动
* --unset：删除设置

### 查看diff信息的工具

```
[diff]
    tool = default-difftool
[difftool "default-difftool"]
    cmd = code --wait --diff $LOCAL $REMOTE
```

## 记录

* git blame [filename]：查看文件每一行是由谁在哪次commit中修改的, 按q退出，-w忽略空格变更
* git show [hash]或[branchname]:[filename]：查看某次修改的记录，或其它branch中的文件，加上重定向即可保存到当前分支里；不加参数就是看上一次提交的
* git log --stat：查看提交信息及更新的文件，--shortstat只显示变化了的文件数量和行号
* git log --graph --oneline --decorate --all：通过 ASCII 艺术的树形结构来展示所有的分支
* git shortlog -sn：显示各个作者的提交次数
* git archive --format tar --output /path/to/file.tar master：将master以tar格式打包到指定文件
* git diff --check：检查行尾有没有多余的空白；--name-only --diff-filter=U显示冲突文件列表
* cat .git/HEAD：显示HEAD的指向
* git tag [tagname] [hash] 新建tag，-n显示tag及commit信息，-d删除；git push --tags：推送所有标签；删除远端标签：git push origin :refs/tags/v0.9
* git reflog：查看所有记录，包括reset的
* git log branch1...branch2：显示branch2比branch1多了哪些提交
* git whatchanged xxx --since='2 weeks ago'：查看某文件的修改历史
* [彻底删除文件](https://www.cnblogs.com/shines77/p/3460274.html)：`git filter-branch -f --index-filter 'git rm -r --cached --ignore-unmatch 文件路径' --prune-empty HEAD`；加--all修改所有的分支，prune empty会去掉删除文件后没有任何更改的提交，不加-f在不加-d时会直接失败，`--tag name filter cat --`会不更改tag的名字，-d指定临时操作目录，ignore-unmatch忽略文件不存在时报错失败；如果文件路径里有空格，把外层改成双引号，路径用单引号
* 彻底重命名且不会丢失历史：`git filter-branch -f --tree-filter 'git mv -k 原文件名 新文件名' --prune-empty HEAD`；-k忽略文件不存在时报错失败；会修改本分支所有提交；https://stackoverflow.com/questions/3142419 给了一个用index-filter的示例；如果要移动到之前不存在的文件夹中，命令要加`mkdir -p xxx;`
* git clone --depth=1指的不是只clone根文件夹，而是不clone之前的记录，当前提交还是完整的
* git diff：比较工作目录和staged之间的内容，即add了的与没有add之间的比较，如果没有任何add，就与git diff HEAD一样了。git diff --cached/--staged比较的是add了的与HEAD之间的差别。默认会把修改了的内容都显示出来，加--stat可以只显示文件和变化行数
* git diff master [patch]：比较当前分支/patch与master/目标分支的差别。可以重定向到.patch中，用git apply恢复
* `git config remote.origin.fetch "+refs/heads/*:refs/remotes/origin/*"; git fetch origin`：恢复--single-branch
* git format-patch HEAD^：生成最近一次提交的patch；sha1..sha2生成从前者到后者的patch，每次commit都会对应一个，自动命名；--root可以把整个仓库都patch上。之后可以用git am依次打上，apply的没有记录
* git bundle create repo.bundle HEAD/master可以把当前分支（可同时指定多个分支）整个打包成一个二进制文件，之后路径可以直接当作仓库fetch和clone
* 现在好像出了个替代filter-branch的工具：https://github.com/newren/git-filter-repo

## Pull&Push

* 远端分支（o/master）其实是真·远端在本地的镜像，fetch后就是更新的它
* git push [-u] [-f]：u如果分支与多个远端关联或者没有任何关联，可以用这个设定默认push的主机，等于--set-upstream；f可以强制push，无视远端先于本地和其他冲突，如果不加会被拒绝，只 pull
* git push origin \<source\>:\<destination\>：冒号前是源分支（本地分支，其实是refspec），后是目标分支（远端分支的名字，无需前缀，否则会作为名字的一部分），如果两者不同或者远端不存在那个名字的分支，可以不省略目标分支，把未跟踪的分支push过去，但是不会解决冲突
* git pull也可以指定主机、source（远端分支）、destination（本地分支）。效果是fetch source，把source合并到destination里（如果无法快速前进会被拒绝），再merge（或指定rebase） destination进当前分支（或HEAD）；所以destination不能是HEAD、merge不会改变destination的指针（此时跟source一致），但会改变当前分支
* git fetch/pull origin :branchname：在HEAD处创建本地分支

## 其它git命令

* git checkout HEAD~3表示把HEAD往回移动3次提交，^2用于父提交不止一个的时候移动到分支上。可以链式操作，如git checkout HEAD~3^2
* bash的感叹号有特殊作用，如果commit message里要用，可以用单引号包裹
* git reflog expire --expire-unreachable=now --all显示不在分支上的提交（悬挂提交）；git gc --prune=now：手动清理它们
* git clean -df：删除未跟踪的文件，-x无视gitignore（例如bin），-X只清除ignore的
* src refspec master does not match any：没有任何commit就push
* git bisect：以二分的方式找需要的记录，[https://www.worldhello.net/2016/02/29/git-bisect-on-git.html](https://www.worldhello.net/2016/02/29/git-bisect-on-git.html)
* 在文件夹中添加一个.gitkeep可以上传空文件夹；没有内容的提交：--allow-empty，没有信息的提交：--allow-empty-message
* 列出所有项目中忽略的文件：git ls-files --others --ignored --exclude-standard
* git rebase --rebase-merges/-r、rebase --preserve-merges/-p：没看懂
* git help -g：显示一些内置的教程，git help -a：显示所有的git命令
* git update-ref -d HEAD：把所有的改动都放回工作区并清空所有的commit
* git for-each-ref --sort=-committerdate --format='%(refname:short)' refs/heads/：以最后提交的顺序列出所有分支

## Syncing Fork

* git remote add upstream *url*
* git fetch upstream/--all/git remote update + (git checkout master) + git merge upstream/master
* 但如果本地没有修改，直接用git pull [upstream master]就可以快速前进，不会产生merge commit
* 一次性同步所有远端分支：原生没有这个命令，git pull -all会fetch所有但只会更新HEAD

## git stash

* 用于在有 uncommitted changes 的情况下切换branch，并把改动应用到新的branch上
* git config rebase.autoStash true：每次pull的时候会自动stash当前本地的改动，不用手动stash，并在pull之后stash pop本地更改；不知对普通的rebase是否有效？普通的有--autostash这个参数

```bash
git stash # -u会保存untracked的文件
git checkout [branchname]
git stash pop
git checkout stash@{n} -- file-path # 从stash中拿出某一个文件

git stash list
git stash clear
git show stash@{0} # 查看stash缓存区顶部的改动
git stash apply stash@{1} # 将版本号为stash@{1}的改动应用（不会删除）到当前branch
git stash save "work in progress for foo feature" # 为当前未提交改动加一个注释，并保存到stash
```

## Submodule

* git clone --recursive --shallow-submodules可以自动拉取子模块；否则用git submodule update --init --recursive --depth=1，其中后者的--recursive用于子模块也用了submodule
* git submodule add [-b 目标分支] url [文件夹]：主动添加
* git submodule update --recursive：相当于cd子模块然后checkout，要加--remote才相当于fetch+checkout
* git submodule foreach [--recursive]：因为update和clone已经有了recursive，没必要用这个，只有自己想对所有子模块用别的命令时才用
* 删除submodule：https://stackoverflow.com/questions/1260748；不存在rm命令
* git submodule sync：若`.gitmodules`中的url发生了变化，需要使用此命令把信息更新到`.git/config`中
* 未读：https://github.github.com/training-kit/downloads/submodule-vs-subtree-cheat-sheet/

### Remarks

* git pull --recurse-submodules只会顺便fetch子模块，不会pull/checkout
* git config status.submodulesummary 1：git status时显示子模块的信息
* 修改了子模块但只push了主模块，其他人会遇到问题。git push --recurse-submodules=on-demand可以一并推送子模块，或者用check只警告
* submodule的meta信息储存在`.gitmodule`中
* 默认是分离模式，更新时直接checkout最新的提交，不会更改branch指针，可以在submodule.$name.update中指定merge/rebase
* submodule.$name.branch为要使用的分支
* submodule改变后不能直接看到内容，所以（其他人）容易误操作把老的hash推上来了
* 会出现reset hard加clean df也还是有未staged的情况；以及在另一个分支上添加了子模块，切换到原分支时子模块文件夹会保留，删掉后切换过去又要update --init
* 实际内容保存在父仓库的.git里，子模块的.git只有一个指针。所以删除子模块的文件夹也没事
* 不支持菱形依赖，会出问题：https://stackoverflow.com/questions/1419498；url写成自己时递归更新会无限循环
* 当切换回原来的某个提交时，必须update一下submodule，否则仍是新的
* recursive和recurse-submodules应该是同一个命令，但是现在文档中只有后者

### 比较

* 适用于子仓库是独立封装好了的情形（模块/组件），只在父仓库中引用子仓库的某一个提交。在子仓库目录中时，就相当与在一个单独的仓库内，对父仓库完全不可见。push父仓库时不会push子仓库的代码；如果对子仓库（的引用）做了改变，会显示**一个**名字为子仓库名的**文件**改变了
* subtree适用于进行系统级的开发的情形，会把子仓库的历史直接全部放到父仓库里（可squash为一个提交），但不会有子仓库的branch。大概相当于全部rebase到自己的仓库里。修改子仓库后在父仓库的status能直接看到
* submodule is link, subtree is copy

## 处理换行符

### 提交时忽略空格更改

* git merge -Xignore-space-change --no-ff patch-1：换行符不会导致自动合并冲突。如果仅仅只有换行符的改变，则不会变；否则**保留更改过后的**。但与`ignore-all-space`参数的区别不明；试过一次没有效果
* `git diff -U0 -w | git apply --cached --ignore-whitespace --unidiff-zero -`：在未add且未commit时使用；用完后会给warning意义不明，去掉-U0会消失，$?仍是0；末尾的横线用处不明；如果没有可以合并的，会报unrecognized input错误，属正常现象，此时$?不是0。如果某一行发生了更改，末尾的空格改变不会忽略；但是现在遇到了`fatal: corrupt patch at line`的问题，且没有`--no-verify`可用

### 全局设置

* Windows下：git config --global core.autocrlf true，提交时转换为LF，检出时转换为CRLF；如果本地已经是LF了，提交不会改变本地文件，但reset时会
* Linux下：git config --global core.autocrlf input，提交时转换为LF，检出时不转换
* 关闭自动转换：git config --global core.autocrlf false
* 允许提交混合换行符：git config --global core.safecrlf true/false/warn

### 设置.gitattributes

* 自动转换：* text=auto
* 整个仓库固定CRLF/LF：* text=crlf/lf；指定文件：*.txt eol=crlf
* 不进行自动转换：*.txt binary
* 修改后可用`git add --renormalize .`改变已有文件的换行；使用前需要把当前文件都提交

## SSH & GPG Keys

### [GPG](https://help.github.com/cn/articles/about-commit-signature-verification)

1. gpg2 --list-key *邮件地址*
2. git config user.signingkey ...
3. 需要用的时候必须git commit -S

### [SSH](https://help.github.com/en/articles/connecting-to-github-with-ssh)

1. ssh-keygen -o -t rsa -b 4096 -C "email@example.com"
2. cat ~/.ssh/id_rsa.pub | clip
3. Add your public SSH key to your GitLab account
4. ssh -T git@gitlab.com

## Oh My Zsh Alias

* gaa='git add --all'
* gcam='git commit -am'
* gcb='git checkout -b'
* gcf='git config --list'
* gco='git checkout'
* gcp='git cherry-pick'
* glods='git log --graph --pretty
* glum='git pull upstream master'
* gpsup='git push --set-upstream origin $(git_current_branch)'
* grhh='git reset --hard'
* gst='git status'
* gwch='git whatchanged -p --abbrev-commit --pretty=medium'

## .gitignore

* git status --ignored：显示忽略的文件
* git check-ignore -v file：如果提交不了特定的文件，可以用此命令查看文件匹配到了哪个规则；或者可以add -f
* 同一仓库可以在不同文件夹下有不同的.gitignore文件，所有的“全局”只会在同级和子目录生效，无法对父目录起作用；以斜杠开头也表示.gitignore文件所在的目录 （但其实是下一条的特例）
* 当pattern中间不含有斜杠（非路径）时，匹配是全局的（相当于 以**/开头 ）；如果有，则隐式在最前面加斜杠，此时以**/开头会全局匹配
* 星号匹配多个字符（会一直匹配到斜杠才结束），问号匹配单个字符，用方括号表示单个字符匹配列表；以斜杠结尾表示要匹配的是目录（以及里面的），但注意会全局匹配
* .*会匹配以点开头的（不管后面有多少个点），*.*则只不会匹配无后缀的
* 当**在中间时，可以匹配那一部分有或没有的路径，理论上可以匹配多层，但实际有不行的
* 注释用井号，匹配井号用转义，其他的类似
* 使用于被版本控制的情形，用户自己单独定义可以用$GIT_DIR/info/exclude和core.excludesFile
* 如果仓库原本没有此文件，则可以不提交就忽略自己；如果原本有，则不能不提交就忽略自己

### 不忽略 （即要包含）

* 在开头使用叹号
* 无法递归生效；如果父目录被排除，子目录无法再被包含
* 网上的文章说是从上到下解析？但下面这个例子却是从下到上的解析规则
* 只有用了忽略，结尾不加*、加*、加**才会不同

```
# 排除所有内容，除了foo/bar：（必须这么写）
/*
!/foo
/foo/*
!/foo/bar
# 前两行是因为无法递归生效，必须要包含foo目录
# 再排除foo下所有文件，再包含bar（这两个也不可以互换）
```

## 杂项技巧

### 在issue中标签隐藏过长的代码

* https://gist.github.com/ericclemmons/b146fe5da72ca1f706b2ef72a20ac39d
* summary和内容之间需要一个空行，否则markdown样式无法生效

```html
<details>
 <summary>Title</summary>

collapsable content
</details>
```

### 显示代码片段

```
[显示名称，可以是文件名](带hash的文件名#L5-L10)
```

### code owners

* https://help.github.com/en/articles/about-code-owners
* 把`CODEOWNERS`文件放到`/`、 `.github/` 下即可启用，仅对存在此文件的分支的PR有效
* 文件匹配+空格+@用户；从后往前匹配，所以单独一个*要放到最前
* *.js匹配所有js；开头不加/则匹配所有的；末尾为/*不会递归生效，为/才会

## License

* https://zhuanlan.zhihu.com/p/56759711 Github协议详解，详细又易懂
* No License：https://choosealicense.com/no-permission/ 保留所有权利
* CC：https://www.zhihu.com/question/265416787
* 选择开源协议：https://choosealicense.com/

## bare和mirror

* git init --bare xxx.git：用作同步中心，不包含工作区，可以**接受**push，不能使用平常的git命令；命名按照习惯以.git结尾，实际上下下来的就是.git文件夹
* git clone --mirror：隐含bare；普通的clone会把origin作为直接上游，会有跟踪远端的本地分支，没有origin的上游信息；bare直接是本地分支，没有跟踪分支，也没有origin的上游；mirror则有origin的上游，运行git remote update会覆盖所有的refs，与删掉再clone一致
* 不通过fork创建重复的仓库：https://help.github.com/cn/github/creating-cloning-and-archiving-repositories/duplicating-a-repository

## [hub命令行工具](https://github.com/github/hub)

* sudo apt install hub; hub version
* 第一次使用会提示输入用户名和密码，可设置`GITHUB_USER`和`GITHUB_PASSWORD`避免交互，会创建OAuthToken放在`~/.config/hub`；也可用`GITHUB_TOKEN`，这种不会写入config，但可能就每次都要设置（与Actions结合比较方便）
* 好像会默认使用https因为有个issue说它可以用gcm ；config中设置`hub.protocol ssh`可指定协议为ssh
* (commit); hub create; git push -u origin HEAD：自动创建GitHub上的仓库，也可用hub init
* hub clone myrepo：无需写完整UR
* hub clone user/repo; (cd;checkout;commit); hub fork --remote-name origin; hub pull-request：clone别人仓库后方便地fork出自己的，并打开editor提交PR
* hub sync：fast-forward all local branches to match the latest state on the remote
* hub browse -- issues：浏览当前仓库的issue区，如果是其它仓库就把横杠换掉；但我运行的时候说`Please set $BROWSER to a web launcher`
* hub gist create --copy build.log：自动把文件变成gist
* hub ci-status --verbose
* hub release create --copy -F release-notes.txt v2.3.0
* 还有一些操作issue和pr的功能，包括浏览（可指定过滤条件）、创建（可assign）、close、merge；宣传说所有原本的git命令都可用，hub只是做了扩充，可以直接设为git的alias

## 其它问题

### fatal: index file corrupt

删掉`.git/index`再reset一下就好，不过可能未提交的会丢失。

## 参考

* https://rogerdudler.github.io/git-guide/index.zh.html\
* http://www.boydwang.com/2014/01/git-notes/\
* https://www.lugir.com/git-basic.html\
* https://rogerdudler.github.io/git-guide/index.zh.html\
* https://learngitbranching.js.org/\
* https://www.cnblogs.com/kidsitcn/p/4513297.html\
* https://www.liaoxuefeng.com/wiki/0013739516305929606dd18361248578c67b8067c8c017b000\
* https://medium.com/therobinkim/git-add-everything-but-whitespace-changes-deec3dce39f
* https://www.cnblogs.com/kevingrace/p/5690241.html （有错误）
* https://pdf-lib.org/Home/Details/407 （有错误）
* https://git-scm.com/doc
* https://github.com/521xueweihan/git-tips
