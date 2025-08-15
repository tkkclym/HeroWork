这是 Git GUI 中的提交相关区域，各按钮作用如下：



1. **Rescan**：重新扫描工作区与暂存区的差异，会重新检测文件变动，更新 Git GUI 中显示的文件状态，比如有文件修改但没被识别时，点它重新获取变化。
2. **Stage Changed**：将工作区中已修改、未暂存（Unstaged Changes 区域）的文件，移动到暂存区（Staged Changes 区域 ），和 `git add` 部分功能类似，准备提交这些变动。
3. **Sign Off**：在提交信息里自动添加 `Signed-off-by` 行，格式一般是 `Signed-off-by: 你的姓名 <你的邮箱>` ，用于记录提交者身份，常配合开源项目的贡献流程，证明提交符合贡献协议。
4. **Commit**：结合暂存区文件与填写的提交信息（Commit Message），创建新的提交（即生成一个 commit 对象），把暂存区改动持久化到本地版本库，类似 `git commit` 命令 。
5. **Push**：把本地已提交的 commits，推送到远程版本库（比如 GitHub、GitLab 等远程仓库），和 `git push` 命令功能一致，同步本地改动到远程。
6. **Amend Last Commit**（复选框，非按钮）：勾选后，新提交会修改上一次的提交（修改提交信息、补充 / 调整暂存区文件 ），相当于执行 `git commit --amend`，常用于修正上一次提交的小问题。