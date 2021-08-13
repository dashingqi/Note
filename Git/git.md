## Git操作

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



###### 由https切换为ssh

```
git remote set-url origin ssh连接地址
```

检验与之关联的远程地址链接

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

###### 
