# Claude Code + N8N 工作流教程

> 来源：Jason's Notion 公开文档  
> 抓取日期：2026-03-31

## 概述

n8n 工作流没有被淘汰，它成为了智能体的底层执行搭档。通过 n8n MCP 和 n8n Skills 协议，让 Claude Code 或 OpenCode 这种终端智能体自主完成工作流的创建与修改。

**GitHub仓库：** `czlonkowski/n8n-mcp`， `czlonkowski/n8n-skills`

---

## 环境准备与依赖安装

### n8n 安装

两种方式：**npm 全局安装** 和 **Docker 容器部署**。

#### npm 安装

```bash
npm install -g n8n
```

设置环境变量（Windows cmd）：
```bash
setx NODES_EXCLUDE "[]"
setx N8N_ENABLE_EXECUTE_COMMAND "true"
setx N8N_BLOCK_ENV_VARS_IN_EXECUTE_COMMAND "false"
setx N8N_COMMUNITY_PACKAGES_ALLOW_TOOL_USAGE "true"
setx N8N_RUNNERS_ENABLED "true"
setx N8N_ENFORCE_SETTINGS_FILE_PERMISSIONS "true"
```

#### Docker 部署

创建 `compose.yaml`：

```yaml
services:
  n8n:
    image: docker.n8n.io/n8nio/n8n
    container_name: n8n
    user: root
    restart: unless-stopped
    ports:
      - "5678:5678"
    environment:
      - TZ=Asia/Shanghai
      - GENERIC_TIMEZONE=Asia/Shanghai
      - N8N_RUNNERS_ENABLED=true
      - N8N_ENFORCE_SETTINGS_FILE_PERMISSIONS=true
      - N8N_BLOCK_FS_WRITE_ACCESS=false
      - NODES_EXCLUDE=[]
      - NODE_FUNCTION_ALLOW_BUILTIN=*
      - NODE_FUNCTION_ALLOW_EXTERNAL=*
      - N8N_UNVERIFIED_PACKAGES_ENABLED=true
    volumes:
      - n8n_data:/home/node/.n8n
      - "D:/repo/daily-info:/data"

volumes:
  n8n_data:
```

启动：`docker-compose up -d`  
访问：`http://localhost:5678/`

### n8n 前置配置

1. **生成 n8n API 密钥**：在设置面板创建 API 凭证
2. **创建 Credential**：需要手动创建 OpenAI 凭证（API Key + Base URL）

### Claude Code 部署与模型转接

```bash
npm install -g @anthropic-ai/claude-code
```

**Windows（cmd）：**
```bash
setx ANTHROPIC_BASE_URL "https://openrouter.ai/api"
setx ANTHROPIC_MODEL "moonshotai/kimi-k2.5"
setx ANTHROPIC_AUTH_TOKEN "你的API Key"
setx ANTHROPIC_API_KEY ""
```

**Mac（Terminal）：**
```bash
echo 'export ANTHROPIC_BASE_URL="https://openrouter.ai/api"' >> ~/.zshrc
echo 'export ANTHROPIC_MODEL="minimax/minimax-m2.7"' >> ~/.zshrc
echo 'export ANTHROPIC_AUTH_TOKEN="你的API Key"' >> ~/.zshrc
echo 'export ANTHROPIC_API_KEY=""' >> ~/.zshrc
source ~/.zshrc
```

> ANTHROPIC_API_KEY 必须强制置空，防止触发原生 Anthropic 鉴权拦截。

**兼容模型推荐：**
- 智谱 GLM：`https://open.bigmodel.cn/api/anthropic`
- DeepSeek：`https://api.deepseek.com/anthropic`

### MCP 与 Skills 配置

**Skills 目录：** `~/.claude/skills/`（放入 n8n Skills 文件）

**MCP 配置（~/.claude.json）：**
```json
{
  "mcpServers": {
    "n8n-mcp": {
      "command": "npx",
      "args": ["-y", "n8n-mcp@latest"],
      "env": {
        "MCP_MODE": "stdio",
        "N8N_API_URL": "http://localhost:5678",
        "N8N_API_KEY": "你的API密钥",
        "WEBHOOK_SECURITY_MODE": "moderate",
        "LOG_LEVEL": "error",
        "DISABLE_CONSOLE_OUTPUT": "true"
      }
    }
  }
}
```

**OpenCode 配置（~/.config/opencode/opencode.json）：**
```json
"n8n-mcp": {
  "type": "local",
  "command": ["npx", "-y", "n8n-mcp@latest"],
  "environment": {
    "MCP_MODE": "stdio",
    "N8N_API_URL": "http://localhost:5678",
    "N8N_API_KEY": "你的API密钥",
    "WEBHOOK_SECURITY_MODE": "moderate",
    "LOG_LEVEL": "error",
    "DISABLE_CONSOLE_OUTPUT": "true"
  }
}
```

---

## 连接验证

发送以下指令验证 MCP 与 n8n 的交互状态：

```markdown
执行以下探测任务：
1. 列出你目前从 n8n-mcp 服务器获取到的所有可用工具（Tools）。
2. 调用 n8n_health_check 或使用 search_nodes 搜索 'PostgreSQL' 节点。
3. 列出你所拥有的n8n相关skills。
4. 尝试读取我本地 n8n 实例的系统版本信息。
```

---

## 实战：构建 AI 资讯处理工作流

**目标：** 构建『AI 论文情报站』自动化工作流

**业务逻辑：**
- 触发：手动触发
- 数据采集：抓取 Google DeepMind RSS（https://deepmind.google/blog/rss.xml）最新10条
- AI处理：OpenAI节点翻译 title 和 description 为中文学术语言
- 数据聚合：整理论文名称、摘要、原始链接、发布日期
- 保存：存入 n8n 内置 Data Tables

**交付规范：**
- 先调用 `n8n-mcp_get_node` 确认节点参数 Schema
- 严禁生成新凭证，必须引用提供的 Credential ID
- 先展示逻辑计划和关键节点代码草案，收到"确认"后再部署

---

## 工程价值对比矩阵

| | 智能体直接执行 | n8n + 智能体 |
|---|:---:|:---:|
| **密钥安全性** | 高风险，凭证明文暴露 | 隔离管理，存储在 n8n 保险库 |
| **可观测性** | 黑盒运行，定位困难 | 节点级监控，可追踪每个步骤 |
| **算力消耗** | 每次重复任务消耗 Token | 初次构建后零模型成本 |

---

## 关键资源

- n8n MCP：`czlonkowski/n8n-mcp`
- n8n Skills：`czlonkowski/n8n-skills`
- n8n 官网：https://n8n.io
- Claude Code：https://github.com/anthropics/claude-code
