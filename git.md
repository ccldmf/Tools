### 简介
Git 是一个开源的分布式版本控制系统，用于敏捷高效地处理任何或小或大的项目。

Git 是 Linus Torvalds 为了帮助管理 Linux 内核开发而开发的一个开放源码的版本控制软件。
* Git 是分布式的
* Git 把内容按元数据方式存储
* Git 没有一个全局的版本号

### 安装配置
Git 目前支持 Linux/Unix、Solaris、Mac和 Windows 平台上运行。

### 基本操作
* 创建仓库
使用当前文件夹作为git仓库，我们需要初始化它
```
git init
```
* 克隆
```
git clone
```
* 提交与修改
```
git add                             添加文件到仓库
git status                          查看当前仓库状态
git diff                            比较文件的不同
git commit                          提交暂存区到本地仓库
git commit --amend                  修改历史log
git reset                           回退版本
git rm                              删除工作区文件
git mv                              移动或者重命名工作区文件
git log                             查看历史提交log
git log --oneline                   查看历史记录的简洁版本
git log --graph                     查看历史记录中出现的分支，合并情况
git log --reverse                   逆向查看历史日志
git log --author=username --oneline 查看指定用户提交的日志信息
git blame <file>                    查看指定文件的历史修改记录
```
* 远程操作
```
git remote                          远程仓库操作
git fetch                           从远程仓库获取代码库
git pull                            下载远程代码并合并
git push                            上传远程代码并合并
```
* 分支管理
```
git branch (branch_name)            创建分支
git checkout (branch_name)          切换分支
git merge (branch_name)             合并branch_name分支到当前工作分支
git branch                          列出分支
git checkout -b (branch_name)       创建分支并切换到该分支
git branch -d (branch_name)         删除指定分支
```
### 标签
达到一个重要的阶段，并希望记住那个特别的提交快照，可以使用 git tag 给它打上标签。
```
git tag -a V1.0                     -a是可选的，添加该参数是为了添加一个带注释的标签
git tag -a v0.9 85ff45              在85ff45点追加一个tag
```
