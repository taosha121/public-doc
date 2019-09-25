# GIT最佳实践

## GIT原理

![img](img/git-structure.jpg)

- Workspace：工作区
- Index / Stage：暂存区
- Repository：仓库区（或本地仓库）
- Remote：远程仓库

## GIT工作流程
1. 本地初始化一个仓库
	- git init
	- git add/commit
	- git remote add origin https://github.com/xjtustt/test.git
	- git push --set-upstream origin master
2. clone一个远程仓库
	- git clone https://github.com/xjtustt/test.git
3. 检查分支，直接在master上工作
	- git branch
	- git add/commit/push
4. 查看log
	- git log --graph --pretty=oneline
5. git commit --amend -m "xxx"
	- 使用一次新的commit，替代上一次提交
	- 如果代码没有任何新变化，则用来改写上一次commit的提交信息
6. push前更新远程的提交
	- git pull和git pull --rebase的区别
	- git pull
		- 自动merge成功
		- 出现conflict手动解决，然后git add -> git commit
	- git pull --rebase
		- 自动rebase成功
		- 出现conflict手动解决，然后git add -> git rebase --continue
7. 新建feature分支
	- git checkout -b feature-branch
	- git add/commit
	- git push --set-upstream origin feature-branch

## 常见问题
1. 修改的东西不想要了
	- 还在工作区？ git checkout
	- 已经到了暂存区？ `git reset HEAD xx.txt`把已经到暂存区的修改撤回到工作区, 后续可以`git checkout -- xx.txt`彻底撤销修改
	- 已经到本地仓库了？
		- `git revert` 撤销 某次操作，此次操作之前和之后的commit和history都会保留，并且把这次撤销
		- git revert 和 git reset的区别
2. PR发现out of date
	- git fetch
	- git rebase origin/master
	- git push -f
3. PR合并commit
	- git rebase -i HEAD~3
	- git push -f
4. 修改之前提交的commit history里面的author和email
	1. `git clone --bare https://github.com/user/repo.git`
	2. run this shell script
	```sh
	#!/bin/sh

	git filter-branch --env-filter '

	OLD_EMAIL="your-old-email@example.com"
	CORRECT_NAME="Your Correct Name"
	CORRECT_EMAIL="your-correct-email@example.com"
	
	if [ "$GIT_COMMITTER_EMAIL" = "$OLD_EMAIL" ]
	then
    		export GIT_COMMITTER_NAME="$CORRECT_NAME"
    		export GIT_COMMITTER_EMAIL="$CORRECT_EMAIL"
	fi
	if [ "$GIT_AUTHOR_EMAIL" = "$OLD_EMAIL" ]
	then
    		export GIT_AUTHOR_NAME="$CORRECT_NAME"
    		export GIT_AUTHOR_EMAIL="$CORRECT_EMAIL"
	fi
	' --tag-name-filter cat -- --branches --tags
	```
	3. `git push --force --tags origin 'refs/heads/*'`
