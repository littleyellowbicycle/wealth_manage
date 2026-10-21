---
title: Cloudflare 多阶段安全审计 Skill
source: raw_20261020_233106_054929c5
source_url: https://github.com/cloudflare/security-audit-skill
created: '2026-10-20'
updated: '2026-10-21'
tags:
- 开源项目
- AI
- 风险控制
- 网络安全
- 代码审计
- 漏洞挖掘
status: processed
note_type: content
related:
- AI Job Search 求职自动化工作流
- ai-memory：跨编码代理的长期记忆层
- freehire：开源 IT 职位搜索与申请平台
- Graphify：本地优先的代码知识图谱
- Headroom：AI Agent 上下文压缩层
- Kronos：面向金融K线序列的首个开源基础模型
- Orca 多智能体开发编排工具
- Ponytail：AI 编程代理的最小实现规则集
- Ruflo：Claude Code 与 Codex 的智能体元编排框架
- Supabase 架构与能力
---

# Cloudflare 多阶段安全审计 Skill

## 核心结论
> Cloudflare 的 security-audit-skill 将编码代理编排成一套六阶段、覆盖驱动且可重复的安全审计流程，把侦察、漏洞候选验证、结构化记录和独立复核串成机器可验证的证据链。其核心方法是用隔离代理、覆盖率账本和“发现者与验证者分离”降低误报；只有具备完整源码追踪和有界影响的事实才能标为 confirmed，未决或被证伪的线索分别保留为 needs_validation 和 rejected。

## 详细内容
## 核心结论
Cloudflare 的 security-audit-skill 将编码代理编排成一套六阶段、覆盖驱动且可重复的安全审计流程，把侦察、漏洞候选验证、结构化记录和独立复核串成机器可验证的证据链。其核心方法是用隔离代理、覆盖率账本和“发现者与验证者分离”降低误报；只有具备完整源码追踪和有界影响的事实才能标为 confirmed，未决或被证伪的线索分别保留为 needs_validation 和 rejected。

## 项目定位
该项目是一个面向编码代理的开源安全审计 Skill，源自 Cloudflare 漏洞发现测试框架的演进版本。它不直接生成带有攻击载荷的报告，而是围绕架构理解、系统化搜寻、对抗式验证和可追溯记录组织审计过程。

项目采用 MIT 许可证，原始来源为 https://github.com/cloudflare/security-audit-skill；材料记录的 GitHub Stars 为 33,438。

## 六阶段审计流程

1. **[[侦察阶段]]**：分析目标系统的架构、信任边界、输入面、既有证据和可重复的覆盖范围，产出 `architecture.md` 与 `coverage-ledger.json`。
2. **[[覆盖驱动的漏洞搜寻]]**：按照覆盖账本中的工作单元分配相互隔离的 hunter，分别记录检查结果，再由覆盖批评代理寻找遗漏区域。
3. **[[候选漏洞验证]]**：每个唯一候选漏洞都交给新的验证代理，验证者以证伪为目标，而不是简单重复发现者的判断。
4. **[[结构化安全输出]]**：将结果写入 `findings.json`，并依据 `report-schema.json` 校验 `confirmed`、`needs_validation` 和 `rejected` 三种状态。
5. **[[独立记录复核]]**：由新的代理重新检查最终记录中的源码主张；如果记录被替换，替换内容还要再次接受独立验证。
6. **[[目标无关报告]]**：只依据已复核的记录和覆盖账本生成 `REPORT.md`、`FINDINGS-DETAIL.md` 与 `NEEDS-VALIDATION.md`，避免报告依赖具体目标或代理实现。

主代理会在创建或更新覆盖账本后运行 `validate-coverage-ledger.cjs`，并在第四阶段以及每次第五阶段记录替换后运行 `validate-findings.cjs`。

## 判定标准与证据链

- `confirmed`：必须具备完整的源码追踪和有界的实际观察结果。
- `needs_validation`：必须明确指出尚未解决的具体事实，并且不预先赋予严重性。
- `rejected`：表示候选漏洞已经被证伪，但仍保留被否决的记录。

这种设计将“发现线索”“确认事实”和“报告输出”分开，减少单个代理的判断偏差。[[严重性评级]]只依据实际影响，采用可能性与影响的组合，而不是因为某项检查清单未完成就提高等级。

## 持续审计与增量覆盖

针对同一代码库的多次运行可以叠加结果。后续运行会读取既有覆盖账本和漏洞记录，优先填补缺口、重新验证发生变化的源码，并继承仍然适用于当前版本的证据；过期或未解决的线索不会被误认为已经覆盖。

项目测试中，单次运行发现的漏洞约为重复运行累计发现量的一半，因此多轮执行和覆盖反馈是重要能力，而不是简单的重复扫描。

## 安全执行要求

该 Skill 适合代码审计、漏洞发现和代码渗透测试等场景，尤其适用于支持工具调用与并行子代理的编码代理。运行环境还需要 Node.js，以执行零依赖的账本和发现校验器。

目标代码、构建、测试、浏览器、模拟器、模糊测试器和 fixture 都必须运行在操作系统强制隔离的沙箱中，并满足以下条件：

- 禁用外部网络访问；
- 使用经过清理的白名单环境变量；
- 限制 CPU、内存、进程和其他资源；
- 只允许写入指定的临时工作目录。

如果缺少这些控制，工作流不会执行目标代码，而是让相关线索保持 `needs_validation` 状态。

## 方法原则

- **只确认已成立的边界失败**：源码支持但仍缺少关键事实的线索不能直接升级为漏洞。
- **[[对抗式验证]]**：发现漏洞的代理不能成为最终验证者。
- **影响优先**：防御纵深缺口或单纯偏离最佳实践，通常属于加固建议而非漏洞。
- **[[多轮覆盖]]**：重复运行应利用历史证据扩大覆盖面，而不是从头开始。

## 审计范围

仓库提供按目标类型拆分的攻击面提示，覆盖 [[内存安全与二进制安全]]、[[AI 与 LLM 安全]]、[[Web 与身份协议安全]]、[[客户端安全]]、[[供应链与发布安全]]、[[云与部署安全]]、[[协议、RPC 与消息安全]]、[[资源耗尽与可用性]]、[[数据隔离与生命周期安全]]以及 [[桌面、移动与本地 IPC 安全]]等领域。

## 使用方式

可通过 Skills CLI 安装：`npx skills add https://github.com/cloudflare/security-audit-skill --skill security-audit`；添加 `--global` 可安装到用户级环境。

在目标代码库中启动编码代理后，可使用 `security audit this codebase`、`find security vulnerabilities in ./src` 或指定输出目录的安全审计请求。未指定目录时，完整审计默认写入 `~/security-audit-skill/<repo-name>/run-<N>`；只有明确选择且被版本控制忽略时，工作流才会在目标仓库内写入文件。

## 相关笔记
- [[AI Job Search 求职自动化工作流]]
- [[ai-memory：跨编码代理的长期记忆层]]
- [[freehire：开源 IT 职位搜索与申请平台]]
- [[Graphify：本地优先的代码知识图谱]]
- [[Headroom：AI Agent 上下文压缩层]]
- [[Kronos：面向金融K线序列的首个开源基础模型]]
- [[Orca 多智能体开发编排工具]]
- [[Ponytail：AI 编程代理的最小实现规则集]]
- [[Ruflo：Claude Code 与 Codex 的智能体元编排框架]]
- [[Supabase 架构与能力]]