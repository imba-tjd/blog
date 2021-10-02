---
title: Git/GitHub笔记
---

## OverView

* git init/git clone git@github.com:user/repo [本地目标文件夹/.]
* git pull [--rebase] [upstream master] = git fetch + git merge（到当前分支）；rebase会先stash当前更改，更新后再pop
* git add -A（所有修改）/.或*（包括修改和新建，不包括删除）/-u（更新，包括修改和删除，不包括新建）
* git status；加-uno会不检测未stage的文件，大大加快执行速度；-s显示简单信息，且clean时没有输出
* git commit -am "message"
* git push -u origin master

## Branch

* git checkout [-f] dev：检出到远端分支时HEAD会变成分离状态。发生冲突且不想要当前时-f放弃当前内容，或用--merge/-m相当于stash-pop
* git checkout [-b/B] dev [from]：[新建并]切换分支，from可填远端分支或用相同名字，会自动跟踪相当于push -u。-B一般要加from，相当于先branch -f，用于把未提交的改为基于另一个位置
* git checkout -：切换到上一个分支
* git branch [-avv]：无参显示本地，r仅显示remote，a显示所有，v额外显示最后一次commit信息，vv显示对应的远端分支
* git branch -d/D dev：删除分支
* git branch [-f] dev [refspec]：新建分支。或者如果分支已存在，用-f可把指定分支强制移动到当前[/指定]位置
* git branch -m [old] dev：重命名当前[/指定]分支；重命名远端的只能先删除
* 清理在远端删除了的分支：见remote部分
* 删除远端分支：git push origin --delete dev或git push origin :dev
* 把当前内容建立为一个没有历史的分支：git checkout --orphan new；但注意原来commit了的内容会自动stage，注意gitignore
* 可用git switch [-c]替代git checkout [-b]，需手动指定-d才能进入分离模式

### 合并分支

* git merge patch [master]：把patch分支合并到**当前分支**/master里，patch分支指针不变；`--no-ff`可以强制不快速前进；分支名用`-`可指代上一次切换过来的分支
* git cherry-pick [refspec1 ...]：选择某几次改动复制到当前branch/HEAD；可用`A..B`表示范围，A应该更老，但此范围不包括A，如果要包括就用A^
* git rebase master [patch]：把**patch**/当前分支依次复制提交合并到master里，并移动**patch/当前分支**到master前面，master分支指针不变
* git rebase -i HEAD~n：处理当前分支的最后n个提交，只会更改当前分支；如果为refspec，不会修改它，只会改到它后一个；squash最前面必须是pick，直接修改message不会生效要用reword
* 快速前进是切换到落后的分支，merge/rebase先进的分支，两者一样是因为rebase没有要复制的，只利用它移动当前到指定之后
* git的rebase和GitHub的不同：都是在目标分支上再现，但git会移动当前分支到目标分支前面，目标分支不动；GitHub则是目标分支移动，当前分支不变
* 如果要继续在子分支上开发，最好选择merge，这样才能有公共的父结点；否则下一次合并的时候会再把之前的比较一遍，一旦master有提交，就会产生冲突。另一种方法是squash前把master merge进dev，这一步可能产生冲突，是正常现象，否则合并到master本来也会冲突；这样就会产生一个公共结点，再把dev squash进master；如果担心污染dev此处也可以用squash
* 删除已经合并到了 master 的分支：`git branch --merged master | grep -v '^\*\|  master' | xargs -n 1 git branch -d`
* 如果有其它分支依赖了当前分支，不要rebase，否则其它分支就依赖不存在的节点了
* git merge --squash：不会进行提交，只会add

### 冲突

* 发生冲突时，文件状态为unmerged changes，像是staged和local的中间态
* 解决冲突后需要add，或rm，然后git merge --continue，不用加分支名
* git和GitHub解决冲突不同：后者是把base往fork合并一次，再在PR中无冲突合并回base；前者直接在merge commit中解决了
* checkout对于新文件不会有问题。对于local修改了的文件，如果HEAD和目标之间那些文件未修改，也没有问题；但如果修改过，即使不存在冲突，也无法直接切换，此时可用-m尝试合并和解决冲突，但staged可能会丢失。对于staged，如果存在冲突，用-m也无法进行切换，如果不存在冲突，会相当于自动合并一次，如果此时再切换会原分支，staged可能和之前不一样（如果不换回原分支应该没问题）。总之最好不要在有staged的情况下切换分支

