---
name: cc-switcher
description: |
  Use when: 用户提到切换提供商、配置 API Key、添加 DeepSeek/Qwen/OpenRouter/Gemini 等第三方提供商、让 Claude Code 使用第三方 API、配置 cc 命令。
  DO NOT use when: 用户只是在问 Claude 模型的能力对比，或只是切换 /model 而不涉及 API Key 配置。
---

## 意图判断

**在做任何事之前**，判断属于哪种场景：

- **首次配置**：providers 目录不存在（macOS/Linux：`~/.claude/providers/`，Windows：`%USERPROFILE%\.claude\providers\`），或启动脚本不存在（macOS/Linux：`~/.claude/launch.sh`，Windows：`%USERPROFILE%\.claude\launch.ps1`）
- **添加新提供商**：目录和脚本已存在，用户只需新增一个 provider 文件
- **删除提供商**：用户提到"删除"、"移除"、"卸载"某个提供商

根据判断结果，执行对应工作流。

**平台检测**：读取系统上下文的 `Platform` 字段——`darwin`/`linux` → 走 macOS/Linux 分支；`win32` → 走 Windows 分支。每步只向用户展示对应平台的命令。

---

## 工作流 A：首次配置

### 步骤 0（你静默执行，不向用户展示命令）

编辑或新增 `.claude.json`，确保包含 `"hasCompletedOnboarding": true`：

- macOS / Linux：`~/.claude.json`
- Windows：`%USERPROFILE%\.claude.json`

操作规则：

- 文件**不存在** → 用 Write 工具创建，内容为 `{ "hasCompletedOnboarding": true }`
- 文件**已存在** → 用 Edit 工具追加该字段，保留其他字段

### 步骤 1：创建目录

**macOS / Linux**
```bash
mkdir -p ~/.claude/providers
```

**Windows PowerShell**
```powershell
New-Item -ItemType Directory -Force "$env:USERPROFILE\.claude\providers"
```

### 步骤 2：创建提供商配置文件

1. **联网查询**（不向用户询问）：获取该提供商的 `ANTHROPIC_BASE_URL` 及各档位模型 ID（档位含义见末尾「参考：配置文件格式」字段说明表），必须从官方文档获取，不得猜测
2. **索要 API Key**：向用户展示查到的 Base URL 和模型名，然后说："请提供你的 [提供商名] API Key，其余配置已自动获取。"
3. **创建文件**（按末尾「参考：配置文件格式」）：
   - macOS / Linux：`~/.claude/providers/<公司名>.json`
   - Windows：`%USERPROFILE%\.claude\providers\<公司名>.json`
   - 文件名使用**公司名**大小写，不使用模型名

### 步骤 3：复制启动脚本

**macOS / Linux**
```bash
cp ~/.claude/skills/cc-switcher/scripts/launch.sh ~/.claude/launch.sh
```

> 此脚本依赖 `fzf` 和 `jq`。如未安装，先运行 `brew install fzf jq`。

**Windows PowerShell**
```powershell
Copy-Item "$env:USERPROFILE\.claude\skills\cc-switcher\scripts\launch.ps1" "$env:USERPROFILE\.claude\launch.ps1"
```

> PowerShell 原生支持 JSON，无需额外工具。`fzf` 可选，未安装时自动降级为数字列表。

### 步骤 4：赋予执行权限

**macOS / Linux**
```bash
chmod +x ~/.claude/launch.sh
```

**Windows PowerShell**
```powershell
# 允许运行本地脚本（只需一次）
Set-ExecutionPolicy -Scope CurrentUser RemoteSigned
```

### 步骤 5：添加 alias

**macOS / Linux（写入 ~/.zshrc）**
```bash
echo "alias cc='~/.claude/launch.sh'" >> ~/.zshrc && source ~/.zshrc
```

**Windows PowerShell（写入 $PROFILE）**
```powershell
Add-Content $PROFILE "`nfunction cc { & `"$env:USERPROFILE\.claude\launch.ps1`" @args }"
. $PROFILE
```

### 完成后告知用户

> 配置完成。以后启动 Claude Code 只需输入 `cc`，脚本会列出所有提供商供你选择。
>
> 需要透传参数时直接附加，例如：`cc --dangerously-skip-permissions`
>
> **注意**：切换提供商后必须**重启 Claude Code 窗口**才会生效。同时打开多个窗口时，请先**退出所有窗口**再运行 `cc`，否则可能导致 API Key 与模型不匹配。

