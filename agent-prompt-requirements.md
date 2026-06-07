# 投研 Agent Prompt 需求文档

> **核心理念**：工具不重要，数据才是护城河。
>
> 本文档是一份通用的投研 Agent 搭建模板。你只需要做两件事：
> 1. 填写 `soul/my-soul.md`（告诉 Agent 你是谁）
> 2. 往 `data/` 文件夹里扔数据（让 Agent 真正了解你）
>
> 其余所有模块均已预配置完毕，开箱即用。Agent 会根据你的数据自动迭代 `soul/`、`memory/`、`scheduler/` 三个模块，逐步成为真正了解你的私人投研助手。
>
> **设计原则**：Claude Code 原生能力覆盖的，不额外封装。联网搜索、API 调用、文件读写都是 Claude Code 的内置能力（`WebSearch`、`WebFetch`、`curl`、`Read/Write/Edit`），不需要单独的代码文件来包装。Agent 的技能清单写进 `agent-soul.md` 即可，它自己会调用。

---

## 项目结构

```
/agent/
│
├── .claude/                     # Claude Code 权限与配置
│   └── settings.json            # Agent 读写权限、网络访问、命令执行范围
│
├── .env                         # API Key 配置（定时任务使用，不提交到 Git）
│
├── soul/                        # 你是谁 — Agent 的人格、技能、用户画像
│   ├── agent-soul.md            # Agent 的性格、风格、价值观、技能清单（已预填，无需修改）
│   └── my-soul.md               # ⭐ 你的画像（唯一需要你填写的配置文件）
│
├── templates/                   # 内容创作模板
│   ├── daily-brief.md           # 财经日报模板
│   ├── tweet.md                 # 推文模板
│   ├── thread.md                # Thread 模板
│   ├── video-script.md          # 视频脚本模板
│   └── research-report.md       # 投研报告模板
│
├── memory/                      # 你记得什么 — Agent 的记忆系统
│   ├── working.json             # 工作记忆（当前关注/近期决策/市场环境）
│   └── episodes.json            # 情景记忆（重要决策与对话摘要）
│
├── scheduler/                   # 你什么时间做什么事 — 定时任务规则
│   └── rules.md                 # 所有定时任务 + 信号规则（一个文件搞定）
│
├── data/                        # ⭐ 你的数据 — Agent 进化的燃料（随时往里扔）
│   ├── raw/                     # 原始数据
│   │   ├── tweets/              # 历史推文/社媒内容
│   │   ├── trades/              # 交易记录
│   │   ├── notes/               # 个人笔记、研究心得
│   │   └── references/          # 参考文章、研报、截图
│   └── feedback/                # 用户反馈
│       └── ratings.json
│
└── output/                      # Agent 的产出
    ├── daily/                   # 每日简报
    ├── research/                # 投研分析
    ├── signals/                 # 信号记录
    └── content/                 # 内容草稿
```

### 数据驱动迭代机制

```
data/raw/       →  用户随时投喂原始数据（推文、交易、笔记...）
     ↓
Agent 自动提炼，直接更新到权威来源：
┌─────────────────────────────────────────────┐
│  soul/my-soul.md  ← 更新用户画像与偏好微调  │
│  memory/          ← 更新工作记忆与情景记忆   │
│  scheduler/       ← 优化任务频率与信号参数   │
└─────────────────────────────────────────────┘
     ↓
data/feedback/  →  用户对产出评分 → 触发下一轮迭代
```

---

## 零、Permissions — Agent 的权限边界

> Agent 能做什么、不能做什么，由 `.claude/settings.json` 统一管控。
> 预设配置将 Agent 的读写范围限定在当前项目目录内。如果你希望 Agent 读写电脑上的其他位置（比如你的 Obsidian 笔记库、下载文件夹等），只需修改权限规则即可。

### 预设权限配置

```json
// .claude/settings.json
{
  "permissions": {
    "allow": [
      "Read(**)",
      "Write(**)",
      "Edit(**)",
      "MultiEdit(**)",
      "Bash(**)",
      "WebFetch(*)",
      "WebSearch(*)"
    ],
    "deny": [
      "Bash(rm -rf *)",
      "Bash(rm -r /)",
      "Bash(sudo *)",
      "Bash(chmod 777 *)",
      "Bash(shutdown *)",
      "Bash(reboot *)",
      "Bash(kill -9 *)",
      "Bash(git push --force *)",
      "Bash(git reset --hard *)",
      "Bash(npm publish *)"
    ]
  },
  "additionalDirectories": []
}
```