## Remote

* git remote add/remove/set-url origin server_url
* git remote show origin：显示一些基本信息，包括url和分支和跟踪情况
* git remote prune origin：清理在远端删除了的分支，但本地分支只会显示未发布，不会主动删除；还可用fetch -p或remote update -p
* git branch [-u] origin/master [foo]：让foo[/当前分支]跟踪**远端**的master分支，如果不用-b新建时指定而是之后修改就要这样；也可在config中修改

## 撤销更改

* git reset --soft：只把分支（和HEAD）往回移，staged/index和local都不变，一般用HEAD^相当于把提交还原到staged中
* git reset --mixed：默认选项，把分支和staged往回移，local不变。如果用HEAD或不加且此时local没有修改，相当于unadd；实际是清除staged，不会把staged中的覆盖local中又修改了的；如果用HEAD^相当于把提交还原到local中
* git reset --hard：完全与回移到的地点一致，会丢弃本地更改，但不会丢弃未跟踪的文件
* git reset --merge：保留staged和local之间的差异文件，把分支和staged回移（staged会丢失），对local中的其它文件hard reset；如果回移中的提交也修改了那些差异文件，会abort；或者可以看成把staged的内容提交了再reset --keep
* git reset --keep：保留HEAD和local之间的差异，相当于先unadd，也相当于stash；我觉得并不类似于checkout -B
* git revert --no-edit pushed：在当前分支上创建一个撤销pushed分支最后一次更改的更改，总之此命令只跟单个提交，不是进行比较。若要在一个提交中撤销多个，可以`git revert --no-commit HEAD~n..`再手动提交；也可以先hard reset到要撤销的提交前，再soft reset要撤销的提交后，再commit
* git commit --amend：修补最后一次的提交（但hash会变），可以用-m参数只修改信息，或--no-edit只修改提交内容；可以先git rebase -i HEAD~n把之前需要修改的放到最后（用edit），修改后再放回去
* git commit --fixup refspec：把add了的自动写提交信息作为指定refspec的修正，之后rebase -i --autosquash自动把这些提交放到合适的位置合并
* git checkout -- filename：将staged的文件恢复到local中，未add那个文件或加了refspec时就相当于hard reset；git reset -- filename：将已提交的文件覆盖到staged中，相当于unadd。新文件不用担心消失，会报错。实际上不加--也能生效，导致可能与分支弄混。现在被git restore --source=refspec f d替代，默认--worktree对应checkout，--staged对应reset，两者同时用对应hard reset
* git reset --hard upstream/master：好像会重新完整释放一遍？如果是就很耗资源
* 如果reset的不是当前分支，则会进入分离模式
* 找回删掉了的文件：一般先用git rev-list -n 1 HEAD -- file_path找到删除那个文件的commit，再用refspec^

## Config

