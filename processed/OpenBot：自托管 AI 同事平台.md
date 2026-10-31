---
title: OpenBot：自托管 AI 同事平台
source: raw_20261029_233117_d0a86411
source_url: https://github.com/CopilotKit/OpenBot
created: '2026-10-29'
updated: '2026-10-31'
tags:
- 开源项目
- AI
- 风险控制
- 智能体平台
- 私有化部署
status: processed
note_type: content
related:
- OpenShell 自主 AI 智能体安全运行时
- SkillSpector：AI Agent 技能安全扫描器
- Cloudflare 多阶段安全审计 Skill
- DeepTutor 开源智能导师系统
- Claude-Mem：AI Agent 持久记忆层
- Impeccable：AI 编程代理设计控制工具
- Lightpanda：面向 AI 智能体的无头浏览器
- OpenMAIC：多智能体互动课堂平台
- T3 Code：跨端 AI 编程代理控制台
- Claude Code Game Studios 多智能体游戏开发框架
---

# OpenBot：自托管 AI 同事平台

## 核心结论
> [[OpenBot]] 是可自行部署和改造的开源 [[AI 智能体]] 平台：每个 Bot 拥有独立浏览器、文件空间与受控工具，并通过 [[AG-UI]] 接入不同框架的智能体。其关键差异不是能够调用工具，而是所有动作都先经过 [[策略网关]] 校验并写入 [[审计日志]]，从而把自主执行纳入可审批、可拒绝、可追责的治理体系。项目仍处于 Beta 且定位为模板，适合重视数据自主权的团队试点，但上线前必须补齐身份认证、密钥、策略与生产运维。

## 详细内容
## 核心结论
[[OpenBot]] 是可自行部署和改造的开源 [[AI 智能体]] 平台：每个 Bot 拥有独立浏览器、文件空间与受控工具，并通过 [[AG-UI]] 接入不同框架的智能体。其关键差异不是能够调用工具，而是所有动作都先经过 [[策略网关]] 校验并写入 [[审计日志]]，从而把自主执行纳入可审批、可拒绝、可追责的治理体系。项目仍处于 Beta 且定位为模板，适合重视数据自主权的团队试点，但上线前必须补齐身份认证、密钥、策略与生产运维。

## 项目定位
- **形态**：MIT 许可证下的开源项目，而非注册即用的 SaaS。用户需要克隆仓库或运行容器镜像，并自行替换示例租户包、配置同事、技能与渠道。
- **运行方式**：支持 [[自托管]]，核心组件通过 Docker Compose 启动，业务数据保存在 PostgreSQL；也可以使用单个 Docker 镜像运行应用、API、Bot 浏览器及可选的嵌入式 PostgreSQL。
- **模型选择**：项目不内置模型。管理员自行提供模型供应商和密钥，凭据采用 [[密钥管理|加密存储]]，不会写入日志。
- **典型角色**：初始提供通用助手、知识助手和风险分析员，示例目录还包含费用审核、会议跟进、发布说明、工单分流、值班交接、续约准备等岗位型 Bot。

## 核心架构
### 1. 每个 Bot 拥有一台独立计算机
- 每个 Bot 由 [[容器隔离]] 承载，拥有独立的 Chromium、工作区卷和浏览器配置，可保留各自的网站登录状态。
- Bot 不仅能够浏览网页，还能执行命令行、处理文件、安装依赖并调用获准工具。
- 可选的 gVisor 与 Chromium 沙箱进一步缩小宿主机暴露面。

### 2. AG-UI 解耦智能体框架
- 任何实现 [[AG-UI]] 协议的端点都可以注册为 Bot，不依赖 LangGraph、Mastra、CrewAI、Pydantic AI 或 Google ADK 等特定框架。
- 自定义智能体端点必须通过目标地址校验；私网地址需要加入 `AGENT_ENDPOINT_ALLOWED_HOSTS`，且匹配规则不支持通配符。
- [[CopilotKit Intelligence]] 负责持久化会话线程与记忆，可使用托管服务，也支持自行部署。

### 3. 网关先决策、再记录、后执行
正常产品路径中的调用顺序为：

`用户请求 → 服务端与 AG-UI → 策略网关 → 权限判定 → 写入审计记录 → 执行动作`

- 网关统一处理浏览器、文件、Shell、[[MCP]] 和组件调用，避免智能体绕过治理直接操作环境。
- 策略采用 [[CEL 策略]]，可以检查工具名称、意图、Bot、操作用户、页面域名、命令、文件、MCP 服务及任务发起者等字段。
- 规则遵循 [[Fail Closed]]：拒绝规则优先于允许规则，缺少策略时默认不许可，规则损坏时拒绝执行而非放行。

