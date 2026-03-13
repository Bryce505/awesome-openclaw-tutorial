## 1.同时配置多agents和多模型切换

```

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
          "fallbacks": ["kimi/moonshot-v1", "openai/gpt-4"]
        },
        "tools": {
          "profile": "full"  // 完整工具集
        },
        "system": "你是一个全能的AI助手。你可以帮助用户处理文件、知识管理、日程安排等各种任务。保持友好、专业的态度。",
        "feishu": {
          "app_id": "cli_a123456789abcdef",
          "app_secret": "AbCdEfGhIjKlMnOpQrStUvWxYz"
        },
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
          "profile": "coding"  // 编程工具集
        },
        "system": "你是一位资深的代码审查专家和编程助手。你擅长代码生成、Bug修复、代码审查、性能优化。提供清晰的代码注释和最佳实践建议。",
        "feishu": {
          "app_id": "cli_b123456789abcdef",
          "app_secret": "BcDeFgHiJkLmNoPqRsStUvWxYzA"
        },
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
        "weixin": {  // 可选：企业微信集成
          "webhook_url": "https://qyapi.weixin.qq.com/cgi-bin/webhook/send?key=xxxxx"
        },
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
        "qq": {  // 可选：QQ集成
          "bot_token": "xxxxx",
          "group_id": "xxxxx"
        },
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
    "profile": "full"  // 全局工具配置
  },

  "workspace": {
    "main": {
      "description": "主工作空间"
    },
    "coding": {
      "description": "编程工作空间"
    },
    "content": {
      "description": "内容创作工作空间"
    },
    "news": {
      "description": "资讯工作空间"
    }
  }
}
```
## 切换模型和agent
在 OpenClaw 中，快速切换模型或 Agent 的主要方法是使用命令行工具进行配置调整。以下是步骤：

### 切换模型
1. **设置目标模型**: 使用 `openclaw config set` 命令指定首选模型。
   ```bash
   # 切换到 DeepSeek 模型
   openclaw config set agents.defaults.model.primary "deepseek/deepseek-chat"

   # 切换到 Claude Haiku 模型
   openclaw config set agents.defaults.model.primary "maynor/claude-haiku-4-5-20251001"
   ```

2. **重启生效**: 切换模型后，需要通过以下命令重启网关以使更改生效：
   ```bash
   openclaw gateway restart
   ```

3. **手动查看或编辑配置文件**: 配置文件存储在 `~/.openclaw/openclaw.json`，可以直接编辑该文件进行高级设置。

---

### 切换 Agent
1. **为特定 Agent 配置模型**:
   ```bash
   # 为技术开发 Agent 配置一个模型
   openclaw config set models.providers.anthropic.apiKey "sk-ant-xxx" --agent tech-dev

   # 为内容撰写 Agent 使用不同的模型
   openclaw config set models.providers.openai.apiKey "sk-yyy" --agent content-writer
   ```

2. **验证和测试配置**:
   ```bash
   # 验证 Agent 配置
   openclaw config get --agent tech-dev

   # 测试 Agent 的功能
   openclaw agent --message --agent tech-dev "Hello"
   ```

---

详细说明与更多设置技巧请参考以下文档内容：
- [快速切换模型的文档](https://github.com/Bryce505/awesome-openclaw-tutorial/blob/5e5dd5c293752518b2b2f2da4b8c4de621f5f36d/docs/01-basics/03-quick-start.md#L729-L822)
- [多 Agent 配置示例](https://github.com/Bryce505/awesome-openclaw-tutorial/blob/5e5dd5c293752518b2b2f2da4b8c4de621f5f36d/appendix/E-config-templates.md#L566-L683)

