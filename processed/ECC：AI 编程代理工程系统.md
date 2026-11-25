---
title: ECC：AI 编程代理工程系统
source: raw_20261117_233130_40b45467
source_url: https://github.com/affaan-m/ECC
created: '2026-11-17'
updated: '2026-11-25'
tags:
- 开源项目
- AI
- 风险控制
- 上下文工程
status: processed
note_type: content
related:
- 闲鱼 AI 智能监控系统
- Claude Trading Skills：个人投资者 AI 交易工作流
- OpenBot：自托管 AI 同事平台
- OpenShell 自主 AI 智能体安全运行时
- SkillSpector：AI Agent 技能安全扫描器
- Claude-BugHunter 开源安全技能包
- Cloudflare 多阶段安全审计 Skill
- 移动应用商店合规预提交检查
- Firecrawl 面向 AI 代理的知识基础设施
- AutoClip：本地优先的 AI 视频切片工具
---

# ECC：AI 编程代理工程系统

## 核心结论
> ECC 将 AI 编程代理从一次性代码生成器升级为遵循“计划—测试—实现—评审—验证—记忆—改进”的工程系统。其核心资产是 72 个专业代理、327 项技能、Hooks、规则、记忆与 AgentShield，并以 MIT 开源方式覆盖 Claude Code、Codex 及多种适配器。实际采用应坚持每个 Harness 只选一种安装路径、仅使用官方来源并检查能力边界，避免重复配置与第三方镜像风险。

## 详细内容
## 项目定位

[[ECC]] 是一个面向 [[AI 编程代理]] 的开源工程化系统，目标不是单纯增强模型，而是为代理提供统一的计划、测试、评审、验证、记忆与持续改进机制。其设计原则是“优化上下文窗口，其余内容持久化”：将高价值规则保留在上下文中，把技能、配置、经验和安全策略沉淀为可复用层。

