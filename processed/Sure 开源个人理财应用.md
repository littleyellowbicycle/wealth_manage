---
title: Sure 开源个人理财应用
source: raw_20261110_233132_692bb41e
source_url: https://github.com/we-promise/sure
created: '2026-11-10'
updated: '2026-11-13'
tags:
- 开源项目
- 个人理财
- AI
- 金融
- 自托管
status: processed
note_type: content
related:
- Kronos：面向金融K线序列的首个开源基础模型
- Buzz 人类与 AI 智能体协作工作区
- Wealthfolio：本地优先的个人财务追踪开源工具
- Claude Trading Skills：个人投资者 AI 交易工作流
- Chrome DevTools MCP：AI 代理的浏览器调试工具
- Cloudflare 多阶段安全审计 Skill
- Codex Security 漏洞扫描与修复工作流
- LLM 驱动的多市场股票分析工作台
- Modern Web Guidance 编码代理知识库
- OpenBot：自托管 AI 同事平台
---

# Sure 开源个人理财应用

## 核心结论
> Sure 是已停止维护的 Maybe Finance 项目的社区分支，以 AGPLv3 许可延续其个人财务与财富管理能力。项目可通过 Docker 自托管或云平台一键部署，并支持网页、桌面、移动端、API 客户端与 LLM 代理；持续发展的关键在于性能优化、社区协作、许可与商标合规，以及探索可持续的运营模式。

## 详细内容
## 项目概览

- **项目地址**：[we-promise/sure](https://github.com/we-promise/sure)
- **材料采集时数据**：11,578 Stars；该数字会随时间变化。
- **项目定位**：面向个人用户的开源 [[个人理财]] 与 [[财富管理]] 应用，覆盖现金流、净资产、支出和资金流出等视图。
- **项目来源**：Sure 是已归档的 Maybe Finance 项目的社区分支，由社区继续维护和开发。
- **主要形态**：既可通过 [[Docker]] 进行 [[自托管部署]]，也提供 Render、PikaPods、Hostim 等平台的部署模板。

## 核心能力

1. **综合财务看板**：界面集中展示现金流、净资产、支出与资金流出，并可借助 [[AI 助手]] 回答储蓄等财务问题。
2. **多终端访问**：支持浏览器、macOS 桌面端、移动应用、[[API]] 客户端和 [[LLM 代理]]，便于将财务数据接入自动化工作流。
3. **灵活的 AI 配置**：部署模板分为无 AI、基于 OpenAI 的简单 AI，以及使用外部助手和 AlphaClaw 管理 OpenClaw 网关的外部 AI 三类。
4. **自主管理数据与部署**：用户可以自行维护运行环境，减少对单一商业托管服务的依赖。
5. **社区协作**：项目公开性能面板和调用栈，邀请开发者共同定位数据密集型请求的性能瓶颈。

## 项目沿革

Maybe Finance 团队在 2021—2022 年投入近 100 万美元开发个人财务与财富管理应用，产品曾包含连接真实 CFP/CFA 顾问的咨询服务。该商业方向未能持续运营，团队于 2023 年中停止开发并开源代码，随后短暂推出过托管版本，但同样未能形成可持续的 B2C 业务。

Sure 因此承担了延续代码库和探索新运营路径的职责。它保留了 AGPLv3 开源基础，同时以独立社区项目身份重新组织开发、部署和贡献流程。

## 部署与开发

### Docker 自托管

完整的自托管流程参见 [Docker 部署指南](https://github.com/we-promise/sure/blob/main/docs/hosting/docker.md)。项目可作为完整应用独立运行，并允许选择是否启用 AI 能力。

### 云平台模板

- **Sure - No AI**：部署 Web 应用、Sidekiq Worker、[[PostgreSQL]] 和 Render Key Value，AI Token 留空。
- **Sure - Simple AI**：增加 OpenAI 配置，并提供支持 [[pgvector]] 的 PostgreSQL；部署时需要填写 OpenAI Token。
- **Sure - External AI**：接入外部助手，并通过 AlphaClaw 管理部署在 Render 上的 OpenClaw 网关。
- **其他平台**：仓库还提供 PikaPods 和 Hostim 的一键部署入口。

### 本地开发

本地开发的核心依赖如下：

- Ruby：版本以仓库中的 `.ruby-version` 为准。
- PostgreSQL：要求 9.3 以上，推荐使用最新稳定版。
- Redis：要求 5.4 以上，推荐使用最新稳定版。

初始化与启动命令：

`cd sure` → `cp .env.local.example .env.local` → `bin/setup` → `bin/dev`

应用默认运行于 `http://localhost:3000`。如需演示数据，可执行 `rake demo_data:default`；README 提供的本地演示账户为 `user@example.com` / `Password1!`。

## 性能与贡献

Sure 已被明确描述为数据密集型应用，复杂请求可能出现性能问题。项目通过 [Skylight 性能面板](https://oss.skylight.io/app/applications/s6PEZSKQcklL/recent/6h/endpoints)公开问题请求与调用栈，以便贡献者定位慢查询和高负载接口。

性能改进是当前最明确的贡献方向之一；潜在工作包括优化数据库查询、减少重复计算、改进缓存策略，以及降低 [[可观测性]] 面板所暴露的高延迟请求。

## 许可与商标

- Sure 与原 Maybe Finance 代码均采用 [[AGPLv3]] 许可证；使用、再分发或部署前应审查并履行相应义务。
- 派生仓库必须保留原始 AGPLv3 许可证，并明确声明其基于 Maybe Finance，但与 Maybe Finance Inc. 不存在关联或背书关系。
- Maybe 是 Maybe Finance Inc. 的注册商标，派生仓库不得继续使用 Maybe 名称或相关 Logo。
- Sure 是该社区分支所使用的项目名称。

## 适用性判断

**适合**：希望自行托管财务数据、具备 Ruby 与数据库运维能力，或希望通过 API、桌面端、移动端和 LLM 代理使用统一财务系统的个人及开发者。

**主要风险**：原项目的商业化已经受挫，当前又面临 [[性能优化]]、社区持续性、基础设施维护和 AI 服务成本等挑战。若需求是成熟的商业托管、稳定 SLA 或专业顾问服务，现有材料并未提供相应保证。

## 来源

- [Sure GitHub 仓库](https://github.com/we-promise/sure)
- [Sure 客户端说明](https://github.com/we-promise/sure/blob/main/docs/clients.md)
- [Docker 自托管指南](https://github.com/we-promise/sure/blob/main/docs/hosting/docker.md)

## 相关笔记
- [[Kronos：面向金融K线序列的首个开源基础模型]]
- [[Buzz 人类与 AI 智能体协作工作区]]
- [[Wealthfolio：本地优先的个人财务追踪开源工具]]
- [[Claude Trading Skills：个人投资者 AI 交易工作流]]
- [[Chrome DevTools MCP：AI 代理的浏览器调试工具]]
- [[Cloudflare 多阶段安全审计 Skill]]
- [[Codex Security 漏洞扫描与修复工作流]]
- [[LLM 驱动的多市场股票分析工作台]]
- [[Modern Web Guidance 编码代理知识库]]
- [[OpenBot：自托管 AI 同事平台]]