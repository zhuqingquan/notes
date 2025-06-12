# 经验
1. 代码达到提测标准后，拉release分支进行提测。提测过程中修复bug也是在release分支。
2. 开发的feature分支，先合并到develop，然后再从develop拉出release分支进行提测。

# Git命令备忘

## 换行符

如果同一份代码checkout到移动硬盘，然后希望同时在Windows与linux系统下都能用，这时需要关闭Windows下自动转换为CR LF。[参考](https://deepinout.com/git/git-questions/300_git_how_do_i_recheckout_all_files_in_git_to_convert_from_crlf_to_lf.html#:~:text=Windows%E7%B3%BB%E7%BB%9F%E9%BB%98%E8%AE%A4%E4%BD%BF%E7%94%A8CRLF%E4%BD%9C%E4%B8%BA%E8%A1%8C%E5%B0%BE%E6%A0%87%E8%AF%86%E3%80%82%20%E5%A6%82%E6%9E%9C%E6%82%A8%E5%9C%A8Windows%E7%B3%BB%E7%BB%9F%E4%B8%8A%E4%BD%BF%E7%94%A8%20Git%20%EF%BC%8C%E5%8F%AF%E4%BB%A5%E9%80%9A%E8%BF%87%E4%BB%A5%E4%B8%8B%E5%91%BD%E4%BB%A4%E5%B0%86%20core.autocrlf,%E8%AE%BE%E7%BD%AE%E4%B8%BAtrue%E6%9D%A5%E8%87%AA%E5%8A%A8%E5%B0%86CRLF%E8%BD%AC%E6%8D%A2%E4%B8%BALF%EF%BC%9A%20git%20config%20--global%20core.autocrlf%20true)
```
git config --global core.autocrlf input
```

当已有的代码是在Windows checkout成了CR LF。在linux下打开这些代码git diff将很大差异。此时可以设置autocrlf之后重新checkout。
重新checkout当前代码
```
// 删除所有缓存的文件，但保留在工作目录中的文件
git rm --cached -r .
// 重置当前分支，将其与上游分支相同。这将清除工作目录中的所有更改
git reset --hard
// 重新检出所有文件，以应用配置的行尾转换。这将根据core.autocrlf的配置将CRLF转换为LF
git checkout .
```

[参考](http://kuanghy.github.io/2017/03/19/git-lf-or-crlf)
```
# 提交时转换为LF，检出时不转换
git config --global core.autocrlf input
# 拒绝提交包含混合换行符的文件
git config --global core.safecrlf true
```
cherry-pick或者merge时忽略行尾的换行符或者空格
```
// 支持的-X参数 ignore-space-change, ignore-all-space, ignore-space-at-eol, ignore-cr-at-eol
git cherry-pick -Xignore-cr-at-eol 5b6bcb36eebfac232eb7d22a4d9b8912ac53e0ec
```

## filemode变化的处理
git diff时提示
old mode 100644
new mode 100755
```
git config --add core.filemode false
```


## 查看远程仓库地址
```
git remote -v
```

## 改变commit记录的author提交者
```
// 需要先用 git config user.name/email 设置用户名和邮箱，然后再
git commit --amend --reset-author
```

## 显示commit的详细修改内容，类似diff
```
git show <COMMIT>
git diff <COMMIT>^!
```

### patch或者diff
+ diff可以不提交而直接生成差异文件给别人。patch需要先提交。

使用diff
```
// 将未staged的修改生成diff文件
git diff > modify.diff
// 将已经staged的修改生成diff文件
git diff --staged > modify.diff
git diff  【commit sha1 id】 【commit sha1 id】 >  【diff文件名】
```

使用format-patch
```
//某次提交（含）之前的几次提交：
git format-patch 【commit sha1 id】-n
//某个提交的patch：
git format-patch 【commit sha1 id】-1
//某两次提交之间的所有patch:
git format-patch 【commit sha1 id】..【commit sha1 id】 
```

检查或者应用patch
```
// 加--check就是只检查
git apply --check 【path/to/xxx.patch】
git apply --check 【path/to/xxx.diff】
// for format-patch
git  am 【path/to/xxx.patch】
```

### 版本回滚
#### 将当前Commit到指定SHA的Commit都回滚，再Push
```
git reset --hard dbf5efdb3cd8ea5d576f2e29fe0db1951d0e3e3b
 
# 强制推送到远程分支, 会抹去远程库的提交信息, 不要这么干，可能把代码弄掉
git push -f origin master
```
 
#### 只回滚某个特定版本的修改
```
# 回退到指定版本, 需要解决冲突
git revert e7c8599d29b61579ef31789309b4e691d6d3a83f
 
# 放弃回退(加--hard会重置已 commit和工作区 的内容)
# git reset --hard origin/master 
```
#### 将文件回滚到指定的提交
```
// commit id可以通过git log path获得文件的提交记录
git checkout <commit id> filepath
// 或者
git reset <commit id> fileName
```
#### 回滚一次merge操作
```
git revert -m 1 HEAD 新建一个commit，并且回到合并之前的状态
```

### push
git push origin master

### 初次连接到远程仓库，并将代码同步到远程仓库
```
git remote add origin <git url>  
// 可以先将本地分支mater与远端分支master绑定
git branch --set-upstream-to=origin/master master
git push -u origin master
```
### 删除远程的分支
```
git push origin --delete branch-name
```
### 将本地分支推送到远端（创建）
```
git push -u origin feature/upgrade_new
```
### 同步远程分支列表到本地
```
git remote update origin --prune
```

### 更换远程仓库地址
```
git remote -v  #查看远端地址
git remote #查看远端仓库名
git remote set-url origin https://gitee.com/xx/xx.git (新地址)
```

### 重命名本地分支
```
git branch -m new-name
```

### 删除本地分支
```
git branch -d branch-name
```
___
### fatal: refusing to merge unrelated histories

如我在Github新建一个仓库，写了License，然后把本地一个写了很久仓库上传。这时会发现 github 的仓库和本地的没有一个共同的 commit 所以 git 不让提交，认为是写错了 origin ，如果开发者确定是这个 origin 就可以使用 --allow-unrelated-histories 告诉 git 自己确定

遇到无法提交的问题，一般先pull 也就是使用 git pull origin master 这里的 origin 就是仓库，而 master 就是需要上传的分支，因为两个仓库不同，发现 git 输出 refusing to merge unrelated histories 无法 pull 内容

因为他们是两个不同的项目，要把两个不同的项目合并，git需要添加一句代码，在 git pull 之后，这句代码是在git 2.9.2版本发生的，最新的版本需要添加 --allow-unrelated-histories 告诉 git 允许不相关历史合并

假如我们的源是origin，分支是master，那么我们需要这样写git pull origin master --allow-unrelated-histories 如果有设置了默认上传分支就可以用下面代码

git pull --allow-unrelated-histories
___

### 将文件恢复为非stage状态  
```
git reset -- file
```
___

## git difftool 
将默认的difftool修改为vimdiff
```
git config --global diff.tool vimdiff
```
### vimdiff
+ dp 将当前文件中的修改复制到另一个文件中
+ do 将另一个文件的修改复制到当前文件
+ [c 上一个差异处
+ ]c 下一个差异处
+ Ctrl + w, w  两个文件之间来回切换

### 比较2个分支的差异 git diff
```
git diff branch1 branch2
```

## pull、merge合并时处理冲突
#### 冲突时使用远程分支的代码
```
git checkout --theirs -- somefile.dll
git checkout --theirs file
```

## checkout
### 获取、切换到某次特定的提交
+ 从提交的log中获取到特定提交对应的SHA1码
+ 使用**git checkout <SHA1>**执行checkout特定revision的代码

### git describe
```
$ git describe 5f6ba67e3f8cb59cb9a2f4db22f12e55326a182d
$ git checkout kors-2757-g5f6ba67
```

### 放弃本地所有的修改（未staged的）
```
git checkout .
```

## 关于detached HEAD
[git问题记录--如何从从detached HEAD状态解救出来](https://www.jianshu.com/p/ae4857d2f868)

## 如果将master分支合并到develop分支
```
git checkout master
git pull --rebase
git checkout develop
git pull --rebase
git merge master
```

## 将分支A中的本地修改提交到分支B
```
git stash
git checkout B
git stash pop
```

## 取消上一次Commit（在push之前）
```
git reset --soft HEAD^ //撤销 缓存区

git reset --hard HEAD^
```

## 取消上一次Commit（在push之后）
```
git revert HEAD
```

## 放弃本地的rebase
当feature分支从develop分支rebase了修改。
在push feature分支到远端之前，如果想要放弃rebase操作。
```
// rebase操作
git checkout feature
git pull --rebase
git rebase develop
```

```
// 取消rebase
git checkou feature
git reflog 
//找到rebase时本地的最后一次提交的hash
git reset HEAD commit-hash
```

## submodule
```
# add
git submodule add https://github.com/uNetworking/uWebSockets.git third_party/uWebSockets
# update
git submodule update --init --recursive
# 删除submodule
git submodule deinit submodule_name
git rm submodule_path
git commit -m "Remove submodule"
```

## 关于查看日志
```
// 查看当前分支日志，一条线
git log --oneline --decorate --graph
//
git reflog
```

## 设置git命令行通过SSR代理
+ SSR开启全局代理模式，查看下使用的代理端口。默认为1080.
```
# 设置代理
git config --global http.proxy socks5://127.0.0.1:1086
git config --global https.proxy socks5://127.0.0.1:1086
git config --global http.sslVerify false

#取消代理
git config --global --unset http.proxy 
git config --global --unset https.proxy
```

## 放弃本地的Commit
```
// 先查询到需要放弃的Commit的前一次的Commit ID
git reflog 
git reset --hard <Commit ID>
```

## 配置用户名、邮箱
```
// 如果想要配置全局的就使用 --global 选项
git config user.name "zhuqingquan"
git config user.email "zqq_222@163.com"
```

## 保存用户名、密码
```
git config credential.helper store
```

## 清楚缓存的用户名密码
```
git credential-manager uninstall
// 或者
git config --system --unset credential.helper
```

## 修改最后一次提交的用户名和邮箱
```
git commit --amend --author="userName <userEmail>"
# 注意邮箱外的‘<>’是必须的
```

## 设置使用vim作为编辑器
```
git config --global core.editor "vim"
```

## 删除某个文件，包括这个文件在所有分支中的提交产生的引用文件。彻底删除
```
git filter-branch --tree-filter "rm -f {filepath}" -- --all
```

## tag操作
```shell
# 显示所有tag
git tag --list
# 显示tag对应的详细信息和提交log
git show <tag-name>
# 创建tag
git tag -a <tag-name> <commit-id> -m "备注信息"
# 将tag推送到远端
git push origin <tag-name>
```

## lfs
[很好的参考文档](https://zzz.buzz/zh/2016/04/19/the-guide-to-git-lfs/)
- 在已有的git仓库中将之前普通提交的文件转换为使用lfs
```
git lfs track "file"
git lfs status # 查看是否生效
git add -u
git add .gitattributes
git commit -m ""
git push

# 对于新增加的或者从非lfs转换到lfs的文件，pull这些修改的地方需要显式调用下面命令拉取文件
git lfs pull
```
常用命令
```
# 查看当前使用 Git LFS 管理的匹配列表。显示的是规则列表。
git lfs track

# 使用 Git LFS 管理指定的文件
git lfs track "*.psd"

# 不再使用 Git LFS 管理指定的文件
git lfs untrack "*.psd"

# 类似 `git status`，查看当前 Git LFS 对象的状态
git lfs status

# 枚举目前所有被 Git LFS 管理的具体文件
git lfs ls-files

# 检查当前所用 Git LFS 的版本
git lfs version

# 针对使用了 LFS 的仓库进行了特别优化的 clone 命令，显著提升获取
# LFS 对象的速度，接受和 `git clone` 一样的参数。 [1] [2]
git lfs clone https://github.com/user/repo.git
```

## clone失败或者update submodule失败
错误信息：error: RPC 失败。HTTP 504 curl 22 The requested URL returned error: 504
可以使用--depth 1指定深度，减少拉数据的数量。
```
git clone --depth 1 xxx
git submodule update --init --recursive --depth 1
```
方式2：使用ssh拉代码。**（推荐）**