---
title: 闲鱼 AI 智能监控系统
source: raw_20261103_233133_b4e1a9a6
source_url: https://github.com/Usagi-org/ai-goofish-monitor
created: '2026-11-03'
updated: '2026-11-06'
tags:
- 开源项目
- AI
- 风险控制
- Playwright
- 闲鱼监控
- Docker
status: processed
note_type: content
related:
- OpenShell 自主 AI 智能体安全运行时
- OpenBot：自托管 AI 同事平台
- SkillSpector：AI Agent 技能安全扫描器
- Cloudflare 多阶段安全审计 Skill
- Google Agent Skills 云端 AI 工作流技能库
- TradingAgents 多智能体金融交易框架
- Kronos：面向金融K线序列的首个开源基础模型
- NVIDIA Auto Ontology 企业语义层
- Ruby on Rails 框架概览
- 面向真实工程实践的 Agent Skills 技能库
---

# 闲鱼 AI 智能监控系统

## 核心结论
> ai-goofish-monitor 通过 [[Playwright]] 与 [[多模态模型]] 将闲鱼商品搜索、筛选、分析和通知串成闭环，支持多任务并发、[[定时调度]]及 [[Docker]] 自托管。它适合需要持续筛选特定商品的监控场景，但部署时必须妥善处理 [[登录态]]、默认密码、接口密钥、平台风控与请求频率合规问题。

## 详细内容
## 项目定位

[[ai-goofish-monitor]] 是一套基于 [[Python]]、[[Playwright]] 和 [[AI]] 的闲鱼商品监控系统，通过浏览器自动化获取商品信息，再利用支持图片输入的模型进行筛选和推荐。项目提供完整的 [[FastAPI]] 后端与 Web 管理界面，面向商品监控、技术研究和个人自动化场景；原始材料记录其 GitHub Stars 数量为 16,231。

系统的核心价值不是单纯采集商品，而是将“搜索—筛选—分析—通知—记录”整合为可持续运行的自动化闭环。

## 核心能力

| 能力 | 说明 |
|---|---|
| Web 可视化管理 | 管理任务、闲鱼账号、AI 分析标准、运行日志和监控结果 |
| AI 任务创建 | 使用自然语言描述需求，由 [[多模态模型]] 异步生成商品分析标准 |
| 关键词任务 | 直接配置关键词规则，无需经过 AI 标准生成流程 |
| [[多任务并发]] | 每个任务可独立设置关键词、价格、发布时间、区域和 Prompt |
| [[高级筛选]] | 支持包邮、发布时间范围，以及省、市、区三级 [[区域筛选]] |
| [[定时调度]] | 通过 Cron 周期性执行监控任务 |
| [[即时通知]] | 支持 ntfy、Gotify、企业微信、Bark、Telegram 和 [[Webhook]] |
| 账号与 [[代理轮换]] | 支持多闲鱼账号、任务绑定、代理池轮换、失败重试与黑名单机制 |
| [[Docker]] 部署 | 官方镜像内置 Chromium，可通过 Docker Compose 快速启动 |

## 任务创建与筛选

任务支持两种创建方式：

1. **AI 判断**：填写详细需求后，系统在后台 Job 中生成分析标准，再创建监控任务。创建接口返回 `202` 和 Job ID，前端通过独立进度弹窗轮询处理状态。
2. **关键词判断**：直接填写关键词规则并创建任务，不调用 AI 生成标准，适合规则明确且追求稳定性的场景。

一个任务可组合以下条件：

- 关键词或 AI 生成的分析标准；
- 价格区间与包邮要求；
- 商品发布时间范围；
- 省、市、区三级区域；
- 指定闲鱼账号或由系统自动选择；
- Cron 定时规则和独立的 AI Prompt。

区域筛选会显著缩小结果集。若目标是先观察整体市场，建议暂时留空；只有明确关注特定地区时再启用。

## 监控工作流程

1. 用户手动启动或由 [[定时调度]] 触发任务。
2. 系统选择闲鱼账号和代理配置，开始搜索商品并处理分页。
3. 发现新商品后，抓取商品详情、卖家信息并下载图片。
4. 调用支持图片输入的 AI 模型，对商品是否符合分析标准进行判断。
5. 推荐商品通过通知渠道推送；无论是否推荐，任务结果都会写入 [[SQLite]]。
6. 若遇到登录失效、平台风控或网络异常，系统执行账号或代理轮换，并按配置进行 [[失败重试]]。

## 快速部署

推荐使用 Docker Compose：

```bash
git clone https://github.com/Usagi-org/ai-goofish-monitor
cd ai-goofish-monitor
cp .env.example .env
vim .env
docker compose up -d
docker compose logs -f app
```

