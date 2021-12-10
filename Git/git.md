## Git操作

#### 配置以及基本命令

###### 配置用户名和邮箱

```
git config --global user.name ""

git config --global user.email ""
```

###### 查看本地git配置

```
cat ~/.gitconfig
```

###### 本地初始化一个仓库

```
git init
```

###### Mac终端创建一个文件

```
touch Demo.java
```

###### 查看仓库的状态

```
git status
```

###### 暂存文件（暂存区）

```
git add Demo.java

// 暂存所有文件
git add .  / git add -A
```

###### 从缓存去移除文件

```
git rm --cached Demo.java
```

###### 查看隐藏文件

```
ls -la 
```

###### 提交文件到Git本地仓库

```
git commit -m"[update]"

git commit --amend

git commit --amend --no-edit
```

###### 与远程仓库进行连接

```
git remote add origin https://github.com/dashingi/*git

git remote add origin git@github.com:dashingqi/DQMVVMComponent.git
```

###### 删除与远程仓库的链接

```java
git remote rm origin
```

###### 查看与远程仓库连接关系

```
git remote -v
```

###### 推送本地仓库到远程仓库

```
git push origin master -u

git push
```

克隆远程仓库

```
git clong *.git
```

###### 拉取远程仓库代码

```
git pull --rebase
```

#### 分支操作

###### 创建分支

```
git branch dev
```

###### 查看分支

```
git branch

git branch -r //查看远程分支
```

###### 切换分支

```
git checkout dev
```

###### 创建并切换到dev分支

```
git checkout -b dev
```

###### 删除分支

```
// 当前dev分支代码合并到master分支，在master分支上删除dev分支的代码
git branch -d dev

// 强制删除
git branch -D dev
```

###### 合并代码

```
git checkout master

git merge dev
```

###### 将本地分支推送到远端，并在远端创建一个新的分支dev

```
git push origin dev
```

###### 删除远端分支dev

```
git push origin :dev
```

将本地分支推送到远端，并在远端创建一个新的分支develop

```
git push origin dev:develop
```

#### 合并分支的操作

###### 查看分支提交记录

```
git log // 全量查询

git log --oneling // 查看每条记录的第一行

```

###### 查看具体某一次提交的具体内容

```
git show id
```

###### 非 fast Forword 模式

```
git merge dev --no-ff
```

###### rebase

```
git rebase dev
```

###### git mergetool  # vimdiff

```
git mergetool
vimdiff
```

#### 回滚和撤销

###### 撤回到上次的commit（不会删除文件）

```
git reset dev^

// 切回到上上次的commit
git reset dev^^

// 切回到具体某一个的commit
git reset commitId
```

###### 撤回到某一次的提交（将暂存区中的文件删除）

```
git reset --hard commitid
```

###### revert 团队开发

```
git revert commitid

// 撤销本次revert
git revert --abort
```

#### 文件忽略以及同步forked仓库代码

###### .gitgnore文件

```
// 生成网站
https://www.toptal.com/developers/gitignore
```

###### 同步forked的仓库源代码

```
git remote -v

git remote add upstream 上游仓库(之前forked的仓库地址)

git remote -v

git fetch upstream 分支名（默认是master）

pull = fetch + merge

// 如果你是贡献 你需要使用 merge


```

#### 免密登陆

###### 参考文章

```
https://www.jianshu.com/p/63edbb08bd5f
```



###### 由https切换为ssh

```
git remote set-url origin ssh连接地址
```

###### 检验与之关联的远程地址链接

```
git remote -v
```

###### 查看当前本地分支与远程分支的关联关系

```
git branch -vv
```

###### 清除Git的提交缓存

```
git rm -r --cached .
```