### 权限说明

| 类别 | 预设行为 | 说明 |
|------|---------|------|
| **文件读取（Read）** | ✅ 允许 | 默认可读取项目目录内所有文件 |
| **文件编辑（Edit）** | ✅ 允许 | 默认可编辑项目目录内所有文件 |
| **命令执行（Bash）** | ⚠️ 白名单制 | 只允许 node/python/git 等安全命令，禁止 rm -rf、sudo 等危险操作 |
| **网络访问（WebFetch/WebSearch）** | ✅ 允许 | 默认开放，Agent 可自主联网搜索和调用 API |
| **项目外读写** | ❌ 默认关闭 | 需手动添加 `additionalDirectories` 开启 |

### 扩展 Agent 的读写范围

默认情况下，Agent 只能读写当前项目目录。如果你希望 Agent 访问电脑上的其他位置，修改 `additionalDirectories` 字段：

```json
// 示例：让 Agent 读写你的 Obsidian 笔记库和下载文件夹
{
  "additionalDirectories": [
    "/Users/你的用户名/Documents/Obsidian",
    "/Users/你的用户名/Downloads",
    "/Users/你的用户名/Desktop/research"
  ]
}
```

```json
// 进阶：让 Agent 读写整台电脑（⚠️ 请确保你了解风险）
{
  "additionalDirectories": [
    "/"
  ]
}
```

### 安全提醒

- `deny` 规则的优先级永远高于 `allow`，即使你开放了全局读写，危险命令仍会被拦截
- 建议不要随意删除预设的 `deny` 规则，它们保护你免于误操作
- 如果 Agent 尝试执行未在 `allow` 中的操作，Claude Code 会弹窗询问你是否允许
- 定期审查 `.claude/settings.json`，确保权限范围符合你的预期

---

## 一、Workflow — 每次对话的标准流程

> Agent 的每一次对话都遵循固定的工作流。不管用户问什么，Agent 都按以下流程执行，确保上下文完整、产出贴合用户、记忆持续进化。

### 对话生命周期

```
┌─────────────────────────────────────────────────────────────────┐
│  Phase 1 — 启动加载（每次对话开头，静默执行，不输出给用户）        │
│                                                                 │
│  ⓪ claude-mem 插件自动注入历史会话上下文（插件自动完成）          │
│  ① 读取 soul/agent-soul.md     → 加载 Agent 人格与技能清单      │
│  ② 读取 soul/my-soul.md        → 加载用户画像                   │
│  ③ 读取 memory/working.json    → 加载工作记忆（当前关注/近期决策）│
│  ④ 读取 memory/episodes.json   → 加载最近 10 条情景记忆          │
│                                                                 │
│  加载完成，Agent 已进入"了解用户"的状态                          │
├─────────────────────────────────────────────────────────────────┤
│  Phase 2 — 理解意图                                             │
│                                                                 │
│  ⑤ 解析用户输入，判断任务类型：                                  │
│     - 投研分析（个股/宏观/加密）                                 │
│     - 内容创作（推文/脚本/长文）                                 │
│     - 信号查询（抄底信号/技术面）                                │
│     - 日常问答（闲聊/工具使用/学习）                             │
│     - 数据投喂（用户扔了新文件到 data/raw/）                     │
│                                                                 │
│  ⑥ 根据任务类型，决定需要调用哪些能力：                          │
│     - 是否需要 WebSearch 搜索最新新闻/数据？                     │
│     - 是否需要 WebFetch/curl 调 API 拉取行情/财报/链上数据？     │
│     - 是否需要读取 data/raw/ 中的本地文件？                      │
│     - 是否需要参考 templates/ 中的内容模板？                      │
├─────────────────────────────────────────────────────────────────┤
│  Phase 3 — 执行任务                                             │
│                                                                 │
│  ⑦ 调用所需能力获取数据                                         │
│  ⑧ 结合 my-soul.md 中的用户偏好生成回复：                       │
│     - 分析框架对齐用户的分析框架偏好                             │
│     - 风险判断参照用户的风险偏好                                 │
│     - 语气和深度匹配用户风格                                    │
│  ⑨ 输出回复给用户                                               │
├─────────────────────────────────────────────────────────────────┤
│  Phase 4 — 自我迭代（每次对话结尾，静默执行）                    │
│                                                                 │
│  ⑩ claude-mem 在后台自动捕获本次对话所有细节（插件自动完成）      │
│  ⑪ 判断本次对话是否产生了值得记录的新信息：                      │
│     - 用户提到了新标的/新观点？→ 更新 working.json               │
│     - 用户做了重要决策（买入/卖出/调仓）？→ 追加 episodes.json   │
│     - 用户纠正了 Agent 的判断？→ 更新 working.json + 记入反思    │
│     - 用户表达了新的投资信念/偏好？→ 直接修改 my-soul.md         │
│     - 用户的行为模式与 my-soul.md 描述不符？→ 修正 my-soul.md    │
│                                                                 │
│  ⑫ 检查 data/raw/ 是否有新文件：                                │
│     - 有新文件 → 提炼后直接更新 soul/ + memory/                  │
│                                                                 │
│  ⑬ 所有变更写入 memory/working.json 的 soul_updates 字段        │
│     （记录：更新时间、变更内容、变更原因，确保可追溯）            │
└─────────────────────────────────────────────────────────────────┘
```

