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