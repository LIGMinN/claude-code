# Claude-code 仓库 Git SOP

本仓库是 [`anthropics/claude-code`](https://github.com/anthropics/claude-code) 的个人 fork，
约定 `main` 作为**纯净上游镜像**，本地工作走 `custom-<机器名>` 分支隔离。

---

## 远端结构

```
upstream  → anthropics/claude-code     只读源
origin    → LIGMinN/claude-code (fork)
            ├── main                   ← 镜像 upstream，禁止本地 commit
            └── custom-<host>          ← 本地工作分支，多机隔离
```

当前已有的 custom 分支：

| 分支名               | 机器              | 备注       |
| -------------------- | ----------------- | ---------- |
| `custom-ligminn-mbp` | LIGMinN MacBook Pro | 主力开发机 |

新机器加入时，**新建** `custom-<host>` 分支，**不要复用**已有 custom 分支。

---

## 三条核心铁律

1. **`main` 永远不接受本地 commit**。任何本地改动一律走 `custom-<host>`。
2. **`main` 只做 fast-forward**。同步上游用 `--ff-only`，绝不 merge commit。
3. **`custom-<host>` 永不 merge 回 `main`**。要回流上游应通过 PR 直接发到 anthropics。

---

## 工作流

### 同步上游（任意机器）

```bash
git checkout main
git fetch upstream
git merge --ff-only upstream/main
git push origin main
```

### 推送当前机器的本地工作

```bash
git checkout custom-ligminn-mbp
git add -A && git commit -m "..."
git push origin custom-ligminn-mbp
```

### 另一台机器拉取本机工作

```bash
git fetch origin
# 首次：
git checkout custom-ligminn-mbp
# 已存在则：
git pull --ff-only origin custom-ligminn-mbp
```

### 让 custom 分支跟上最新上游

```bash
git checkout custom-ligminn-mbp
git fetch upstream
git rebase upstream/main          # 推荐 rebase，保持线性
git push --force-with-lease origin custom-ligminn-mbp
```

> 注意：`--force-with-lease` 比 `--force` 安全，会拒绝覆盖远端他人的新 commit。

### 想给 anthropics 发 PR

```bash
git checkout -b pr/<topic> main   # 基于纯净 main 切 PR 分支
# ... 改代码 ...
git push origin pr/<topic>
gh pr create --repo anthropics/claude-code --base main
```

**不要**直接从 `custom-ligminn-mbp` 发 PR——里面会带其他本地特异内容。

---

## 常见错误自检

| 现象                              | 处理                                                  |
| --------------------------------- | ----------------------------------------------------- |
| `main` 误 commit 了本地内容       | `git reset --hard upstream/main` 后重新 push          |
| `main` 出现 merge commit          | 同上，强制对齐 upstream                               |
| custom 分支与 main 分叉太远难合并 | `git rebase upstream/main`，逐个解决冲突             |
| 多机 push 冲突                    | 先 `pull --rebase` 再 push，**不要** `--force`        |

---

## 备份/状态参考

- 本仓库初始化日期：2026-06-13
- 初始 HEAD：`ca9f604` (chore: Update CHANGELOG.md and feed.xml)
- 创建方式：`gh repo clone LIGMinN/claude-code`（含完整历史与 tags）
