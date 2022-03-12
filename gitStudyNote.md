
# git常用命令
***
| 作用 | 命令|
| :-------------------: | :-------------------: |
|设置git用户名和邮箱| ` git config --global user.name yuqian` <br> `git config --global user.email okyyqq@gmail.com `
| 初始化git仓库 | `git init` |
| 添加新文件 | `git add filename` |
| 提交版本 | `git commit -m "xxxxxx"` |
|查看版本信息 | ` git reflog `|
|查看版本详细信息 | ` git log `|
|版本穿梭| `git reset --hard 版本号`|
|本地创建sshkey|`ssh-keygen -t rsa -C "your_email@youremail.com"`|
|验证是否成功|`ssh -T git@github.com`|
***

# git分支操作
| 作用 | 命令|
| :-------------------: | :-------------------: |
| 创建分支 | `git branch 分支名`|
|查看分支 | `git branch -v` | 
|切换分支|`git checkout 分支名`|
|把指定分支合并到==当前==分支上| `git merge 分支名`|

# git创建别名
| 作用 | 命令|
| :-------------------: | :-------------------: |
| 创建别名| `git remote add 别名 git@github.com:yourName/yourRepo.git`|
|查看别名 | `git remote -v `|
|push到远程仓库|`git push 别名 分支名`|
|解决一些错误|`git config --global http.sslVerify "false"`|
|更新本地仓库至最新的改动| `git pull`|

# git相关概念
git有本地文件，暂存区，本地库，远程仓库
add命令将本地文件存放到暂存区，暂存区通过commit提交到本地库，本地库push到远程仓库

