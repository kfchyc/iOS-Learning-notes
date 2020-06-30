# 用Terminal进行stash
## 步骤
1. 添加到工作区
`git add -all`
2. 提交到本地
`git commit -m "<#你的commit说明#>"`
3. 拉取远端代码
`git pull`
4. 检查并解决可能存在的冲突
5. 提交到远端
`git push`

## 注意
1. 每一步都要注意检查当前分支和状态
	1. 检查当前状态：`git status`
	2. 检查当前所在分支：`git branch`