---
title: SkillSpector：AI Agent 技能安全扫描器
source: raw_20261026_233111_d019ff65
source_url: https://github.com/NVIDIA/SkillSpector
created: '2026-10-26'
updated: '2026-10-26'
tags:
- AI
- 开源项目
- 风险控制
- 供应链安全
status: processed
note_type: content
related:
- OpenShell 自主 AI 智能体安全运行时
- Cloudflare 多阶段安全审计 Skill
- Agent Reach：AI Agent 多平台只读访问层
- Cindy 开源 AI Agent 客户端
- Hallmark：反 AI 模板化网页设计技能
- ECC：AI 编程代理工程优化系统
- Huashu Design 3.0：Agent 原生设计交付工作流
- Impeccable AI 前端设计工作流
- OpenBot：本地优先的持久 AI 队友工作台
- Karpathy 启发的 Claude Code 编码准则
---

# SkillSpector：AI Agent 技能安全扫描器

## 核心结论
> SkillSpector 是 NVIDIA 开源的 [[AI Agent 技能]] 安全扫描器，用于在安装前识别提示词注入、数据外泄、越权、恶意代码与供应链风险。它以快速 [[静态分析]] 配合可选的 [[大语言模型]] 语义分析，输出 0–100 风险分、处置建议及 JSON、Markdown、SARIF 报告，适合接入 [[持续集成]]。README 所述 31,132 个技能样本中有 26.1% 含漏洞、5.2% 呈现疑似恶意意图，表明技能生态不能依赖隐式信任，应将扫描、[[安全基线]]、人工复核和 [[MCP]] 安装门禁组合使用。

## 详细内容
## 项目定位

SkillSpector 面向 [[Claude Code]]、[[Codex CLI]]、[[Gemini CLI]] 等使用的 [[AI Agent 技能]]，解决技能在隐式信任环境中安装、运行且缺乏充分审查的问题。它既可作为命令行安全工具，也可作为 [[MCP]] 服务供智能体在安装技能时调用。

该工具是 [[NVIDIA Verified Skills]] 流水线的扫描与评估组件：通过检查、评分和签名的技能才会进入 NVIDIA 技能目录。其目标是在技能进入运行环境前回答：这个技能是否安全，以及风险是否可接受。

## 风险依据

README 引用的研究数据子集包含 31,132 个技能，其中 26.1% 被检测出漏洞，5.2% 呈现疑似恶意意图。该数据说明技能供应链中存在显著风险，但比例仅适用于对应研究样本，不应直接外推到整个技能生态。

## 核心能力

| 能力 | 说明 |
|---|---|
| 多格式输入 | 支持 Git 仓库、URL、ZIP、目录和单文件 |
| 规则检测 | 提供 71 种漏洞模式，覆盖 17 个风险类别 |
| 两阶段分析 | 快速 [[静态分析]]，以及可选的 [[大语言模型]] 语义分析 |
| 依赖情报 | 通过 OSV.dev 查询实时 CVE，并在离线时回退 |
| 风险评估 | 输出 0–100 风险分、严重级别和处置建议 |
| 报告输出 | 支持终端、JSON、Markdown 和 SARIF |
| 误报治理 | 通过指纹或 Glob 规则建立 [[安全基线]]，仅突出新增问题 |
| 批量扫描 | 并行扫描技能目录，并支持中文、日文、韩文检测 |

## 检测范围

- [[提示词注入]]：识别指令覆盖、隐藏指令、行为操纵、有害内容及利用空白区域藏匿指令。
- [[反拒绝与越狱]]：发现禁止拒绝、删除免责声明和绕过安全策略等模式。
- [[数据外泄]]：检测向外部地址传输数据、收集环境变量、枚举敏感文件及泄露对话上下文。
- [[权限提升与凭证访问]]：覆盖过度权限、sudo/root 执行以及 SSH 密钥、口令和令牌读取。
- [[供应链安全]]：检查依赖未锁定、远程脚本执行、混淆代码、已知 CVE、废弃依赖、仿冒包和依赖源重定向。
- [[过度自治]]：发现无约束工具访问、缺少人类审批的高影响决策、范围蔓延及无界资源消耗。
- [[输出处理与系统提示词泄露]]：识别未经校验的模型输出、跨信任边界输出以及直接或间接泄露系统提示词。
- [[记忆投毒]]：检测持久化恶意指令、上下文窗口填充和智能体存储状态篡改。
- 代码与工具风险：覆盖工具滥用、失控智能体、触发滥用、[[AST 分析]]、[[污点分析]]、[[YARA 规则]]、[[MCP 最小权限]] 和 MCP 工具投毒。

## 安装与扫描

项目要求 Python 3.12 及以上，采用 Apache 2.0 许可证，也提供基于 Python 3.12 slim 镜像的 Docker 构建方式。

