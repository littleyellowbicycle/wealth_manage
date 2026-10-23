---
title: OpenShell 自主 AI 智能体安全运行时
source: raw_20261023_233106_90e2b0e8
source_url: https://github.com/NVIDIA/OpenShell
created: '2026-10-23'
updated: '2026-10-23'
tags:
- 开源项目
- AI
- 风险控制
- 沙箱隔离
- 形式化验证
status: processed
note_type: content
related:
- Cloudflare 多阶段安全审计 Skill
- Claude 知识工作插件体系
- Claude-Red 渗透测试技能库概览
- Kronos：面向金融K线序列的首个开源基础模型
- VoxCPM2 多语言 TTS 与语音克隆
- 紫微斗数开源排盘引擎与解读数据集
- AI 供应链卡脖子环节投资框架
- AI 时代个人量化交易可行路径与核心铁律
- AI供应链卡脖子环节挖掘框架
- AI时代个人量化生存指南
---

# OpenShell 自主 AI 智能体安全运行时

## 核心结论
> OpenShell 是 NVIDIA 开源的自主 AI 智能体安全运行时，通过访问策略同时提供文件、软件包、API 与凭据使用能力，并限制其越界访问。它依靠沙箱内的内核级强制、端点限定凭据和策略变更的形式化验证，把运行时隔离与变更审批结合起来。该项目适合需要真实工具能力，但不能让智能体无限制访问主机、数据、密钥或网络的团队。

## 详细内容
## 项目概览

[[OpenShell]] 是面向自主 AI 智能体集群的安全、私有运行时。它允许智能体读取文件、安装软件包、调用 API 并使用凭据，同时依据 [[访问策略]] 限制其可访问的数据、资源和网络目标，目标是让智能体具备实际工作能力而不获得宿主机的不受控权限。

- 原始来源：<https://github.com/NVIDIA/OpenShell>
- 许可证：Apache License 2.0
- 支持环境：Linux、macOS Apple Silicon，以及实验性的 Windows WSL 2
- 基础依赖：Docker、Podman 或宿主机虚拟化
- 0.1.x 版本线重点引入稳定的发布节奏、新的隔离机制、扩展接口与 API

## 核心架构

系统由 [[OpenShell 网关]]、[[监督器]] 和 [[沙箱]] 等组件协同组成。网关是管理沙箱、策略与访问权限的控制面，本地安装后会同时配置命令行工具和本地网关。

[[OpenShell 网关]] 负责连接应用与运行环境，[[SDK]] 则用于让 Python、TypeScript、Go 和 Rust 应用接入网关；SDK 本身不负责安装命令行工具，SDK 与网关应尽量使用相同的 OpenShell 版本。

## 安全机制

### 内核级运行时隔离

每个智能体都运行在独立的 [[沙箱隔离]] 环境中。内核控制负责限制智能体可以访问的文件和调用的系统调用；每次网络连接在离开沙箱前都必须通过策略检查，从执行层阻止未经授权的横向访问。

### 端点限定凭据

智能体不会直接看到真实凭据。[[OpenShell 网关]] 只会在请求目标属于已批准端点时注入相应凭据，从而将凭据的使用范围限制在特定服务或 API，避免一次泄漏演变为全局权限泄漏。

### 形式化验证与人工审核

在策略变更正式生效前，[[形式化验证]] 会检查新策略可能授予的访问权限，并标记高风险变化，例如允许智能体携带凭据访问新主机，或调用新的 API 方法。这些变更需要等待人工审核，形成“策略编写、自动检查、人工批准”的变更流程。

## 策略体系

[[访问策略]] 覆盖以下控制维度：

- 文件系统规则：控制可读取和写入的文件范围。
- 网络规则：控制允许连接的主机、端点与网络行为。
- 进程规则：约束智能体可以执行的系统调用和进程操作。
- 凭据规则：限定凭据可以注入的请求及其目标端点。

策略配套的 [[策略顾问]] 与 [[策略证明器]] 用于审阅变更，帮助团队识别新增访问范围是否符合预期。

## 快速开始

安装 CLI 和本地网关：

```bash
curl -LsSf https://raw.githubusercontent.com/NVIDIA/OpenShell/main/install.sh | sh
openshell sandbox create --name demo
```

默认沙箱镜像采用最小化 Ubuntu，且不预装智能体。官方入门流程会运行 OpenCode，并连接免费的 OpenRouter 模型，演示如何在智能体需要新权限时审核并批准对应策略。

## 扩展与 SDK

OpenShell 支持通过中间件、拦截器和计算驱动扩展运行环境，并提供公开的智能体技能包，帮助编码智能体操作 CLI、编写沙箱策略及排查网关与推理路由问题：

```bash
npx skills add NVIDIA/OpenShell
```

各语言 SDK 的安装方式如下：

| 语言 | 安装命令 |
|---|---|
| Python | `uv add openshell` |
| TypeScript | `npm install @nvidia/openshell-sdk` |
| Go | `go get github.com/NVIDIA/OpenShell/sdk/go@latest` |
| Rust | `cargo add openshell-sdk --git https://github.com/NVIDIA/OpenShell --tag <release-tag>` |

## Kubernetes 部署

OpenShell 网关可以通过 [[Helm]] 部署到 [[Kubernetes]]。集群使用的 CNI 必须能够执行 [[NetworkPolicy]]，否则 Kubernetes 层面的网络策略无法提供预期约束。

## 遥测与隐私

按项目说明，OpenShell 默认仅收集匿名的运行类别与计数，用于改进项目，不收集沙箱名称、主机名、文件路径、提示词、凭据、服务商或模型名称以及用户内容。

可通过网关环境变量关闭遥测：

```bash
OPENSHELL_TELEMETRY_ENABLED=false
```

Helm 部署可设置 `server.telemetryEnabled=false`；项目也支持在编译时完全移除遥测代码。

## 适用场景与落地检查

OpenShell 特别适合需要读取业务文件、调用外部 API 或使用服务凭据，同时又要求权限隔离、变更审计和人工审批的 [[AI Agent]] 团队。落地时应重点确认：

1. 操作系统、容器运行时与虚拟化环境是否处于官方支持范围。
2. 集群 CNI 是否真正支持并执行网络策略。
3. SDK、网关和扩展组件是否使用兼容版本。
4. 文件、进程、网络和凭据策略是否覆盖智能体的完整工作路径。
5. 高风险策略变更是否保留人工审核，并按需关闭遥测。

## 许可与责任边界

项目采用 Apache License 2.0。OpenShell 可能自动获取或访问外部材料，这些材料不随软件一并分发，使用者需要自行审查并遵守相关条款、许可和安全要求。软件按“AS IS”方式提供，不附带明示或默示担保，实际使用风险由使用者承担。

## 相关笔记
- [[Cloudflare 多阶段安全审计 Skill]]
- [[Claude 知识工作插件体系]]
- [[Claude-Red 渗透测试技能库概览]]
- [[Kronos：面向金融K线序列的首个开源基础模型]]
- [[VoxCPM2 多语言 TTS 与语音克隆]]
- [[紫微斗数开源排盘引擎与解读数据集]]
- [[AI 供应链卡脖子环节投资框架]]
- [[AI 时代个人量化交易可行路径与核心铁律]]
- [[AI供应链卡脖子环节挖掘框架]]
- [[AI时代个人量化生存指南]]