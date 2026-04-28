# 量化交易策略回测系统 - 项目结构

## 1. 项目概览
- **名称**: 量化交易策略回测系统
- **技术栈**: Python + FastAPI + Vue 3 + Element Plus + ECharts
- **数据源**: akshare（A股行情数据）

## 2. 目录结构（精简版）
.
├── AGENTS.md
├── README.md
├── PROJECT_STRUCTURE.md
├── quant_backend
│   ├── README.md
│   ├── app
│   │   ├── api/            # API路由层
│   │   │   ├── endpoints.py         # 核心API（回测、数据、指标、搜索、策略）
│   │   │   └── optimizer_api.py     # 策略参数优化API
│   │   ├── core/           # 核心配置
│   │   │   └── config.py            # 应用设置（端口、SSL、CORS等）
│   │   ├── main.py         # FastAPI应用入口（启动、中间件、路由注册）
│   │   ├── models/         # Pydantic数据模型
│   │   │   ├── BacktestRequest.py       # 回测请求参数（策略、权重、风控等12项）
│   │   │   ├── BacktestResponse.py      # 回测结果（统计+交易记录+图表数据）
│   │   │   ├── ComprehensiveChartsData.py
│   │   │   ├── OptimizationRequest.py
│   │   │   ├── OptimizationResult.py
│   │   │   ├── Statistics.py
│   │   │   ├── StockData.py
│   │   │   ├── StockDataResponse.py
│   │   │   ├── TechnicalIndicator.py
│   │   │   ├── TechnicalIndicatorsResponse.py
│   │   │   └── TradeRecord.py
│   │   ├── services/       # 业务逻辑层
│   │   │   ├── backtest_service.py     # 策略信号生成 + 回测引擎
│   │   │   ├── data_service.py         # 股票数据加载（akshare）
│   │   │   ├── indicator_service.py    # 技术指标计算（MA/MACD/RSI/布林/量/动量）
│   │   │   ├── optimizer_service.py    # 优化器核心（遗传/网格/随机/贝叶斯）
│   │   │   └── strategy_optimizer.py   # 优化器封装（参数重要性、样本外测试）
│   │   └── utils/          # 工具函数
│   │       ├── helpers.py             # JSON编码器、股票代码前缀处理、格式化
│   │       └── logger.py              # 日志配置
│   ├── requirements.txt
│   ├── run.py             # uvicorn启动入口
│   └── ssl/               # SSL证书目录
├── quant_frontend
│   ├── README.md
│   ├── index.html
│   ├── package.json
│   ├── vite.config.js
│   ├── public/
│   └── src
│       ├── App.vue              # 根组件（Header + RouterView）
│       ├── main.js
│       ├── router/index.js      # 路由配置（单页Home）
│       ├── services/api.js      # Axios API封装（7个接口）
│       ├── utils/
│       │   ├── helpers.js       # 格式化工具（百分比、货币、颜色）
│       │   └── tradeProfit.js   # 交易配对与盈亏计算
│       ├── views/Home.vue       # 主页面（表单 + 结果展示布局）
│       └── components/
│           ├── BacktestForm.vue       # 策略参数配置表单（股票搜索、指标权重、风控）
│           ├── ResultsDisplay.vue     # 回测结果展示（统计卡片 + 图表 + 交易记录）
│           ├── ComprehensiveCharts.vue # 综合图表（价格/MACD/RSI/净值/回撤/收益分布）
│           ├── OptimizerPanel.vue     # 策略参数优化面板（4种优化方法、进度轮询）
│           ├── RevenueCharts.vue      # 独立图表组件（净值/收益分布/月度收益）
│           └── TradeTable.vue         # 交易记录表（含配对盈亏列）
└── .claude/                # Claude Code配置

## 3. Use Case 总览

### 3.1 已实现的 Use Case

