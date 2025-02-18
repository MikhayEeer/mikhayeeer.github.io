<header>
_ Welcome to mikhayeeer's tech blog _
</header>

@[TOC]
# 0. 简单认识Git
## 0.1 起源
  最初是由linux开发者linus用两周时间用c语言编写而成。github就是使用git系统对网站进行管理，github只支持git分布式系统，故名为github。

## 0.2 分布式
  svn和cvs也是版本控制系统，是集中式的，集中式的在每次写代码时都需要从服务器上拉取一份，如果服务器丢失，那么所有的都丢失，本机客户端仅仅保存当前版本信息。所有的回滚操作都需要服务器支持。
  而git是分布式的，每个电脑都是服务器，可以自由在本地回滚。

## 0.3 最高频操作
私以为最高频的应该就是远程仓库和本地仓库的拉取与推送了
```shell
git add .
git commit -m "This is description"
git push origin master

git pull origin master

git status
git log
```
# 1. 介绍
git 官网: https://git-scm.com
```shell
git help
```
## 1.1 安装
Linux安装
```shell
sudo apt install git
```
windows
[Git - Downloading Package (git-scm.com)](https://git-scm.com/download/win)

## 1.2 基本配置
查看git信息
```shell
git -V
git version
```
配置config，设置自己的用户名和邮箱，这些是每次提交的时候都会用到的信息
```bash
git config --global user.name "User Name"
git config --global user.email "UserEmail@Example.com"
# --global表示配置全局，整个git环境


# 查询git config的信息
git config --list
git config -l #两种都行
  # 按 q 退出查看
```
取消配置
```shell
git config --global --unset user.name
```
配置git使用的代理（根据个人需求设置）
```shell
git config --global http.proxy "http://127.0.0.1:7890"
git config --global https.proxy "http://127.0.0.1:7890"
```
## 1.3 架构
在git的过程中，主要分为两个大区
> 工作区和版本库，版本库又分为暂存区和分支

- **版本库**：存在于工作区中的一个隐藏目录`.git`，该目录不属于工作区，是git的版本库，是git管理的所有内容；
- **工作区**：是你撰写代码的地方，修改完成后，通过add操作将工作区的修改放到暂存区；一般是最新的版本代码
- **暂存区**：是版本库的一个临时区域，保存下一步将要提交的文件；
- **分支**：版本库有很多分支，提交的文件就存储在分支中，例如master和main分支
- **仓库**：所有提交都在这，包括历史版本

```
Git
|————————WorkSpace:工作区
|————————Index/Stage:暂存区/缓存区
|————————Repository:本地仓库/仓库区
|————————Remote:远程仓库
```
###  .git
Index:缓存区，存在于.git中

# 2. 仓库

一个仓库对应一个目录，该目录中的所有文件被git管理
## 2.1 新建仓库
```bash
git init
# 需要在空目录
```
执行`git init`操作的目录即为**工作区**，在操作之后，该目录下会有`.git`目录，是版本库
.git是git的配置文件目录，是隐藏文件夹，普通的ls命令无法查看
```bash
ls
# 用-lha参数来查看隐藏的目录
ls -lha
```
> 插播一句废话：`-lha`是命令`ls`的三个可选参数，分别代表显示详细信息、以便于人类可读的方式显示大小、显示所有文件（包括隐藏）


# 3. 基本操作
```bash
#查看仓库状态
git status
# 能看到是否有改动，untracked未跟踪代表未提交

#暂存文件，将工作区文件存入暂存区/缓存区
git add .
git add --all  # 添加所有，基本同git add . ， 但是会记录删除操作
git add filname.txt  # 添加某文件

#提交文件，将暂存区文件存入分支，形成一个版本
git commit -m "提交的描述信息"

# 删除文件
git rm filename
```
push和pull操作会放到下文远程仓库中
## 3.2 日志
```shell
# 查看历史提交日志
git log
# log的日志格式
# commit <hash id> (HEAD -> master)
# Author : 提交者(user.name)
# Date : 提交时间
#  myBase add new file "readme.md"

git log filename
# 查看某一文件的回滚版本

# 查看提交历史：  包括回滚操作
git reflog

git log --pretty=oneline #单行显示

git log --help
#会打开本地的log命令选项文档
```
### 日志帮助文档
```shell
#合并merge
git log --merge #仅展示合并的commit
git log --no-merge #不展示合并的commit
```

# 4. 远程仓库

前面提到的仓库是本地仓库，当多人进行协同开发工作的时候，每个人都在自己的本地仓库维护版本，但是工作需要多人之间共享代码、合并代码，就需要用到远程仓库
## 4.1 工作模式
程序员在本地仓库进行add操作将工作区文件存入暂存区，通过commit操作将暂存区文件存入分支，形成一个版本。
然后通过`clone`,`push`,`pull`操作与远程仓库进行交互

* 常见的远程仓库是**github和gitee**
* 一些企业也会有自己的远程仓库

## 4.2 基础操作
```bash
# 当前仓库名
git remote 

#本地关联远程仓库
git remote add origin https://gitee.com/Username/case.git

#查看远程仓库地址
git remote -v

git remote show origin

# 使用prune参数刷新本地分支仓库
git remote prune origin

git remote rename <old name> <new name>

git remote remove <remote_name>
```

## 4.3 推送 克隆 拉取
```bash
git push origin master

git clone https://gitee.com/username/case.git 

git clone https://gitee.com/username/case.git .
# 有 . 在末尾，及克隆到当前目录，最好保证是一个空目录

git pull origin master

# 单一分支的时候可以省略后面的参数
git push
git pull
```

|命令|描述|
|--|--|
|`git remote add <标识> 远程仓库地址`|本地仓库关联远程仓库|
|`git push <标识> master`|将本地仓库上传到远程仓库的master分支|
|`git pull <标识> master`|从远程仓库master分支拉取到本地仓库|
|`git clone 地址`|将远程仓库复制到本地，并形成本地仓库|
## 4.4 同步
```shell
git fetch
# 拉取远程全部分支，包括这些分支的仓库版本和日志记录
```
在进行PR操作前，需要与远程仓库的最新进展进行同步，就可以使用fetch。
该操作比较适用于本地仓库有多个远程仓库，通过`remote`查看。
例如fork一个主要仓库，在自己的仓库修改后，想要提交一个PR合入到主要仓库。
```bash
git remote add upstream <仓库地址>

git fetch upstream # 拉取upstream仓库的更新

git pull upstream master

git push origin feature
```

## 4.5 Github
想要直接与Github仓库进行交互使用，可以通过SSH对github仓库进行访问和写入数据。
SSH连接需要使用密钥进行身份验证，也就需要在本地电脑上生成SSH密钥并把公钥添加到github账户中。
详细操作可以直接参考[github的文档](https://docs.github.com/zh/authentication/connecting-to-github-with-ssh/generating-a-new-ssh-key-and-adding-it-to-the-ssh-agent#generating-a-new-ssh-key)
```shell
ssh-keygen -t ed25519 -C "your_email@example.com"
```

测试与github的连接
```shell
ssh -T git@github.com
```

github相关内容不是本篇博客重点，点到为止。更多问题可以直接搜索关键字github ssh
# 5. 分支
## 5.1 分支简介
分支，是一个个版本存储的最终位置；
一个分支相当于一条时间线，每一次commit都会形成一个版本，一个个版本依次存储在分支的一个个提交点上，分支上有个指针默认指向最新的提交点。

## 5.2 分支的操作
```bash
#查看分支
git branch

#查看所有分支
git branch -a # 包括远程

# 创建分支
git branch dev #创建名为dev的分支，只会创建，不会切换
git checkout -b dev #创建dev分支

git checkout -b previous upstream/previous
# 基于远程仓库upstream/previous创建previous分支

#切换分支
git checkout dev

#删除本地分支
git branch -D <branch_name>

#删除远程分支
git push origin --delete <remote_branch_name>
```

* 查看分支的差异
```shell
git diff A B filename
# 查看filename文件在分支A和分支B的差异

gitk --all
# 会唤起一个图形化浏览器展示commit
```

### 5.2.1 分支的细节原理

### 5.2.2 分支切换时，保存当前工作
如果修改了文件，是不可能直接切换到其他分支，是git为了防止丢失当前工作区的内容
使用命令`git stash`保存状态
```shell
# 保存当前工作状态
git stash

# 查看存储的工作状态
git stash list

# 在其他分支完成工作后，切换为刚刚的分支，可以恢复状态
git stash pop # 将list也删除了
git stash apply # 不删除list，默认恢复第一个
git stash apply list_name # 恢复指定的list

git stash drop list_name
git stash clear

# 查看新保存的stash和当前目录的差异
git stash show
```

该命令是解决git不提交代码不能切换分支的问题，有时候并不一定需要把代码push到远程

## 5.3 分支合并
```shell
# 分支合并
git merge branchB
# 当前的分支是A，上面的命令是在A分支下合并分支B

# 强制合并分支
git merge branchB --allow-unrelated-histories
```
合并原理：
1. 快速合并： 如果分支B的基于分支A的修改，则仅仅只是合并分支B的新内容，相当于移动指针到分支B的最新提交点；
2. 三方合并： 两个分支都有修改，则累加修改在一起，形成一个全新的提交点，作为合并后分支的指针位置
> 可能会造成冲突

### 5.3.1 冲突
当两个分支对同一个文件进行了修改，会出现冲突，冲突后，git会将冲突的内容都展示在文件中，用`<<<< ==== >>>>`来进行隔开

### 5.3.2 别的分支修改转移到当前分支
将A分支的某一修改，合并到当前的B分支
```shell
git checkout branch_A

git log
#查看需要合并的分支A的具体commit

git checkout branch_B
#切换到合并的分支

git cherry-pick <commit_hash_id>
# 合并具体的commit
```

### 5.3.3 预览合并的冲突
```shell
git checkout main

# 模拟合并
git merge --no-commit dev

# 取消合并
git merge --abort

# 解决冲突，继续合并
git add .
git commit -m "merge"
```
vscode中进行冲突的管理
vscode在发现合并带来的冲突时候，会把更改分为两类，一种是合并更改，一种是暂存更改，暂存更改的意思是来自于合并的分支，但是没有造成冲突，而合并更改中，就是合并的分支和当前的分支产生了冲突，需要人为去管理

## 5.4 恢复删除
> 起因，错误删除了远程原仓库中的文件，并在本地修改后提交到了自己的仓库，随后发起了pull request，需要恢复这个删除的文件。
> 可以直接复制原本的内容，然后创建新的文件，但是也可以利用其他方法

```shell
git fetch # 拉取所有远程仓库为最新
git fetch upstream

git pull upstream main # 但是被删除的commit还是存在，不会直接恢复

git checkout <branch-name>  -- <file-name>

git checkout upstream/main -- "Reports/season 3/successfulbarrier/[WeeklyReport]2024.07.15~2024.07.28.md"
git add .
git commit -m "Restore deleted files from remote repository"
```

# 6. 进阶操作

## 6.1 回滚仓库代码
- 常规回滚操作
```bash
# 查看提交日志
git log
# 在日志中找到需要回滚到的一次提交的id

#  --hard  一切全部恢复，add的暂存区消失，代码全部恢复到以前状态
git reset --hard <hash id>

#  --soft  仅仅恢复头指针，已经add的暂存区以及工作空间所有东西都不变
git reset --soft <hash id>

#  --mixed  头指针移动，add暂存区消失，工作空间代码不变
git reset --mixed <hash id>
```

* 利用`HEAD`来回滚仓库版本
```bash
# HEAD 指向当前仓库，是头指针

git reset --hard HEAD^
# ^代表上一次提交的版本

git reset --hard HEAD~3
# ~3以当前版本为基数，回滚几次； ~1表示回滚1次，同^； 这个表示回滚3次
```

* 回滚某一文件
```bash
git log filename

git reset <hash id> filename
```

## 6.2 Github使用
[[Github]]（这个博客还在写，准备写一些开发中常用到的功能）
### 常见问题
遇到拒绝访问
可能是token过期了，需要重新生成一个token，并且记得给token权限

默认的github端口是22，更换端口为443
```json
Host github.com
	HostName ssh.github.com
	Port 443
```
## 6.3 子模块
```shell
git submodule add address

git submodule init
git submodule update
```
## 6.4 新增分支操作
适用于git 2.23之后版本，新增`switch`和`restore`
```shell
git switch dev

git switch -b dev # 合并
git switch -c dev # 创建

git restore filename # 撤销提交修改，会从缓存区删除

```

## 6.5 Git本地服务器
> 以linux下为例
```shell
sudo apt install git

git adduser git

su git

ssh-keygen -t rsa -C "email@example.com"

cd ~
cd .ssh

touch authorized_keys
# 存放他人的公钥，存放后该服务器对公钥的用户开启权限，免密登录，拥有推送拉取权
```

## 6.6 忽略 .gitignore
忽略一些文件或者文件夹
在仓库中右键git bash here
```shell
touch .gitignore
```
编辑文件`.gitignore`
```shell
dir/       // 忽略dir文件夹
dir/*.txt  //忽略dir目录下所有.txt文件
           //并不会忽略 dir/dir2/a.txt这类文件
*.py       //忽略所有.py文件

*.gz       //忽略所有.gz结尾文件
!hhh.gz    // hhh.gz文件除外

abc        //忽略abc文件

/todo      //忽略根目录的todo文件

**/__pycache__/
```
再次`git status`就发现忽略的文件，没了
```shell
git status --ignored
```

.gitignore对已经追踪的文件是无效的，所以需要线清除一下相关缓存状态

### 忽略失败
.gitignore 文件只是 ignore 没有被 staged(cached) 文件，对于已经被 staged 的文件，加入 ignore 文件时一定要先从 staged 移除。
```shell
git rm --cached <file>
git rm --cached .obsidian/workspace.json
git rm -r --cached .obsidian   # 忽略文件夹
# 使用-r参数递归忽略整个文件夹

git add .
git commit -m "ignore workspace.json"
```

## 6.7 规则
在`.git/hooks/`下添加文件`pre-commit`或者`pre-commit.sh` 如果是linux还需要赋予该文件执行权限。
然后在文件中写下脚本，用来判断一些文件的存储规则：
```bash
#!/bin/bash

CURRENT_DIR=$(pwd)
HOOK_DIR=$(dirname "$0")
cd "$HOOK_DIR/../../.." || exit 1

FILES_TO_COMMIT=$(git diff --cached --name-only --diff-filter=ACM)

INVALID_FILES=""
for FILE in $FILES_TO_COMMIT; do
    if [[ "$FILE" =~ \.(png|jpg|jpeg|gif)$ ]]; then
        if [[ ! "$FILE" =~ ^z_LinkSource(/|$) ]]; then
            INVALID_FILES="$INVALID_FILES $FILE"
        fi
    fi
done

if [ -n "$INVALID_FILES" ]; then
    echo "ERROR:Image files must be stored in the z_LinkSource/ directory or its subdirectories.\n The following files need to be moved to the correct directory before committing:"
    echo "$INVALID_FILES"
    exit 1
fi

echo "Pre-commit Checking Passed!"

exit 0
```

<footer>

</footer>