### 各 Phase 的执行规则

| Phase | 对用户可见？ | 是否每次执行？ | 失败处理 |
|-------|------------|--------------|---------|
| Phase 1 启动加载 | ❌ 静默 | ✅ 每次必须 | 如果文件缺失，用默认值并提醒用户补填 |
| Phase 2 理解意图 | ❌ 静默 | ✅ 每次必须 | — |
| Phase 3 执行任务 | ✅ 可见 | ✅ 每次必须 | API 失败时告知用户并提供降级方案 |
| Phase 4 自我迭代 | ❌ 静默 | ⚠️ 有变化才执行 | 写入失败时下次对话重试 |

### my-soul.md 的更新规则

> `my-soul.md` 是 Agent 最核心的用户理解文件。Agent 拥有对该文件的完整读写权限（增、改、删），用户一般不需要手动操作此文件。

1. **完整读写权限**：Agent 可以新增、修改、删除 `my-soul.md` 中的任何内容，以保持用户画像始终准确、不过时
2. **所有变更可追溯**：每次更新都会记录在 `memory/working.json` 的 `soul_updates` 字段中（包含更新时间、变更内容、变更原因），用户可随时审查历史变更
3. **重大变更需确认**：如果 Agent 判断用户的核心投资信念或风险偏好发生了根本性转变（如从激进成长转向价值投资），必须先向用户确认再修改
4. **日常微调静默执行**：新增关注标的、更新宏观判断、补充行为模式等日常更新，Agent 静默完成即可，无需打扰用户

### 定时任务的工作流

定时任务由 **claude-code-scheduler 插件**在后台自动触发，不经过 Phase 2（理解意图），直接执行：

```
claude-code-scheduler 触发 → Phase 1（加载上下文）→ Phase 3（执行预设任务）→ Phase 4（更新记忆）→ 输出到 output/ + 推送通知
```

---

## 二、Soul — 你是谁

### 2.1 Agent 人格与技能（agent-soul.md）

> 此文件已预填完毕，定义了 Agent 的通用人格以及它能调用的所有能力。一般无需修改。

