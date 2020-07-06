# 用Terminal进行stash
## 步骤
1. 添加到工作区
  `git add -all`

2. 提交到本地
  `git commit -m "<#你的commit说明#>"`

3. 存储当前工作目录
  `git stash`

4. 拉取远端代码
  `git pull`

5. 检查并解决可能存在的冲突

6. 查看之前存储的所有版本列表
  `git stash list`

7. 恢复某一次的版本（若不指定id，则默认恢复最新的存储记录）
  `git stash pop [stask_id]`

  > e.g. `git stash pop stash@{0}`

8. 检查文件状态和修改内容。

9. 提交到远端
   `git push`

## 注意
1. 每一步都要注意检查当前分支和状态
	1. 检查当前状态：`git status`
	2. 检查当前所在分支：`git branch`