## 治理与安全
- **审计追踪**：允许、拒绝和失败的动作都会进入 `/admin/audit`；拒绝记录包含具体命中的规则。
- **人工接管**：遇到登录或双因素认证时，Bot 可请求人工接管。接管、释放及相关事件均被记录，人工控制期间 Bot 动作会被拒绝而不是排队执行。
- **密钥保护**：管理员凭据为只写、静态加密，API 不返回明文，审计事件只记录秘密被请求及使用时长，不记录秘密内容。
- **网络隔离**：Bot 计算机默认绑定 `127.0.0.1`，每个容器还需提供独立令牌；监督器使用共享令牌保护 Docker 管理接口。
- **身份与权限**：支持 Google、Microsoft、Okta、企业 SAML 和 OIDC，并通过成员角色、审批规则与边界策略控制访问。
- **连接器治理**：MCP 工具按 Bot 授权；未知工具和自定义服务器工具默认按写操作处理，避免智能体绕过目录直接访问供应商网站。

## 主要使用界面
| 界面 | 用途 |
| --- | --- |
| `/channel/:id` | 与 Bot 对话、观看屏幕、查看其运行记录 |
| `/bots` | 创建并配置同事、角色、端点与可见范围 |
| `/bots/:id/responsibilities` | 设置长期目标和自主工作边界 |
| `/bots/:id/approvals` | 配置哪些操作可以免审批执行 |
| `/admin/boundaries` | 管理浏览器、文件与 MCP 动作策略 |
| `/admin/audit` | 查阅允许、拒绝和失败的动作 |
| `/admin/credentials` | 管理加密凭据 |
| `/admin/plugins` | 配置 MCP 服务及其工具授权 |
| `/routines` | 管理周期性任务，异常连续失败后自动停用 |

## 快速启动
运行前需要 Docker、Bun 1.4+、CopilotKit Intelligence 项目或许可，以及模型 API 密钥。基本流程为：

1. 执行 `cp .env.example .env` 与 `bun install`。
2. 登录并选择 Intelligence 项目，再运行 `bun scripts/setup-learning.ts`。
3. 填写模型密钥；生产环境需重新生成 `KEY_ENCRYPTION_KEY`，示例值仅适合本地使用。
4. 执行 `bash scripts/start.sh`，访问 `http://localhost:3010`。
5. 停止服务可执行 `bash scripts/stop.sh`；数据库、Bot 文件和浏览器配置均保存在卷中。

示例配置默认启用 `OPENBOT_SINGLE_USER=true`，会将所有请求视为同一管理员。仅可将其用于本地体验；在其他用户可访问前，必须关闭该选项并配置 [[身份认证]]，非本机部署还应在入口启用 TLS。

## 适用场景与限制
- **适合**：需要保留数据和运行控制权、希望接入多种智能体框架，并要求工具调用可审计、可审批的组织。
- **主要价值**：框架无关、逐 Bot 隔离、策略前置、全量审计，以及能够观察和接管的执行界面。
- **当前限制**：项目仍是 Beta，预期存在缺陷和接口变化；它提供的是可改造起点，而非开箱即用的企业产品。
- **运维成本**：部署者需要负责模型接入、身份提供方、密钥轮换、策略设计、数据库、容器资源及升级维护。
- **本地 Intelligence 限制**：自托管 Intelligence 的本地评估方案目前面向带 Docker Desktop 的 macOS，属于预览能力，不应直接视为生产安装方案。

## 项目信息
- 原始来源：[GitHub - CopilotKit/OpenBot](https://github.com/CopilotKit/OpenBot)
- 许可证：MIT
- 当前状态：Beta
- 原始材料采集时：约 7,879 Stars，并曾位列当日仓库趋势第 3；该数据仅反映当时热度，不代表生产成熟度。

## 相关笔记
- [[OpenShell 自主 AI 智能体安全运行时]]
- [[SkillSpector：AI Agent 技能安全扫描器]]
- [[Cloudflare 多阶段安全审计 Skill]]
- [[DeepTutor 开源智能导师系统]]
- [[Claude-Mem：AI Agent 持久记忆层]]
- [[Impeccable：AI 编程代理设计控制工具]]
- [[Lightpanda：面向 AI 智能体的无头浏览器]]
- [[OpenMAIC：多智能体互动课堂平台]]
- [[T3 Code：跨端 AI 编程代理控制台]]
- [[Claude Code Game Studios 多智能体游戏开发框架]]