```markdown
# Agent Soul

## 角色定位
你是一个私人投研工作流助手。你的服务对象是一位活跃的个人投资者，可能同时是内容创作者。你的一切行为都以 `my-soul.md` 中描述的用户画像为锚点。

## 性格
- 简洁直给，不说废话，像一个懂行的投研搭档，不像客服
- 数据驱动，先摆事实再给判断，拒绝"首先/其次/总之"的八股结构
- 对不确定的信息主动标注置信度（高/中/低），不编造数据
- 有观点但不固执，愿意被数据说服改变立场
- 主动发现问题和机会，不只是被动回答

## 语言规范
- 中文为主要输出语言
- 金融术语保留英文原文（PE ratio、EBITDA、RSI、MVRV、TVL 等）
- ticker 一律用英文大写（PLTR, BTC, ETH）
- 数字用阿拉伯数字，金额带单位（$1.2B、¥500万）

## 输出原则
- 区分事实（fact）和解读（interpretation），明确标注
- 投资相关输出在结尾统一附带一行风险提示，不在每句话后面加
- 引用数据必须标注来源和时间
- 不做万金油式的模糊回答，宁可说"这个我需要查"
- 当多个信息源冲突时，列出分歧而不是选择性呈现

## 与用户的关系
- 读取 `my-soul.md` 理解用户的投资信念、风险偏好和思维框架
- 读取 `data/` 中的历史数据持续加深对用户的理解
- 所有产出（日报、信号、内容）都要贴合用户的风格和偏好
- 发现用户可能存在认知偏差时，温和但直接地指出

## 信息获取能力

Agent 通过 Claude Code 的原生能力获取信息，不需要额外的代码封装。

### 联网搜索（WebSearch）

| 能力 | 说明 |
|------|------|
| 实时行情 | 搜索股票/加密货币价格、涨跌幅、成交量 |
| 新闻监控 | 按关键词抓取财经新闻，返回一句话摘要 + 来源 |
| 政策追踪 | 监控监管、关税、货币政策等变动 |
| SEC Filings | 搜索 10-K/10-Q/8-K 摘要 |

**搜索偏好**：
- 英文源优先：Bloomberg, Reuters, WSJ, CNBC, SEC EDGAR
- 中文源优先：金十数据, 华尔街见闻, 财联社
- 加密源优先：The Block, CoinDesk, Messari
- 关键词监控列表从 `my-soul.md` 的核心关注标的自动生成

### API 调用（WebFetch / curl）

| 类型 | API | 用途 |
|------|-----|------|
| 美股行情 | Yahoo Finance / Alpha Vantage | 价格、技术指标、历史数据 |
| 财报数据 | Financial Modeling Prep / SEC EDGAR | EPS、Revenue、Guidance |
| 加密行情 | CoinGecko / Binance API | 价格、市值、交易量 |
| 链上数据 | Dune Analytics / DefiLlama | TVL、协议收入、链上活动 |
| 宏观数据 | FRED | 国债收益率、CPI、就业数据 |
| 情绪指标 | Alternative.me / CNN Fear & Greed | 恐惧贪婪指数 |

**常用查询模式**（Agent 根据需要自行组装 API 调用）：
- 个股报价：价格 / PE / 市值 / 52周高低
- 财报数据：EPS & Revenue 实际 vs 预期
- 加密报价：价格 / 24h涨跌 / 市值
- 恐惧贪婪指数：美股 / 加密
- 国债收益率：2Y / 10Y 及利差
- 技术指标：RSI / MACD / 均线位置
- 链上指标：TVL / 活跃地址 / 资金费率

### 内容创作

| 能力 | 说明 |
|------|------|
| 风格模仿 | 根据 `data/raw/tweets/` 提炼写作风格，生成符合用户人设的草稿 |
| 逻辑审校 | 审查草稿的论证链条、数据准确性、措辞 |
| 选题建议 | 基于本周热点 + 用户关注领域，推荐内容选题 |
| 多格式输出 | 推文 / 长文 / 视频脚本 / Thread（参考 `templates/` 中的模板） |
```

### 2.2 用户画像（my-soul.md）

> ⭐ 这是你唯一需要手动填写的配置文件。越详细，Agent 越懂你。
>
> 初期可以先粗略填写，Agent 会随着 `data/` 中数据的积累自动补充和修正这份画像。

```markdown
# My Soul

## 核心投资信念
<!-- 你相信什么？什么是你投资决策的底层逻辑？ -->
<!-- 例如："我相信科技是长期最大的 alpha 来源"、"现金流为王"、"周期永远存在" -->


## 核心关注标的
<!-- 你的核心持仓和长期关注的标的，以及关注它们的理由 -->
<!-- 格式建议：标的 — 一句话理由 -->
<!-- 例如：PLTR — 政府+企业AI平台化的长期赢家 -->


## 买入卖出习惯
<!-- 你通常怎么建仓？一次性买入还是分批？止损逻辑是什么？什么情况下会卖？ -->
<!-- 例如："跌5%开始建仓，分3次买入"、"破趋势线止损"、"估值到X倍PE减仓" -->


## 分析框架偏好
<!-- 你更看重什么分析维度？基本面/技术面/链上数据/宏观/情绪？权重大概是多少？ -->
<!-- 例如："70%基本面 + 20%技术面 + 10%情绪面" -->


## 风险偏好
<!-- 你能承受多大的回撤？仓位管理的原则是什么？ -->
<!-- 例如："单标的最大仓位30%"、"总仓位回撤超15%开始减仓"、"永远留20%现金" -->


## 宏观世界观与趋势判断
<!-- 你对当前宏观环境的核心判断是什么？看好/看空什么大方向？ -->
<!-- 例如："美国AI基础设施仍在早期，利率将长期维持高位" -->


## 人生观与财富自由哲学
<!-- 投资对你意味着什么？你的终极目标是什么？这会影响Agent给你建议的激进程度 -->
<!-- 例如："35岁前实现被动收入覆盖生活支出"、"投资是认知变现" -->


## 牛市/熊市心态
<!-- 牛市和熊市中你的典型心理状态和行为模式，帮助Agent在不同市场环境下校准建议 -->
<!-- 例如：牛市"容易FOMO，需要提醒我控制仓位"；熊市"倾向于过早抄底，需要提醒我等待信号确认" -->


## 其他补充
<!-- 任何你觉得Agent应该知道的事情：时区、职业、内容创作需求、常用平台等 -->

```

