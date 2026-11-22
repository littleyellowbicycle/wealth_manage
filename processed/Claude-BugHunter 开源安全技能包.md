---
title: Claude-BugHunter 开源安全技能包
source: raw_20261117_233107_6110015a
source_url: https://github.com/elementalsouls/Claude-BugHunter
created: '2026-11-17'
updated: '2026-11-22'
tags:
- AI
- 开源项目
- 风险控制
- 网络安全
- 漏洞赏金
- 红队测试
status: processed
note_type: content
related:
- Cloudflare 多阶段安全审计 Skill
- SkillSpector：AI Agent 技能安全扫描器
- Claude Trading Skills：个人投资者 AI 交易工作流
- OpenShell 自主 AI 智能体安全运行时
- ECC：AI 编程代理工程系统
- OpenBot：自托管 AI 同事平台
- 移动应用商店合规预提交检查
- 闲鱼 AI 智能监控系统
- AFFiNE 本地优先开源知识库与协作画布
- Diagram Design：AI 编辑型图表生成技能
---

# Claude-BugHunter 开源安全技能包

## 核心结论
> Claude-BugHunter 将漏洞赏金、外部红队与安全报告流程封装为 83 个可按主题自动加载的 [[Claude Code]] 技能，覆盖 Web 应用、企业身份平台、云基础设施、侦察与报告交付。其核心价值不只是提供检测知识，还通过范围控制、验证门禁、证据卫生和标准化工作流降低漏报与误报。使用时必须限定于获得授权的资产；该工具集不覆盖内部域渗透、后渗透、横向移动、C2 与规避技术，不能替代完整的红队平台。

## 详细内容
## 项目概览

