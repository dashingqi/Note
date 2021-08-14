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
```

###### 与远程仓库进行连接

```
git remote add origin https://github.com/dashingi/*git
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

git status

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