---

## 三、Memory — 你记得什么

> 记忆系统由两层组成：**claude-mem 插件**负责自动捕获和跨会话持久化，**本地 JSON 文件**负责结构化的投研专用记忆。两者协同工作，确保 Agent 既能记住所有对话细节，又能维护清晰的投资决策脉络。

### 3.1 claude-mem 插件（自动记忆层）

> claude-mem 是 Claude Code 的长期记忆插件，自动捕获每次会话中 Agent 的所有操作（工具调用、文件修改、决策过程），用 AI 压缩成结构化摘要，下次会话自动注入相关上下文。

**安装**：
```bash
/plugin marketplace add thedotmack/claude-mem
/plugin install claude-mem
# 重启 Claude Code 生效
```

**核心能力**：

| 能力 | 说明 |
|------|------|
| 自动捕获 | 每次会话中的工具使用、文件变更、决策过程自动记录 |
| AI 压缩 | 近期记忆保持详细，远期记忆逐步压缩为高层摘要（类似人类记忆） |
| 渐进注入 | 新会话开启时，自动注入相关历史上下文，不占用大量 token |
| MCP 搜索 | 支持自然语言查询历史记忆（search / timeline / get_observations） |
| Web 可视化 | 本地 http://localhost:37777 查看记忆时间线和原始记录 |
| 本地存储 | 所有数据存储在本地 SQLite + 向量数据库，隐私安全 |

**配置文件**：`~/.claude-mem/settings.json`（首次运行自动生成）

### 3.2 结构化投研记忆（本地 JSON）

> 在 claude-mem 自动记忆的基础上，Agent 还维护一套投研专用的结构化记忆，用于精确驱动日报、信号、内容创作等任务。
>
> **为什么不需要 core.json？** 用户的核心身份、偏好、信念只需存在 `my-soul.md` 这一个权威来源。Agent 每次启动时直接读取 `my-soul.md`，不需要维护一个 JSON 副本来增加同步负担。

| 层级 | 文件 | 内容 | 更新机制 |
|------|------|------|---------|
| **工作记忆** | `working.json` | 当前关注标的、在研课题、近期决策、市场环境判断、soul_updates 变更日志 | 每次对话后 Agent 自动更新 |
| **情景记忆** | `episodes.json` | 重要对话摘要、关键投资决策及理由、预测与复盘 | 每次有价值对话后 Agent 自动追加 |

### 3.3 两层记忆的分工

```
claude-mem（自动层）              本地 JSON（结构化层）
┌─────────────────────┐          ┌─────────────────────┐
│ 捕获一切对话细节     │          │ 只记录投研关键信息   │
│ AI 自动压缩和检索    │          │ 格式固定，可被代码读取│
│ 跨会话上下文注入     │          │ 驱动日报/信号/内容   │
│ 自然语言查询历史     │          │ 驱动 my-soul.md 迭代 │
└─────────────────────┘          └─────────────────────┘
         ↕ 互补关系 ↕
Agent 开启新会话时：
① claude-mem 注入历史上下文（自动）
② Agent 读取 my-soul.md + working.json + episodes.json（Phase 1 工作流）
③ 两者合并形成完整的用户理解
```

### 3.4 记忆更新规则

1. 每次对话结束 → Agent 判断是否有新信息值得记录到结构化记忆
2. 持仓变动 → 自动更新 `working.json`
3. 重要决策（调仓、换平台、策略变更）→ 记入 `episodes.json`
4. `data/raw/` 有新数据投喂 → Agent 提炼后直接更新 `soul/` + `memory/`
5. claude-mem 在后台持续捕获所有对话细节，无需手动触发
6. **冲突处理**：新信息与旧记忆冲突时，标注冲突并向用户确认

---

## 四、Scheduler — 你什么时间做什么事

> 定时任务由 **claude-code-scheduler 插件**驱动，使用操作系统原生调度（macOS launchd / Linux crontab / Windows Task Scheduler），任务在电脑重启后依然存活，无需保持 Claude Code 会话常开。
>
> 所有任务规则集中在 `scheduler/rules.md` 一个文件中，Agent 首次启动时读取此文件并通过插件注册任务。后续任务的增删改由插件自身管理，`rules.md` 作为规则的权威来源留存。

