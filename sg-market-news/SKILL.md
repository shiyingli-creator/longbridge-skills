---
name: sg-market-news
description: >
  每日搜索并汇总新加坡股市、宏观经济政策的重要新闻，按重要程度筛选，
  生成双语（中英）市场简报；用户追加港美股关注事件后，自动生成完整的
  英文社区讨论帖子（含标题、摘要、正文、3大问题、市场回顾）。支持中英文触发。
  触发词：新加坡市场新闻、今天新加坡股市、SG market news、Singapore market update、
  给我新加坡新闻、daily SG briefing、新加坡今日要闻、sg-market-news。
  当用户想了解新加坡股市、宏观经济、上市公司动态，或在SG简报基础上生成社区讨论帖时，必须使用此skill。
---

# SG Market News Skill

三阶段工作流：
- **Phase 1**：搜索并汇总当日新加坡市场新闻，输出双语简报
- **Phase 2**：用户追加港美股事件后，合并所有信息，生成完整英文社区讨论帖
- **Phase 3**：自动生成 5 个新加坡散户视角的社区互动帖（可选 1 个为博主风格）

## 依赖说明

- **必需工具**：`web_search` 和 `web_fetch`（Phase 1 抓取新闻所需）
- **配套 Skill**：`blogger-writer`（Phase 3 博主风格帖会引用其博主索引 `/mnt/skills/user/blogger-writer/INDEX.md`）。若未安装，Phase 3 仍可运行，但博主风格帖会自动回退为第 5 个散户视角帖。
- **运行环境**：建议使用支持联网搜索的 Claude（claude.ai 或带 web 工具的 API 配置）

---

## Phase 1：新加坡市场简报

### Step 1：并行搜索（9条）

同时执行以下搜索，覆盖所有关键领域。**个股搜索最重要，务必多搜**：

**宏观 / 政策（3条）**
1. `Singapore economic policy MAS announcement today`
2. `Singapore property HDB market news today`
3. `Singapore GDP trade inflation news today`

**个股动态（6条）— 重点**
4. `SGX listed companies earnings results today`
5. `Singapore listed company contract win acquisition today`
6. `Singapore listed company CEO CFO director change today`
7. `Singapore REIT distribution results today`
8. `Singapore blue chip DBS OCBC UOB SIA Singtel results news today`
9. `SGX company announcement corporate action today`

对搜索结果中出现的具体公司，**使用 web_fetch 抓取原文页面**以获取更完整的数据（股价、具体金额、百分比等）。

**可信来源白名单**（优先抓取，排除其他）：

*本地媒体*
- The Business Times (businesstimes.com.sg) — 首选，moomoo News SG 主要引用来源
- The Straits Times (straitstimes.com)
- Channel NewsAsia / CNA (channelnewsasia.com)

*官方机构*
- SGX (sgx.com) — 官方公告、月度市场数据
- MAS (mas.gov.sg) — 货币政策、通胀数据
- HDB (hdb.gov.sg) — 房产数据
- MTI (mti.gov.sg) — GDP、贸易数据
- DOS (singstat.gov.sg) — 统计数据

*财经聚合 / 数据*
- SGinvestors.io — SGX公告聚合，用于发现个股新闻线索
- ShareInvestor (shareinvestor.com) — STI点位、市场数据
- moomoo News SG (moomoo.com/community) — 参考选题方向，不直接引用

*国际媒体（新加坡相关报道）*
- Reuters Singapore
- Bloomberg Singapore

### Step 2：筛选与评分

从搜索结果中筛选新闻，按以下标准打分，**总计不超过10条**：

| 优先级 | 类型 | 示例 |
|--------|------|------|
| ⭐⭐⭐ 高 | 政策/央行/MAS声明、重大财报（蓝筹股）、并购 | MAS加息、DBS季报 |
| ⭐⭐ 中 | 中型公司财报、高管变动、大额合同、REIT分派 | REITs季度业绩、CEO离职 |
| ⭐ 低 | 小公司动态、分析师评级调整 | 小市值股票评级 |

