#### 1. 基础配置与初始化
在使用 Git 前，首先要告诉系统你是谁。
- **配置用户信息**：`git config --global user.name "你的名字"` 和 `git config --global user.email "你的邮箱"`
- **新建仓库**：在项目文件夹运行 `git init` 即可将其变为 Git 仓库。
- **克隆现有项目**：`git clone <远程仓库地址>`

#### 2. 文件的“保存点”管理（暂存与提交）
- **查看状态**：`git status`（最常用的命令，随时查看文件变动）。
- **添加暂存**：`git add <文件名>`（或 `git add .` 添加所有变动）。
- **正式提交**：`git commit -m "本次修改的描述语"`。
- **查看历史**：`git log --oneline`（简洁查看提交记录）。

#### 3. 分支管理（并行开发的利器）
- **创建并切换分支**：`git checkout -b <分支名>`。
- **查看所有分支**：`git branch`。
- **合并分支**：先切回主分支 `git checkout main`，再执行 `git merge <功能分支名>`。
- **删除分支**：`git branch -d <分支名>`。

#### 4. 远程同步（多人协作）
- **拉取更新**：`git pull`（同步远程代码到本地）。
- **推送代码**：`git push origin <分支名>`。
- **查看远程库**：`git remote -v`。
- **新增远程库**：`git remote add [-options] <repoName> <repoURL>`

#### 5. 撤销与“后悔药”
- **放弃工作区修改**：`git checkout -- <文件名>`。
- **撤销暂存区文件**：`git reset HEAD <文件名>`。
- **修正最后一次提交**：`git commit --amend`。