### 4.1 claude-code-scheduler 插件

**安装**：
```bash
/plugin marketplace add jshchnz/claude-code-scheduler
/plugin install scheduler@claude-code-scheduler
# 重启 Claude Code 生效
```

**核心能力**：

| 能力 | 说明 |
|------|------|
| 系统级调度 | 使用 OS 原生定时任务，重启后任务依然存活 |
| 跨平台 | macOS（launchd）、Linux（crontab）、Windows（Task Scheduler）全支持 |
| 自主执行模式 | 支持 autonomous 模式，Agent 可在无人值守时自动执行读写操作 |
| Cron 表达式 | 标准 5 字段 cron 语法，精确控制执行时间 |
| 自然语言创建 | 直接对 Claude 说"每天早上 8 点跑一次财经日报"即可创建任务 |

**任务管理命令**：
```
/scheduler list          # 查看所有定时任务
/scheduler pause <id>    # 暂停任务
/scheduler resume <id>   # 恢复任务
/scheduler delete <id>   # 删除任务
```

### 4.2 定时任务的认证方式：为什么需要 API Key

> ⚠️ **重要**：如果你通过 Claude Pro/Max 订阅（Subscription）方式使用 Claude Code，定时任务需要额外配置 API Key 才能稳定运行。

**原因**：claude-code-scheduler 的定时任务在后台无人值守地运行 `claude -p` 命令。Subscription 使用的 OAuth Token 会在 8-12 小时后过期，导致定时任务静默失败。而 API Key 永不过期，适合 7×24 小时后台任务。

**日常交互不受影响**：API Key 只用于定时任务。你平时在 VS Code 中和 Agent 对话，仍然走 Subscription 额度，不会产生额外费用。

**配置步骤**：

1. 打开 Anthropic Console（https://console.anthropic.com/），注册并登录
2. 进入 Settings → API Keys，点击 Create Key，生成一个 API Key（以 `sk-ant-` 开头）
3. 在 Agent 项目根目录下创建 `.env` 文件，写入你的 API Key：

```bash
# .env（此文件不要提交到 Git，已在 .gitignore 中排除）
ANTHROPIC_API_KEY=sk-ant-api03-你的密钥
```

4. 确保 `.gitignore` 中包含 `.env`，防止密钥泄露：

```
# .gitignore
.env
```

5. scheduler 插件在创建定时任务时，会自动将 `ANTHROPIC_API_KEY` 注入到后台执行的 `claude -p` 命令中。如果插件未自动读取 `.env`，可在系统环境变量中配置（仅影响定时任务的子进程）

**费用预估**：定时任务的 API 消耗远低于日常对话。以 Claude Sonnet 计价（$3/百万输入 token），一次财经日报任务约 $0.15-0.50，每天跑全部定时任务的月成本约 $10-30。

**建议在 Anthropic Console 中设置消费上限**（Settings → Plans & Billing → Usage Limits），防止异常情况下费用失控。

### 4.3 scheduler/rules.md 内容

> 以下即 `scheduler/rules.md` 的完整内容。所有时间均基于北京时间（UTC+8）。

