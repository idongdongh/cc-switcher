# cc-switcher

> 在 Claude Code 中快速切换 AI 提供商 —— 支持 Anthropic、DeepSeek、Qwen、OpenRouter、Gemini 等任意 Anthropic 兼容 API

![License](https://img.shields.io/badge/license-MIT-blue.svg)
![Platform](https://img.shields.io/badge/platform-macOS%20%7C%20Linux%20%7C%20Windows-lightgray.svg)

## 它能做什么

通过 Claude Code 对话，自动完成提供商的配置、切换和管理，无需手动编辑配置文件。

- **首次配置**：自动创建目录结构、查询官方文档获取 Base URL 和模型 ID、写入配置文件、安装 `cc` 启动命令
- **添加提供商**：联网查询新提供商的配置信息，只需提供 API Key
- **删除提供商**：安全删除，若删除的是当前活跃提供商会提前提示
- **一键切换**：运行 `cc` 通过 fzf 交互菜单选择提供商启动 Claude Code

## 安装

```bash
# macOS / Linux
git clone https://github.com/idongdongh/cc-switcher.git ~/.claude/skills/cc-switcher

# Windows PowerShell
git clone https://github.com/idongdongh/cc-switcher.git "$env:USERPROFILE\.claude\skills\cc-switcher"
```

安装完成后无需额外操作，在 Claude Code 中对话即可触发。

### 依赖

**macOS / Linux**：需要 `fzf` 和 `jq`

```bash
brew install fzf jq
```

**Windows**：无强制依赖，PowerShell 原生支持 JSON；`fzf` 可选，未安装时自动降级为数字列表。

## 使用方法

### 首次配置

在 Claude Code 中输入：

```
帮我配置 Claude Code 提供商
```

或直接说你想切换的目标：

```
我想用 DeepSeek
```

Skill 会自动完成以下所有步骤，中途只需你提供 API Key：

1. 创建 `~/.claude/providers/` 目录
2. 联网查询提供商官方文档，获取 Base URL 和模型 ID
3. 向你确认配置信息并索要 API Key
4. 写入提供商配置文件
5. 安装启动脚本（`~/.claude/launch.sh` 或 `launch.ps1`）
6. 配置 `cc` 命令别名

配置完成后，以后启动 Claude Code 只需运行 `cc`。

### 添加新提供商

```
帮我添加 OpenRouter 提供商
```

### 删除提供商

```
删除 OpenRouter 提供商
```

### 切换提供商

**方式一（推荐）**：在终端运行 `cc`，通过交互菜单选择

**方式二**：在 Claude Code 中说

```
切换到 Qwen 提供商
```

> **注意**：切换后必须**重启 Claude Code 窗口**才会生效。同时打开多个窗口时，请先退出所有窗口再运行 `cc`，否则可能导致 API Key 与模型不匹配。

## 配置文件格式

提供商配置文件位于 `~/.claude/providers/<公司名>.json`（Windows：`%USERPROFILE%\.claude\providers\`）：

```json
{
  "env": {
    "ANTHROPIC_AUTH_TOKEN": "<API Key>",
    "ANTHROPIC_BASE_URL": "<Anthropic 兼容端点>",
    "ANTHROPIC_DEFAULT_HAIKU_MODEL": "<轻量模型 ID>",
    "ANTHROPIC_SMALL_FAST_MODEL": "<轻量模型 ID>",
    "ANTHROPIC_DEFAULT_OPUS_MODEL": "<旗舰模型 ID>",
    "ANTHROPIC_DEFAULT_SONNET_MODEL": "<主力模型 ID>",
    "ANTHROPIC_MODEL": "<主力模型 ID>",
    "API_TIMEOUT_MS": "3000000",
    "CLAUDE_CODE_DISABLE_NONESSENTIAL_TRAFFIC": "1"
  }
}
```

| 字段 | 说明 |
|------|------|
| `ANTHROPIC_AUTH_TOKEN` | API Key（Anthropic 官方可省略；第三方必须填写） |
| `ANTHROPIC_BASE_URL` | 提供商的 Anthropic 兼容端点 |
| `ANTHROPIC_DEFAULT_HAIKU_MODEL` | 轻量任务（后台操作、自动补全） |
| `ANTHROPIC_SMALL_FAST_MODEL` | 同 Haiku，两者同时填写保证兼容性 |
| `ANTHROPIC_DEFAULT_SONNET_MODEL` | 主对话（日常默认档位） |
| `ANTHROPIC_DEFAULT_OPUS_MODEL` | 复杂任务（用户手动选择时） |
| `ANTHROPIC_MODEL` | 总默认值，通常与 Sonnet 保持一致 |

提供商只有一个模型时，四个模型字段填同一值即可。

## 支持的提供商

理论上支持任意 Anthropic API 兼容服务，已验证可用：

- Anthropic（官方）
- DeepSeek
- Alibaba Cloud（通义千问）
- OpenRouter
- Google Gemini

## 项目结构

```
cc-switcher/
├── SKILL.md           # Skill 定义文件（Claude Code 自动加载）
├── README.md          # 本文件
└── scripts/
    ├── launch.sh      # macOS / Linux 启动脚本
    └── launch.ps1     # Windows PowerShell 启动脚本
```

## 许可证

MIT License