---

## 工作流 B：添加新提供商

### 步骤 1：联网查询配置信息

使用联网搜索获取以下信息（**不向用户询问**）：

- `ANTHROPIC_BASE_URL`：该提供商支持 Anthropic 兼容格式的端点地址
- 可用模型 ID：按档位分别列出（档位含义见末尾「参考：配置文件格式」字段说明表），至少找到主力模型（sonnet 档位）

> 必须从官方文档获取，不得猜测或假设模型名称。

### 步骤 2：向用户确认并索要 API Key

将查询到的配置信息（Base URL、模型名）展示给用户确认，然后说：

> "请提供你的 [提供商名] API Key，其余配置已自动获取。"

### 步骤 3：创建配置文件

按参考格式创建配置文件（格式见末尾）：

- macOS / Linux：`~/.claude/providers/<公司名>.json`
- Windows：`%USERPROFILE%\.claude\providers\<公司名>.json`

`launch.sh` 和 `launch.ps1` 均动态扫描 providers 目录，新建文件后直接运行 `cc` 即可看到新条目，**无需修改脚本**。

### 完成后告知用户

> [提供商名] 已添加。运行 `cc` 重新启动即可选择新提供商。
>
> **注意**：必须**重启 Claude Code 窗口**后新配置才会生效。

---

## 工作流 C：删除提供商

### 步骤 1（静默执行）：检测是否为当前活跃提供商

读取并比对以下两个文件的 `env.ANTHROPIC_BASE_URL` 字段：

- 待删除 provider 文件：
  - macOS/Linux：`~/.claude/providers/<公司名>.json`
  - Windows：`%USERPROFILE%\.claude\providers\<公司名>.json`
- settings.json：
  - macOS/Linux：`~/.claude/settings.json`
  - Windows：`%USERPROFILE%\.claude\settings.json`

若两者相同 → 标记「活跃冲突」，执行步骤 2；否则跳到步骤 3。

### 步骤 2：提示用户（仅当活跃冲突时）

> "你当前正在使用 [模型名称]，确认删除吗？"

等待用户确认后继续。

### 步骤 3：删除 provider 文件

- macOS/Linux：`~/.claude/providers/<公司名>.json`
- Windows：`%USERPROFILE%\.claude\providers\<公司名>.json`

### 完成后告知用户

> [提供商名] 已删除，启动菜单中该选项已移除。
> [若有活跃冲突] 下次运行 `cc` 时选择其他提供商即可，settings.json 会自动更新。

---

## 参考：配置文件格式

```json
{
  "env": {
    "ANTHROPIC_AUTH_TOKEN": "<API Key>",
    "ANTHROPIC_BASE_URL": "<Anthropic 兼容端点>",
    "ANTHROPIC_DEFAULT_HAIKU_MODEL": "<轻量模型ID>",
    "ANTHROPIC_SMALL_FAST_MODEL": "<轻量模型ID>",
    "ANTHROPIC_DEFAULT_OPUS_MODEL": "<旗舰模型ID>",
    "ANTHROPIC_DEFAULT_SONNET_MODEL": "<主力模型ID>",
    "ANTHROPIC_MODEL": "<主力模型ID>",
    "API_TIMEOUT_MS": "3000000",
    "CLAUDE_CODE_DISABLE_NONESSENTIAL_TRAFFIC": "1"
  }
}
```

**字段说明：**

| 字段 | 用途 |
|------|------|
| `ANTHROPIC_AUTH_TOKEN` | API Key（Anthropic 官方可省略，使用内置 OAuth；第三方必须填写） |
| `ANTHROPIC_BASE_URL` | 提供商的 Anthropic 兼容端点 |
| `ANTHROPIC_DEFAULT_HAIKU_MODEL` | 轻量任务（后台操作、自动补全） |
| `ANTHROPIC_SMALL_FAST_MODEL` | 同 Haiku，部分版本使用此字段名，两者同时填写保证兼容 |
| `ANTHROPIC_DEFAULT_SONNET_MODEL` | 主对话（日常默认档位） |
| `ANTHROPIC_DEFAULT_OPUS_MODEL` | 复杂任务（用户手动选择时） |
| `ANTHROPIC_MODEL` | 总默认值，通常与 Sonnet 保持一致 |

提供商只有一个模型时，四个模型字段填同一值即可。
