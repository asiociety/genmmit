# genmmit

AI 驱动的 Git Commit 消息生成器。

在执行 `git commit` 时自动调用大模型 API，根据 staged changes 生成 commit 消息。

## 特性

- **Git Hook 集成** - 通过 `prepare-commit-msg` hook 无缝融入 git 工作流
- **OpenAI 兼容** - 支持任何 OpenAI 兼容 API（OpenAI、Claude proxy、Ollama 等）
- **可配置风格** - 内置 Conventional Commits、Angular、简洁中文等模板
- **自定义 Prompt** - 支持自定义模板或内联 prompt
- **零依赖** - 纯 Bash 脚本，仅需 `curl` 和 `jq`
- **静默失败** - 生成失败不阻塞提交，错误写入日志

## 安装

```bash
# 1. 克隆仓库
git clone https://github.com/yourname/genmmit.git
cd genmmit

# 2. 创建配置目录
mkdir -p ~/.config/genmmit/templates

# 3. 复制模板和配置
cp templates/*.txt ~/.config/genmmit/templates/
cp config.example ~/.config/genmmit/config

# 4. 编辑配置，填入 API 信息
vim ~/.config/genmmit/config

# 5. 在目标项目中安装 hook
cp genmmit /path/to/your/project/.git/hooks/prepare-commit-msg
chmod +x /path/to/your/project/.git/hooks/prepare-commit-msg
```

### 全局安装（可选）

如果你使用了 `core.hooksPath`：

```bash
cp genmmit ~/.config/git/hooks/prepare-commit-msg
chmod +x ~/.config/git/hooks/prepare-commit-msg
```

## 配置

### 全局配置 `~/.config/genmmit/config`

```bash
# API 设置（必填）
GENMMIT_API_URL="https://api.openai.com/v1/chat/completions"
GENMMIT_API_KEY="sk-your-api-key"
GENMMIT_MODEL="gpt-4o"

# 模板（可选，默认 conventional.txt）
GENMMIT_TEMPLATE="conventional.txt"

# 超时秒数（可选，默认 30）
GENMMIT_TIMEOUT=30

# 日志（可选）
GENMMIT_LOG_ENABLED=true
GENMMIT_LOG_LEVEL="error"  # error | warn | info | debug
```

### 项目级配置 `.genmmit`

在项目根目录创建 `.genmmit` 文件覆盖全局配置：

```bash
# 使用简洁中文风格
GENMMIT_TEMPLATE="simple.txt"

# 或使用内联 prompt
GENMMIT_PROMPT="用一句话描述这次改动，中文，不超过 30 字"
```

### 环境变量

环境变量优先级最高，适合 CI 或临时覆盖：

```bash
GENMMIT_TEMPLATE="angular.txt" git commit
```

## 内置模板

| 模板 | 说明 | 示例 |
|------|------|------|
| `conventional.txt` | Conventional Commits 格式 | `feat(auth): add login endpoint` |
| `angular.txt` | Angular 风格 | `fix(core): resolve memory leak` |
| `simple.txt` | 简洁中文 | `修复登录页面样式问题` |

### 自定义模板

在 `~/.config/genmmit/templates/` 创建 `.txt` 文件：

```
根据以下 diff 生成 commit 消息。

要求：
- 格式：emoji + 中文描述
- 示例：✨ 新增用户注册功能

只输出 commit 消息，不要解释。

---
Diff:
```

然后配置使用：

```bash
GENMMIT_TEMPLATE="my-style.txt"
```

## 使用

```bash
# 正常使用 - 自动生成消息
git add .
git commit

# 跳过生成 - 手动指定消息
git commit -m "手动写的消息"

# 修改不满意的消息
git commit --amend
```

## 日志

日志文件位于 `~/.config/genmmit/error.log`：

```
[2026-01-05 10:30:22] [ERROR] API error: HTTP 401
[2026-01-05 10:32:10] [WARN] Diff truncated: 8521 -> 4000 chars
[2026-01-05 10:33:05] [INFO] Commit message generated (23 chars)
```

## 故障排除

**Hook 没有触发**

检查是否设置了全局 `core.hooksPath`：

```bash
git config --global core.hooksPath
```

如果有，需要将 genmmit 安装到该目录。

**生成失败但没有报错**

查看日志：

```bash
cat ~/.config/genmmit/error.log
```

**手动测试**

```bash
./genmmit /tmp/test-msg && cat /tmp/test-msg
```

## License

MIT
