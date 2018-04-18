## 原理
- 工作区 —— 暂存区 —— 本地仓库 —— 远程仓库

## 命令行
- `git init`
- `git add/rm fileName`
- `git commit -m "messagecontent"` `git commit --amend`
- `git long [--graph] [--pretty=oneline]` `git reflog`
- 回退恢复
  - `git reset --hard HEAD^|HEAD^^|HEAD~100|commit-id` 回退
  - `git reset HEAD fileName` 暂存区 => 工作区
  - `git checkout -- fileName` 丢弃工作区
- 分支
  - `git checkout -b branchName` = `git branch branchName` + `git checkout branchName`
  - `git merge branchName [--no-ff]` ff表示Fast-Forawrad方式合并
