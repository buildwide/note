# Https 记住密码
## 永久记住密码
git config --global credential.helper store
## 临时记住密码
git config –global credential.helper cache
git config credential.helper ‘cache –timeout=3600’

## 忘记密码
git config --system --unset credential.helper

# merge 

- 默认行为（等同于 --ff）：允许快进，必要时创建 merge commit
```bash
git merge feature
```
- 强制快进，否则失败
```bash
git merge --ff-only feature
```
- 强制创建 merge commit（保留合并记录）
```bash
git merge --no-ff -m "Merge feature: 描述" feature
```
- squash（把所有改动合并为工作区改动，不记录为 merge；需手工 commit）
```bash
git merge --squash feature
git commit -m "合并为一条提交的信息"
```
- 不自动提交（合并变更到工作区/索引，允许你调整再提交）
```bash
git merge --no-commit feature
git commit -m "自定义信息"
```
- 编辑合并消息
```bash
git merge --edit feature
```
- 在合并消息中追加被合并的提交列表（常与 --no-ff/--ff 等配合）
```bash
git merge --log feature
```
- 指定合并策略或策略选项（例：recursive + theirs）
```bash
git merge -s recursive -X theirs feature
```
- 允许合并没有共同历史的分支
```bash
git merge --allow-unrelated-histories other-root-branch
```
- 取消正在进行的合并（遇冲突想放弃）
```bash
git merge --abort
```

# reset

- 撤销最近一次提交，但保留改动为已暂存（staged）：
````bash
git reset --soft HEAD~1
````

- 撤销最近一次提交，但保留改动为未暂存（working directory，推荐）：
````bash
git reset --mixed HEAD~1
# 等同于
git reset HEAD~1
````

- 撤销最近一次提交并丢弃改动（不可恢复）：
````bash
git reset --hard HEAD~1
````

- 撤销指定提交但以新提交的形式反向提交（安全且适用于已推送到远端的情况）：
````bash
git revert <commit-sha>
git push origin <branch>
````

- 已推送到远端且必须重写远端历史（危险，需小心并通知团队）：
````bash
git reset --hard <target-commit-or-HEAD~1>
git push --force-with-lease origin <branch>
````

说明：把 HEAD~1 换成 HEAD~N 或具体 commit SHA 以撤销多个或指定提交。选择前请确认是否已推送以及团队协作影响。

# branch
- 查看远程分支
git branch -r 

- 查看本地对应远程分支
git branch -vv