* --global user.email、user.name
* global的设置在~/.gitconfig里，system的设置在/etc/gitconfig里，local的设置在git仓库的.git/config里；-e打开文本编辑器进行操作
* color.ui：彩色的git输出，默认为auto无需更改
* --global credential.helper store：储存密码，输入一次以后就会明文放在~/.git-credential里；如果安装了GitGUI则默认为manager，要求输入账户密码时是打开一个网页，Win下可设为wincred，这俩都会储存到控制面板的账户里的Credential Manager中，后者可用git credential-wincred管理
* gc.pruneexpire "30 days"：不在branch上的30天后清理；gc.auto 0：关闭gc
* core.quotePath false：当路径出现中文时，不会进行转义，即能显示中文
* core.ignorecase false：默认情况下，已经push到远端的文件夹，在本地只修改文件名大小写是不会被检测的；但启用后仅仅只是会push一个另一个大小写的文件过去？可以考虑在Linux端改名，Win端直接删除文件，然后pull
* rebase.autosquash true：rebase -i时自动移动fixup提交的位置
* --global core.editor "code --wait"：编辑信息时要使用的编辑器；不过普通WSL下会有问题，要Remote扩展才行
* --global https.https://github.com.proxy socks5://127.0.0.1:1080：这样可以只访问github时代理。但对ssh无效，ssh要修改`~/.ssh/config`
* --global http.postBuffer 10485760：每块数据的接收大小，默认不超过1M，此为10M，有人说能加速传输，有人说无效
* --get-all remote.origin.url：获取对应section的值，效果与--list中看到的一样。主要是有的无法被设置，只能用这个看
* feature.manyFiles/experimental true：启用实验性功能
* core.fileMode false：不再将权限变化视为改动
* --unset：删除设置
* pull.rebase true：pull自动-r
* help.autocorrect：打错字时自动使用建议的内容

### 查看diff信息的工具

```conf
[diff]
    tool = default-difftool
[difftool "default-difftool"]
    cmd = code --wait --diff $LOCAL $REMOTE
```

## 记录