| 编号 | 用例名称 | 描述 | 涉及端点/组件 |
|------|---------|------|-------------|
| UC-01 | 股票代码搜索 | 根据代码或名称模糊搜索A股，支持最近搜索历史 | `GET /api/v1/stocks/search` → `BacktestForm.vue` |
| UC-02 | 股票行情数据获取 | 获取指定时间段的OHLCV日线数据，支持前复权/后复权/不复权 | `GET /api/v1/stock-data` → `DataService` |
| UC-03 | 技术指标计算 | 计算双均线(MA)、MACD、RSI(含多时间框架)、布林带(含挤压检测)、成交量比率、动量指标 | `GET /api/v1/technical-indicators` → `IndicatorService` |
| UC-04 | 单股策略回测 | 选择策略类型，设置12项参数，运行回测并返回完整结果（统计+图表数据+交易记录） | `POST /api/v1/backtest` → `BacktestService` |
| UC-05 | 高胜率策略 | 基于6项技术指标加权打分，含买入/卖出意愿阈值、布林挤压过滤、波动率过滤、价格偏离过滤 | `BacktestService.generate_high_win_rate_signal()` |
| UC-06 | 趋势跟踪策略 | 基于趋势强度(MACD/RSI/布林中线)的趋势跟踪买卖信号 | `BacktestService._generate_trend_following_signal()` |
| UC-07 | 均值回归策略 | 在RSI超卖+布林下轨处买入，RSI超买+布林上轨处卖出，含持仓3日反转止损 | `BacktestService._generate_mean_reversion_signal()` |
| UC-08 | 回测引擎 | 逐日模拟交易，支持初始资金/手续费/止损止盈，生成净值曲线和基准对比 | `BacktestService.run_backtest()` |
| UC-09 | 回测统计报告 | 计算总收益、基准收益、年化收益、最大回撤、交易次数、胜率、平均盈亏、盈亏比、夏普比率、最终资产 | `BacktestService.calculate_statistics()` |
| UC-10 | 价格走势可视化 | ECharts渲染：收盘价+双均线+布林带+买卖信号标记，含缩放和数据zoom | `ComprehensiveCharts.vue` (price tab) |
| UC-11 | 成交量可视化 | 红涨绿跌柱状图+成交量均线 | `ComprehensiveCharts.vue` (volume chart) |
| UC-12 | MACD指标可视化 | DIF/DEA/柱状图三合一展示 | `ComprehensiveCharts.vue` (technical tab) |
| UC-13 | RSI指标可视化 | RSI曲线+超买(70)/超卖(30)/中线(50)标记 | `ComprehensiveCharts.vue` (technical tab) |
| UC-14 | 净值曲线对比 | 策略净值 vs 买入持有基准，标注买卖点 | `ComprehensiveCharts.vue` (performance tab) |
| UC-15 | 回撤曲线分析 | 动态回撤面积图，展示策略风险特征 | `ComprehensiveCharts.vue` (performance tab) |
| UC-16 | 日收益率分布 | 直方图+均值线，展示收益分布形态 | `ComprehensiveCharts.vue` (performance tab) |
| UC-17 | 交易记录配对展示 | 买入-卖出自动配对，计算每笔盈亏百分比 | `TradeTable.vue` + `tradeProfit.js` |
| UC-18 | 策略参数优化 | 支持遗传算法/随机搜索/网格搜索/贝叶斯优化，后台异步执行，前端轮询进度 | `POST /api/v1/optimizer/start` + `GET /api/v1/optimizer/status/{id}` |
| UC-19 | 优化历史可视化 | 优化过程中的分数收敛曲线 | `OptimizerPanel.vue` history chart |
| UC-20 | 参数重要性分析 | 基于优化历史计算各参数与分数的相关系数，排序展示 | `StrategyOptimizer.get_parameter_importance()` |
| UC-21 | 样本外测试 | 将数据按7:3分割为训练集/测试集，在测试集上验证优化参数 | `StrategyOptimizer.test_optimized_strategy()` |
| UC-22 | 策略列表查询 | 返回系统支持的3种策略的id/名称/描述 | `GET /api/v1/strategies` |
| UC-23 | 系统健康检查 | 服务运行状态检测 | `GET /health`、`GET /api/v1/health` |

### 3.2 建议补充的 Use Case

以下用例基于当前架构自然延伸，按优先级分为 P0/P1/P2。

#### P0 — 核心能力增强（短期）

