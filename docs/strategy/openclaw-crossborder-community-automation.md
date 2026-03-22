# OpenClaw 跨境社群获客自动化项目说明（交接文档）

## 1. 项目背景

当前希望用 OpenClaw 帮助一个跨境独立站做**社群发现 → 社群筛选 → 内容切入 → 导流承接 → 私域沉淀**这条链路。

现有情况：

- 独立站已存在：`https://dysonclean.net/`
- 当前站内实际售卖产品以**女包 / 单肩包 / crossbody bag / 日常搭配包**为主
- 处于**冷启动阶段**
- 目标是通过 Facebook Groups、Reddit、Discord 等社区找到精准用户，再通过内容和轻导流把用户引到独立站、waitlist、邮箱或 WhatsApp 等私域承接点

## 2. 一个重要现实判断

当前网站的**域名/产品/关键词**存在明显错位：

- 域名：`dysonclean.net`
- 实际产品：女包
- 用户曾考虑的部分关键词：Dyson 相关关键词（如 `dyson clearance sale`、`dyson 90% off` 等）

结论：

- **包类关键词**可以作为主战场，适合直接做社群导流和内容运营
- **Dyson 相关关键词**如果要做，只能做成**信息型内容流量入口**（真假判断、骗局识别、翻新是否值得买等），**不适合**直接把 Dyson 交易意图流量导向当前卖包的站点承接页
- 若继续做 Dyson 高交易意图词，会带来严重的信任、转化、品牌和平台风险

因此，当前自动化项目的**主线**应聚焦于：

> 基于当前网站实际产品（包类）做关键词库、社群发现、社群评分、内容生成和后续导流。

## 3. 当前主目标

用 OpenClaw 实现一套稳健版自动化系统，日常输出：

1. 关键词更新
2. 社群发现
3. 社群评分
4. 内容草稿生成
5. 日报复盘

人工负责：

1. 确认进哪些群
2. 审核发什么
3. 决定什么时候导流
4. 看哪些评论值得继续跟

## 4. 为什么后续改由 Mac 上的 OpenClaw 继续做

当前 Linux 侧 OpenClaw 存在这些限制：

- 虽然已经打通了 headless Chrome，可浏览普通网页
- 但对需要用户登录态、复杂交互的网站（如 Google Sheets、部分 X/Twitter 页面、部分受限页面）仍不够自然
- 无法直接稳定编辑用户的 Google Sheet
- 无法直接复用用户本机浏览器会话

Mac 上的 OpenClaw 具备天然优势：

- 有本机浏览器
- 有用户登录态
- 可以直接操作 Google Sheets
- 后续更适合接手：
  - 社群搜索与浏览
  - Google Sheet 自动写入
  - Facebook/Reddit/Discord 的半自动操作
  - 需要登录态的操作

因此，这份文档的目的就是：

> 把当前项目的背景、边界、策略、表结构和 SOP 讲清楚，方便 Mac 上的 OpenClaw 无缝接手。

---

# 5. 项目范围（Scope）

## 主范围（当前要做）

围绕包类产品，搭建一套社区获客系统：

- 关键词库建设
- 社群搜索词库
- 社群池管理
- 社群评分
- 内容切入草稿
- 导流承接页建议
- 日报 / 周报

## 暂不做 / 不建议做

- 大规模自动加群
- 大规模自动私信
- 大规模自动发帖
- 批量账号矩阵发广告
- Dyson 高交易意图词直接导流到当前包站
- 任何明显违反平台规则、隐私规则或反垃圾规则的自动化骚扰行为

---

# 6. 产品与受众定位（当前阶段建议）

## 产品侧

当前站内可见产品倾向：

- 女包
- 单肩包
- crossbody bag
- camera bag 风格
- 轻量、日常搭配、通勤/出行场景

## 受众侧（建议优先测试）

1. 女性日常穿搭人群
2. 通勤场景用户
3. 轻旅行 / 城市出行用户
4. 喜欢“好看 + 实用”的包类用户
5. 收纳 / EDC / 日常携带需求人群

---

# 7. 关键词策略

## 7.1 包类主关键词池（主战场）

### 核心交易词

- crossbody bag for women
- women shoulder bag
- minimalist crossbody bag
- camera bag for women
- lightweight shoulder bag
- women underarm bag
- everyday crossbody bag
- small shoulder bag for women
- fashion crossbody bag
- casual shoulder bag for women

