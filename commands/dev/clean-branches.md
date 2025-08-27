# 清理分支命令

清理已合并和过时的 git 分支

## 说明

请按照以下系统性方法清理 git 分支：**$ARGUMENTS**

1. **仓库状态分析**
   - 检查当前分支和未提交的更改
   - 列出所有本地和远程分支
   - 确定主分支名称
   - 查看最近的分支活动和合并历史记录

   ```bash
   # 检查当前状态
   git status
   git branch -a
   git remote -v

   # 检查主分支名称
   git symbolic-ref refs/remotes/origin/HEAD | sed 's@^refs/remotes/origin/@@'
   ```

2. **安全预防措施**
   - 确保工作目录干净
   - 切换到主/master 分支
   - 从远程拉取最新更改
   - 如有需要，创建当前分支状态的备份

   ```bash
   # 确保状态干净
   git stash push -m "清理分支前备份"
   git checkout main # 或 master
   git pull origin main
   ```

3. **识别已合并的分支**
   - 列出已合并到主分支的分支
   - 排除受保护的分支（main、master、develop）
   - 检查本地和远程的合并分支
   - 验证合并状态以避免意外删除

   ```bash
   # 列出已合并的本地分支
   git branch --merged main | grep -v "main\\|master\\|develop\\|\\*"

   # 列出已合并的远程分支
   git branch -r --merged main | grep -v "main\\|master\\|develop\\|HEAD"
   ```

4. **识别过时分支**
   - 查找最近没有活动的分支
   - 检查每个分支的上次提交日期
   - 识别超过指定时间范围（例如 30 天）的分支
   - 考虑功能/修补程序分支的命名模式

   ```bash
   # 按上次提交日期列出分支
   git for-each-ref --format='%(committerdate) %(authorname) %(refname)' --sort=committerdate refs/heads

   # 查找超过 30 天的分支
   git for-each-ref --format='%(refname:short) %(committerdate)' refs/heads | awk '$2 < "'$(date -d '30 days ago' '+%Y-%m-%d')'"'
   ```

5. **交互式分支审查**
   - 删除前审查每个分支
   - 检查分支是否有未合并的更改
   - 验证分支的用途和状态
   - 删除前请求确认

   ```bash
   # 检查未合并的更改
   git log main..branch-name --oneline

   # 显示分支信息
   git show-branch branch-name main
   ```

6. **受保护分支配置**
   - 识别不应删除的分支
   - 为重要分支配置保护规则
   - 记录分支保护策略
   - 为新仓库设置自动保护

   ```bash
   # 受保护分支示例
   PROTECTED_BRANCHES=("main" "master" "develop" "staging" "production")
   ```

7. **本地分支清理**
   - 安全删除已合并的本地分支
   - 移除过时的功能分支
   - 清理已删除远程分支的跟踪分支
   - 更新本地分支引用

   ```bash
   # 删除已合并的分支（交互式）
   git branch --merged main | grep -v "main\\|master\\|develop\\|\\*" | xargs -n 1 -p git branch -d

   # 必要时强制删除（谨慎使用）
   git branch -D 分支名称
   ```

8. **远程分支清理**
   - 移除已合并的远程分支
   - 清理远程跟踪引用
   - 删除过时的远程分支
   - 更新远程分支信息

   ```bash
   # 修剪远程跟踪分支
   git remote prune origin

   # 删除远程分支
   git push origin --delete branch-name

   # 删除已删除远程分支的本地跟踪
   git branch -dr origin/branch-name
   ```

9. **自动清理脚本**

  ```bash
   #!/bin/bash
   
   # Git branch cleanup script
   set -e
   
   # Configuration
   MAIN_BRANCH="main"
   PROTECTED_BRANCHES=("main" "master" "develop" "staging" "production")
   STALE_DAYS=30
   
   # Functions
   is_protected() {
       local branch=$1
       for protected in "${PROTECTED_BRANCHES[@]}"; do
           if [[ "$branch" == "$protected" ]]; then
               return 0
           fi
       done
       return 1
   }
   
   # Switch to main branch
   git checkout $MAIN_BRANCH
   git pull origin $MAIN_BRANCH
   
   # Clean up merged branches
   echo "Cleaning up merged branches..."
   merged_branches=$(git branch --merged $MAIN_BRANCH | grep -v "\\*\\|$MAIN_BRANCH")
   
   for branch in $merged_branches; do
       if ! is_protected "$branch"; then
           echo "Deleting merged branch: $branch"
           git branch -d "$branch"
       fi
   done
   
   # Prune remote tracking branches
   echo "Pruning remote tracking branches..."
   git remote prune origin
   
   echo "Branch cleanup completed!"
   ```

10. **团队协调**
    - 清理共享分支前通知团队
    - 检查分支是否被他人使用
    - 协调分支清理计划
    - 记录分支清理流程

11. **分支命名规范清理**
    - 识别命名不规范的分支
    - 清理临时或实验性分支
    - 删除旧的修补程序和功能分支
    - 强制执行一致的命名规范

12. **验证和确认**
    - 确认重要分支是否仍然存在
    - 检查是否删除了任何活跃工作
    - 验证远程分支同步
    - 确认团队成员没有问题

    ```bash
    # 验证清理结果
    git branch -a
    git remote show origin
    ```

13. **文档和报告**
    - 记录已清理的分支
    - 报告发现的任何问题或冲突
    - 更新团队关于分支生命周期的文档
    - 创建分支清理计划和策略

14. **回滚程序**
    - 记录如何恢复已删除的分支
    - 使用 reflog 查找已删除的分支提交
    - 创建紧急恢复程序
    - 设置分支恢复脚本

    ```bash
    # 使用 reflog 恢复已删除的分支
    git reflog --no-merges --since="2 weeks ago"
    git checkout -b recovered-branch commit-hash
    ```

15. **自动化设置**
    - 设置自动分支清理脚本
    - 配置用于分支清理的 CI/CD 流水线
    - 创建计划清理作业
    - 实施分支生命周期策略

16. **最佳实践实施**
    - 建立分支生命周期指南
    - 设置自动合并检测
    - 配置分支保护规则
    - 实施代码审查要求

**高级清理选项：**

```bash
# 清理所有合并的分支（受保护的分支除外）
git branch --merged main | grep -E "^ (feature|hotfix|bugfix)/" | xargs -n 1 git branch -d

# 交互式清理并确认
git branch --merged main | grep -v "main\|master\|develop" | xargs -n 1 -p git branch -d

# 批量删除远程分支
git branch -r --merged main | grep origin | grep -v "main\|master\|develop\|HEAD" | cut -d/ -f2- | xargs -n 1 git push origin --delete

# 清理早于特定日期的分支
git for-each-ref --format='%(refname:short) %(committerdate:short)' refs/heads | awk '$2 < "2025-01-01"' | cut -d' ' -f1 | xargs -n 1 git 分支 -D
```

请记住：
- 清理前务必备份重要分支
- 删除共享分支前与团队成员协调
- 先在安全环境中测试清理脚本
- 记录所有清理流程和政策
- 制定定期清理计划，防止代码累积