[Claude-BugHunter](https://github.com/elementalsouls/Claude-BugHunter) 是由 Sachin Sharma 开发的开源安全技能包，面向 [[漏洞赏金]]、[[Web 应用安全测试]] 和 [[外部红队]]。项目采用 MIT 与 CC BY 4.0 双许可证：源代码使用 MIT，技能方法、词表、正则规则、严重性评级与文档使用 CC BY 4.0。

该项目被收录于 GitHub Trending，快照为 8,487 Stars。README 当前标注 83 个技能、15 个斜杠命令、681 个来自公开披露报告的检测模式，其中 433 个已单独引用并可审计，覆盖 24 类核心漏洞。原始 Trending 描述仍写作 82 个技能，说明摘要数据可能未同步更新。

## 核心能力

整个技能包分为四个相互衔接的层次：

1. **思考与纪律**：通过 `bb-methodology` 和 `redteam-mindset` 建立假设驱动、非线性的测试方法，并强调范围意识与操作纪律。
2. **Web 应用狩猎**：58 个 `hunt-*` 技能覆盖 XSS、SQL 注入、SSRF、IDOR、文件上传、GraphQL、OAuth、JWT、CORS 等常见漏洞类别，并提供检测逻辑、载荷参考、绕过检查表和组合链模板。
3. **企业边界测试**：覆盖 [[Microsoft 365]]、[[Microsoft Entra ID]]、[[Okta]]、[[VMware vCenter]]、SSL VPN、SharePoint、云 IAM 与 Android 应用等外部暴露面，纳入了 2024—2026 年企业平台漏洞链和凭据后 escalation 模型。
4. **验证与交付**：通过“7 问门禁”、基于 [[VRT]] 的严重性判断、范围外结论反驳、个人信息脱敏和证据留存规范，将测试结果转化为 [[漏洞报告]] 或客户交付物。

## 工作机制

技能以标准 [[Agent Skills]] 格式编写，用户用自然语言描述测试目标后，系统会按主题自动选择相关技能，无需逐个记诵命令名称。确定性参与引擎还会根据目标攻击面进行路由，把发现项交给对应技能继续处理。

README 将流程描述为非线性的“侦察 → 映射与排序 → 测试 → 验证 → 报告”，并同时声称其为六阶段流程，但正文仅列出五个命名阶段；具体阶段划分应以 `docs/architecture.md` 为准。

项目支持两种驱动方式：

- **自然语言模式**：适合临时探索和专题验证，由系统自动加载技能。
- **`/hunt` 与 `cbh` 模式**：创建参与目录、目标范围、状态和证据结构，并通过命令行协调侦察、分类、验证与报告。

## 安装与兼容性

推荐通过 Claude Code 插件市场安装，技能与命令会保存在 `claude-bugunter:` 命名空间下，并随插件版本更新：

`/plugin marketplace add elementalsouls/Claude-BugHunter`

`/plugin install claude-bugHunter@elementalsouls`

复制安装方式适合不支持插件系统或需要固定版本的环境，可在克隆仓库后运行 `bash scripts/install.sh` 或 `pwsh ./scripts/install.ps1`。两种方式的能力并不完全相同：插件方式支持技能和 15 个命令，`cbh` CLI 可通过 `pipx` 单独安装，但 `hunt` 项目脚手架仅随仓库副本提供。

知识层采用开放的 `SKILL.md` 格式，可安装到 [[Claude Code]]、[[OpenCode]]、[[OpenAI Codex CLI]]、[[Hermes Agent]] 和 [[Google AntiGravity]]。其中斜杠命令与 `/hunt` 引擎仍以 Claude Code 为唯一运行目标。安装脚本还支持配置 [[Burp Suite]] 的 [[MCP]] 集成。

## 适用范围

该项目聚焦互联网可访问的外部攻击面，主要适用于：

- Web、API、SaaS、GraphQL、OAuth 与云应用的漏洞测试；
- 面向 M365、Entra ID、Okta、SharePoint、vCenter 和 SSL VPN 等企业平台的外部评估；
- 公共云存储、实例元数据、STS AssumeRole 与跨账户权限问题；
- 子域名枚举、证书透明度、JavaScript 分析、密钥扫描与 [[OSINT]]；
- HackerOne、Bugcrowd、Intigriti、Immunefi 以及企业红队报告交付。

## 明确排除的范围

以下内容属于有意设置的能力边界，而非尚未实现的普通功能：

- 内部 Active Directory 攻击，包括 Kerberos、票据伪造、凭据转储与 DCSync；
- 后渗透、持久化、横向移动和 C2 框架；
- 恶意软件开发、0-day 武器化与大规模目标基础设施；
- AMSI、ETW、AV 或 EDR 规避；
- iOS、硬件、射频、ICS，以及二进制漏洞、内核和浏览器内部机制研究。

外部边界失陷后的内部活动需要交由 BloodHound、Impacket、NetExec、Rubeus 等专用工具处理。

## 授权与安全约束

技能只应用于自有资产、获得书面授权的渗透测试、赏金计划范围内目标、CTF 或公开训练环境。内置验证门禁会确认目标是否在授权范围内、影响是否被项目接受，以及测试是否具备可修复性；证据卫生模块则用于避免在截图和报告中泄露 Cookie、令牌或受害者个人信息。

Anthropic 的运行时网络安全策略可能拒绝部分漏洞利用或进攻性工具请求。合法的授权安全工作应按官方要求申请 [[Cyber Verification Program]]，而不是通过改写描述来规避安全分类器；批量数据窃取和勒索软件开发仍属于不可调整的禁止用途。

## 适用场景与局限

该项目适合希望把临时提示词转化为可重复安全流程的个人研究者、赏金猎人和红队团队。其突出特点是知识库较完整、范围规则内置、验证与报告环节受到重视，并能复用现成的 [[OWASP]] 映射、公开披露报告和 [[ProjectDiscovery]] 工具链。

实际效果仍取决于目标规则是否准确录入、工具链是否配置完成、CVE 数据是否及时刷新，以及专业人员对结果的复核。项目 README 声称这些技能经过授权参与和公开训练平台验证，但仓库指标本身不能替代对输出结论的独立验证；涉及企业环境的自动化结果应由合格安全人员审核后再采取行动。

## 发展路线

后续计划包括 HackerOne MCP 集成、跨项目参与记忆、项目规则自动解析器、行业专用技能、企业平台扩展和季度漏洞矩阵更新。当前已发布金融 GraphQL 专用技能，而医疗 FHIR、政府合规、部分商业 VPN 与 AD CS 能力仍在路线图中。

## 相关笔记
- [[Cloudflare 多阶段安全审计 Skill]]
- [[SkillSpector：AI Agent 技能安全扫描器]]
- [[Claude Trading Skills：个人投资者 AI 交易工作流]]
- [[OpenShell 自主 AI 智能体安全运行时]]
- [[ECC：AI 编程代理工程系统]]
- [[OpenBot：自托管 AI 同事平台]]
- [[移动应用商店合规预提交检查]]
- [[闲鱼 AI 智能监控系统]]
- [[AFFiNE 本地优先开源知识库与协作画布]]
- [[Diagram Design：AI 编辑型图表生成技能]]