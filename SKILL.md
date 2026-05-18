---
name: blogger-writer
description: |
  按照真实博主的风格写文章（第一人称、面向新加坡读者）。
  用法：/blogger-writer [博主名] [主题]，不指定博主则根据主题自动匹配。

  可选博主（直接在消息里写名字即可）：
  • Lenny Rachitsky — 产品管理、增长、AI 工具
  • Chris Zheng — Tesla/EV、自动驾驶、供应链
  • Nick Timiraos — 美联储、货币政策、利率
  • Lance Roberts — 宏观周期、Fed policy、组合风险
  • Meet Kevin — 美股热点快评、美联储即时解读
  • Rayner Teo — 技术分析、交易策略
  • Joseph Carlson — 美股长期投资、成长股
  • The Plain Bagel — 金融教育、行为金融
  • InvestNotBet — 长期投资、ETF、避免交易陷阱
  • Michael Kramer — 期权结构、VIX/gamma、S&P 500 日评
  • 小lin说 — 财经科普、宏观经济、商业故事（中文）
  • 美投讲美股 — 美股个股、财报解读、估值（中文）
  • SG Stocks Investing — SG REITs、SGX、CPF
  • TradingKey — 全球股票、IPO、加密、商品
  • Investmoolah — SG REITs、股息投资、个人组合
  • Fifth Person — 价值成长、SG/Asia REITs、股息组合
allowed-tools:
  - Read
  - Write
  - Glob
  - WebSearch
  - Bash
---

# Blogger Writer — 博主风格文章生成

> 学习真实博主的写作风格、内容领域和语言习惯，输出对应风格的文章。所有输出面向**新加坡读者**，默认以**第一人称"我"**写作。

---

## 已收录博主（16 位）

完整索引和自动匹配规则见 `{baseDir}/bloggers/INDEX.md`。下表为快速查询：

| 博主ID | 博主 | 平台 | 擅长领域 | 原文语言 |
|--------|------|------|---------|---------|
| `lenny` | Lenny Rachitsky | Substack | 产品管理、增长、PM 职业、AI 工具 | EN |
| `chris-zheng` | Chris Zheng | X | Tesla 中国、EV、自动驾驶、供应链 | EN/中 |
| `nick-timiraos` | Nick Timiraos | X + WSJ | 美联储、货币政策、利率、通胀 | EN |
| `joseph-carlson` | Joseph Carlson | YouTube + X | 美股长期投资、成长股、个人组合 | EN |
| `rayner-teo` | Rayner Teo | X + Blog | 技术分析、交易策略、风险管理 | EN |
| `xiao-lin-shuo` | 小lin说 | YouTube | 财经科普、宏观经济、商业故事 | 中 |
| `meitou-meiguu` | 美投讲美股 | YouTube | 美股个股、财报解读、估值 | 中 |
| `the-plain-bagel` | The Plain Bagel | YouTube | 金融教育、行为金融、投资原理 | EN |
| `meet-kevin` | Meet Kevin | YouTube | 美股热点快评、美联储即时解读 | EN |
| `sg-stocks-investing` | SG Stocks Investing | Blog | SG REITs、SGX、CPF、Singapore property | EN |
| `trading-key` | TradingKey | Web + App | 全球股票、IPO、加密、商品、宏观 | EN |
| `lance-roberts` | Lance Roberts | TalkMarkets | 宏观周期、Fed policy、组合风险 | EN |
| `michael-kramer` | Michael Kramer | TalkMarkets | 期权结构、VIX/gamma、S&P 500 日评 | EN |
| `investmoolah` | Investmoolah | Blog | SG REITs、股息投资、个人组合日记 | EN |
| `investnotbet` | InvestNotBet | Blog | 长期投资、避免交易陷阱、ETF | EN |
| `fifth-person` | The Fifth Person | Blog + YouTube | 价值成长、SG/Asia REITs、股息组合 | EN |

---

## 触发方式

用户可以这样说（自然语言，不需要严格的命令格式）：

- "用 Lenny 的风格写一篇关于 AI agents 产品策略的文章"
- "模仿 Chris Zheng 写小米 SU7 最新销量数据"
- "按照 sg-stocks-investing 的语气写一篇 Mapletree REITs 的分析"
- "帮我写一篇关于美联储降息的短评"（未指定博主，自动匹配 → `nick-timiraos`）

---

## 执行流程

### Step 1: 确定博主风格

**指定了博主**：读取 `{baseDir}/bloggers/{id}.md` 获取风格档案。

**未指定**：读取 `{baseDir}/bloggers/INDEX.md` 末尾的"自主选择规则"，根据主题匹配。匹配后**先告知用户**："I'll write this in [博主名]'s style because the topic best fits their domain." 让用户有机会换人。

---

### Step 2: 主题调研（如涉及时事/具体公司/数据）

用 WebSearch 补充：最新动态、关键数字、官方引语、事件时间线。把调研到的数据点先列出来，写作时引用，事实核查时核对。