原始来源为 [GitHub 仓库](https://github.com/affaan-m/ECC)。Trending 抓取快照记录了 305,142 个 Star，并将当日趋势语言标为 JavaScript；README 同时列出 Shell、TypeScript、Python、Go、Java、Perl 等技术栈，因此它并非单一语言工具，而是一套跨代理平台的工程基础设施。

## 核心工作流

ECC 将代理行为组织为以下闭环：

`计划 → 测试 → 实现 → 评审 → 验证 → 记忆 → 改进`

- **计划**：先澄清需求、研究现状并形成可执行方案，避免直接进入编码。
- **测试与实现**：围绕验收条件推进 [[测试驱动开发]]，将通用流程沉淀为 [[技能]]。
- **评审与验证**：从新的上下文重新检查产物，通过测试、构建或人工确认降低自我审查偏差。
- **记忆与改进**：利用 [[持续学习]]、会话总结和本能规则保存高价值经验，再将重复成功转化为可复用技能与工作流。

## 能力构成

| 组件 | 数量或形态 | 主要作用 |
| --- | --- | --- |
| [[专业代理]] | 72 个 | 覆盖规划、评审、构建修复、安全、架构和垂直领域任务 |
| [[技能库]] | 327 项 | 覆盖 TDD、研究、安全、文档、前端、数据、机器学习与运维等工作 |
| [[命令兼容层]] | 98 个 | 提供旧命令入口，逐步向技能优先的交互方式迁移 |
| [[Hooks 运行时]] | 运行时能力 | 执行策略约束、会话总结、持续学习、本能规则与上下文控制 |
| [[规则引擎]] | 按需选择 | 按语言或项目加载始终生效的工程规范 |
| [[AgentShield]] | 内置安全组件 | 扫描提示词、Hooks、[[MCP 配置]]、权限、密钥和代理文件 |

具体平台可用的组件子集可能不同，不能仅依据仓库总览数字假设所有 Harness 都具备完整功能。

## 平台支持

- [[Claude Code]] 是当前体验最完整的平台，可使用原生 `ecc@ecc` 插件。
- [[Codex]] 支持原生仓库市场插件，旧版文件同步流程仅作为兼容方案保留。
- Kimi Code 提供引导式安装，Qoder 提供原生插件路径。
- [[Cursor]]、[[OpenCode]]、[[Gemini CLI]]、[[Zed]]、[[GitHub Copilot]]、[[Antigravity]] 等通过能力受限的托管适配器接入。
- 适配器主要用于迁移工作流与技能，不能假设其具备与 Claude Code 相同的 Hooks、阻塞式工具拦截或原生技能发现能力。

## 安装与配置

推荐使用 ECC 2.2.3 的通用引导安装：

```bash
npx ecc-universal@2.2.3 setup
```

需要同时配置 Claude Code、Codex 与 Kimi Code 时，可运行多 Harness 向导：

```bash
npx ecc-universal@2.2.3 install --guided
```

通用安装要求 Node.js 18 或更高版本；Claude Code 插件安装还要求 Git 和 Claude Code 2.1 或更高版本位于 `PATH` 中。安装前可使用 `--dry-run` 检查目标目录和文件变更计划，安装后使用 `doctor` 与平台插件列表命令验证状态。

关键约束如下：

1. **每个 Harness 只选择一种安装方式**。可以同时安装到多个 Harness，但不能在同一 Harness 上叠加原生插件、完整手动安装和旧同步流程，否则可能产生重复技能、命令、Hooks 或配置。
2. Claude Code 插件不能分发 `rules`，应仅按项目需要添加 `rules/common` 和实际使用的语言规则包。
3. 低上下文环境可使用 `minimal` 配置；任何可能启用 Hooks 运行时的配置都应明确选择 `--enable-hooks` 或 `--no-hooks`。
4. `ecc-install` 是 `ecc-universal` 内部二进制名称，不是独立发布的 npm 包，不应直接通过 `npx ecc-install` 安装。

## 安全与供应链治理

项目明确要求仅从官方渠道获取 ECC：GitHub 仓库、npm 上的 `ecc-universal` 与 `ecc-agentshield`、官方 GitHub App、插件标识 `ecc@ecc` 以及项目官网。第三方转载和未审查镜像不受项目维护，可能包含恶意代码。

版本锁定只能提高可复现性，并不等于安全审计；执行包代码前仍应检查发布来源、包管理器完整性和实际文件变更。ECC 不会自动上传诊断信息，提交问题前应由用户主动检查并提供必要信息。

## 采用建议

- 首次接入时先选择一个主力 Harness 和一个项目，避免一次覆盖全部平台。
- 优先启用计划、测试、评审与验证流程，再按实际需求逐步增加 [[持续学习]]、自动 Hooks 和安全扫描。
- 先以项目级规则验证团队接受度，再决定是否迁移为全局规范。
- 升级前运行诊断和干运行，卸载或重置时使用官方清理流程，避免误删用户自行维护的文件。

## 开源与商业边界

仓库采用 MIT 许可证，核心开源版本可持续免费使用；ECC Pro 则是面向私有仓库的托管 GitHub App。赞助者和 Pro 订阅用于支持项目维护，官方称其由单一维护者按周跨多个 Harness 交付。

## 相关笔记
- [[闲鱼 AI 智能监控系统]]
- [[Claude Trading Skills：个人投资者 AI 交易工作流]]
- [[OpenBot：自托管 AI 同事平台]]
- [[OpenShell 自主 AI 智能体安全运行时]]
- [[SkillSpector：AI Agent 技能安全扫描器]]
- [[Claude-BugHunter 开源安全技能包]]
- [[Cloudflare 多阶段安全审计 Skill]]
- [[移动应用商店合规预提交检查]]
- [[Firecrawl 面向 AI 代理的知识基础设施]]
- [[AutoClip：本地优先的 AI 视频切片工具]]