**数量目标**：
- 🇸🇬 Breaking News：1–3条（宏观/政策，质量优先）
- 🔔 Stocks to Watch：**5–8条**（个股为主体，尽量覆盖不同板块）
- 合计不超过10条

**过滤规则**：

- **时效范围**：优先收录当日新闻；前一交易日收盘后发布的新闻（如盘后财报、盘后公告）同样可收录，但超过1个交易日的旧闻一律排除

- **核心原则：每个热点默认只展示1天**。一个事件在某日简报/帖子中出现过后，次日起**默认不再收录**，除非满足下方"再次收录条件"。这是为了避免社区帖子重复，保持每日新鲜感。

  *再次收录条件*（满足任一即可）：
  - **结果落地**：从"预期/传闻/拟议"进入"确认/落地"阶段。例：拟收购 → 完成交割、传闻加息 → MAS正式宣布
  - **进展更新**：事件本身有新的实质性进展。例：贸易谈判昨天提及但未达成，今天有新协议草案
  - **数据修正**：核心财务数据被修正，或出现重大反转
  - **市场反应剧烈**：财报次日股价异动超过 ±5%、出现监管/官方回应、引发同行业连锁反应
  
  *判断示例*：
  - ❌ "DBS 今天发了财报，明天再讲一遍财报内容" → 重复，不收录
  - ✅ "DBS 财报次日股价跌 7%、分析师下调评级" → 是市场新反应，可收录
  - ❌ "特朗普访华行程消息昨天已发" → 重复，不收录
  - ✅ "特朗普访华今天有新的协议草案/联合声明" → 是新进展，可收录
  - ❌ "MAS 上周已宣布的政策" → 旧闻，不收录
  - ✅ "MAS 政策落地后首份通胀数据出炉" → 是后续数据点，可收录

  **执行方式**：生成 Phase 1 简报或 Phase 2 帖子前，回顾最近1-2个工作日已发布的简报（如有记忆/上下文），逐条核查是否构成重复。无法核查时，对"看起来眼熟"的事件（尤其是大公司财报、宏观政策）保持警觉，优先采用"是否有新进展"作为筛选标准。

- 排除重复新闻（同一事件只保留最权威来源）
- 排除非新加坡核心来源的消息
- Catalist小市值股票：仅收录有实质性事件（财报、合同、高管变动）的，不收录纯行政公告

### Step 3：分类整理

**板块A：🇸🇬 Breaking News**
- 宏观经济、政策、房产、MAS、政府数据
- 每条包含：核心数据点、背景、市场影响

**板块B：🔔 Stocks to Watch**
- 个股动态：财报、合同、高管变动、诉讼、并购
- 每条必须包含：股票代码（格式见下方）、价格变动

### Step 4：输出 Phase 1 简报

格式见下方「Phase 1 输出模板」。

**输出完毕后**，主动提示用户：
> "简报已完成 ✅ 如需生成社区讨论帖，请继续输入今日港股 / 美股的关注事件，我将合并整理为完整帖子。"

---

## Phase 2：社区讨论帖生成

**触发条件**：用户在 Phase 1 简报完成后，追加输入港股或美股的事件、数据或要点。

### Step 0：输入核验（生成帖子前必须执行）

在整合内容前，先对用户输入的所有标的进行市场归属核验：

**保留**：
- 🇺🇸 美股（NYSE / Nasdaq / AMEX 上市）
- 🇭🇰 港股（HKEX 上市，股票代码为数字.HK）
- 🇸🇬 新加坡股（SGX 上市）

**摘除**：
- 🇨🇳 A股（沪深两市上市，股票代码为6位数字，交易所为 SSE / SZSE）
- 任何仅在中国大陆市场上市的标的（包括北交所）

> 注意：同一公司若同时有港股和A股（如腾讯、比亚迪），**保留港股部分，摘除A股部分**。若用户输入的是A股事件但该公司有港股，自动转换为港股视角处理。

核验完成后，**不需要向用户列出被摘除的标的**，直接用保留的内容生成帖子。

### 整合逻辑

将以下内容合并为一篇帖子：
1. Phase 1 已整理的新加坡新闻（宏观 + 个股）
2. 用户输入中**核验保留**的港美股事件

