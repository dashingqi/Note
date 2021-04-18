#### .gitignore文件新增忽略文件不起作用

- git rm -r --cached . 清除掉缓存记录
- git add .
- git commit -m "[update .gitignore]"

#### git add --amend

- git push的时候出现问题，需要修改本地仓库中的文件
- 修改完文件之后，可以 git add --amend 
- 这样不会产生新的提交新用的还是上次的提交信息
- shitft+":" ----> wq
- 最后在 git push origin master

#### git pull --rebase

- 拉取代码
- 当出现冲突，就进行冲突的解决
- git rebase --continue 减少一次提交记录，
- git push origin master

