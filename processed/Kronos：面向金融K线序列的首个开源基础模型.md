---
title: Kronos：面向金融K线序列的首个开源基础模型
source: raw_20261020_014255_c4757d03
source_url: https://github.com/shiyu-coder/Kronos
created: '2026-10-20'
updated: '2026-10-22'
tags:
- AI
- 交易
- 开源项目
- 回测
- 技术分析
- 金融
status: processed
note_type: content
related:
- 基于 bt 与 alphalens 的多因子回测框架实践
- K线圆弧角度分析的核心观点
- K线圆弧角度理论 金融市场的半圆数学
- Qbot 开源量化交易平台与经典技术指标策略实现
- Supabase：基于 PostgreSQL 的开源应用开发平台
- 分钟级K线涨跌跟随策略的回测：高频交易是负和博弈
- 2万元交易员付费课性价比存疑 B站免费资源更优
- Anthropic 知识工作插件体系
- BOLL短线两种高胜率策略与仓位管理
- Claude Code 渐进式学习与自动化指南
---

# Kronos：面向金融K线序列的首个开源基础模型

## 核心结论
> Kronos 是首个专为金融 K 线（OHLCV）序列设计的开源基础模型，采用「专用 tokenizer + 自回归 Transformer」两阶段框架，在 45+ 全球交易所数据上预训练，可作为统一模型服务多种量化任务。该工作已被 AAAI 2026 接收，并配套提供从预测、微调到回测的完整 A 股示例流程。

## 详细内容
## 模型概述

[[Kronos]] 是首个面向金融市场的开源 [[基础模型]]，专注于将 K 线（即 candlesticks）序列视为一种"语言"进行建模。与通用时序基础模型不同，它专门针对金融数据高噪声、强随机性的特性设计。训练数据覆盖全球 45+ 交易所的 K 线序列，论文已发布于 arXiv（2508.02739），并被 [[AAAI 2026]] 接收。

## 两阶段框架

Kronos 的核心是一个 decoder-only 自回归 Transformer，配合两阶段流水线：
1. **专用 Tokenizer**：将连续多维 OHLCV 数据量化为层级离散 token，压缩信息并降低噪声敏感度。
2. **自回归预训练**：在大规模 K 线 token 序列上预训练 Transformer，使其能作为统一模型处理多种下游任务（预测、生成、回测信号等）。

## 模型家族

项目在 Hugging Face 上以 NeoQuasar 命名空间开源了多档模型：

| 模型 | 参数量 | 上下文长度 | 是否开源 |
|---|---|---|---|
| Kronos-mini | 4.1M | 2048 | ✅ |
| Kronos-small | 24.7M | 512 | ✅ |
| Kronos-base | 102.3M | 512 | ✅ |
| Kronos-large | 499.2M | 512 | ❌ |

small 与 base 的 max_context 为 512，超出会被 KronosPredictor 自动截断。

## 快速预测流程

通过 KronosPredictor 类，几行代码即可完成从原始 K 线到未来预测的全流程，核心步骤：
1. 从 Hugging Face 加载对应的 KronosTokenizer 与 Kronos 模型。
2. 实例化 KronosPredictor(model, tokenizer, max_context=512)，指定设备。
3. 准备输入：DataFrame 需含 ['open','high','low','close']（volume/amount 可选），加上 x_timestamp 与 y_timestamp。
4. 调用 predict(df, x_timestamp, y_timestamp, pred_len, T, top_p, sample_count)，可通过温度与 [[nucleus sampling]] 控制采样。

支持批量版本 predict_batch，对多个资产并行推理，所有串行需保持相同 lookback 与 pred_len。

## 微调与回测示例

项目提供了基于 [[Qlib]] 的 A 股端到端微调 pipeline，流程分四步：
1. **配置**：finetune/config.py 中设置 qlib_data_path、dataset_path、save_path 等路径与超参。
2. **数据准备**：运行 qlib_data_preprocess.py 生成 train/val/test.pkl。
3. **微调**：分两阶段——先用 torchrun 微调 tokenizer，再微调 predictor；均支持多卡。
4. **回测**：qlib_test.py 输出预测信号并跑简单 top-K 策略对比基准。

需注意：示例中的"原始信号"还需经过组合优化与风险因子中性化（market beta、风格因子等）才能提取 [[纯Alpha]]，且示例未建模交易成本、滑点等损耗，不构成生产级 [[量化交易]] 系统。

## 局限与生产化建议

- **信号到组合**：模型输出仅是 raw signals，生产环境需接入组合优化、风险中性化、动态仓位、止损/止盈等模块。
- **数据适配**：QlibDataset 仅是示例，外部数据源需自行改造加载与预处理逻辑。
- **回测保真度**：高保真回测必须精细建模交易成本、滑点与市场冲击，示例中的简单 top-K 仅作演示。

## 引用

```
@misc{shi2025kronos,
  title={Kronos: A Foundation Model for the Language of Financial Markets},
  author={Yu Shi and Zongliang Fu and Shuo Chen and Bohan Zhao and Wei Xu and Changshui Zhang and Jian Li},
  year={2025},
  eprint={2508.02739},
  archivePrefix={arXiv},
  primaryClass={q-fin.ST}
}
```

License：MIT。

## 相关笔记
- [[基于 bt 与 alphalens 的多因子回测框架实践]]
- [[K线圆弧角度分析的核心观点]]
- [[K线圆弧角度理论 金融市场的半圆数学]]
- [[Qbot 开源量化交易平台与经典技术指标策略实现]]
- [[Supabase：基于 PostgreSQL 的开源应用开发平台]]
- [[分钟级K线涨跌跟随策略的回测：高频交易是负和博弈]]
- [[2万元交易员付费课性价比存疑 B站免费资源更优]]
- [[Anthropic 知识工作插件体系]]
- [[BOLL短线两种高胜率策略与仓位管理]]
- [[Claude Code 渐进式学习与自动化指南]]