### 帖子写作原则

- **语言**：全英文
- **语气**：活泼、有观点、像一个熟悉全球市场的投资者朋友在聊天，不是机构报告
- **目标**：吸引社区用户互动评论，问题要有争议性或开放性，让人忍不住想回答
- **精简但有料**：正文导语控制在5句以内，突出最重磅的2–3个事件
- **3大问题**：每个问题必须跨越"事实描述"，聚焦"判断/预测/策略"，让读者真正需要思考才能回答；问题质量检验：读者应能说出一个具体的立场或策略，而不只是复述新闻
- **市场回顾**：按 🌍 Global → 🇺🇸 US → 🇭🇰 HK/China → 🇸🇬 SG 分区块，每区块3–6句，有数据支撑

### Phase 2 输出模板

```
☕️ [Task Coins Giveaway] Daily Market Talk

🇺🇸🇭🇰🇸🇬 Big moves across US, HK, SG markets. Join the chat & earn Task Coins!

[导语：2–3句，点出当日最重磅的2–3个跨市场事件，语气轻松有力，以"Midday update:"开头，不铺陈背景，直接说事]

💬 Today's 3 Big Questions
1. [事件背景 — 核心判断/策略问题？]
2. [事件背景 — 核心判断/策略问题？]
3. [事件背景 — 核心判断/策略问题？]

👇 Like, drop your take (20+ words), click "Repost" & earn 288 Task Coins ✨
🚫 NO duplicates or similar content
⏰ Deadline: [当日日期], 11:59 PM (SGT)

📊 Quick Market Recap

🌍 Global Macro & Markets
• [宏观事件1：地缘/大宗/央行，1–2句，含关键数据]
• [宏观事件2]
• [如有更多，继续追加；无则省略]

🇺🇸 US Markets: [小标题反映当日主题，如"AI Demand Fuels Tech Surge"]
• 📈 [指数表现：S&P 500 / Nasdaq / Dow，含涨跌幅]
• [公司名] — [核心事件+关键数据，1–2句]
• [公司名] — [核心事件+关键数据，1–2句]
• [如有更多个股，继续追加]

🇸🇬 Singapore Markets
• 📈 [STI点位与涨跌；如无数据则省略此条]
• [宏观数据条目，如通胀/MAS政策，1–2句]
• [公司名] — [核心事件+关键数据，1–2句]
• [公司名] — [核心事件+关键数据，1–2句]
• [如有更多个股，继续追加]

🇭🇰 Hong Kong & China Markets
• 📈 [指数表现：HSI / Hang Seng Tech，含涨跌幅]
• [公司名] — [核心事件+关键数据，1–2句]
• [如有更多个股，继续追加]

📅 Key Events Tonight / This Week
• 🇺🇸/🇭🇰/🇸🇬 [事件名] — [时间 SGT]
• 🇺🇸/🇭🇰/🇸🇬 [事件名] — [时间 SGT]
• [如有更多，继续追加；无则省略]

Disclaimer: For reference only, not investment advice.
```

帖子输出完毕后，另起一段输出标的代码汇总：

```
📎 相关标的代码汇总
$公司全称 (代码.US)$
$公司全称 (代码.HK)$
$公司全称 (代码.SG)$
[按帖子中出现顺序列出所有标的，每个占一行；代码不确定的写 $公司名$，绝不填错误代码]
```

---

## 股票代码格式

使用长桥/Longbridge社区格式：

```
$公司全称 (股票代码.SG)$   ← 新加坡股
$公司全称 (股票代码.US)$   ← 美股
$公司全称 (股票代码.HK)$   ← 港股
```

例：
- `$DBS Group Holdings (D05.SG)$`
- `$Intel Corporation (INTC.US)$`
- `$Tencent Holdings (700.HK)$`

如代码不确定，写 `$公司名$` 省略代码，绝不填错误代码。

---

## Phase 1 输出模板