---

### Step 3: 写作

按所选博主的风格档案执行。除了忠实还原档案里的**语气、结构、句式、要避免的写法**之外，**以下七条全局规则适用于所有博主**：

#### 规则 1: 第一人称视角

**默认用 "我"**，不要用"我们"指代博主本人。

**为什么**：这些都是**个人博主**，不是机构或团队。把博主本人说成"我们"会让文章读起来像券商研报或公司公告，瞬间丢失个人语气。
- 中文："我觉得 / 我注意到 / 我手上这只 REIT"
- 英文："I think / I noticed / the REIT I hold"

**"我们"的允许场景**：当 "我们 / we / our / us" 是**作者 + 读者的共指**（博主在跟读者群体说话），是允许的，而且通常是好的：它拉近距离，营造对话感。

判断方法：把这句话里的"我们"换成"我和你"，意思通不通？
- ✅ 通："这对我们普通人意味着什么？" → "这对我和你这样的普通人意味着什么？"（成立）
- ✅ 通："Our local rates feel it." → "My rate and your rate both feel it."（成立）
- ❌ 不通："我们认为 MIT 值得持有" → "我和你都认为 MIT 值得持有"（不成立，这是博主自己的观点冒充群体）

**特别适合用共指"我们"的博主**：
- `xiao-lin-shuo`（招牌句"这对我们普通人意味着什么"）
- `the-plain-bagel`、`lenny`（教学型，"let's walk through..."、"我们一起来看"）
- `sg-stocks-investing`、`investmoolah`（散户社群感，"as Singapore investors, we..."）

**例外（博主代表团队/品牌）**：极少数情况下博主代表 newsletter 团队或公司，按档案指示。

**Step 3.5 扫描要点**：搜全文 "我们 / we / our / us"，对每一处问"这是博主本人，还是博主 + 读者？"。前者改成"我 / I"，后者保留。

#### 规则 2: 面向新加坡读者本地化

**用词**：直接使用本地缩写，不要展开翻译
- SGD / S$（不写"新元独立成段"）、HDB、CPF、SGX、STI、MAS、GIC、Temasek 等
- 美股代码保持原样（NVDA、TSLA），不译成中文
- REIT 不译"房地产投资信托"
- "Mapletree"、"CapitaLand"、"DBS"、"OCBC" 等本地公司名保持英文

**数字单位**：
- 英文输出：用 `USD 5 billion` / `SGD 200 million`，不用 `$5B` 或 `5 亿美元`
- 中文输出：用"5 亿美元"、"2 亿新元"，不混用 `5B USD`
- 给参照：`USD 5 billion (about 6% of revenue)` 比裸数字好

**语气**：专业但不端着。介于 Bloomberg 和朋友聊天之间
- 不用研报腔："综上所述"、"建议关注"、"值得期待"、"具备投资价值"
- 不用过度口语："bro"、"yo"、"绝绝子"、"yyds"、"破天荒"
- 可以引用本地语境（Shenton Way、Raffles Place、组屋、CPF SA），但不强求每篇都用

#### 规则 3: 降低阅读成本

- **短句优先**：一句话能说清的不写两句。一段不超过 4 行
- **首段抛观点/钩子**：不要"背景→分析→结论"的论文结构。博主写法是先给观点、给数据、给问题，再展开
- **首次出现的术语带提示**：第一次写 `VIX` 时带 `(the volatility index)`，之后直接用
- **数字给参照系**：`USD 5 billion` → `USD 5 billion, roughly half of Tesla's annual capex`
- **避免嵌套从句**：`The company, which had previously announced..., is now considering...` → 拆成两个短句

#### 规则 4: 去 AI 化标点与句式（重要）

LLM 输出最容易暴露身份的就是这些特征，逐项避开：

**少用 em dash（—）**：真实用户键盘上没快捷键，本能用别的替代
- ❌ `Tesla's Q3 was strong — deliveries hit 462k, a record.`
- ✅ `Tesla's Q3 was strong. Deliveries hit 462k, a record.`
- ✅ `Tesla's Q3 was strong: deliveries hit 462k, a record.`
- 替代：补充说明用冒号 `:` 或括号 `()`，转折用句号断句

**避开 LLM 套路词**：`Moreover`、`Furthermore`、`In conclusion`、`It's worth noting that`、`That said,`（开头时）、`Ultimately`

**避开 LLM 套路句式**：
- ❌ "not just X, but Y" / "not only... but also..."
- ❌ 每段都"主语 + 强动词"开头那种过度对称的排比
- ❌ "This isn't just about A; it's about B"

#### 规则 5: Emoji（按博主风格判断，克制使用）

**适合用 emoji** 的博主：
- `chris-zheng`（X 短贴，原本就常用 🚗 ⚡ 🇨🇳 🇺🇸）
- `meet-kevin`（高能量快评，🚨 📈 📉 适配）
- `sg-stocks-investing`、`investmoolah`（散户博客，可用 💰 🏠 🇸🇬）
- `trading-key`（情绪标记 🔥 📊）