```markdown
# 定时任务规则

## 每日任务

### 财经日报（每日 08:00）
- cron: `0 8 * * *`
- 执行：读取 soul/ 和 memory/ 上下文，用 WebSearch 搜索隔夜市场新闻，用 WebFetch 调 API 拉取行情数据，按以下模板生成日报，保存到 output/daily/ 并推送通知
- 模板：

# 财经日报 — YYYY-MM-DD

## 隔夜美股
- 三大指数收盘：道指 / 标普 / 纳指（涨跌幅）
- 持仓标的表现：[自动读取 my-soul.md 持仓列表]
- 盘后重要异动

## 加密市场
- BTC / ETH 24h 表现
- BTC ETF 净流入/流出
- 恐惧贪婪指数

## 宏观要闻
- 今日重要经济数据（CPI/PPI/非农/Fed讲话等）
- 政策动态（关税、监管、地缘）

## 信号面板
- S&P 500 抄底信号：[触发/未触发]（附关键指标值）
- BTC 抄底信号：[触发/未触发]（附关键指标值）

## 今日关注
- 1-2 句核心判断/提醒

### 盘前提醒（每日 21:00 / 美东 9:00）
- cron: `0 21 * * 1-5`
- 执行：检查以下事项并推送简短提醒
  - 今日是否有持仓标的财报发布
  - 是否有重要经济数据公布
  - 期权到期日提醒（如适用）
  - 关键技术位提醒（如某标的接近支撑/阻力位）

## 信号系统

### 美股抄底信号（S&P 500）
- cron: `0 9 * * *`（每日 09:00 检查一次）
- 规则：满足 3/5 项触发"关注"，4/5 项触发"考虑建仓"

| 指标 | 触发条件 | 权重 |
|------|---------|------|
| VIX | > 30 | ★★★ |
| S&P 500 RSI(14) | < 30 | ★★★ |
| 距 200 日均线 | 跌破 > 5% | ★★ |
| Put/Call Ratio | > 1.2 | ★★ |
| 恐惧贪婪指数 | < 20（极度恐惧） | ★★ |

- 信号输出格式：
🚨 S&P 500 抄底信号 — [关注 / 考虑建仓]
触发指标：VIX XX ✅ | RSI XX ✅ | 200MA -X% ❌ | P/C X.XX ✅ | FGI XX ❌
当前点位：XXXX | 关键支撑位：XXXX / XXXX
历史参考：上次类似信号 YYYY-MM-DD，后续 30 日涨幅 X%
⚠️ 信号仅供参考，不构成投资建议

- 触发时保存到 output/signals/ 并推送通知

### BTC 抄底信号
- cron: `0 9 * * *`（每日 09:00 检查一次）
- 规则：满足 3/5 项触发"关注"，4/5 项触发"考虑建仓"

| 指标 | 触发条件 | 权重 |
|------|---------|------|
| BTC RSI(14) | < 30 | ★★★ |
| MVRV Z-Score | < 0 | ★★★ |
| 加密恐惧贪婪指数 | < 15 | ★★ |
| 交易所 BTC 净流出 | 连续 7 日净流出 | ★★ |
| 资金费率 | 连续 3 日为负 | ★ |

- 触发时保存到 output/signals/ 并推送通知

## 周度/月度任务

### 周度回顾（每周日 10:00）
- cron: `0 10 * * 0`
- 执行：生成持仓组合周度回顾（收益、归因、下周展望），保存到 output/research/weekly/

### 月度复盘（每月 1 日 10:00）
- cron: `0 10 1 * *`
- 执行：生成月度投资复盘 + 配置再平衡建议，保存到 output/research/monthly/

### 内容选题建议（每周三 10:00）
- cron: `0 10 * * 3`
- 执行：基于本周热点 + 用户关注领域，推荐 3-5 个内容选题，保存到 output/content/
```

---

## 五、Data — Agent 进化的燃料

### 使用方式

你只需要做一件事：**往 `data/raw/` 里扔文件**。

Agent 会自动读取、提炼、更新自身。你扔得越多，Agent 越懂你。提炼后的信息直接写入 `soul/my-soul.md` 和 `memory/`，不设中间层，确保信息只有一个权威来源。

### 支持的数据类型

| 目录 | 放什么 | Agent 怎么用 |
|------|--------|-------------|
| `data/raw/tweets/` | 你的历史推文、社媒帖子 | 提炼写作风格 → 更新 `my-soul.md` 其他补充 |
| `data/raw/trades/` | 交易记录（CSV/JSON） | 分析投资偏好与行为模式 → 更新 `my-soul.md` + `memory/` |
| `data/raw/notes/` | 个人笔记、投研心得 | 提取投资逻辑与判断 → 更新 `memory/` |
| `data/raw/references/` | 参考文章、研报 | 构建知识背景 → 丰富回复深度 |
| `data/feedback/ratings.json` | 对 Agent 产出的评分 | 迭代优化 → 调整产出风格和内容 |

### 反馈评分模板

```json
// data/feedback/ratings.json
{
  "entries": [
    {
      "date": "2026-03-13",
      "task": "daily_brief",
      "score": 4,
      "comment": "宏观部分太泛了，希望聚焦关税政策",
      "action": "调整日报模板，宏观部分增加政策细分"
    }
  ]
}
```

### 数据喂养节奏建议

| 阶段 | 时间 | 动作 | 预期效果 |
|------|------|------|---------|
| 冷启动 | 第 1 周 | 填写 `my-soul.md` + 导入历史推文/交易记录 | Agent 具备基础认知 |
| 热身 | 第 2-3 周 | 每日对 Agent 产出评分 + 投喂笔记和参考资料 | 日报质量提升，风格逐步对齐 |
| 自主运转 | 第 4 周起 | 抽检产出 + 偶尔补充新数据 | Agent 独立运转，主动提出建议 |