```
📅 Singapore Market Briefing — [日期，例：Monday, April 28, 2025]

━━━━━━━━━━━━━━━━━━━━━━━━━━━━
🇸🇬 Breaking News
━━━━━━━━━━━━━━━━━━━━━━━━━━━━

[英文标题（加粗）]
[英文正文：2–4句，包含关键数据]
📌 [中文摘要：1–2句核心要点]

[如有多条，重复以上格式]

━━━━━━━━━━━━━━━━━━━━━━━━━━━━
🔔 Stocks to Watch
━━━━━━━━━━━━━━━━━━━━━━━━━━━━

$公司名 (代码.SG)$ [涨跌幅，例：▲1.7% S$2.41]
[英文正文：2–3句，说明核心事件]
📌 [中文摘要：1句]

[如有多条，重复以上格式]

━━━━━━━━━━━━━━━━━━━━━━━━━━━━
📊 Market Snapshot（可选，如有数据）
━━━━━━━━━━━━━━━━━━━━━━━━━━━━
STI: [点位] [涨跌]
USD/SGD: [汇率]
```

---

## Phase 3：社区互动帖生成（散户视角为主）

**触发条件**：Phase 2 帖子及标的代码汇总输出完毕后，自动执行 Phase 3，无需用户额外指令。

**核心定位**：模拟真实新加坡散户在社区随手发的帖子。**不是新闻复述，不是博主分析报告**，而是普通投资者看到这条消息后的第一反应、吐槽、提问、晒单、求安慰、调侃同行——像 Reddit r/singaporeinvestments 或 Hardwarezone Money Mind 板块里的真实留言。

### Step 1：提取标的与事件清单

从 Phase 2 内容中提取所有**标的和对应事件**，每个标的/事件作为一个独立写作单元。宏观事件（如 FOMC、PCE、MAS政策）无标的代码，同样纳入清单。

### Step 2：为每个事件生成 5 个帖子

每个标的/事件生成 **5 个独立帖子**，其中：
- **默认 4 个为散户视角**（第一人称"I/me/my"，普通投资者口吻）
- **1 个可选为博主风格**（从 blogger-writer 的 INDEX.md 中选 1 位最契合该事件的博主，用其英文表达风格写）。若事件不适合任何博主，5 个全部为散户视角。

### Step 3：散户视角写作规则

#### 语言与语气

- **全英文**，但保留新加坡本地化用词，自然出现即可：`HDB`, `CPF`, `SGX`, `STI`, `REIT`, `coffee shop talk`, `kopi`, `auntie/uncle`, `boss`, `bro`, `sia`（仅作为感叹词偶尔出现，不滥用）
- **不写中文，不中英夹杂成句**（保留本地化用词除外）
- **第一人称视角**："I just...", "My portfolio...", "Bought at...", "Holding since...", "Just sold my..."

#### 篇幅与节奏

- **每个帖子 1–3 句话**，长短不一：
  - 极短型（1句）：吐槽、感叹、晒结果，常带 emoji
  - 中等型（2句）：观点 + 一个具体动作或问题
  - 稍长型（3句）：背景 + 个人决策 + 留个问题
- **5 个帖子之间篇幅必须有明显差异**，不能全是 2 句的"标准长度"

#### 风格变化（5 个帖子覆盖至少 3 种）

| 风格类型 | 特征 | 例子方向 |
|---------|------|---------|
| 😅 吐槽党 | 抱怨、自嘲、苦中作乐 | "Bought DBS at $48, now $42. RIP my CPF top-up plan 😭" |
| 🚀 看多党 | 兴奋、看好、加仓 | "Adding more on this dip 💪 long-term story still intact" |
| 🤔 提问党 | 真诚求教、不懂就问 | "Anyone know if this dividend qualifies for SRS tax relief? 🙏" |
| 📊 分析党 | 摆数据、谈估值，但很短 | "P/E now at 9x, below 5-year avg of 12x. Cheap or value trap? 🤷" |
| 🎯 晒单党 | 报告自己的操作 | "Sold half my position at $5.20. Locking in some gains ✅" |
| 😏 调侃党 | 拿同行/朋友/auntie 开玩笑 | "My kopi uncle predicted this two weeks ago. Should've listened 🤣" |
| 🛋️ 躺平党 | 不操作、长期持有、看戏 | "Just sitting tight on my REIT bag, the distribution still comes monthly 💤" |

