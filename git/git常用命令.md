# git 常用命令

```bash
git init 						# 初始化本地git环境
git clone [url]	new_dir			# 克隆一份代码到本地目录
git pull 						# 把远程库的代码更新到工作台
git pull --rebase origin master # 强制把远程库的代码更新新到当前分支上面
git fetch 						# 把远程库的代码更新到本地库
git add . 						# 把本地的修改加到stage中
git commit -m 'comments here' 	# 把stage中的修改提交到本地库
git commit --amend				# 撤消刚才的提交操作，重新提交
git push 						# 把本地库的修改提交到远程库中
git push origin master 			# 把本地库的修改提交到远程master中
git branch -r/-a 				# 查看远程分支/全部分支
git branch -v 					# 可以看见每一个分支的最后一次提交
git branch -d test 				# 删除test分支
git branch -r -d origin/dev		# 删除远程分支dev
git push origin :test			# 删除远程分支test
git checkout -- file 			# 撤销文件修改，commit前
git checkout master/branch 		# 切换到某个分支
git checkout -b test 			# 新建test分支
git merge master 				# 假设当前在test分支上面，把master分支上的修改同步到test分支上
git merge tool 					# 调用merge工具
git stash 						# 把未完成的修改缓存到栈容器中
git stash list 					# 查看所有的缓存
git stash pop 					# 恢复本地分支到缓存状态
git blame someFile 				# 查看某个文件的每一行的修改记录（）谁在什么时候修改的）
git status 						# 查看当前分支有哪些修改
git log 						# 查看当前分支上面的日志信息
git diff 						# 查看当前没有add的内容
git diff --cache 				# 查看已经add但是没有commit的内容
git diff HEAD 					# 上面两个内容的合并
git reset --hard id/HEAD 		# 撤销本地修改,git的HEAD变了,文件也变了
git reset --mixed id/HEAD		# git的HEAD变了,但文件并没有改变,取消了commit和add的内容
git reset --soft id/HEAD		# git的HEAD变了,但文件并没有改变,取消了commit的内容,重新add了一次
git revert id/HEAD				# 撤销某次操作,重新commit 了 old_commit一次
git mv new old 					# 重命名
git tag -l 'v1.*'				# 列出标签
git tag -a new_tag -m 'new_tag'	# 创建一个带说明的新标签
git show net_tag				# 查看相应标签的版本信息
git tag -a new_tag commit_id	# 后期加注标签
git push origin new_tag			# 推送标签，默认 git push 不会推送 tag
git push origin --tags			# 一次推送所有本地新增的标签上去
git tag -d v1.0					# 删除本地标签
git push origin :[tag]			# 删除远程标签（先删除本地标签）
git push origin :refs/tags/v1.0	# 删除远程标签（先删除本地标签）
git reflog						# 历史记录
```