### 场景关键词

- best bag for daily commute
- women commute bag
- best everyday bag for women
- bag for office and casual use
- travel crossbody bag women
- lightweight everyday bag
- best small bag for daily use
- one bag for work and weekend

### 对比 / 决策关键词

- crossbody vs shoulder bag
- best crossbody bag for everyday use
- how to choose a women shoulder bag
- what size crossbody bag is best
- small bag vs tote for daily use

### 痛点词

- bag too heavy for daily use
- best lightweight bag for women
- shoulder bag hurts shoulder
- bag with good compartments women
- stylish but practical bag women

## 7.2 Dyson 关键词处理策略

如果未来仍要利用 Dyson 相关流量，只能用于**信息型内容页**：

可保留的方向：

- cheap dyson vacuum legit
- dyson warehouse sale real or fake
- dyson refurbished worth it
- dyson under $100 real
- best dyson deals reddit
- dyson outlet legit
- how to tell if dyson sale is fake

但这些词**不能直接承接到包类产品页**。

---

# 8. 社群方向

## 8.1 包类主线社群方向

### Facebook Groups

适合搜索的方向：

- women fashion accessories
- handbag lovers
- shoulder bag lovers
- minimalist fashion women
- capsule wardrobe women
- office style women
- travel accessories women
- everyday carry women

### Reddit

适合观察和参与的方向：

- 女性时尚 / 穿搭建议类 subreddit
- handbag / bags 相关 subreddit
- travel gear 相关 subreddit
- everyday carry / organization 相关讨论区

### Discord

作为后续补充：

- fashion / lifestyle / travel gear / EDC 方向服务器

## 8.2 不建议优先投入的社群方向

- 纯卖家群
- 泛 coupon/deal 群
- 明显广告泛滥的垃圾群
- 与产品实际场景弱相关的泛流量群

---

# 9. 当前建议的自动化模式：稳健版

## 自动后台（由 OpenClaw 负责）

每天自动生成：

1. 新关键词
2. 新社群候选
3. 社群评分与分层
4. 社群内容草稿
5. 日报

## 人工前台（由人负责）

每天处理：

1. 看哪些群值得加入
2. 审核今天发什么
3. 决定是否导流
4. 看哪些评论值得继续跟

---

# 10. Google Sheet 结构（已规划）

目前计划使用 5 张表：

1. `keywords`
2. `communities`
3. `content_drafts`
4. `engagements`
5. `daily_reports`

## 10.1 keywords 字段

- keyword
- category
- intent
- priority
- source
- status
- last_checked
- notes

## 10.2 communities 字段

- platform
- community_name
- url
- source_keyword
- theme
- audience_guess
- member_count
- activity_level
- content_friendliness
- commercial_potential
- risk_level
- overall_score
- tier
- status
- notes
- last_checked

## 10.3 content_drafts 字段

- date
- community_name
- platform
- content_angle
- draft_title
- draft_body
- cta_type
- risk_note
- review_status
- publish_status
- post_link
- notes

## 10.4 engagements 字段

- date
- community_name
- platform
- post_link
- comment_summary
- user_signal
- followup_needed
- followup_type
- lead_potential
- action_status
- notes

## 10.5 daily_reports 字段

- date
- new_keywords_count
- new_communities_count
- a_tier_count
- b_tier_count
- drafts_generated
- approved_posts
- posted_count
- high_signal_comments
- top_insights
- next_actions

---

# 11. 每日运行 SOP（V1）

## Step 1：关键词更新

目标：

- 每天扩 10–20 个新词
- 优先扩：场景词、痛点词、社群搜索词

输出写入：`keywords`

## Step 2：社群发现

目标：

- 每天新增 10–20 个候选社群/方向

搜索方式：

- Facebook Groups 搜索
- Reddit 搜索
- 必要时借助 Google `site:` 搜索

输出写入：`communities`

## Step 3：社群评分

评分维度：

- 人群匹配度
- 活跃度
- 内容友好度
- 商业潜力
- 风险等级

结果分：

- A / B / C

输出写入：`communities`

## Step 4：内容草稿生成

针对 A/B 类社群生成草稿。

内容类型：

- discussion
- comparison
- tips
- feedback_request

