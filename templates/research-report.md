# 投研报告模板

> Agent 使用说明：用 WebSearch 搜索最新新闻，用 WebFetch 调 Yahoo Finance / Financial Modeling Prep / CoinGecko 拉取数据，结合 my-soul.md 中的分析框架偏好，生成完整投研报告，保存到 output/research/。

---

## 报告基本信息

- **标的**：{{TICKER}} — {{COMPANY_NAME}}
- **报告类型**：☐ 个股分析  ☐ 宏观专题  ☐ 加密资产  ☐ 行业研究
- **评级**：{{RATING}}（买入 / 持有 / 减仓 / 观察）
- **目标价/位**：{{TARGET_PRICE}}
- **报告日期**：{{REPORT_DATE}}
- **置信度**：{{CONFIDENCE}}（高 / 中 / 低）

---

## 执行摘要（Executive Summary）

> 200 字以内，让读者 1 分钟内理解核心论点和结论

**核心结论**：{{CORE_THESIS}}

**关键催化剂**：{{KEY_CATALYSTS}}

**主要风险**：{{KEY_RISKS}}

**建议行动**：{{RECOMMENDED_ACTION}}

---

## 一、基本面分析

### 1.1 业务概览

| 维度 | 内容 |
|------|------|
| 主营业务 | {{BUSINESS_OVERVIEW}} |
| 商业模式 | {{BUSINESS_MODEL}} |
| 竞争优势 | {{MOAT}} |
| 市场地位 | {{MARKET_POSITION}} |

### 1.2 财务数据

| 指标 | 最近季度 | 同比 | 预期 | 实际 vs 预期 |
|------|---------|------|------|------------|
| 营收（Revenue） | {{REV}} | {{REV_YOY}} | {{REV_EST}} | {{REV_BEAT}} |
| EPS | {{EPS}} | {{EPS_YOY}} | {{EPS_EST}} | {{EPS_BEAT}} |
| 毛利率 | {{GROSS_MARGIN}} | {{GM_YOY}} | — | — |
| 自由现金流 | {{FCF}} | {{FCF_YOY}} | — | — |

### 1.3 估值

| 指标 | 当前值 | 行业均值 | 历史均值（5年） | 判断 |
|------|--------|---------|--------------|------|
| PE（TTM） | {{PE}} | {{SECTOR_PE}} | {{HIST_PE}} | {{PE_JUDGE}} |
| PS | {{PS}} | {{SECTOR_PS}} | {{HIST_PS}} | {{PS_JUDGE}} |
| EV/EBITDA | {{EV_EBITDA}} | — | — | — |
| PEG | {{PEG}} | — | — | — |

### 1.4 管理层与治理

- CEO/创始人背景：{{MGMT_BACKGROUND}}
- 近期重要表态：{{MGMT_STATEMENT}}
- 内部人持股/增减持：{{INSIDER_ACTIVITY}}

---

## 二、技术面分析

| 指标 | 当前值 | 信号 |
|------|--------|------|
| 当前价格 | {{PRICE}} | — |
| RSI(14) | {{RSI}} | {{RSI_SIGNAL}} |
| MACD | {{MACD}} | {{MACD_SIGNAL}} |
| 50 日均线 | {{MA50}} | 价格{{MA50_POS}} |
| 200 日均线 | {{MA200}} | 价格{{MA200_POS}} |
| 52 周高低 | {{52W_HIGH}} / {{52W_LOW}} | 当前位于{{52W_POS}} |

**关键支撑位**：{{SUPPORT_LEVELS}}
**关键阻力位**：{{RESISTANCE_LEVELS}}
**技术面小结**：{{TECH_SUMMARY}}

---

## 三、宏观与行业背景

- **行业趋势**：{{INDUSTRY_TREND}}
- **宏观影响**：{{MACRO_IMPACT}}（利率 / 汇率 / 政策）
- **竞争格局**：{{COMPETITIVE_LANDSCAPE}}
- **主要竞争对手比较**：

| 公司 | 市值 | 营收增速 | PE | 竞争优势 |
|------|------|---------|-----|---------|
| {{TICKER}} | {{MKTCAP}} | {{REV_GROWTH}} | {{PE}} | {{MOAT}} |
| {{COMP1}} | {{COMP1_CAP}} | {{COMP1_GROWTH}} | {{COMP1_PE}} | {{COMP1_MOAT}} |
| {{COMP2}} | {{COMP2_CAP}} | {{COMP2_GROWTH}} | {{COMP2_PE}} | {{COMP2_MOAT}} |

---

## 四、催化剂与风险

### 催化剂（看多理由）

| 催化剂 | 预期时间 | 概率 | 影响程度 |
|--------|---------|------|---------|
| {{CAT1}} | {{TIME1}} | {{PROB1}} | {{IMPACT1}} |
| {{CAT2}} | {{TIME2}} | {{PROB2}} | {{IMPACT2}} |

### 风险（看空理由）

| 风险 | 概率 | 影响程度 | 对冲方式 |
|------|------|---------|---------|
| {{RISK1}} | {{RPROB1}} | {{RIMPACT1}} | {{HEDGE1}} |
| {{RISK2}} | {{RPROB2}} | {{RIMPACT2}} | {{HEDGE2}} |

---

## 五、投资结论

**评级**：{{FINAL_RATING}}

**操作建议**：
- 若 [条件 A]：[行动 X]
- 若 [条件 B]：[行动 Y]
- 止损位：{{STOP_LOSS}}
- 分批建仓计划：{{POSITION_PLAN}}

**复盘节点**：下次财报（{{NEXT_EARNINGS}}）/ {{RECHECK_DATE}} 复盘本报告

---

*数据截止日期：{{DATA_DATE}} | 来源：{{SOURCES}}*
*⚠️ 本报告仅为个人投研记录，不构成投资建议。投资有风险，决策需谨慎。*