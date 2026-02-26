# Git完整教程

> Git是分布式版本控制系统，DevOps工作流的基础
>
> @author erik.zhou

## 📋 目录

- [Git基础](#git基础)
- [分支管理](#分支管理)
- [协作开发](#协作开发)
- [高级特性](#高级特性)
- [Git工作流](#git工作流)
- [最佳实践](#最佳实践)

## 🎯 学习目标

- [ ] 掌握Git基本操作
- [ ] 熟练使用分支管理
- [ ] 理解Git工作流程
- [ ] 掌握团队协作方法
- [ ] 使用Git高级特性
- [ ] 遵循Git最佳实践

## Git基础

### 安装配置

```bash
# 安装Git
# Ubuntu/Debian
sudo apt-get install git

# CentOS/RHEL
sudo yum install git

# macOS
brew install git

# 验证安装
git --version

# 配置用户信息
git config --global user.name "Erik Zhou"
git config --global user.email "erik.zhou@example.com"

# 配置编辑器
git config --global core.editor vim

# 配置默认分支名
git config --global init.defaultBranch main

# 查看配置
git config --list
git config --global --list

# 配置别名
git config --global alias.st status
git config --global alias.co checkout
git config --global alias.br branch
git config --global alias.ci commit
git config --global alias.lg "log --graph --pretty=format:'%Cred%h%Creset -%C(yellow)%d%Creset %s %Cgreen(%cr) %C(bold blue)<%an>%Creset' --abbrev-commit"
```

### 创建仓库

```bash
# 初始化新仓库
git init
git init my-project

# 克隆现有仓库
git clone https://github.com/user/repo.git
git clone https://github.com/user/repo.git my-folder

# 克隆指定分支
git clone -b develop https://github.com/user/repo.git

# 浅克隆（只克隆最近的提交）
git clone --depth 1 https://github.com/user/repo.git
```

### 基本操作

```bash
# 查看状态
git status
git status -s  # 简洁输出

# 添加文件到暂存区
git add file.txt
git add *.js
git add .  # 添加所有文件

# 提交更改
git commit -m "提交信息"
git commit -am "提交信息"  # 添加并提交已跟踪文件

# 查看提交历史
git log
git log --oneline
git log --graph --oneline --all
git log -p  # 显示差异
git log --stat  # 显示统计信息
git log --since="2 weeks ago"
git log --author="Erik"

# 查看差异
git diff  # 工作区vs暂存区
git diff --staged  # 暂存区vs仓库
git diff HEAD  # 工作区vs仓库
git diff branch1 branch2  # 比较分支

# 撤销更改
git checkout -- file.txt  # 撤销工作区更改
git restore file.txt  # 新版本命令
git reset HEAD file.txt  # 取消暂存
git restore --staged file.txt  # 新版本命令

# 删除文件
git rm file.txt
git rm --cached file.txt  # 只从Git删除，保留本地文件

# 移动/重命名文件
git mv old.txt new.txt
```

### .gitignore

```bash
# .gitignore示例
# 忽略所有.log文件
*.log

# 忽略node_modules目录
node_modules/

# 忽略所有.env文件
.env
.env.local
.env.*.local

# 忽略IDE配置
.vscode/
.idea/
*.swp
*.swo

# 忽略操作系统文件
.DS_Store
Thumbs.db

# 忽略构建输出
dist/
build/
*.o
*.so

# 但不忽略特定文件
!important.log

# 忽略特定目录下的文件
doc/*.txt

# 忽略所有子目录下的文件
**/temp/
```

## 分支管理

### 分支操作

```bash
# 查看分支
git branch  # 本地分支
git branch -r  # 远程分支
git branch -a  # 所有分支

# 创建分支
git branch feature-login
git checkout -b feature-login  # 创建并切换
git switch -c feature-login  # 新版本命令

# 切换分支
git checkout main
git switch main  # 新版本命令

# 删除分支
git branch -d feature-login  # 安全删除
git branch -D feature-login  # 强制删除

# 重命名分支
git branch -m old-name new-name

# 查看分支详情
git branch -v  # 显示最后一次提交
git branch -vv  # 显示跟踪分支
```

### 合并分支

```bash
# 合并分支（Fast-forward）
git checkout main
git merge feature-login

# 合并分支（创建合并提交）
git merge --no-ff feature-login

# 合并分支（压缩提交）
git merge --squash feature-login
git commit -m "合并feature-login"

# 解决冲突
# 1. 编辑冲突文件
# 2. 标记为已解决
git add conflicted-file.txt
# 3. 完成合并
git commit

# 中止合并
git merge --abort

# 查看合并状态
git status
```

### 变基操作

```bash
# 变基到main分支
git checkout feature-login
git rebase main

# 交互式变基
git rebase -i HEAD~3

# 变基操作选项
# pick: 保留提交
# reword: 修改提交信息
# edit: 修改提交内容
# squash: 合并到前一个提交
# fixup: 合并到前一个提交，丢弃提交信息
# drop: 删除提交

# 继续变基
git rebase --continue

# 跳过当前提交
git rebase --skip

# 中止变基
git rebase --abort
```

## 协作开发

### 远程仓库

```bash
# 查看远程仓库
git remote
git remote -v

# 添加远程仓库
git remote add origin https://github.com/user/repo.git

# 修改远程仓库URL
git remote set-url origin https://github.com/user/new-repo.git

# 删除远程仓库
git remote remove origin

# 重命名远程仓库
git remote rename origin upstream

# 查看远程仓库信息
git remote show origin
```

### 推送和拉取

```bash
# 推送到远程仓库
git push origin main
git push -u origin main  # 设置上游分支

# 推送所有分支
git push --all origin

# 推送标签
git push origin v1.0.0
git push --tags

# 强制推送（危险操作）
git push -f origin main

# 拉取远程更改
git pull origin main
git pull --rebase origin main  # 使用rebase

# 获取远程更改（不合并）
git fetch origin
git fetch --all

# 查看远程分支
git branch -r

# 跟踪远程分支
git checkout -b feature origin/feature
git branch --set-upstream-to=origin/feature feature
```

### 标签管理

```bash
# 创建轻量标签
git tag v1.0.0

# 创建附注标签
git tag -a v1.0.0 -m "版本1.0.0"

# 为特定提交打标签
git tag -a v0.9.0 9fceb02

# 查看标签
git tag
git tag -l "v1.*"

# 查看标签信息
git show v1.0.0

# 推送标签
git push origin v1.0.0
git push origin --tags

# 删除标签
git tag -d v1.0.0  # 本地删除
git push origin :refs/tags/v1.0.0  # 远程删除
git push origin --delete v1.0.0  # 远程删除（新版本）

# 检出标签
git checkout v1.0.0
git checkout -b version1 v1.0.0  # 基于标签创建分支
```

## 高级特性

### Stash（储藏）

```bash
# 储藏当前更改
git stash
git stash save "工作进行中"

# 查看储藏列表
git stash list

# 应用储藏
git stash apply  # 应用最近的储藏
git stash apply stash@{2}  # 应用特定储藏

# 应用并删除储藏
git stash pop

# 删除储藏
git stash drop stash@{0}
git stash clear  # 删除所有储藏

# 从储藏创建分支
git stash branch feature-branch stash@{0}

# 储藏未跟踪的文件
git stash -u

# 储藏所有文件（包括忽略的）
git stash -a
```

### Cherry-pick

```bash
# 挑选特定提交
git cherry-pick abc123

# 挑选多个提交
git cherry-pick abc123 def456

# 挑选提交范围
git cherry-pick abc123..def456

# 只应用更改，不提交
git cherry-pick -n abc123

# 解决冲突后继续
git cherry-pick --continue

# 中止cherry-pick
git cherry-pick --abort
```

### Reset和Revert

```bash
# Reset（重置）
git reset --soft HEAD~1  # 保留更改在暂存区
git reset --mixed HEAD~1  # 保留更改在工作区（默认）
git reset --hard HEAD~1  # 丢弃所有更改

# 重置到特定提交
git reset --hard abc123

# Revert（回退）
git revert HEAD  # 回退最近的提交
git revert abc123  # 回退特定提交

# 回退合并提交
git revert -m 1 abc123
```

### Reflog

```bash
# 查看引用日志
git reflog

# 查看特定分支的reflog
git reflog show main

# 恢复丢失的提交
git reset --hard HEAD@{2}

# 恢复删除的分支
git checkout -b recovered-branch HEAD@{2}
```

### Submodule

```bash
# 添加子模块
git submodule add https://github.com/user/lib.git libs/lib

# 克隆包含子模块的仓库
git clone --recursive https://github.com/user/repo.git

# 初始化子模块
git submodule init
git submodule update

# 更新子模块
git submodule update --remote

# 查看子模块状态
git submodule status

# 删除子模块
git submodule deinit libs/lib
git rm libs/lib
rm -rf .git/modules/libs/lib
```

## Git工作流

### Git Flow

```
Git Flow分支模型

main (生产分支)
  │
  ├─── hotfix/xxx (紧急修复)
  │
release/x.x.x (发布分支)
  │
develop (开发分支)
  │
  ├─── feature/login (功能分支)
  ├─── feature/payment
  └─── feature/search
```

**操作流程**
```bash
# 初始化Git Flow
git flow init

# 开始新功能
git flow feature start login
# 工作...
git flow feature finish login

# 开始发布
git flow release start 1.0.0
# 准备发布...
git flow release finish 1.0.0

# 紧急修复
git flow hotfix start critical-bug
# 修复...
git flow hotfix finish critical-bug
```

### GitHub Flow

```
GitHub Flow工作流

main (主分支)
  │
  ├─── feature-branch-1
  ├─── feature-branch-2
  └─── bugfix-branch
```

**操作流程**
```bash
# 1. 创建分支
git checkout -b feature-login

# 2. 提交更改
git add .
git commit -m "实现登录功能"

# 3. 推送分支
git push -u origin feature-login

# 4. 创建Pull Request（在GitHub上）

# 5. 代码审查和讨论

# 6. 合并到main
# （在GitHub上合并）

# 7. 删除分支
git branch -d feature-login
git push origin --delete feature-login
```

### GitLab Flow

```
GitLab Flow工作流

main → pre-production → production
  │
  ├─── feature-branch-1
  └─── feature-branch-2
```

## 最佳实践

### 提交规范

**Conventional Commits**
```bash
# 格式
<type>(<scope>): <subject>

<body>

<footer>

# 类型
feat: 新功能
fix: 修复bug
docs: 文档更新
style: 代码格式（不影响代码运行）
refactor: 重构
perf: 性能优化
test: 测试
chore: 构建过程或辅助工具的变动

# 示例
feat(auth): 添加用户登录功能

实现了基于JWT的用户认证系统
- 添加登录API
- 添加token验证中间件
- 添加用户会话管理

Closes #123

fix(api): 修复用户注册接口500错误

当用户名包含特殊字符时会导致数据库错误

Fixes #456
```

### 分支命名

```bash
# 功能分支
feature/user-authentication
feature/payment-integration

# 修复分支
bugfix/login-error
hotfix/critical-security-issue

# 发布分支
release/v1.0.0
release/v2.1.0

# 文档分支
docs/api-documentation
docs/readme-update
```

### 代码审查

```bash
# 创建Pull Request前
# 1. 确保代码通过测试
npm test

# 2. 确保代码符合规范
npm run lint

# 3. 更新文档
# 4. 编写清晰的PR描述

# Pull Request模板
## 变更说明
简要描述本次变更的内容

## 变更类型
- [ ] 新功能
- [ ] Bug修复
- [ ] 文档更新
- [ ] 代码重构
- [ ] 性能优化

## 测试
描述如何测试这些变更

## 截图（如适用）

## 相关Issue
Closes #123
```

### 安全实践

```bash
# 1. 不要提交敏感信息
# 使用.gitignore忽略敏感文件
.env
*.key
*.pem
config/secrets.yml

# 2. 如果不小心提交了敏感信息
# 从历史中删除文件
git filter-branch --force --index-filter \
  "git rm --cached --ignore-unmatch path/to/sensitive-file" \
  --prune-empty --tag-name-filter cat -- --all

# 或使用BFG Repo-Cleaner
bfg --delete-files sensitive-file.txt

# 3. 使用Git Secrets防止提交敏感信息
git secrets --install
git secrets --register-aws

# 4. 签名提交
git config --global user.signingkey <GPG-KEY-ID>
git config --global commit.gpgsign true
git commit -S -m "签名提交"
```

### 性能优化

```bash
# 1. 浅克隆
git clone --depth 1 https://github.com/user/repo.git

# 2. 部分克隆
git clone --filter=blob:none https://github.com/user/repo.git

# 3. 清理仓库
git gc
git gc --aggressive --prune=now

# 4. 查看仓库大小
git count-objects -vH

# 5. 压缩仓库
git repack -a -d --depth=250 --window=250

# 6. 删除未使用的对象
git prune

# 7. 清理reflog
git reflog expire --expire=now --all
```

### 常用脚本

**自动化提交脚本**
```bash
#!/bin/bash
# git-commit.sh

# 检查是否有更改
if [[ -z $(git status -s) ]]; then
    echo "没有需要提交的更改"
    exit 0
fi

# 显示状态
git status

# 添加所有更改
git add .

# 提示输入提交信息
read -p "提交信息: " message

# 提交
git commit -m "$message"

# 推送
read -p "是否推送到远程? (y/n): " push
if [[ $push == "y" ]]; then
    git push
fi
```

**批量更新子模块**
```bash
#!/bin/bash
# update-submodules.sh

git submodule foreach git pull origin main
git add .
git commit -m "更新子模块"
git push
```

## 故障排查

### 常见问题

**1. 合并冲突**
```bash
# 查看冲突文件
git status

# 编辑冲突文件，解决冲突标记
<<<<<<< HEAD
当前分支的内容
=======
合并分支的内容
>>>>>>> feature-branch

# 标记为已解决
git add conflicted-file.txt

# 完成合并
git commit
```

**2. 撤销操作**
```bash
# 撤销最后一次提交（保留更改）
git reset --soft HEAD~1

# 修改最后一次提交
git commit --amend

# 撤销推送的提交
git revert HEAD
git push
```

**3. 恢复删除的文件**
```bash
# 查找删除文件的提交
git log --all --full-history -- path/to/file

# 恢复文件
git checkout <commit-hash>^ -- path/to/file
```

## 📚 学习检查清单

- [ ] 掌握Git基本操作
- [ ] 熟练使用分支管理
- [ ] 理解Git工作流
- [ ] 掌握团队协作方法
- [ ] 使用Git高级特性
- [ ] 遵循Git最佳实践

## 🔗 参考资源

- [Pro Git Book](https://git-scm.com/book/zh/v2)
- [Git官方文档](https://git-scm.com/doc)
- [GitHub Guides](https://guides.github.com/)

---

@author erik.zhou