输出写入：`content_drafts`

## Step 5：日报复盘

每天总结：

- 新增关键词数
- 新增社群数
- A/B 类社群数
- 草稿数
- 已发数
- 高信号评论数
- 今日洞察
- 明日动作

输出写入：`daily_reports`

---

# 12. 内容策略（冷启动阶段）

## 当前推荐内容类型

### 讨论型
- What matters more in a daily bag: weight, compartments, or style?
- Crossbody vs shoulder bag — which one do you actually use more?

### 对比型
- Tote vs crossbody for commuting — which is more practical?
- Lightweight vs structured bags — what do you prefer for daily use?

### 痛点型
- What is the most annoying thing about daily bags?
- What makes a bag look stylish but still practical?

### 技巧型
- What actually matters in a daily-use women’s bag?
- How do you organize essentials in a small bag?

## 当前不建议

- 一上来发产品图
- 一上来发购买链接
- 一上来发群邀请
- 一上来发折扣信息

---

# 13. 导流策略（冷启动阶段）

## 优先级顺序

1. 社群内容互动
2. 独立站场景页 / 指南页
3. waitlist / 邮箱 / WhatsApp 留资
4. 后续再考虑 VIP 群 / Telegram 群

## 暂不建议

- 直接把人拉去 Telegram 群
- 直接扔产品页
- 直接上强 CTA

---

# 14. 为什么不建议一开始全自动写 Google Sheet

虽然最终长期目标是自动写表，但当前建议先确认：

1. 字段是否稳定
2. 流程是否合理
3. 这些数据是否真有价值

不过由于当前项目要交给 Mac 上的 OpenClaw 接手，而 Mac 具备浏览器和用户登录态，因此后续确实可以考虑：

- 直接编辑 Google Sheets
- 或通过 Apps Script / API 自动写入

但在项目接手初期，**优先先让 Mac OpenClaw 能稳定读表、填表、操作浏览器**。

---

# 15. Mac 上的 OpenClaw 接手建议

Mac 侧 OpenClaw 接手后，建议优先完成以下任务：

## 第一阶段

1. 打开现有 Google Sheet
2. 校验 5 张表结构是否正确
3. 把示例数据和下拉规则补齐
4. 读取网站实际产品页，进一步细化关键词库

## 第二阶段

1. 真实搜索 Facebook Groups / Reddit
2. 把第一批真实社群填入 `communities`
3. 给社群打分
4. 生成第一批 `content_drafts`

## 第三阶段

1. 建立每日跑批节奏
2. 输出日报
3. 跟踪真实发帖和评论
4. 写入 `engagements`

---

# 16. Mac OpenClaw 的优先任务清单

请 Mac 上的 OpenClaw 按以下顺序继续：

## Task 1
读取当前 Google Sheet，确认以下 sheet 已存在：

- keywords
- communities
- content_drafts
- engagements
- daily_reports

## Task 2
为这 5 张表补充：

- 表头
- 下拉选项
- 筛选器
- 条件格式（如果方便）

## Task 3
基于当前站点 `https://dysonclean.net/` 的实际产品内容，重新整理：

- 核心交易词
- 场景词
- 痛点词
- 社群搜索词

写入 `keywords`

## Task 4
开始真实搜索：

- 30 个 Facebook Groups 候选
- 20 个 Reddit 方向

写入 `communities`

## Task 5
对第一批社群打分并分 A/B/C

## Task 6
对 A/B 类社群生成第一批内容草稿，写入 `content_drafts`

## Task 7
生成第一份日报，写入 `daily_reports`

---

# 17. 最终目标（不是短期噪音，而是可复用系统）

这个项目最终不是要“刷一批群”，而是要沉淀出：

1. 可复用关键词体系
2. 可复用社群池
3. 可复用评分机制
4. 可复用内容模板
5. 可复用日报 / 周报机制
6. 可逐步升级为真正自动化写表、自动调度和半自动运营的系统

---

# 18. 一句话交接总结

> 当前项目应从“包类产品的社区获客系统”出发，使用 OpenClaw 做关键词更新、社群发现、社群评分、内容草稿生成和日报复盘；Google Sheet 作为中台管理面板；Mac 上的 OpenClaw 因具备浏览器和登录态，更适合接手表格编辑、社区搜索和后续半自动运营。

