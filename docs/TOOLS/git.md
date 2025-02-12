# 基本命令：
`git init`：把当前目录变成git可以管理的库

`git add [file]`：将修改了的文件添加到仓库

`git commit -m` "说明内容"

`git commit -a` -a 参数设置修改文件后不需要执行 git add 命令，直接来提交

`git config user.name` 查看提交时用户名
`git config user.email` 查看提交时邮箱

`git config --global user.name "your name"` 修改用户名
`git config --global user.email "your email"` 修改邮箱


`git status`

`git log` 查看提交记录

`git reset --hard HEAD^` 回退到当前版本的上一个版本

`git reflog`

`git checkout -- [file]`当你改乱了工作区某个文件的内容，想直接丢弃工作区的修改时

`git reset HEAD [file]`当你不但改乱了工作区某个文件的内容，还添加到了暂存区时，想丢弃修改

`git rm [file]`删除版本库里的文件

# 分支管理：
`git branch`查看分支

`git branch <name>`创建分支

`git checkout <name>`或者`git switch <name>`切换分支

`git checkout -b <name>`或者`git switch -c <name>`创建+切换分支

`git merge <name>`合并某分支到当前分支

`git branch -d <name>`删除分支

# 相关链接：
[Git教程-廖雪峰的官方网站](https://www.liaoxuefeng.com/wiki/896043488029600)