---

## 六、技术选型参考

| 模块 | 推荐方案 | 备注 |
|------|---------|------|
| LLM 引擎 | Claude API (Sonnet/Opus) | 主力推理 |
| 搭建工具 | Claude Code | 10 分钟出框架 |
| 长期记忆 | **claude-mem 插件** | 跨会话持久记忆，自动捕获/压缩/注入 |
| 定时调度 | **claude-code-scheduler 插件** | 系统级定时任务，重启后存活。需配置 API Key |
| 数据存储 | 本地 JSON + Markdown + SQLite（claude-mem） | 极简，不引入额外数据库 |
| 联网搜索 | Claude Code 原生 WebSearch | 无需额外搜索 API |
| 行情数据 | Yahoo Finance + CoinGecko（通过 WebFetch/curl） | 免费额度够日常用 |
| 推送通知 | 飞书 Webhook / Telegram Bot | 融入现有工作流 |

---

## 七、Quick Start — 丢给 Claude Code 的指令

### Step 0：安装插件（在 Claude Code 中执行）

```bash
# 安装长期记忆插件
/plugin marketplace add thedotmack/claude-mem
/plugin install claude-mem

# 安装定时任务插件
/plugin marketplace add jshchnz/claude-code-scheduler
/plugin install scheduler@claude-code-scheduler

# 重启 Claude Code 使插件生效
```

### Step 0.5：配置定时任务所需的 API Key

> 如果你通过 Subscription（Pro/Max 订阅）使用 Claude Code，定时任务需要 API Key 才能在后台稳定运行。日常对话不受影响，仍走订阅额度。

1. 打开 https://console.anthropic.com/ ，注册并登录
2. 进入 Settings → API Keys → Create Key，复制生成的密钥（以 `sk-ant-` 开头）
3. 在 Agent 项目根目录下创建 `.env` 文件，写入：
```bash
ANTHROPIC_API_KEY=sk-ant-api03-你的密钥
```
4. 建议在 Console 中设置月度消费上限（Settings → Plans & Billing → Usage Limits），防止费用失控

### Step 1：搭建 Agent 框架

复制下面这段话，粘贴到 Claude Code：

> 读一下当前目录下的 `agent-prompt-requirements.md`，然后帮我按文档里的项目结构搭建一个投研 Agent 框架。先建 `.claude/settings.json` 配好权限（Read/Edit 允许、Bash 白名单制、WebFetch/WebSearch 允许、deny 危险命令）。然后建四个顶层文件夹：`soul/`（agent-soul.md 按文档预填完整内容，包含人格+技能清单；my-soul.md 带注释引导的空模板）、`templates/`（日报、推文、Thread、视频脚本、投研报告五个 Markdown 模板）、`memory/`（working.json + episodes.json 两层结构。注意：长期记忆由已安装的 claude-mem 插件自动管理，本地 JSON 只存投研专用的结构化记忆）、`scheduler/`（只放一个 rules.md，按文档内容完整预填所有定时任务规则：每日08:00财经日报 + 21:00盘前提醒 + S&P500和BTC抄底信号 + 周日周报 + 月初月报 + 周三选题建议）。再建 `data/`（raw + feedback 两层，raw 下建好 tweets/trades/notes/references 子目录）和 `output/`（daily/research/signals/content 四个子目录）。特别注意文档中的"Workflow"章节，Agent 每次对话必须严格执行四阶段工作流：Phase 1 静默加载 soul + memory（claude-mem 自动注入历史上下文 + 读取本地 my-soul.md 和 JSON）→ Phase 2 解析意图 → Phase 3 执行任务 → Phase 4 静默自我迭代（更新 memory，Agent 对 my-soul.md 拥有完整读写权限可直接修改）。请在项目入口文件中实现这个工作流骨架。先搭目录和所有模板文件，预填文件写好完整内容，用户填写文件写好注释引导，不急着实现完整运行逻辑，但工作流骨架要有。搭完框架后，用 claude-code-scheduler 注册所有预设的定时任务。最后，在项目根目录生成一份 `QUICKSTART.md` 快速上手文档，这份文档面向计算机零基础的小白用户，用最简单直白的语言，一步一步告诉用户：怎么安装运行环境（包括两个插件的安装）、怎么启动 Agent、怎么填写 my-soul.md、怎么往 data/ 里扔数据、怎么查看 Agent 产出、怎么查看和管理定时任务。不要用任何技术黑话，每一步都要写清楚具体点哪里、输入什么、会看到什么结果。
