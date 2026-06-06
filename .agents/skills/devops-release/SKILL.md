---
name: devops-release
description: Use ONLY when publishing a release for qtcloud-write or any of its submodules. Covers the full release workflow: pre-check, submodule commits, version bump, changelog, tag, and push.
---

# DevOps Release

## 子模块 / Sub-crate 地图

| 路径 | 独立仓库 | 类型 |
|------|---------|:--:|
| `docs/gallery` | quanttide-gallery-of-narrative-engineering | 子模块 |
| `docs/journal` | quanttide-journal-of-narrative-engineering | 子模块 |
| `docs/library` | quanttide-library-of-narrative-engineering | 子模块 |
| `docs/profile` | quanttide-profile-of-narrative-engineering | 子模块 |
| `docs/report` | quanttide-report-of-narrative-engineering | 子模块 |
| `docs/roadmap` | quanttide-roadmap-of-narrative-engineering | 子模块 |
| `docs/vision` | quanttide-vision-of-narrative-engineering | 子模块 |
| `assets/fiction` | quanttide-fiction-of-founder | 子模块 |
| `assets/brochure` | quanttide-brochure-of-business-entity | 子模块 |
| `assets/history` | quanttide-history-of-business-entity | 子模块 |
| `assets/vision` | quanttide-vision-of-business-entity | 子模块 |
| `examples/default` | quanttide-laboratory-of-narrative-engineering | 子模块 |
| `apps/qtcloud-write` | quanttide/qtcloud-write | 子模块 |
| `apps/qtcloud-write/src/provider` | quanttide/qtcloud-write | sub-crate |
| `apps/qtcloud-write/src/studio` | quanttide/qtcloud-write | sub-crate |
| `apps/qtcloud-write/src/cli` | quanttide/qtcloud-write | sub-crate |

## 发布工作流

### 首次 clone 后初始化

```bash
git submodule update --init --recursive
```

### 预检查

```bash
# 1. 确认当前分支
git branch --show-current  # 应为 main

# 2. 拉取最新
git pull --rebase origin main

# 3. 检查所有子模块状态
git submodule status
# 有 + 前缀表示子模块有新提交未纳入父仓库
```

### 子模块提交

对每个有变更的子模块，按顺序执行：

```bash
# 1. 进入子模块
cd <submodule-path>

# 2. 确保在分支上（子模块默认 detached HEAD）
git checkout main

# 3. 查看状态
git status

# 4. 暂存并提交
git add <files>
git commit -m "<type>(<scope>): <description>"

# 5. 推送（如被拒绝，先 git pull --rebase origin main）
git push origin main

# 6. 回到父仓库
cd -
```

### 父仓库更新

```bash
# 1. 暂存子模块引用变更
git add <submodule-path>

# 2. 提交
git commit -m "chore: update <submodule> submodule ref (<description>)"

# 3. 推送
git push origin main
```

### 版本标记（正式发布时）

**父仓库发布**：

```bash
git tag -a v<version> -m "release: v<version>"
git push origin v<version>
```

**子模块 break change 发布**（如 gallery 字段重命名）：

```bash
# 1. 子模块内打 tag
cd docs/gallery
git tag -a v<version> -m "release: v<version>"
git push origin v<version>
cd -

# 2. 父仓库更新引用
git add docs/gallery
git commit -m "chore: update docs/gallery submodule ref (v<version>)"

# 3. 父仓库打 tag
git tag -a v<version> -m "release: v<version>"
git push origin v<version>
```

### 回滚

```bash
# 删除本地和远程 tag
git tag -d v<bad-version>
git push origin :refs/tags/v<bad-version>

# 回滚父仓库到上一个稳定 commit
git reset --hard <previous-commit>
git push --force origin main
```

## 注意事项

- 子模块默认处于 detached HEAD。进入子模块后先 `git checkout main` 再提交，否则 commit 会丢失引用。
- 子模块推送被拒绝时（`! [rejected]`），在**子模块内**执行 `git pull --rebase origin main` 再 push，然后回到父仓库更新引用。
- 父仓库 `git diff --stat <submodule>` 只显示子模块 commit hash 变更，不显示内部 diff。确认变更需进入子模块目录查看。
- 多个子模块有变更时，每个子模块单独 commit 和 push，最后在父仓库一次性提交所有子模块引用变更。
- 子模块间有依赖关系时（如 gallery 的 style/motif/story 被多个实验和工具依赖），break change 需先发布被依赖的子模块，再更新依赖方。
