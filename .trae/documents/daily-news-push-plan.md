# 每日 AI 新闻推送自动化方案

## 概述

通过 TRAE 定时任务（Schedule），每日自动搜索 AI 行业热点新闻，整理为结构化报告，推送到 GitHub 仓库并提供多渠道推送能力。

## 当前状态分析

- **工作区**: `/workspace`，已有 `README.md`（DailyNews_TraeVersion 项目）
- **可用插件**: GitHub 插件（MCP 工具齐全）、Lark 插件（用户无飞书账号，暂不适用）
- **可用工具**: WebSearch（新闻搜索）、Schedule（定时任务）、RunCommand（curl 调用 webhook）
- **用户约束**: 无飞书账号、仅推送给自己、有部分 Webhook 通道

## 方案设计

### 架构概览

```
TRAE Schedule（每日定时触发）
    │
    ├── 1. WebSearch × N → 搜索 AI 新闻
    ├── 2. 格式化 → 生成结构化 Markdown 报告
    ├── 3. GitHub → 存入仓库（每日一个 .md 文件）
    └── 4. Webhook → 推送到钉钉/企业微信等平台
```

### 推送渠道矩阵

| 渠道 | 实现方式 | 前提条件 |
|------|---------|---------|
| GitHub 仓库 | MCP `create_or_update_file` / `push_files` | 需授权 GitHub 连接 |
| 钉钉机器人 | `curl` POST 到 Webhook URL | 需提供 Webhook URL |
| 企业微信机器人 | `curl` POST 到 Webhook URL | 需提供 Webhook URL |
| Server酱（微信） | `curl` POST 到 SendKey URL | 需提供 SendKey |
| 邮件 | `curl` 调用 SMTP API（如 Resend/SendGrid） | 需提供 API Key |

### 新闻搜索策略

分 3 轮搜索，覆盖用户要求的三个方面：

1. **产品发布/功能更新**: 搜索 "OpenAI Google Anthropic AI product launch update today"
2. **社会重要新闻**: 搜索 "AI artificial intelligence major news today"
3. **技术突破/论文**: 搜索 "AI research breakthrough paper published today"

输出格式：
```markdown
# AI 行业日报 — YYYY-MM-DD

## 🔬 技术突破与论文
| 序号 | 标题 | 摘要 | 来源 |
|------|------|------|------|

## 🚀 产品发布与功能更新
| 序号 | 标题 | 摘要 | 来源 |
|------|------|------|------|

## 📰 社会热点
| 序号 | 标题 | 摘要 | 来源 |
|------|------|------|------|
```

## 实施步骤

### Step 1: 创建 GitHub 仓库用于存储每日新闻
- 使用 GitHub MCP `create_repository` 创建仓库（如 `daily-ai-news`）
- 仓库结构：`/YYYY/MM/YYYY-MM-DD.md`

### Step 2: 收集用户 Webhook 配置
- 确认用户有哪些 Webhook URL（钉钉/企业微信/Server酱等）
- 确认是否需要邮件推送（需提供 SMTP/API 配置）

### Step 3: 创建 TRAE 定时任务
- 使用 `Schedule` 工具创建每日定时任务
- 时间建议：每日 08:00（北京时间）
- 任务 `message` 包含完整的新闻搜索 + 格式化 + 推送指令

### Step 4: 验证与测试
- 手动触发一次任务，验证全流程
- 确认 GitHub 仓库文件生成正确
- 确认 Webhook 推送成功

## 假设与决策

1. **GitHub 作为内容中枢**: 即使部分 Webhook 推送失败，新闻内容也会持久化在 GitHub 仓库中
2. **Webhook 推送为尽力交付**: 通过 `curl` 调用，失败时记录日志但不阻断流程
3. **新闻来源**: 主要依赖 WebSearch 工具，辅以 WebFetch 获取详情
4. **推送格式**: GitHub 用完整 Markdown，Webhook 用精简文本（受限于消息长度）

## 验证步骤

1. 手动触发任务后检查 GitHub 仓库是否有当日 `.md` 文件
2. 检查 Webhook 平台是否收到推送消息
3. 检查新闻内容格式是否符合要求（按类型分类、含标题/摘要/来源链接）