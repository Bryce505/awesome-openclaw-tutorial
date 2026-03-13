# 如何快速切换 Agent 和模型

> 🚀 **本篇目标**：掌握在 OpenClaw 中快速切换 Agent 和模型的各种方法，根据不同场景灵活选择最合适的 AI。

## 快速导航

- 🔄 [快速切换模型](#快速切换模型)
- 🤖 [配置多个 Agent](#配置多个-agent)
- ⚡ [快速切换 Agent](#快速切换-agent)
- 📋 [速查命令表](#速查命令表)

---

## 快速切换模型

### 方法1：命令行切换（最快）

```bash
# 切换默认模型
openclaw config set agents.defaults.model.primary "模型ID"

# 重启生效
openclaw gateway restart
```

**常用模型切换示例**：

```bash
# 切换到 DeepSeek（性价比最高）
openclaw config set agents.defaults.model.primary "deepseek/deepseek-chat"
openclaw gateway restart

# 切换到 Claude Haiku（速度最快）
openclaw config set agents.defaults.model.primary "anthropic/claude-haiku-4-5"
openclaw gateway restart

# 切换到 Claude Sonnet（质量与速度平衡）
openclaw config set agents.defaults.model.primary "anthropic/claude-sonnet-4-5"
openclaw gateway restart

# 切换到 Kimi（超长上下文，适合长文档）
openclaw config set agents.defaults.model.primary "kimi/moonshot-v1-128k"
openclaw gateway restart

# 切换到 Gemini（支持图片识别）
openclaw config set agents.defaults.model.primary "google/gemini-2.0-flash-exp"
openclaw gateway restart
```

> 💡 **小技巧**：可以把常用的切换命令保存为 Shell 别名，一个字母完成切换。
>
> ```bash
> # 加入 ~/.bashrc 或 ~/.zshrc
> alias oc-ds='openclaw config set agents.defaults.model.primary "deepseek/deepseek-chat" && openclaw gateway restart'
> alias oc-claude='openclaw config set agents.defaults.model.primary "anthropic/claude-sonnet-4-5" && openclaw gateway restart'
> ```

### 方法2：Web UI 切换

1. 打开 OpenClaw Web UI：`http://127.0.0.1:18789/?token=你的token`
2. 点击左侧菜单 **「配置」**（Config）
3. 找到 `agents.defaults.model.primary` 字段
4. 修改模型 ID
5. 点击 **「保存」**，然后 **「重启 Gateway」**

### 方法3：临时切换（不改全局配置）

如果只想临时用某个模型处理一条消息，不想改变默认配置：

```bash
# 临时使用指定模型发送消息
openclaw agent --model "anthropic/claude-opus-4-6" --message "解释一下量子纠缠"

# 临时使用 Gemini 识别图片
openclaw agent --model "google/gemini-2.0-flash-exp" --message "描述这张图片" --image ./photo.jpg
```

### 方法4：查看当前使用的模型

```bash
# 查看当前默认模型
openclaw config get agents.defaults.model.primary

# 查看所有可用模型列表
openclaw models list
```

---

## 配置多个 Agent

多 Agent 配置让你可以为不同场景设置专门的助手，每个 Agent 可以有独立的模型、工具集和提示词。

### 完整的多 Agent 配置示例

在配置文件 `~/.openclaw/openclaw.json` 中添加以下内容：

```json
{
  "env": {
    "DEEPSEEK_API_KEY": "sk-xxxxxxxx",
    "KIMI_API_KEY": "sk-xxxxxxxx",
    "OPENAI_API_KEY": "sk-xxxxxxxx",
    "GLM_API_KEY": "xxxxxxxx"
  },

  "models": {
    "mode": "merge",
    "providers": {
      "deepseek": {
        "baseUrl": "https://api.deepseek.com",
        "apiKey": "${DEEPSEEK_API_KEY}",
        "auth": "api-key",
        "api": "openai-chat"
      },
      "kimi": {
        "baseUrl": "https://api.moonshot.cn/v1",
        "apiKey": "${KIMI_API_KEY}",
        "auth": "api-key",
        "api": "openai-chat"
      },
      "openai": {
        "baseUrl": "https://api.openai.com/v1",
        "apiKey": "${OPENAI_API_KEY}",
        "auth": "api-key",
        "api": "openai-chat"
      },
      "glm": {
        "baseUrl": "https://open.bigmodel.cn/api/paas/v4",
        "apiKey": "${GLM_API_KEY}",
        "auth": "api-key",
        "api": "openai-chat"
      }
    }
  },

  "agents": {
    "list": [
      {
        "name": "主助理",
        "description": "全能AI助手，处理日常对话和通用任务",
        "model": {
          "primary": "deepseek/deepseek-chat",
          "fallbacks": ["kimi/moonshot-v1-128k", "openai/gpt-4o"]
        },
        "tools": {
          "profile": "full"
        },
        "system": "你是一个全能的AI助手。你可以帮助用户处理文件、知识管理、日程安排等各种任务。保持友好、专业的态度。",
        "workspace": "main"
      },

      {
        "name": "编程助手",
        "description": "代码生成、审查、Bug修复专家",
        "model": {
          "primary": "openai/gpt-4o",
          "fallbacks": ["deepseek/deepseek-chat"]
        },
        "tools": {
          "profile": "coding"
        },
        "system": "你是一位资深的代码审查专家和编程助手。你擅长代码生成、Bug修复、代码审查、性能优化。提供清晰的代码注释和最佳实践建议。",
        "workspace": "coding"
      },

      {
        "name": "写作助手",
        "description": "文案创作、内容优化专家",
        "model": {
          "primary": "kimi/moonshot-v1-128k",
          "fallbacks": ["glm/glm-4", "deepseek/deepseek-chat"]
        },
        "tools": {
          "profile": "default"
        },
        "system": "你是一位专业的文案创意专家。你擅长创意写作、文案优化、SEO优化、多平台发布策略。帮助用户生成高质量的内容。",
        "workspace": "content"
      },

      {
        "name": "资讯助手",
        "description": "信息聚合、分析、总结专家",
        "model": {
          "primary": "glm/glm-4",
          "fallbacks": ["deepseek/deepseek-chat"]
        },
        "tools": {
          "profile": "full"
        },
        "system": "你是一位信息分析专家。你擅长网络信息聚合、内容总结、趋势分析、热点解读。帮助用户快速了解行业动态。",
        "workspace": "news"
      }
    ],

    "defaults": {
      "model": {
        "primary": "deepseek/deepseek-chat",
        "fallbacks": ["kimi/moonshot-v1-128k"]
      }
    }
  },

  "tools": {
    "profile": "full"
  }
}
```

### Agent 配置字段说明

| 字段 | 说明 | 示例 |
|------|------|------|
| `name` | Agent 的显示名称 | `"编程助手"` |
| `description` | Agent 的功能描述 | `"代码生成、审查专家"` |
| `model.primary` | 主模型（优先使用） | `"deepseek/deepseek-chat"` |
| `model.fallbacks` | 备用模型列表（主模型失败时按序使用） | `["kimi/moonshot-v1-128k"]` |
| `tools.profile` | 工具集配置（`full`/`coding`/`default`/`messaging`） | `"full"` |
| `system` | Agent 的系统提示词（人格和专长定义） | `"你是一个..."` |
| `workspace` | 工作区（文件隔离） | `"coding"` |

### tools.profile 说明

| Profile | 说明 | 适用场景 |
|---------|------|---------|
| `messaging` | 仅聊天，无工具 | 纯对话 |
| `default` | 默认工具集 | 日常使用 |
| `coding` | 编程工具集（代码执行、文件操作等） | 编程任务 |
| `full` | 完整工具集（包括命令执行） | 复杂自动化 |
| `all` | 全开（所有工具） | 高级用户 |

---

## 快速切换 Agent

### 方法1：在对话中直接 @ 指定 Agent

如果你配置了多个 Agent，可以在消息中直接指定使用哪一个：

```text
@编程助手 帮我写一个 Python 爬虫

@写作助手 帮我优化这段文案

@资讯助手 总结一下今天的 AI 行业动态
```

### 方法2：命令行切换默认 Agent

```bash
# 查看所有配置的 Agent
openclaw agents list

# 切换默认 Agent
openclaw config set agents.defaults.id "编程助手"

# 重启使配置生效
openclaw gateway restart
```

### 方法3：Web UI 切换 Agent

1. 打开 Web UI：`http://127.0.0.1:18789/?token=你的token`
2. 点击右上角的 **Agent 选择器**
3. 从下拉列表中选择目标 Agent
4. 立即生效，无需重启

### 方法4：为不同平台配置不同 Agent

可以让飞书机器人使用"主助理"，QQ 机器人使用"资讯助手"：

```json
{
  "channels": {
    "feishu": {
      "agent": "主助理"
    },
    "qq": {
      "agent": "资讯助手"
    }
  }
}
```

---

## 速查命令表

### 模型切换命令

| 操作 | 命令 |
|------|------|
| 查看当前模型 | `openclaw config get agents.defaults.model.primary` |
| 切换到 DeepSeek | `openclaw config set agents.defaults.model.primary "deepseek/deepseek-chat"` |
| 切换到 Claude Sonnet | `openclaw config set agents.defaults.model.primary "anthropic/claude-sonnet-4-5"` |
| 切换到 Claude Haiku | `openclaw config set agents.defaults.model.primary "anthropic/claude-haiku-4-5"` |
| 切换到 Kimi | `openclaw config set agents.defaults.model.primary "kimi/moonshot-v1-128k"` |
| 切换到 Gemini | `openclaw config set agents.defaults.model.primary "google/gemini-2.0-flash-exp"` |
| 临时使用指定模型 | `openclaw agent --model "模型ID" --message "消息内容"` |
| 列出所有可用模型 | `openclaw models list` |
| 重启 Gateway 使配置生效 | `openclaw gateway restart` |

### Agent 管理命令

| 操作 | 命令 |
|------|------|
| 查看所有 Agent | `openclaw agents list` |
| 切换默认 Agent | `openclaw config set agents.defaults.id "Agent名称"` |
| 查看配置文件 | `cat ~/.openclaw/openclaw.json` |
| 备份配置 | `cp ~/.openclaw/openclaw.json ~/.openclaw/openclaw.json.bak` |

---

## 场景化切换建议

| 使用场景 | 推荐 Agent | 推荐模型 | 理由 |
|---------|-----------|---------|------|
| 日常对话 | 主助理 | DeepSeek | 性价比最高 |
| 写代码 / Debug | 编程助手 | GPT-4o / DeepSeek | 代码能力强 |
| 写文章 / 文案 | 写作助手 | Kimi | 长上下文，中文写作好 |
| 看长文档 / PDF | 主助理 | Kimi 128k | 超长上下文 |
| 图片识别 | 主助理 | Gemini Flash | 多模态支持 |
| 复杂推理 / 数学 | 任意 | Claude Opus Thinking | 推理能力最强 |
| 节省成本 | 任意 | DeepSeek / GLM | 国产模型价格低 |

---

## 相关文档

- [第3章：快速上手 → 模型选择指南](01-basics/03-quick-start.md#模型选择指南)
- [第11章：高级配置 → 多模型切换策略](03-advanced/11-advanced-configuration.md#112-多模型切换策略)
- [附录C：API 服务商对比](../appendix/C-api-comparison.md)
- [附录H：配置文件模板](../appendix/H-config-templates.md)
