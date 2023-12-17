# Git 常用命令

#### 配置

```shell
git config --global user.name ""
git config --global user.email ""

# 查看
git config --list --global

# 清除(缺失等同于清除)
git config --unset --global user.name
git config --unset --global user.email

# 添加ssh
ssh-keygen -t rsa -C "13166962097@163.com"
# 查看
 cd ~/.ssh/
 vim id_rsa.pub
```





 ### 删除本地和远程的分支

先退出需要删除的分支

```bash
# 返回主分支
git checkout main
// 删除本地分支
git branch -d localBranchName

# 删除远程分支
git push origin --delete remoteBranchName
# or
git push origin :remoteBranchName

# 同步分支列表（可以不需要）
git fetch -p
```

当一个分支被推送并合并到远程分支后，`-d` 才会本地删除该分支。如果一个分支还没有被推送或者合并，那么可以使用`-D`强制删除它。