| 编号 | 用例名称 | 描述 | 涉及改动 |
|------|---------|------|---------|
| UC-24 | 多股组合回测 | 同时回测多只股票，按等权/市值加权/风险平价分配资金，生成组合净值曲线和持仓明细 | 新增 `POST /api/v1/backtest/portfolio`，前端组合配置面板 |
| UC-25 | 回测结果持久化 | 回测结果存入数据库(SQLite/PostgreSQL)，支持历史记录列表、按股票/策略/时间筛选 | 新增DB模型 + `GET/POST/DELETE /api/v1/backtest-results` |
| UC-26 | 回测结果对比 | 选择2-5条历史回测结果，并列展示关键指标对比表和净值曲线叠加图 | 前端对比面板 + 后端对比数据聚合 |
| UC-27 | 基准指数选择 | 回测时可选基准（沪深300/中证500/中证1000/创业板指），自动获取基准数据并计算相对收益 | 修改 `BacktestRequest` 加 `benchmark` 字段，`DataService` 加指数加载 |
| UC-28 | 数据导出 | 将回测统计、交易记录、净值序列导出为CSV/Excel | 后端加 `/backtest/{id}/export` + 前端下载按钮 |
| UC-29 | 参数模板管理 | 将当前表单参数保存为命名模板，支持加载、更新、删除，减少重复配置 | 前端 `localStorage` 或后端DB存储模板JSON |

#### P1 — 分析深度扩展（中期）

| 编号 | 用例名称 | 描述 | 涉及改动 |
|------|---------|------|---------|
| UC-30 | 选股扫描器 | 按技术条件（金叉/RSI超卖/布林突破/量比>N）全市场扫描，返回符合条件的股票列表 | 新增 `POST /api/v1/screener`，`ScreenerService` |
| UC-31 | 风险分析报告 | 计算VaR(历史模拟法/参数法)、CVaR、最大回撤持续期、Calmar比率、索提诺比率 | 扩展 `calculate_statistics()` 或新增 `RiskService` |
| UC-32 | 蒙特卡洛模拟 | 基于历史收益率分布随机抽样N次，生成净值路径分布和置信区间 | 新增 `MonteCarloService`，前端概率分布图 |
| UC-33 | 参数敏感性分析 | 对每个策略参数在±20%范围内扰动，生成"参数-绩效"曲线，直观展示参数影响 | 新增 `POST /api/v1/sensitivity`，前端龙卷风图 |
| UC-34 | 多周期分析 | 同时展示日线/周线/月线级别的回测结果，比较不同周期的策略表现 | `DataService` 加周期转换，前端多标签切换 |
| UC-35 | 资金管理模型 | 支持固定比例/凯利公式/风险预算/Van Tharp等多种仓位管理方式 | 修改 `run_backtest()` 的仓位计算逻辑 |
| UC-36 | 策略自定义构建器 | 拖拽式组合技术指标条件（如"MACD金叉 AND RSI<30"），无需编码定义策略 | 新增 `POST /api/v1/strategies/custom`，前端可视化条件构建器 |

#### P2 — 平台化能力（长期）

| 编号 | 用例名称 | 描述 | 涉及改动 |
|------|---------|------|---------|
| UC-37 | 用户账户系统 | 注册/登录/个人信息，每个用户独立保存回测记录、参数模板、自定义策略 | 新增用户模型 + JWT认证 + 前端登录页 |
| UC-38 | 回测报告生成 | 一键生成PDF/HTML格式的完整回测报告（含统计、图表截图、交易明细） | 后端报告渲染（WeasyPrint/Jinja2）+ 前端下载 |
| UC-39 | 实时信号推送 | WebSocket连接，定时获取最新行情并检测策略信号，触发浏览器通知 | `main.py` 加WebSocket端点 + 前端通知组件 |
| UC-40 | 定时自动回测 | 配置定时任务（如每日收盘后），自动运行回测并发送报告通知 | 新增 `SchedulerService`（APScheduler） |
| UC-41 | 港股/美股扩展 | 扩展akshare数据源到港股、美股，前端股票搜索国际化 | `DataService` 加 `market` 参数，搜索加市场过滤 |
| UC-42 | 行业/板块分析 | 按申万行业或概念板块聚合股票，计算板块平均指标和轮动信号 | 新增行业分类数据 + `SectorService` |
| UC-43 | 因子归因分析 | 使用Fama-French三因子/五因子模型对策略收益进行归因，分解Alpha和Beta暴露 | 新增 `FactorService`，前端因子暴露柱状图 |
| UC-44 | 策略排行榜 | 所有用户公开策略按夏普/年化收益/Calmar排行，展示Top策略参数 | 新增 `GET /api/v1/leaderboard` |
| UC-45 | 移动端适配 | 响应式布局，适配手机/平板屏幕，重点优化图表触控交互 | 前端CSS媒体查询 + ECharts touch配置 |
