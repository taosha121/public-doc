# GIT最佳实践

## GIT原理

![img](img/git-structure.jpg)

- Workspace：工作区
- Index / Stage：暂存区(git add之后到暂存区)
- Repository：仓库区（或本地仓库）(git commit后到本地仓库)
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
	- 已经到了暂存区？
		- 主要靠`git reset HEAD filename`来回到最近一个commit的版本，其实使用了默认的--mixed参数把修改回复到工作区，然后可以checkout --来彻底删除
	- 已经到本地仓库的
		- 主要使用`git reset [--soft --mixed --hard]`
		- `git reset --hard HEAD~3`之后代码直接变成当前commit往前数3个的状态,不保留改动，工作区干净, git log也看不到那些被消除的commit，但是使用git reflog可以找回之前的状态
		- `git reset --mixed HEAD^`之后代码变成之前一个那个commit的状态，保留改动但是处于**没有被add的状态**，还需要git add和git commit，--mixed是默认，可以不写
		- `git reset --soft HEAD^`之后代码变成之前一个commit的状态，保留改动但是处于**已经add的状态**，需要git commit
	- git revert 和 git reset的区别
		- git revert用的比较少，这是使用一次新的commit来中和掉之前某次commit提交的内容，因为revert commit的时候还涉及revert普通的commit和merge commit的区别，所以一般不太使用这个命令
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
