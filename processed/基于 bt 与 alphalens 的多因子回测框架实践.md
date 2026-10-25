---
title: 基于 bt 与 alphalens 的多因子回测框架实践
source: raw_20260624_162539_368f685f
source_url: https://www.zhihu.com/question/329206671/answer/3402461940
created: '2026-10-17'
updated: '2026-10-25'
tags:
- 交易
- 回测
- 选股
- A股
- Alpha因子
- 技术分析
status: processed
note_type: content
related:
- 均线低吸交易体系核心逻辑
- MACD与布林线实战：短线中线技术形态选股框架
- 筹码分布战法与盘中关键时点观察框架
- 轻量级 Alpha 多因子回测框架实践与量化系统本质思考
- BOLL短线两种高胜率策略与仓位管理
- Kronos：面向金融K线序列的首个开源基础模型
- 涨停板十种形态操盘逻辑与留走策略
- 涨停板操盘密码：10种形态的主力意图与留走策略
- 纳指ETF策略选择方法论：数据驱动的回测决策框架
- A股均线高阶用法实战框架
---

# 基于 bt 与 alphalens 的多因子回测框架实践

## 核心结论
> 作者基于 alphalens、bt、alphas 等开源项目搭建了 [[alpha因子]] 批量回测框架，以 baostock/akshare 为数据源，用 bt 替代原始 backtrader 以实现多因子并行回测。从程序员视角看，量化回测本质是遍历 K 线数据的 for 循环加策略回调，实盘系统只是把数据获取换成实时并对接交易接口；因此短期只需批量回测筛选有效因子，不必使用 wondertrader 这类高性能复杂平台。

## 详细内容
## 工具与数据源

框架整合三个核心仓库：
- [[alphalens]]：Quantopian 出品的 [[alpha因子]] 通用分析工具
- [[bt框架]]：基于 Python 的轻量回测框架，比 [[backtrader]] 更简洁，原生支持多策略并行回测
- [[alphas]]：alpha101 与 alpha191 因子的简单实现

数据源使用：
- [[baostock]]：证券宝，免费开源，无需注册，提供行情与上市公司财务数据
- [[akshare]]：开源金融数据接口，底层抓取东方财富网

主体逻辑沿用 alphas，但将回测框架替换为 bt，并增加了批量处理与界面操作。

## 工作流

1. 运行 datas.py 下载股票数据（暂不纠结前/后复权，原始数据准确度本身是个大问题）
2. 运行 alphas101.py 或 alphas191.py 计算因子
3. 在 1.research.ipynb 分析因子（需对 alphalens 做少量适配）
4. 在 2.1.bt.ipynb 用 bt 替代 backtrader 进行回测（前者基准添加更稳定、批量更友好）
5. 可选：在 factorycompare.ipynb 计算历年因子收益的百分比排名

## bt 回测代码结构

### 1. 获取股票数据

通过 `get_hs300_stocks` 拉取年初 [[沪深300]] 成分股，再用 `get_all_date_data` 取一年期行情，pivot 成宽表后把价格 shift(-1) 模拟次日成交价，缺失值用前值填充。

### 2. 获取基准数据

以 sh000300 作为基准，回测区间与股票数据对齐，同样 pivot 处理。

### 3. 获取因子数据

读取已计算好的 `alpha{:03d}.csv`，按年度切片；通过 `melt` 将宽表转长表（[[alphalens]] 要求的 `date / asset / factor` 格式），并设置二级索引。

### 3.1 批量获取因子信号

循环 101 个 alpha（剔除 48、56、58、59、63、67、69、70、76、79、80、82、87、89、90、91、93、97、100 等无效项），逐个调用 `get_clean_factor_and_forward_returns` 取第 10 分位作为买入信号，存入 `obj_dict` 字典供后续批量回测使用。

### 4. 构造调仓信号表（关键步骤）

`get_clean_factor_and_forward_returns` 配合 `quantiles=10, periods=[1]` 产出因子分位与前瞻收益，筛 `factor_quantile == 10` 的 (date, asset)，标记 close=True，再 pivot 成 `trade_date × sec_code` 的布尔矩阵，缺失用 False 填充。

### 5. bt 自定义策略类

```python
class SelectTrade(bt.Algo):
    def __init__(self, signal, alpha: str):
        self.signal = signal
        self.alpha = alpha

    def __call__(self, target):
        if target.now in self.signal.index:
            sig = self.signal.loc[target.now]
            target.temp['selected'] = list(sig.index[sig])
        else:
            target.temp['selected'] = []
        return True
```

继承 `bt.Algo`，在调仓日从布尔信号表中取出 True 标的，交给后续 `WeighEqually` + `Rebalance`。

### 6. 执行回测

构造两条策略：因子策略（`SelectTrade + WeighEqually + Rebalance`）与基准策略（`RunOnce + SelectAll + WeighEqually + Rebalance`），分别生成 Backtest 后用 `bt.run(t, t2)` 同时运行，`res.plot()` 与 `res.display()` 输出图表与统计表。

### 6.1 全因子批量回测

for 循环把所有 alpha 打包成 `bt_list`，星号解包一次性传入 `bt.run(*bt_list)`。bt 之所以被选用，关键就在于这种多策略并行能力——执行效率很高，适合一次性跑遍所有因子做横向筛选。

## 对量化交易系统的理解

作者从聚宽（云平台回调式策略）到 [[backtrader]]（源码嵌套深、数据结构复杂、实盘接口对接 [[A股]] 困难），再到接触 [[wondertrader]]（C++ 高性能事件驱动平台）后才对量化系统形成清晰认知。

从程序员架构视角：
- 量化回测 ≈ 遍历 K 线数据的 for 循环 + 每轮回调策略函数
- 量化交易 ≈ 把 for 循环的输入换成实时行情，并把策略内的信号换成下单调用

更轻量的方案甚至无需框架：数据库 + 定时任务（基于昨日 K 线计算买卖信号写入结果表）+ miniqmt 等交易程序扫描数据库执行——本质上与互联网公司晨会 review 数据报表后决策投放无异（高频场景除外）。

因此个人短期目标只需：批量回测因子 → 发现有效因子 → 模拟交易 → 实盘。不需要 wondertrader 这种重平台，理解"for 循环 + 回调"这一抽象比选哪个框架更重要。

## 相关笔记
- [[均线低吸交易体系核心逻辑]]
- [[MACD与布林线实战：短线中线技术形态选股框架]]
- [[筹码分布战法与盘中关键时点观察框架]]
- [[轻量级 Alpha 多因子回测框架实践与量化系统本质思考]]
- [[BOLL短线两种高胜率策略与仓位管理]]
- [[Kronos：面向金融K线序列的首个开源基础模型]]
- [[涨停板十种形态操盘逻辑与留走策略]]
- [[涨停板操盘密码：10种形态的主力意图与留走策略]]
- [[纳指ETF策略选择方法论：数据驱动的回测决策框架]]
- [[A股均线高阶用法实战框架]]