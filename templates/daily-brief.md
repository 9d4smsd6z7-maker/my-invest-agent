# 财经日报 — {{DATE}}

> Agent 使用说明：用 WebSearch 搜索隔夜市场新闻，用 WebFetch 调 CoinGecko / Yahoo Finance / Alternative.me API 拉取数据，读取 my-soul.md 中的持仓列表，按以下模板生成完整日报，保存到 output/daily/YYYY-MM-DD.md。

---

## 隔夜美股

| 指数 | 收盘价 | 涨跌幅 |
|------|--------|--------|
| 道琼斯（DJIA） | {{DJIA}} | {{DJIA_CHG}} |
| 标普 500（SPX） | {{SPX}} | {{SPX_CHG}} |
| 纳斯达克（NASDAQ） | {{NASDAQ}} | {{NASDAQ_CHG}} |
| 恐惧贪婪指数（CNN FGI） | {{US_FGI}} | {{US_FGI_LABEL}} |

### 持仓标的表现
<!-- Agent 自动读取 my-soul.md 核心关注标的，逐一填写 -->

| 标的 | 最新价 | 涨跌幅 | 备注 |
|------|--------|--------|------|
| ... | ... | ... | ... |

### 盘后重要异动
- ...

---

## 加密市场

| 标的 | 最新价 | 24h涨跌 |
|------|--------|---------|
| BTC | ${{BTC_PRICE}} | {{BTC_CHG}} |
| ETH | ${{ETH_PRICE}} | {{ETH_CHG}} |

- **BTC ETF 净流量**：{{BTC_ETF_FLOW}}（数据来源：The Block）
- **加密恐惧贪婪指数**：{{CRYPTO_FGI}} / 100 — {{CRYPTO_FGI_LABEL}}

---

## 宏观要闻

### 今日重要经济数据
<!-- 如无重要数据，写"今日无重要经济数据发布" -->
- ...

### 政策动态（关税/监管/地缘）
- ...

---

## 信号面板

### S&P 500 抄底信号

| 指标 | 当前值 | 触发阈值 | 状态 |
|------|--------|---------|------|
| VIX | {{VIX}} | > 30 | {{VIX_STATUS}} |
| S&P 500 RSI(14) | {{SPX_RSI}} | < 30 | {{SPX_RSI_STATUS}} |
| 距 200 日均线 | {{SPX_200MA_PCT}} | < -5% | {{SPX_200MA_STATUS}} |
| Put/Call Ratio | {{PC_RATIO}} | > 1.2 | {{PC_STATUS}} |
| 恐惧贪婪指数 | {{US_FGI}} | < 20 | {{US_FGI_STATUS}} |

**信号结论**：{{SPX_SIGNAL}}（满足 {{SPX_TRIGGER_COUNT}}/5 项）

### BTC 抄底信号

| 指标 | 当前值 | 触发阈值 | 状态 |
|------|--------|---------|------|
| BTC RSI(14) | {{BTC_RSI}} | < 30 | {{BTC_RSI_STATUS}} |
| MVRV Z-Score | {{MVRV}} | < 0 | {{MVRV_STATUS}} |
| 加密恐惧贪婪指数 | {{CRYPTO_FGI}} | < 15 | {{CRYPTO_FGI_STATUS}} |
| 交易所 BTC 净流量 | {{BTC_EXCHANGE_FLOW}} | 连续7日净流出 | {{FLOW_STATUS}} |
| 资金费率 | {{FUNDING_RATE}} | 连续3日为负 | {{FUNDING_STATUS}} |

**信号结论**：{{BTC_SIGNAL}}（满足 {{BTC_TRIGGER_COUNT}}/5 项）

---

## 今日关注

> Agent 核心判断（1-2 句，直接、有观点）：
>
> {{CORE_INSIGHT}}

---

*数据来源：Yahoo Finance / CoinGecko / Alternative.me / CNN Business | 更新时间：{{UPDATE_TIME}} UTC+8*
*⚠️ 以上内容仅供参考，不构成投资建议。*