* git blame [filename]：查看文件每一行是由谁在哪次commit中修改的, 按q退出，-w忽略空格变更
* git show [refspec]:[filename]：查看指定记录或其它branch中的文件，加上重定向即可保存到当前分支里；不加记录就是看上一次提交的
* git log --stat：查看提交信息及更新的文件，--shortstat只显示变化了的文件数量和行号
* git log --graph --oneline --decorate --all：通过 ASCII 艺术的树形结构来展示所有的分支
* git shortlog -sn：显示各个作者的提交次数
* git archive -o repo.zip/.tar.gz HEAD：打包
* git diff --check：检查行尾有没有多余的空白；--name-only --diff-filter=U显示冲突文件列表
* cat .git/HEAD：显示HEAD的指向
* git tag [tagname] [hash] 新建tag，-n显示tag及commit信息，-d删除；git push --tags：推送所有标签；删除远端标签：git push origin :refs/tags/v0.9
* git reflog：查看本地所有变动过的记录，包括不在分支上的；注意clone下来的用此命令只能看到一条clone的记录，远端删除后再clone无法用它恢复；
* git log branch1...branch2：显示branch2比branch1多了哪些提交
* git whatchanged xxx --since='2 weeks ago'：查看某文件的修改历史
* [彻底删除文件](https://www.cnblogs.com/shines77/p/3460274.html)：`git filter-branch -f --index-filter 'git rm -r --cached --ignore-unmatch 文件路径' --prune-empty HEAD`；加--all修改所有的分支，prune empty会去掉删除文件后没有任何更改的提交，不加-f在不加-d时会直接失败，`--tag name filter cat --`会不更改tag的名字，-d指定临时操作目录，ignore-unmatch忽略文件不存在时报错失败；如果文件路径里有空格，把外层改成双引号，路径用单引号
* 彻底重命名且不会丢失历史：`git filter-branch -f --tree-filter 'git mv -k 原文件名 新文件名' --prune-empty HEAD`；-k忽略文件不存在时报错失败；会修改本分支所有提交；https://stackoverflow.com/questions/3142419 给了一个用index-filter的示例；如果要移动到之前不存在的文件夹中，命令要加`mkdir -p xxx;`
* git clone --depth=1指的不是只clone根文件夹，而是不clone之前的记录，当前提交还是完整的
* git diff：比较local和staged之间的内容，如果没有任何add，就与git diff HEAD一样了。git diff --cached/--staged比较的是add了的与HEAD之间的差别。默认会把修改了的内容都显示出来，用--stat可以只显示文件和变化行数，用于获取比status更多的信息
* git diff master [patch]：比较当前分支/patch与master/目标分支的差别。可以重定向到.patch中，用git apply恢复
* `git config remote.origin.fetch "+refs/heads/*:refs/remotes/origin/*"; git fetch origin`：恢复--single-branch
* git format-patch HEAD^：生成最近一次提交的patch；sha1..sha2生成从前者到后者的patch，每次commit都会对应一个，自动命名；--root可以把整个仓库都patch上。之后可以用git am依次打上，apply的没有记录
* git bundle create repo.bundle HEAD/master可以把当前分支（可同时指定多个分支）整个打包成一个二进制文件，之后路径可以直接当作仓库fetch和clone
* 现在好像出了个替代filter-branch的工具：https://github.com/newren/git-filter-repo

## Pull,Push,Fetch

* 远端分支（origin/master）其实是真·远端在本地的镜像，fetch后就是更新的它
* git push --force-with-lease：更安全的rebase后的force push，当本地的远端分支不是最新时不推送，防止覆盖其他人推送了的
* git push origin source:destination：冒号前是源分支（本地分支，其实是refspec），后是目标分支（远端分支的名字，无需前缀，否则会作为名字的一部分），如果两者不同或者远端不存在那个名字的分支，可以不省略目标分支，把未跟踪的分支push过去，但是不会解决冲突
* git pull也可以指定主机、source（远端分支）、destination（本地分支）。效果是fetch source，把source合并到destination里（如果无法快速前进会被拒绝），再merge（或--rebase） destination进当前分支（或HEAD）；所以destination不能是HEAD、merge不会改变destination的指针（此时跟source一致），但会改变当前分支
* fetch默认会取回origin的所有分支，--all取回所有远端；如果指定的分支（包括remote）未关联到本地，不会自动创建分支，只会放到FETCH_HEAD里，需用:localbranch指定目标
* git pull --all会fetch -all，但只会更新HEAD

## 其它git命令

* git checkout HEAD~3表示把HEAD往回移动3次提交，^2用于父提交不止一个的时候移动到分支上。可以链式操作，如git checkout HEAD~3^2
* git reflog expire --expire-unreachable=now --all显示不在分支上的提交（悬挂提交）；git gc --prune=now --aggressive：手动清理它们
* git clean -df：删除未跟踪的文件，-x无视gitignore（例如bin），-X只清除ignore的
* git bisect：以二分的方式找需要的记录，示例https://www.worldhello.net/2016/02/29/git-bisect-on-git.html
* 在文件夹中添加一个.gitkeep可以上传空文件夹；没有内容的提交：--allow-empty，没有信息的提交：--allow-empty-message
* git rebase --rebase-merges/-r、rebase --preserve-merges/-p：没看懂
* git help -g：显示一些内置的教程，git help -a：显示所有的git命令
* git update-ref -d HEAD：把所有的改动都放回local并清空所有的commit
* git for-each-ref --sort=-committerdate --format='%(refname:short)' refs/heads/：以最后提交的顺序列出所有分支
* git worktree add -B gh-pages public upstream/gh-pages：在当前分支的一个文件夹中checkout另一个分支
* git rev-list --all | xargs git fgrep "xxx"：搜索所有历史中指定文字出现地点。git log -S/-G搜索指定内容在哪个提交中变动

## git stash

* 用于在有未提交的情况下切换分支，并把改动应用到新的分支上
* 默认会把HEAD和local之间的修改保存，相当于unadd了
* git config rebase.autoStash true：rebase或pull -r时，如果本地有未提交的内容，自动stash，rebase完后自动pop；否则如果关闭会拒绝执行rebase。可用--autostash单次开启

```bash
git stash
git checkout [branchname]
git stash pop
git checkout stash@{n} -- file-path # 从stash中拿出某一个文件

git stash list
git stash clear
git show stash@{0} # 查看stash缓存区顶部的改动
git stash apply stash@{1} # 将版本号为stash@{1}的改动应用（不会删除）到当前branch
git stash save "work in progress for foo feature" # 为当前未提交改动加一个注释，并保存到stash

git stash -u # 也保存保存untracked的文件，不要用-a，会把ignore的也保存
git stash branch STASHBRANCH # 然而untracked的无法pop，一种办法是此命令建分支，之后好像要discard一下再pop再checkout --merge
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
* 整个仓库固定CRLF/LF：* text=crlf/lf；指定文件：*.bat eol=crlf
* 不进行自动转换：*.txt binary
* 修改后可用`git add --renormalize .`改变已有文件的换行；使用前需要把当前文件都提交

## 仅修改大小写

* 直接只改大小写会检测不到
* git -c core.ignorecase=false <实际命令> 会认为创建了新文件，且不认为删除了老文件
* git mv abc ABC，-n会认为把abc改为ABC/abc，实际会直接失败

## SSH & GPG Keys

### [GPG](https://help.github.com/cn/articles/about-commit-signature-verification)

1. gpg2 --list-key *邮件地址*
2. git config user.signingkey ...
3. 需要用的时候必须git commit -S
4. TODO：https://zhuanlan.zhihu.com/p/34318369 https://zhuanlan.zhihu.com/p/76861431

### SSH

1. ssh-keygen -t ed25519 -C "email@example.com"
2. cat ~/.ssh/id_rsa.pub | clip （Win自带）
3. 把密钥添加到GitHub账户里去
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

* git status --ignored：显示忽略的文件；还有一种git ls-files --others --ignored --exclude-standard
* git check-ignore -v file：如果提交不了特定的文件，可以用此命令查看文件匹配到了哪个规则；或者可以add -f
* 同一仓库可以在不同文件夹下有不同的.gitignore文件，所有的“全局”只会在同级和子目录生效，无法对父目录起作用；以斜杠开头也表示.gitignore文件所在的目录 （但其实是下一条的特例）
* 当pattern中间不含有斜杠（非路径）时，匹配是全局的（相当于 以**/开头 ）；如果有，则隐式在最前面加斜杠，此时以**/开头会全局匹配
* 星号匹配多个字符（会一直匹配到斜杠才结束），问号匹配单个字符，用方括号表示单个字符匹配列表；以斜杠结尾表示要匹配的是目录（以及里面的），但注意会全局匹配
* `.*`会匹配以点开头的（不管后面有多少个点），`*.*`则只不会匹配无后缀的
* 当**在中间时，可以匹配那一部分有或没有的路径，理论上可以匹配多层，但实际有不行的
* 注释用井号，匹配井号用转义，其他的类似
* 个人单独定义自己的可以用$GIT_DIR/info/exclude和core.excludesFile
* 如果仓库原本没有此文件，则可以不提交就忽略自己；如果原本有，则不能不提交就忽略自己
* VSC的搜索会忽略它匹配到的

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

* fatal: index file corrupt：删掉`.git/index`再reset一下，不过可能未提交的会丢失
* src refspec master does not match any：没有任何commit就push
* bash的感叹号有特殊作用，如果commit message里要用，要加上单引号

### 在issue中标签隐藏过长的代码

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
* `*.js`匹配所有js；开头不加/则匹配所有的；末尾为/*不会递归生效，为/才会

### 特殊链接

```
直接从PR获得文本diff：https://patch-diff.githubusercontent.com/raw/<user>/<repo>/pull/<id>.diff
以tar.gz下载源代码：https://github.com/<user>/<repo>/tarball/master
下载最新版release，但文件名需固定；如果是访问页面就去掉download：https://github.com/<user>/<repo>/releases/download/latest/<filename>
```

## License

* http://www.bewindoweb.com/224.html Github协议详解
* No License：https://choosealicense.com/no-permission/ 保留所有权利
* CC：https://www.zhihu.com/question/265416787 https://creativecommons.org/licenses/ https://github.com/creativecommons/creativecommons.org/tree/master/docroot/legalcode；BY是署名/写原作者，SA是允许演绎/再创作且要以相同协议发布，ND是不允许演绎（包括不允许翻译），NC是不用于商业目的
* 选择开源协议：https://choosealicense.com/，放在一起对比https://choosealicense.com/appendix/
* 所有协议：https://opensource.org/licenses/category
* https://www.gnu.org/licenses/gpl-faq.zh-cn.html https://opensource.org/faq

## bare和mirror

* git init --bare xxx.git：用作同步中心，不包含工作区，可以**接受**push，不能使用平常的git命令；命名按照习惯以.git结尾，实际上下下来的就是.git文件夹
* git clone --mirror：隐含bare；普通的clone会把origin作为直接上游，会有跟踪远端的本地分支，没有origin的上游信息；bare直接是本地分支，没有跟踪分支，也没有origin的上游；mirror则有origin的上游，运行git remote update会覆盖所有的refs，与删掉再clone一致
* 不通过fork创建重复的仓库：https://help.github.com/cn/github/creating-cloning-and-archiving-repositories/duplicating-a-repository
* TODO: https://zhuanlan.zhihu.com/p/258961962

## [gh](https://github.com/cli/cli)

* 大部分命令默认都是交互式的
* gh auth login; eval "$(gh completion -s bash)"; gh config set editor "code --wait"; gh ssh-key add ~/.ssh/id_ed25519.pub
* cat cool.txt | gh gist create; gist list; gist view; gist edit
* gh issue view -w
* gh pr checkout xxx
* gh release create v1.2.3 '/path/to/asset.zip#My display label' -F changelog.md
* gh release download --pattern '*.deb' -R user/repo; list; view
* gh repo clone user/repo -- --depth=1（但会自动fetch全部的upstream）
* gh repo create：在有.git的情况下自动创建网页版仓库并添加remote，但不会自动push
* gh repo view user/repo -w -b dev
* gh repo fork：无参时必须在本地repo中调用，用于fork之前clone到本地的非自己repo并自动把origin改成upstream；如果带user/repo参数，则相当于fork并clone，支持depth
* gh api repos/:owner/:repo --jq '.full_name, .description'

## readme渲染顺序

* readme.md -> readme -> readme.txt
* index、index.md、index.html是没用的

## Gist

* readme.md会在打开后自动排到第一，但不影响gist名
* 以`.`开头会排到前面
* 大写字母开头会排在小写字母的前面；但有一次例外：gist名仍用的是大写开头的，而点开后却是小写开头的排在前面，也许是点开后的排序是先字母再大小写？

## 工具

* npx degit user/repo：下最新的tar.gz，缓存起来

## Git客户端

* GitHub Desktop：添加多个repo时，启动后会自动全部fetch，导致电脑很卡
* TortoiseGit：另有SVN版
* Sourcetree：原支持SVN
* VSC
* https://github.com/gitextensions/gitextensions 与Windows Explorer集成
* https://github.com/jesseduffield/lazygit Terminal UI
* https://github.com/FredrikNoren/ungit JS写的，网页界面
* https://aurees.com/ 不开源
* https://www.cycligent.com/git-tool 不开源

收费的：

* https://git-fork.com
* https://www.gitkraken.com/ 学生包里免费
* https://www.sublimemerge.com/
* https://gitblade.com/
* https://www.git-tower.com
* https://www.syntevo.com/smartgit/

## 参考

* https://rogerdudler.github.io/git-guide/index.zh.html
* http://www.boydwang.com/2014/01/git-notes/
* https://www.lugir.com/git-basic.html
* https://learngitbranching.js.org/
* https://www.cnblogs.com/kidsitcn/p/4513297.html
* https://www.liaoxuefeng.com/wiki/0013739516305929606dd18361248578c67b8067c8c017b000
* https://medium.com/therobinkim/git-add-everything-but-whitespace-changes-deec3dce39f
* https://www.cnblogs.com/kevingrace/p/5690241.html （有错误）
* https://git-scm.com/doc
* https://github.com/521xueweihan/git-tips
* https://help.github.com/en/articles/connecting-to-github-with-ssh

### TODO

* git log https://zhuanlan.zhihu.com/p/259380550
* git lfs https://zhuanlan.zhihu.com/p/146683392 https://docs.github.com/cn/github/managing-large-files
* https://git.wiki.kernel.org/index.php/GitHosting
* Git Hooks https://github.com/typicode/husky
* git fetch --deepen有啥区别，如何unshallow指定数量的提交
