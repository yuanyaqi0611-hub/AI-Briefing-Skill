# 🤖 AI Briefing Skill

> Automated AI content briefing skill for Claude — collects from 15+ global & Chinese platforms, generates bilingual EN+ZH digests, and delivers to Feishu, Slack, Email, Telegram, Notion, WeChat, and RSS.

---

## ✨ What It Does

This skill turns Claude into a fully automated AI content briefing system. Give it your sources, preferred language, output format, and delivery target — and it handles everything from collection to delivery.

```
Sources → Collect → Filter & Score → Summarize → Format → Deliver
```

---

## 📡 Sources Supported

### Global
| Platform | Method |
|---|---|
| Twitter / X | API v2 · Nitter RSS |
| RSS / Blogs | feedparser |
| YouTube | yt-dlp + transcript |
| Podcasts | RSS feed |
| GitHub | Releases API |
| arXiv | RSS (cs.AI · cs.LG · cs.CL) |
| HuggingFace Papers | Daily Papers feed |
| Papers With Code | REST API |
| Reddit | praw (r/MachineLearning · r/LocalLLaMA · r/ChatGPT) |
| Hacker News | Firebase API · Algolia search |
| Newsletters | Substack · Beehiiv RSS (The Rundown · Superhuman AI · The Batch · TLDR AI · Ben's Bites · etc.) |
| LinkedIn | web_search proxy |

### 🇨🇳 Chinese Platforms
| Platform | Method |
|---|---|
| 知乎 | RSS · keyword search |
| 小红书 | web_search proxy · session scrape |
| 抖音 / TikTok | yt-dlp · Whisper fallback |
| 微信公众号 | RSSHub · WeChat2RSS |
| B站 (Bilibili) | Bilibili API · yt-dlp |
| 微博 | Official API · RSSHub |
| 得到 / 喜马拉雅 | RSS |

---

## 📄 Output Formats

| Format | Description |
|---|---|
| `feishu_card` | Feishu interactive card with bilingual sections |
| `newsletter_doc` | Bilingual EN+ZH Markdown / Word document |
| `podcast_script` | Two-host conversational script + show notes |
| `presentation` | Slide deck outline (via pptx skill) |
| `slack_message` | Slack Block Kit with clickable links |
| `email_html` | Responsive HTML email with inline CSS |
| `json` | Structured JSON for downstream processing |

All formats include **mandatory source links** on every item.

---

## 🚀 Delivery Targets

| Platform | Notes |
|---|---|
| **Feishu (飞书)** | Webhook · Bot API · DM |
| **Slack** | Incoming Webhook · Web API · threads |
| **Email** | SMTP · SendGrid |
| **Telegram** | Bot API · channel / group |
| **Notion** | REST API · MCP tool |
| **WeChat Article** | 微信公众号 draft + publish |
| **RSS feed** | Self-hosted XML (GitHub Pages · Cloudflare · Nginx) |

---

## 🌍 Language Support

| Config | Output |
|---|---|
| `"bilingual"` | EN + ZH side by side (default) |
| `"zh"` | Chinese only |
| `"en"` | English only |
| Any ISO 639-1 code | `"ja"` · `"ko"` · `"fr"` · `"es"` · etc. |

Technical terms (model names, org names, paper titles) are always preserved in English regardless of language setting.

---

## ⚙️ Configuration

```json
{
  "timezone":         "Asia/Shanghai",
  "cadence":          "daily",
  "cadence_cron":     "0 8 * * *",
  "languages":        ["zh", "en"],
  "primary_language": "zh",
  "sources": {
    "rss":     [{ "url": "https://www.anthropic.com/news/rss.xml", "tier": 1.0 }],
    "twitter": [{ "username": "karpathy", "tier": 1.0 }],
    "arxiv":   [{ "category": "cs.AI", "tier": 0.9 }],
    "zhihu":   [{ "type": "keyword", "keyword": "大模型" }]
  },
  "outputs":  [{ "format": "feishu_card", "enabled": true }],
  "delivery": {
    "targets": [
      { "platform": "feishu", "webhook_url": "${FEISHU_WEBHOOK_URL}" }
    ]
  }
}
```

---

## 🔗 Agent Integration

The pipeline is fully composable with other agents and systems:

### REST API (FastAPI)
```bash
# Trigger a run
POST /briefing
{ "since_hours": 24, "formats": ["json", "feishu_card"], "callback_url": "https://..." }

# Poll status
GET /briefing/{job_id}

# Fetch result
GET /briefing/{job_id}/result?fmt=feishu_card
```

### MCP Server
Register as an MCP tool and let Claude invoke it natively:
```json
{
  "mcpServers": {
    "ai-briefing": {
      "command": "python",
      "args": ["/path/to/mcp_server.py"]
    }
  }
}
```

Available tools: `run_briefing` · `get_briefing_result` · `list_briefing_sources` · `add_briefing_source`

### Webhook Callbacks
Signed HMAC-SHA256 callbacks notify upstream agents when a run completes:
```json
{
  "event":      "briefing.completed",
  "job_id":     "550e8400-...",
  "result_url": "https://your-server/briefing/JOB_ID/result",
  "outputs_available": ["json", "feishu_card", "newsletter_doc"]
}
```

---

## 📦 Installation

1. Download `ai-briefing.skill` from the [Releases](../../releases) page
2. Open **Claude.ai → Settings → Skills → Install from file**
3. Select `ai-briefing.skill`
4. Start using it — try: *"帮我生成今天的 AI 早报"*

---

## 📁 Skill Structure

```
ai-briefing/
├── SKILL.md                        # Main skill instructions
└── references/
    ├── sources.md                  # Per-platform collection code
    ├── output-formats.md           # Format specs with source link rules
    ├── delivery.md                 # Delivery platform setup
    └── agent-integration.md        # REST API · MCP · callbacks
```

---

## 📋 Example Output

```
🤖 AI 早报 · 2026-03-29

🧠 模型与研究 / Models & Research
🔥 [GPT-5.4 支持百万 token 上下文](https://openai.com/...)
   自主执行多步工作流，OSWorld-V 得分 75%，超越人类基准线
   来源: OpenAI Blog

🛠️ 工具与产品 / Tools & Products
[Google Gemini 深度整合 Workspace](https://blog.google/...)
   Docs · Sheets · Gmail 全面接入，新增跨文件 AI 问答
   来源: Google Blog

💡 核心洞察
• AI 正从对话工具向自主执行代理快速演进
• 头部平台营收爆发，OpenAI ARR 突破 250 亿美元

📊 18 条 · 6 个来源
```

---

## 🔒 Security

- All REST endpoints protected with `X-API-Key` header
- Webhook callbacks signed with HMAC-SHA256
- API keys stored in environment variables only — never in `config.json`
- Source URLs validated before any item is published

---

## 📄 License

MIT — free to use, modify, and distribute.
