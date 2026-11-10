---
title: LLM 驱动的多市场股票分析工作台
source: raw_20261109_233123_05fba0bc
source_url: https://github.com/ZhuLinsen/daily_stock_analysis
created: '2026-11-09'
updated: '2026-11-10'
tags:
- AI
- 股票研究
- 开源项目
- Agent
status: processed
note_type: content
related:
- daily_stock_analysis 项目11策略与LLM量化框架解析
- Anthropic Agent Skills 仓库指南
- 用 OpenClaw 搭建个人股票分析 Agent
- 用OpenClaw搭建个人股票分析Agent
- LLM辅助量化交易 daily_stock_analysis 11策略解析
- addyosmani agent-skills：AI 编程工程化工作流
- agent-browser：AI 智能体浏览器自动化 CLI
- Astryx 面向人类与 AI 智能体的开源设计系统
- billion-context 长会话上下文压缩代理
- Claude Quickstarts 示例项目与工程模式
---

# LLM 驱动的多市场股票分析工作台

## 核心结论
> daily_stock_analysis 将多市场行情、新闻、公告、技术指标与大语言模型整合为自动化的股票研究和决策报告流水线。其核心价值是统一数据采集、策略分析、决策仪表盘与多渠道推送，并可通过 GitHub Actions 低成本运行。实际分析质量取决于模型、数据源和策略配置，免费行情源可能受网络与接口变动影响，输出结果仍需人工核验，且不构成投资建议。

## 详细内容
## 核心结论

daily_stock_analysis 将多市场行情、新闻、公告、技术指标与大语言模型整合为自动化的股票研究和决策报告流水线。其核心价值是统一数据采集、策略分析、决策仪表盘与多渠道推送，并可通过 [[GitHub Actions]] 低成本运行。实际分析质量取决于模型、数据源和策略配置，免费行情源可能受网络与接口变动影响，输出结果仍需人工核验，且不构成投资建议。

## 项目定位

[[daily_stock_analysis]] 是一款以 Python 3.10+ 为主要运行环境、采用 MIT 许可证的 [[开源项目]]，通过 [[大语言模型]] 为 A股、港股、美股、日股、韩股、台股及 ETF 生成每日分析报告。材料记录该项目获得 68,587 个 GitHub Star，主要使用场景是自选股研究、每日复盘、风险提示和自动化信息推送。

项目覆盖了从数据获取到报告交付的完整链路：

1. 聚合行情、K 线、新闻、公告、基本面与辅助数据。
2. 调用大语言模型执行 [[技术分析]]、[[基本面分析]]、舆情解读和策略推理。
3. 生成包含结论、评分、趋势、买卖点位、风险警报、催化因素与操作检查清单的 [[决策仪表盘]]。
4. 将结果推送至企业微信、飞书、Telegram、Discord、Slack 或邮箱。

## 核心能力

### AI 决策报告

- 输出核心结论、综合评分、趋势判断与参考买卖点位。
- 汇总风险警报、利好催化、最新动态及后续观察事项。
- 同步生成大盘复盘，包括主要指数、涨跌家数、涨停跌停数量和板块表现。
- 将自然语言分析与结构化数据结合，便于快速检查和追踪。

### 多市场数据聚合

系统按市场聚合不同来源的数据，并为各市场设置独立的能力边界与降级规则。[[多源数据聚合]] 覆盖实时行情、历史价格、K 线、技术指标、新闻、公告、基本面和报告辅助字段，不同数据源之间通过优先级与 fallback 机制补齐信息。

### Web 与桌面工作台

[[Web 工作台]] 提供手动分析、任务进度、历史报告、完整 Markdown 导出、[[回测]]、持仓管理、配置管理以及浅色和深色主题。可通过以下方式启动：

```bash
python main.py --webui
python main.py --webui-only
```

默认访问地址为 `http://127.0.0.1:8000`。系统也支持图片、CSV、Excel 和剪贴板导入，并能根据股票代码、名称、拼音及别名补全标的。

### Agent 策略问股

[[Agent]] 模式支持多轮追问、实时行情与新闻调用、会话导出、后台执行以及发送至通知渠道。内置约 15 种策略，包括均线金叉、[[缠论]]、波浪理论、多头趋势、热点题材、事件驱动、成长质量与预期重估；自定义策略文件和多 Agent 编排仍属于实验性能力。

## 数据与模型架构

### AI 模型

系统兼容 Modelflare、AIHubMix、Gemini、OpenAI 兼容接口、DeepSeek、通义千问、Claude，以及通过 Ollama 运行的本地模型。OpenAI 兼容服务还可配置 `OPENAI_BASE_URL` 和 `OPENAI_MODEL`，以适配不同服务商或代理端点。

### 行情数据