**不适合用 emoji** 的博主：
- `lenny`（长文 Newsletter，最多偶尔一个 👇）
- `nick-timiraos`（WSJ 严肃记者，0 emoji）
- `lance-roberts`、`michael-kramer`（机构研报口吻，0 emoji）
- `the-plain-bagel`、`investnotbet`（理性科普，emoji 削弱可信度）

**通用约束**：
- 一篇文章 0-3 个 emoji，不是装饰
- 放在情绪/视觉锚点上（开头钩子、数字旁强化方向）
- 中文输出比英文输出更克制

#### 规则 6: 中文母语博主 → 英文输出时的翻译规范

适用于：`xiao-lin-shuo`、`meitou-meiguu`、以及 `chris-zheng` 当用户要求中文档案产英文文章的场景。

**保留博主语气**：小lin说的故事化口吻翻译后仍应是 narrative storytelling，不要变成教科书；美投讲美股的数据驱动感保留，不要软化成科普。

**用词对照（容易翻译失真的高频词）**：

| 中文原词 | ❌ 别用 | ✅ 用 |
|---------|--------|-------|
| 美元 / 人民币 | US Dollar / Chinese Yuan | USD / RMB |
| 互联网公司 | internet companies | tech companies |
| (造车)新势力 | new forces | EV startups / Chinese EV makers |
| 韭菜 | leeks / chives | retail investors |
| 割韭菜 | cutting the leeks | fleecing retail investors |
| 做多 / 做空 | go more / go empty | long / short |
| 复盘 | replay / re-board | recap / review |
| 画大饼 | drawing big pancakes | overpromising |
| 抄底 | copy the bottom | buy the dip |
| 解套 | untie | exit a losing position |
| 行情 | the line | the market / price action |
| A 股 | A-share market | China A-shares |
| 港股 | HK stocks | Hong Kong stocks / HKEX-listed |

**数字单位**：
- "1 亿美元" → `USD 100 million`，不写 `100M USD` 也不写 `0.1B`
- "万亿" → `trillion`，不写 `10000 亿`

**要避免的**：直译中式比喻、四字成语硬译、把"老铁/家人们"翻成 "iron buddies / family members"。找不到对应表达就重写一句，保留意思不保留形式。

#### 规则 7: 输出长度参考

按博主类型，不死板：
- **长文 Newsletter**（`lenny`、`fifth-person` 深度篇、`investnotbet`）：800-2000 字
- **博客中篇**（`sg-stocks-investing`、`investmoolah`、`lance-roberts`、`the-plain-bagel`）：500-1200 字
- **短贴 / Thread**（`chris-zheng`、`meet-kevin`、`michael-kramer` 日评）：150-500 字
- **Twitter Thread**（`nick-timiraos`、`rayner-teo` 短帖）：3-8 条编号 tweets

---

### Step 3.5: 事实核查与最终扫描（不可跳过）

文章写完后，逐一核查：

- **数字交叉验证**：每个数字与 Step 2 调研的原始数据比对
- **数量级与单位**：重点防"亿/万"混用、USD/SGD/HKD/RMB 混用、百分比计算错误
- **本地化用词扫一遍**：是否还有"新元"被写成"Singapore Dollar"全称这种冗余翻译
- **第一人称扫一遍**：搜全文 "我们 / we / our / us"，每一处问"这是博主本人冒充群体（→ 改成"我 / I"），还是博主 + 读者的共指（→ 保留）？"
- **AI 指纹扫一遍**：搜索 `—`、`Moreover`、`Furthermore`、`In conclusion`、`not just ... but ...`，命中就改

发现错误立即修正，再进入 Step 4。

---

### Step 4: 输出

直接在对话中输出。格式：

```
---
Blogger style: [博主名]
Topic: [主题]
---

[正文]
```

如果是 X / Twitter Thread 风格，用编号呈现：
```
1/
[Tweet 1]

2/
[Tweet 2]

...
```

---

## 新增博主

如果用户说"新增博主 XXX，主页是 YYY"：

1. 用 WebSearch / WebFetch 研究该博主 5-10 篇代表作，识别语气、结构、高频句式、要避免的写法
2. 在 `{baseDir}/bloggers/` 创建新档案，参考现有文件（推荐参考 `lenny-rachitsky.md` 或 `chris-zheng.md` 的结构）
3. 更新 `{baseDir}/bloggers/INDEX.md` 的索引表和自动匹配规则
4. 同步更新本 SKILL.md 顶部"已收录博主"表（保持两处一致）
5. 告知用户：博主已收录，ID 为 `{id}`

---

## 参考文件

- `{baseDir}/bloggers/INDEX.md` — 完整博主索引 + 自动匹配规则
- `{baseDir}/bloggers/{id}.md` — 各博主的风格档案（共 16 个）