#### Emoji 使用

- **每个帖子至少 1 个 emoji**，但不超过 3 个
- **统一放在帖子末尾**（最后一句的句末，或单独一行收尾），**不要嵌入句子中间**
- 多个 emoji 可以连排在末尾（如 `🚀💪` 或 `😭🤡`），用来强化情绪
- 常用：📈 📉 🚀 💸 😭 🤣 🤔 😅 💪 🙏 ✅ ❌ 🎯 🤷 🔥 💎 🛋️ 💤 ☕ 😏

#### 严格禁止

- ❌ **不用破折号 `—` 或 `–`**（真实用户极少打长破折号，用逗号、句号或换行代替）
- ❌ 不照搬 Phase 2 的句子或数据表述
- ❌ 不用机构报告腔（"analysts expect", "the company reported a strong quarter"）
- ❌ 不捏造具体股价/数字，没把握的就写模糊表述（"around $48", "down quite a bit"）
- ❌ 不写过长的论述段落
- ❌ 不要 5 个帖子语气雷同

### Step 4：博主风格帖（可选第 5 个）

如该事件适合某位博主，从 blogger-writer 的 `INDEX.md`（路径：`/mnt/skills/user/blogger-writer/INDEX.md`）中选 1 位，用其英文风格写 1 个帖子。匹配方向参考：

| 事件类型 | 推荐博主 |
|---------|---------|
| 美联储 / 利率 / 通胀 / 宏观政策 | `nick-timiraos`, `lance-roberts` |
| Tesla / EV / 自动驾驶 | `chris-zheng` |
| 美股长期投资 / 成长股 | `joseph-carlson`, `investnotbet` |
| 技术分析 / 交易策略 | `rayner-teo` |
| 财报快评 / 市场热点 | `meet-kevin`, `michael-kramer` |
| 财报解读 / 估值 | `meitou-meiguu` |
| 商业故事 / 宏观叙事 | `xiao-lin-shuo` |
| 产品 / AI / 科技公司 | `lenny` |
| 金融科普 / 投资原理 | `the-plain-bagel` |
| 新加坡本地股 / REIT | `sg-stocks-investing`, `investmoolah`, `fifth-person` |
| 港股 / 亚洲资讯 | `tradingkey` |

博主风格帖**同样遵守**：全英文、不用破折号、1–3 段（可比散户长一些，但单帖不超过 150 字）、保留博主原本的标志性表达和句式。

### Step 5：输出格式

每个标的/事件为一组：

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━
🎯 [公司名 / 事件名]
━━━━━━━━━━━━━━━━━━━━━━━━━━━━

[散户帖 1，标注风格 emoji，如 😅 吐槽党]
[1–3 句帖子正文]

[散户帖 2，标注风格 emoji]
[1–3 句帖子正文]

[散户帖 3，标注风格 emoji]
[1–3 句帖子正文]

[散户帖 4，标注风格 emoji]
[1–3 句帖子正文]

[博主风格，或第5个散户帖]
[名字（如 nick-timiraos）or 风格 emoji]
[帖子正文]
```

所有事件组逐一输出，组与组之间空一行。

---

## 注意事项

- **严格只引用白名单来源**，不引用论坛、社交媒体、未知博客
- **不得捏造数据**：如找不到某类数据（如STI点位），直接省略该部分
- **中文摘要**要简洁精准，不是机械翻译，要突出对投资者最有用的信息
- 若当天新闻较少（例如假期），如实说明，不凑数
- 股价变动数据以新闻原文为准，不自行推算
- **股票代码**：代码不确定时写 `$公司名$`，绝不填错误代码
- **Phase 2 写作定位**：社区帖子，语气自然亲切有观点；3大问题要有真正的讨论价值，不能只是"你怎么看"式的空洞提问
- **Phase 3 写作定位**：模拟真实新加坡散户在社区随手发帖的语气，第一人称视角，1–3 句不等，加 emoji，**不用破折号**；5 个帖子覆盖至少 3 种风格（吐槽/看多/提问/分析/晒单/调侃/躺平），可选 1 个为博主风格。绝不写成新闻复述或机构报告腔。