内置数据源包括 AkShare、Tushare、Pytdx、Baostock、YFinance 和 Longbridge。AkShare、Baostock、YFinance 等免费源可以零配置运行，适合低成本试用；对于长期定时任务、批量分析或稳定性要求较高的场景，可配置 Tushare、Longbridge 等 token 型数据源。

### 新闻与舆情

新闻搜索支持 Anspire、SerpAPI、Tavily、博查、Brave、MiniMax 和自建 SearXNG。美股场景还可选用 Stock Sentiment API 获取 Reddit、X 与 Polymarket 的社交舆情。新闻源会直接影响事件识别、情绪判断、公告追踪和催化因素提取，因此比单纯的行情数据更直接影响报告质量。

## 部署与配置

### 推荐方式：GitHub Actions

该方式无需服务器，适合零成本定时运行。部署流程为：

1. Fork 仓库。
2. 在仓库 Secrets 中配置模型、通知渠道和自选股。
3. 启用 Actions，并手动运行一次工作流进行验证。

最低配置要求如下：

- **模型服务**：至少配置一个 API Key，例如 `MODELFLARE_KEY`、`AIHUBMIX_KEY`、`GEMINI_API_KEY`、`ANTHROPIC_API_KEY` 或 `OPENAI_API_KEY`。
- **通知渠道**：至少配置一个目标，如 `WECHAT_WEBHOOK_URL`、`FEISHU_WEBHOOK_URL`、`TELEGRAM_BOT_TOKEN` 与 `TELEGRAM_CHAT_ID`，或邮件参数。
- **自选股票**：必须设置 `STOCK_LIST`，代码需带有各市场对应格式。
- **新闻搜索**：建议至少配置 `ANSPIRE_API_KEYS` 或 `SERPAPI_API_KEYS` 等搜索服务。
- **稳定行情**：可选配置 `TUSHARE_TOKEN`、Longbridge OAuth 等参数。

默认工作日 18:00（北京时间）自动执行，并根据 A股、港股和美股节假日规则跳过非交易日；项目也支持强制运行和断点续传。

### 本地与 Docker

本地运行的基本流程为克隆仓库、安装 `requirements.txt`、复制 `.env.example` 并执行主程序：

```bash
git clone https://github.com/ZhuLinsen/daily_stock_analysis.git
cd daily_stock_analysis
pip install -r requirements.txt
cp .env.example .env
python main.py
```

常用命令还包括：

```bash
python main.py --debug
python main.py --dry-run
python main.py --stocks 600519,hk00700,AAPL,2330.TW
python main.py --market-review
python main.py --schedule
python main.py --serve-only
```

除本地方式外，项目还支持 [[Docker]] 部署、FastAPI 服务、[[GitHub Actions]] 和系统定时任务。

## 自动化工作流

典型运行链路可以概括为：

`自选股输入 → 数据源聚合 → 新闻与公告检索 → LLM 策略分析 → 报告生成 → 仪表盘展示 → 通知推送`

其中，[[选股]] 由 `STOCK_LIST` 或手动导入触发；行情、基本面和新闻数据先按优先级进行聚合，再交由模型生成结构化报告。用户既可以在工作台中检查完整 Markdown 报告，也可以只接收仪表盘摘要。

## 风险与使用边界

- 免费行情源可能受到上游限流、接口变更和网络波动影响，不应仅凭单次结果作出交易决策。
- 不同市场的数据覆盖范围并不完全一致，需依据市场支持文档确认字段和分析能力边界。
- 新闻缺失、基本面超时或 fallback 数据源会改变信息完整度，进而影响风险警报与催化因素判断。
- [[大语言模型]] 输出可能存在事实偏差，应核对原始行情、公告和新闻，尤其不能将参考买卖点位视为保证性建议。
- 材料未显示系统包含自动下单能力，其定位主要是研究、报告生成、策略问股、回测与信息推送。
- 项目明确声明仅供学习和研究使用，不构成投资建议；股市存在亏损风险。

## 相关项目

- [[AlphaSift]]：作为选股实现的参考项目。
- [[AlphaEvo]]：用于策略回测、规则验证与自我进化，探索策略参数和组合。

## 来源

- [GitHub：ZhuLinsen/daily_stock_analysis](https://github.com/ZhuLinsen/daily_stock_analysis)

## 相关笔记
- [[daily_stock_analysis 项目11策略与LLM量化框架解析]]
- [[Anthropic Agent Skills 仓库指南]]
- [[用 OpenClaw 搭建个人股票分析 Agent]]
- [[用OpenClaw搭建个人股票分析Agent]]
- [[LLM辅助量化交易 daily_stock_analysis 11策略解析]]
- [[addyosmani agent-skills：AI 编程工程化工作流]]
- [[agent-browser：AI 智能体浏览器自动化 CLI]]
- [[Astryx 面向人类与 AI 智能体的开源设计系统]]
- [[billion-context 长会话上下文压缩代理]]
- [[Claude Quickstarts 示例项目与工程模式]]