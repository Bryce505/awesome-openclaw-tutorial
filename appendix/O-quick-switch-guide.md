# 附录O：快速切换模型和 Agent 指南

> 💡 **本附录目标**：帮助你快速掌握在 OpenClaw 中切换模型和 Agent 的各种方法，从临时切换到永久配置，一步到位。

---

## 📋 目录

- [O.1 三种切换方式概览](#o1-三种切换方式概览)
- [O.2 方式一：临时切换（单次生效）](#o2-方式一临时切换单次生效)
- [O.3 方式二：永久切换默认模型](#o3-方式二永久切换默认模型)
- [O.4 方式三：多模型 + 多 Agent 配置（推荐）](#o4-方式三多模型--多-agent-配置推荐)
- [O.5 常见场景速查](#o5-常见场景速查)
- [O.6 故障排查](#o6-故障排查)

---

## O.1 三种切换方式概览

| 方式 | 适用场景 | 是否需要重启 | 复杂度 |
|------|---------|------------|--------|
| 临时切换（CLI 参数） | 单次任务使用特定模型 | 否 | ⭐ 简单 |
| 永久切换默认模型 | 长期更换主力模型 | ✅ 是 | ⭐⭐ 中等 |
| 多模型 + 多 Agent | 按场景灵活调度 | ✅ 是（仅首次配置） | ⭐⭐⭐ 稍复杂 |

---

## O.2 方式一：临时切换（单次生效）

不修改任何配置，仅对本次请求生效。

### O.2.1 CLI 临时指定模型

```bash
# 临时使用指定模型发送消息
openclaw agent --model "deepseek/deepseek-chat" --message "帮我写一段 Python 爬虫"

# 临时使用图片识别模型（local-google 为通过 Antigravity Manager 本地代理的 Google 模型，详见第11章）
openclaw agent --model "local-google/gemini-3-pro-image" --message "分析这张图片" --image ./photo.jpg

# 临时使用推理模型（local-anthropic-opus 为通过 Antigravity Manager 本地代理的 Claude Opus 模型）
openclaw agent --model "local-anthropic-opus/claude-opus-4-5-thinking" --message "推导贝叶斯定理的完整证明"
```

> **💡 提示**：`local-*` 前缀的 provider 是通过 [Antigravity Manager](../docs/03-advanced/11-advanced-configuration.md#111-antigravity-manager完全配置指南) 本地代理的模型；`deepseek`、`kimi`、`openai` 等是直接调用 API 的模型，两者格式一致，前缀不同仅代表访问方式不同。

### O.2.2 临时指定 Agent

```bash
# 使用指定 Agent 处理任务
openclaw agent --agent "编程助手" --message "帮我审查这段代码有没有 Bug"

# 使用写作助手
openclaw agent --agent "写作助手" --message "帮我优化这篇文章的开头"
```

> **✅ 优点**：零配置，即开即用，不影响其他对话。  
> **❌ 缺点**：每次都要手动指定，不适合频繁使用。

---

## O.3 方式二：永久切换默认模型

修改默认模型，所有后续对话都使用新模型。

### O.3.1 查看当前默认模型

```bash
openclaw config get agents.defaults.model.primary
```

**输出示例**：
```
anthropic/claude-sonnet-4-5
```

### O.3.2 查看已配置的模型列表

```bash
openclaw models list
```

**输出示例**：
```
Model                                       Input       Ctx      Local Auth  Tags
anthropic/claude-sonnet-4-5                 text        195k     yes   yes   default
deepseek/deepseek-chat                      text        64k      yes   yes   configured
kimi/moonshot-v1                            text        128k     yes   yes   configured
openai/gpt-4                                text        128k     yes   yes   configured
```

### O.3.3 切换默认模型

```bash
# 切换到 DeepSeek（成本最低）
openclaw config set agents.defaults.model.primary "deepseek/deepseek-chat"

# 切换到 Kimi（长文档处理）
openclaw config set agents.defaults.model.primary "kimi/moonshot-v1"

# 切换到 GPT-4
openclaw config set agents.defaults.model.primary "openai/gpt-4"

# 切换到 Claude Opus（推理任务）
openclaw config set agents.defaults.model.primary "anthropic/claude-opus-4-6"

# 重启 Gateway 使配置生效
openclaw gateway restart
```

### O.3.4 验证切换结果

```bash
openclaw config get agents.defaults.model.primary
openclaw message send "你好，你是什么模型？"
```

> **✅ 优点**：一次切换，长期生效。  
> **❌ 缺点**：只能有一个默认模型，不同任务需要手动切换。

---

## O.4 方式三：多模型 + 多 Agent 配置（推荐）

一次性配置好所有模型和 Agent，通过 `@提及` 或 `--agent` 参数随时切换，无需重复修改配置。

### O.4.1 配置多个模型提供商

编辑配置文件 `~/.openclaw/openclaw.json`，在 `models.providers` 下添加所有需要的模型：

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
  }
}
```

**配置说明**：
- `mode: "merge"`：将新配置与现有提供商合并，不覆盖已有设置
- `${DEEPSEEK_API_KEY}`：引用 `env` 中定义的环境变量，密钥管理更安全
- `auth: "api-key"`：使用 API Key 鉴权方式

### O.4.2 配置多个 Agent

在同一配置文件中添加 `agents.list`，每个 Agent 可以绑定不同的模型、工具集和系统提示词：

```json
{
  "agents": {
    "list": [
      {
        "name": "主助理",
        "description": "全能AI助手，处理日常对话和通用任务",
        "model": {
          "primary": "deepseek/deepseek-chat",
          "fallbacks": ["kimi/moonshot-v1", "openai/gpt-4"]
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
          "primary": "openai/gpt-4",
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
          "primary": "kimi/moonshot-v1",
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
        "fallbacks": ["kimi/moonshot-v1"]
      }
    }
  },

  "tools": {
    "profile": "full"
  },

  "workspace": {
    "main":    { "description": "主工作空间" },
    "coding":  { "description": "编程工作空间" },
    "content": { "description": "内容创作工作空间" },
    "news":    { "description": "资讯工作空间" }
  }
}
```

**字段说明**：

| 字段 | 说明 |
|------|------|
| `name` | Agent 名称，用于 `@提及` 或 `--agent` 参数 |
| `description` | Agent 简介，帮助记忆用途 |
| `model.primary` | 该 Agent 优先使用的模型 |
| `model.fallbacks` | 主模型不可用时的备用模型列表（按顺序尝试） |
| `tools.profile` | 工具权限集：`messaging`/`default`/`coding`/`full`/`all` |
| `system` | 系统提示词，定义 Agent 的角色和行为 |
| `workspace` | 关联的工作空间（隔离不同 Agent 的数据） |

### O.4.3 应用配置

```bash
# 重启 Gateway 使多 Agent 配置生效
openclaw gateway restart

# 验证 Agent 列表
openclaw agents list
```

**输出示例**：
```
Agent    Description                   Model (Primary)
主助理   全能AI助手，处理日常对话和通用任务  deepseek/deepseek-chat
编程助手  代码生成、审查、Bug修复专家       openai/gpt-4
写作助手  文案创作、内容优化专家           kimi/moonshot-v1
资讯助手  信息聚合、分析、总结专家         glm/glm-4
```

### O.4.4 使用多 Agent

配置完成后，有以下几种方式快速切换：

**方式A：CLI 参数**
```bash
openclaw agent --agent "编程助手" --message "审查这段代码"
openclaw agent --agent "写作助手" --message "帮我写一篇产品介绍"
openclaw agent --agent "资讯助手" --message "总结今天的AI领域新闻"
```

**方式B：对话中 @提及**（在 Web UI 或 IM 平台中）
```text
@编程助手 帮我找一下这段代码的 Bug

@写作助手 帮我优化这篇文章的标题

@资讯助手 今天有什么值得关注的技术新闻？
```

> **✅ 优点**：一次配置，按需调用，不同场景使用最适合的模型和工具集。  
> **✅ 适合**：日常工作中需要频繁在不同 AI 能力之间切换的用户。

---

## O.5 常见场景速查

### 场景1：今天临时用一下 DeepSeek

```bash
openclaw agent --model "deepseek/deepseek-chat" --message "你的问题"
```

### 场景2：长期从 Claude 切换到 DeepSeek 节省成本

```bash
openclaw config set agents.defaults.model.primary "deepseek/deepseek-chat"
openclaw gateway restart
```

### 场景3：图片识别任务

```bash
# local-google 为通过 Antigravity Manager 代理的 Gemini 模型，需先完成第11章配置
openclaw agent --model "local-google/gemini-3-pro-image" --message "描述这张图片" --image ./image.jpg
```

### 场景4：需要深度推理的复杂问题

```bash
# local-anthropic-opus 为通过 Antigravity Manager 代理的 Claude Opus 推理模型
openclaw agent --model "local-anthropic-opus/claude-opus-4-5-thinking" --message "分析这个商业决策的利与弊"
```

### 场景5：切换回默认模型

```bash
# 查看之前的配置（如果有备份）
cat ~/.openclaw/openclaw.json.backup | jq '.agents.defaults.model.primary'

# 恢复默认模型
openclaw config set agents.defaults.model.primary "anthropic/claude-sonnet-4-5"
openclaw gateway restart
```

### 场景6：快速检查当前使用的是哪个模型/Agent

```bash
# 查看默认模型
openclaw config get agents.defaults.model.primary

# 查看所有 Agent
openclaw agents list

# 查看所有可用模型
openclaw models list
```

---

## O.6 故障排查

### 问题1：切换模型后不生效

**原因**：忘记重启 Gateway

**解决方法**：
```bash
openclaw gateway restart
```

### 问题2：指定的 Agent 名称不存在

**原因**：`--agent` 参数的名称与配置文件中 `agents.list[].name` 不一致

**解决方法**：
```bash
# 查看所有 Agent 名称
openclaw agents list

# 使用完全匹配的名称
openclaw agent --agent "编程助手" --message "..."
```

### 问题3：模型 ID 找不到

**原因**：该模型尚未配置或 provider 名称拼写错误

**解决方法**：
```bash
# 列出所有已配置的模型
openclaw models list

# 检查配置文件中的 providers
cat ~/.openclaw/openclaw.json | jq '.models.providers | keys'
```

### 问题4：Fallback 模型未生效

**原因**：fallbacks 字段格式错误，或配置文件修改后未重启

**解决方法**：
```bash
# 检查 fallbacks 配置
openclaw config get agents.defaults.model

# 重启 Gateway
openclaw gateway restart
```

---

## 🔗 相关章节

- [第11章：高级配置（多模型切换/成本优化）](../docs/03-advanced/11-advanced-configuration.md) — 详细的模型配置和容灾策略
- [第9章：多平台集成](../docs/03-advanced/09-multi-platform-integration.md) — 多 Agent 在飞书/企微的配置
- [附录A：命令速查表](A-command-reference.md) — 完整命令参考
- [附录C：API 服务商对比](C-api-comparison.md) — 各模型价格与能力对比
- [附录H：配置文件模板](H-config-templates.md) — 更多开箱即用的配置示例
