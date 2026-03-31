# cc-switcher

> 在 Claude Code 中快速切换 AI 提供商 - 支持 Anthropic、DeepSeek、Qwen、OpenRouter、Gemini 等任意 Anthropic 兼容 API

![License](https://img.shields.io/badge/license-MIT-blue.svg)
![Platform](https://img.shields.io/badge/platform-macOS%20%7C%20Linux%20%7C%20Windows-lightgray.svg)

## 功能特性

- 🔄 **一键切换** - 通过交互式菜单在不同 AI 提供商间快速切换
- 📦 **多提供商管理** - 支持同时配置多个提供商，按需选择使用
- 🖥️ **跨平台支持** - macOS、Linux、Windows 全覆盖
- 🎨 **友好界面** - fzf 模糊搜索菜单，优雅降级体验
- ⚡ **即开即用** - 配置一次，永久生效

## 安装方式

### 方式一：Git 安装（推荐）

```bash
# macOS / Linux
git clone https://github.com/idongdongh/cc-switcher.git ~/.claude/skills/cc-switcher

# Windows PowerShell
git clone https://github.com/idongdongh/cc-switcher.git "$env:USERPROFILE\.claude\skills\cc-switcher"
```

### 方式二：手动下载

1. 下载本仓库代码
2. 将 `cc-switcher` 文件夹放置在 `~/.claude/skills/` 目录下

## 使用方法

### 首次配置

在 Claude Code 中触发 `cc-switcher` skill，或手动运行：

```bash
cc
```

脚本将引导你完成：

1. 创建 providers 目录
2. 配置首个 AI 提供商（需要 API Key）
3. 复制启动脚本
4. 设置 shell alias

### 添加新提供商

配置完成后，在 Claude Code 中告诉它：

```
帮我添加 DeepSeek 提供商
```

系统会自动：

- 查询官方文档获取配置信息
- 向你确认 Base URL 和模型名
- 索要 API Key
- 创建配置文件

### 删除提供商

```
删除 OpenRouter 提供商
```

## 配置文件格式

提供商配置文件位于：

- **macOS / Linux**: `~/.claude/providers/<公司名>.json`
- **Windows**: `%USERPROFILE%\.claude\providers\<公司名>.json`

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

### 字段说明

| 字段 | 用途 |
|------|------|
| `ANTHROPIC_AUTH_TOKEN` | API Key（Anthropic 官方可省略，第三方必须填写） |
| `ANTHROPIC_BASE_URL` | 提供商的 Anthropic 兼容端点 |
| `ANTHROPIC_DEFAULT_HAIKU_MODEL` | 轻量任务（后台操作、自动补全） |
| `ANTHROPIC_SMALL_FAST_MODEL` | 同 Haiku，保证兼容性 |
| `ANTHROPIC_DEFAULT_SONNET_MODEL` | 主对话（日常默认档位） |
| `ANTHROPIC_DEFAULT_OPUS_MODEL` | 复杂任务（用户手动选择时） |
| `ANTHROPIC_MODEL` | 总默认值，通常与 Sonnet 保持一致 |

> **提示**：提供商只有一个模型时，四个模型字段填同一值即可。

## 依赖

- **macOS / Linux**: `fzf`、`jq`
  ```bash
  brew install fzf jq
  ```
- **Windows**: 无强制依赖（PowerShell 原生支持 JSON，fzf 可选）

## 注意事项

1. **重启生效**：切换提供商后需重启 Claude Code 窗口
2. **避免冲突**：多个窗口同时运行时，请先退出所有窗口再运行 `cc`
3. **参数透传**：支持附加参数，如 `cc --dangerously-skip-permissions`

## 支持的提供商

理论上支持任意 Anthropic API 兼容的第三方服务，已验证可用的包括：

- Anthropic（官方）
- DeepSeek
- Alibaba Cloud（通义千问）
- OpenRouter
- Google Gemini
- 其他兼容服务

## 项目结构

```
cc-switcher/
├── SKILL.md           # Skill 定义文件（Claude Code 识别此文件自动加载）
└── scripts/
    ├── launch.sh      # macOS / Linux 启动脚本
    └── launch.ps1     # Windows PowerShell 启动脚本
```

## 许可证

MIT License

## 链接

- [GitHub 仓库](https://github.com/idongdongh/cc-switcher)
- [Claude Code 官方文档](https://github.com/anthropics/claude-code)