默认 Web UI 地址为 `http://127.0.0.1:8000`，官方镜像地址为 `ghcr.io/usagi-org/ai-goofish:latest`。镜像已经包含 Chromium，无需在宿主机额外安装浏览器；如果修改 `SERVER_PORT`，需要同步调整 `docker-compose.yaml` 的端口映射。

### 必要配置

| 配置项 | 用途 |
|---|---|
| `OPENAI_API_KEY` | AI 模型接口密钥 |
| `OPENAI_BASE_URL` | [[OpenAI 兼容 API]] 地址 |
| `OPENAI_MODEL_NAME` | 支持图片输入的模型名称 |
| `WEB_USERNAME` / `WEB_PASSWORD` | Web UI 登录凭据 |

其他重要配置包括 `PROXY_URL`、`PROXY_POOL`、`PROXY_ROTATION_ENABLED`、`PROXY_ROTATION_MODE`、`PROXY_ROTATION_RETRY_LIMIT`、`PROXY_BLACKLIST_TTL`、`TASK_FAILURE_THRESHOLD` 和 `TASK_FAILURE_PAUSE_SECONDS`。

## 首次使用

1. 启动服务并访问 Web UI，使用账号密码登录。
2. 在“闲鱼账号管理”中，通过 Chrome 扩展导出闲鱼 [[登录态]] JSON 并导入。
3. 登录态默认保存在 `state/` 目录，例如 `state/acc_1.json`。
4. 在“任务管理”中创建任务，按需绑定账号、筛选条件、AI Prompt 和定时规则。
5. 运行任务后，在结果页查看商品，在日志页排查登录失效、风控或 AI 调用问题。

## 数据存储与迁移

当前在线主存储为 [[SQLite]]，默认路径为 `data/app.sqlite3`，也可通过 `APP_DATABASE_FILE` 修改。应用启动时会自动建库建表，并尝试一次性导入旧版 `config.json`、`jsonl/` 和 `price_history/` 中的历史数据。

仍保存在文件系统中的内容包括：

- `state/`：闲鱼账号登录状态；
- `prompts/`：任务提示词；
- `logs/`：运行日志；
- `images/`：商品图片与任务临时图片；
- `data/`：任务、结果和价格历史等 SQLite 数据。

商品图片会写入 `images/task_images_<task_name>/`，任务结束后默认清理。升级后应检查 SQLite 数据是否完整，再决定是否移除旧版数据挂载。常规运行从 SQLite 读取任务；只有显式传入 `--config <path>` 时才进入 JSON 兼容模式。

## 开发与接口

本地开发需要 Python 3.10+、Node.js、npm、Playwright CLI 和 Chromium。`start.sh` 会检查浏览器环境、安装依赖、构建前端并启动后端：

```bash
chmod +x start.sh
./start.sh
```

也可以分别启动后端与前端：

```bash
python -m src.app
cd web-ui
npm install
npm run dev
```

主要任务接口包括：

- `POST /api/tasks/generate`：创建 AI 或关键词任务；
- `GET /api/tasks/generate-jobs/{job_id}`：查询 AI 任务生成进度；
- `POST /auth/status`：校验 Web UI 登录凭据。

前端使用 Vite 开发服务器，并将 `/api`、`/auth` 和 `/ws` 代理到后端。FastAPI 同时提供根目录下的前端构建产物和 API 文档 `/docs`。

## 运维与安全

- 默认 Web UI 凭据为 `admin/admin123`，生产环境必须修改；登录状态会保存在浏览器本地，用于路由守卫和 WebSocket 初始化。
- 导入的闲鱼登录态、代理地址、通知令牌和 AI 密钥均属于敏感配置，不应提交到版本库或公开日志。
- 任务日志用于定位登录态失效、平台风控、代理异常和 AI 接口错误，应结合失败阈值与暂停策略进行 [[风险控制]]。
- Docker 环境应保持无头运行，避免因浏览器显示模式增加资源消耗。

## 使用边界

使用时应遵守闲鱼的用户协议与 `robots.txt` 规则，控制请求频率，避免给平台造成负担或导致账号被限制。项目仅适合学习与技术研究，不得用于非法用途；项目采用 MIT 许可证，并按“现状”提供，不附带担保。

## 来源

- GitHub：https://github.com/Usagi-org/ai-goofish-monitor
- 参考项目：`superboyyy/xianyu_spider`

## 相关笔记
- [[OpenShell 自主 AI 智能体安全运行时]]
- [[OpenBot：自托管 AI 同事平台]]
- [[SkillSpector：AI Agent 技能安全扫描器]]
- [[Cloudflare 多阶段安全审计 Skill]]
- [[Google Agent Skills 云端 AI 工作流技能库]]
- [[TradingAgents 多智能体金融交易框架]]
- [[Kronos：面向金融K线序列的首个开源基础模型]]
- [[NVIDIA Auto Ontology 企业语义层]]
- [[Ruby on Rails 框架概览]]
- [[面向真实工程实践的 Agent Skills 技能库]]