```bash
uv tool install git+https://github.com/NVIDIA/skillspector.git

# 仅运行静态分析
skillspector scan ./my-skill/ --no-llm

# 配置模型提供商后运行完整扫描
skillspector scan ./my-skill/

# 生成适合 CI/CD 的 SARIF 报告
skillspector scan ./my-skill/ --format sarif --output report.sarif
```

语义分析支持 OpenAI、Anthropic、AWS Bedrock、Gemini、NVIDIA Build、Ollama 以及其他 OpenAI 兼容端点，可通过 `SKILLSPECTOR_PROVIDER`、模型名称和相应凭证变量配置。

## 推荐工作流

1. 先以 `--no-llm` 完成快速静态检查，快速发现确定性风险。
2. 对高风险或来源不明的技能启用 [[大语言模型]] 语义分析，检查静态规则难以识别的隐蔽意图。
3. 根据风险分、严重级别和具体发现进行人工复核，而不是只依据 SAFE 或 UNSAFE 标签。
4. 对已确认的误报建立 [[安全基线]]，使后续扫描突出新增问题。
5. 在 [[持续集成]] 中执行扫描，并使用 SARIF 将结果送入代码审查或安全平台。
6. 在技能安装入口增加门禁，高风险或扫描不完整时阻止发布与安装。

## MCP 运行时门禁

安装 MCP 扩展后，可以把扫描能力直接暴露给本地智能体：

```bash
uv tool install --force 'skillspector[mcp] @ git+https://github.com/NVIDIA/skillspector.git'
skillspector mcp
claude mcp add skillspector -- skillspector mcp
```

`scan_skill` 工具会返回风险分、严重级别、处置建议、是否可安装、具体发现，以及 `llm_used` 和 `scan_mode`。这些字段能够区分静态扫描与完整扫描，避免把低风险分误认为经过语义分析后的安全结论。

HTTP 模式没有内置身份认证，因此只允许绑定 `127.0.0.1` 或 `::1`，并自动拒绝本地路径和 `file://` URL。远程部署应在监听器前增加带认证的反向代理，例如 nginx 与 mTLS；私有仓库则应使用 CLI 或 stdio 模式。

## 资源与扫描范围控制

- 远程内容、ZIP 解压总量和 Git 克隆后磁盘占用均受 100 MiB 的 `INGEST_MAX_BYTES` 限制。
- 单个 ZIP 最多包含 10,000 个成员，超限后以 `IngestLimitExceededError` 失败关闭。
- 单个待分析文件另有 1 MiB 的读取上限，与内容导入阶段的资源限制相互独立。
- 显式排除文件会使覆盖率变成部分覆盖，并阻止系统给出 SAFE 建议；在 CI 中可启用 `--fail-on-incomplete`。
- [[MCP 注册表快照对比]] 应使用相同选择范围；当前输入中缺失的服务只能判定为本次快照中移除，不能证明其已从上游注册表删除。
- 注册表对比不会抑制发现或改变风险评分，也不会访问或执行服务端点。

## 使用边界

- 静态扫描的低风险分不等于完整扫描通过，应同时检查报告中的扫描模式。
- [[安全基线]] 只管理已接受发现，不会修复技能代码；精确指纹在源码或 SkillSpector 版本变化后会重新生效，需要再次审查。
- 语义分析依赖模型能力和提供商实现，结构化输出失败或模型差异可能影响结果，因此不能替代静态规则与人工判断。
- README 提醒 stdio MCP 传输仍存在初始化挂起问题，追踪于 [Issue #199](https://github.com/NVIDIA/SkillSpector/issues/199)。
- 扫描结果应结合来源信誉、权限需求、数据访问范围和运行环境综合决策，而不应被当作绝对安全证明。

## 项目信息

- 源代码：[NVIDIA/SkillSpector](https://github.com/NVIDIA/SkillSpector)
- 原始材料记录的 GitHub 热度为 21,853 stars，该数字属于动态快照。
- 适用场景：安装前审查、[[持续集成]] 安全门禁、技能目录批量审计、[[MCP]] 运行时拦截以及 NVIDIA Verified Skills 发布流水线。

## 相关笔记
- [[OpenShell 自主 AI 智能体安全运行时]]
- [[Cloudflare 多阶段安全审计 Skill]]
- [[Agent Reach：AI Agent 多平台只读访问层]]
- [[Cindy 开源 AI Agent 客户端]]
- [[Hallmark：反 AI 模板化网页设计技能]]
- [[ECC：AI 编程代理工程优化系统]]
- [[Huashu Design 3.0：Agent 原生设计交付工作流]]
- [[Impeccable AI 前端设计工作流]]
- [[OpenBot：本地优先的持久 AI 队友工作台]]
- [[Karpathy 启发的 Claude Code 编码准则]]