# Reddit 评论自动发送 SOP v2

## 目标

用 OpenClaw 搭一套 **可控、低频、风控优先** 的 Reddit 评论自动发送系统，完成：

1. 发现目标线程
2. 匹配回复策略
3. 生成候选评论
4. 自动筛选可执行项
5. 自动发送低风险评论
6. 回填互动结果
7. 日报复盘

> 这份 SOP 针对的是 **Reddit 评论自动发送**，不是主帖自动发布。

---

## 当前定位

当前阶段主线是：

- **评论优先**
- **低风险、低频率、强相关回复**
- **主帖暂缓**，除非账号信任度和社区适配度已经明显提高
- **只自动发送低风险评论**
- **中高风险项保留人工审核**

原因：

- Reddit 主帖更容易被风控或删除
- 当前账号信任仍偏低
- 评论更适合冷启动阶段积累正常行为、提取真实用户语言
- 自动发送必须优先服从风控，而不是追求量

---

## 执行器定义

### 长期自动化底座
自动发送必须基于：

- **独立浏览器实例**
- **持久登录态**
- **稳定账号会话**
- **表驱动执行**
- **风控优先**

### 不作为长期底座的方案
- Relay / attach-tab
- 临时人工接管浏览器
- 无持久登录态的临时会话

### 执行前必须满足
- Reddit 账号已登录
- 登录态稳定
- 浏览器可正常打开 Reddit 线程页
- 评论框可正常加载

如果以上任一条件不满足：
- 当天**暂停自动发送**
- 只做发现、排队、回填、复盘

---

## 数据结构（Excel 对应表）

### 1. `communities`
用途：目标 subreddit / 社群池

### 2. `posting_rules`
用途：平台与社区动作边界

### 3. `reply_playbooks`
用途：按 subreddit 和 `thread_type` 分类的回复模板库

### 4. `reply_queue`
用途：待执行评论任务队列

### 5. `engagements`
用途：记录评论发出后的真实反馈与跟进价值

### 6. `daily_reports`
用途：每日复盘

### 7. `dashboard`
用途：关键指标总览

---

## 每日执行总流程

每天按这个顺序跑：

1. 读取 `posting_rules`
2. 读取 `reply_playbooks`
3. 读取 `communities`
4. 发现新线程
5. 写入 `reply_queue`
6. 匹配候选回复
7. 做自动发送准入筛选
8. 自动发送低风险评论
9. 回填 `engagements`
10. 更新 `daily_reports`
11. 更新 `dashboard`

---

## 每张表在流程中的职责

### `communities`
- 决定去哪些 subreddit 找线程
- 优先 A/B 类、状态正常的社区

### `posting_rules`
- 决定能不能做 comment / post
- 决定哪些线程不能碰
- 决定是否允许 link drop
- 决定日频控制

### `reply_playbooks`
- 根据 `community_name + thread_type` 匹配模板
- 生成候选评论的基础稿

### `reply_queue`
- 存待执行线程
- 存候选回复
- 存优先级、风险、状态
- 是自动发送执行层的主战场

### `engagements`
- 记录发出后发生了什么
- 记录用户信号、是否值得继续跟
- 是反馈资产表

### `daily_reports`
- 汇总当日发现、执行、结果、风险和下一步动作

### `dashboard`
- 展示今天卡在哪
- 展示当前下一步重点与 blocker

---

## Step 0：系统状态检查

执行前先检查：

- 最近是否出现过 `auto_removed` / `mod_removed`
- 当前账号是否有异常风控迹象
- 昨天评论量是否过高
- 哪些社区当前仍适合评论
- 登录态是否稳定
- 浏览器执行器是否可用

如果删除率异常升高：

- 当天自动降频
- 优先只做发现和排队，不做执行

如果登录态失效或浏览器异常：

- 当天暂停自动发送
- 只保留发现、排队、回填、复盘

---

## Step 1：发现新线程

### 输入
- `communities`
- `keywords`

### 目标
找最近、还活跃、与当前主题高相关的线程。

### 优先主题
- everyday use
- weight
- compartments
- organization
- style vs practicality
- travel carry
- small bag frustration

### 线程新鲜度与可执行性规则
优先：
- 最近发布
- 最近仍有新评论或互动
- 主题明确
- 可自然接话
- 线程未锁定
- 线程未归档

### 过滤掉
- authenticity / legit check
- sale / deal
- 引战
- 太旧的沉帖
- 纯晒图
- 纯买卖
- 已锁帖
- 已归档
- 评论区明显失控或删除严重

### 输出
写入 `reply_queue`：

- `date`
- `platform`
- `community_name`
- `thread_url`
- `thread_title`
- `thread_type`
- `target_comment_summary`
- `priority`
- `risk_level`
- `queue_status = queued`

---

## Step 2：匹配回复模板

### 输入
- `reply_queue`
- `reply_playbooks`
- `posting_rules`

### 目标
为每条候选线程匹配最合适的回复模板。

### 做法
1. 识别 `thread_type`
2. 在对应 subreddit 的 playbook 里找 `approved` 模板
3. 生成 `candidate_reply`
4. 判断风险

### 输出
更新 `reply_queue`：

- `reply_goal`
- `playbook_ref`
- `candidate_reply`
- `risk_level`
- `queue_status`

### 推荐状态（按当前表先用）
- 低风险 + 高匹配：`approved`
- 不确定：`reviewing`
- 待初筛：`queued`

> 当前 `reply_queue.queue_status` 实际已在用的主要是：`queued / reviewing / approved`。
> `skipped`、`scheduled`、`needs_retry` 等状态可作为后续自动发送阶段再启用的扩展状态。

---

## Step 3：自动发送准入筛选

只有满足以下条件才允许进入自动发送：

- `queue_status = approved`
- `risk_level = low`
- 所在 subreddit 在当前允许列表内
- 今日评论总量未超标
- 同 subreddit 今日未过量
- 近 24 小时未出现删除异常
- `candidate_reply` 不含链接
- `candidate_reply` 不含品牌推荐口吻
- `candidate_reply` 不含强 CTA
- 当前线程仍然可回复
- 当前评论链仍然贴题
- 与近 7 天已发评论重复度不高

### 必须拦截的情况
以下任一成立，就不能自动发送：

- 支持型 / 高敏感社区
- 争议贴、引战贴、规则敏感贴
- 线程已锁或接近失效
- 评论链语气明显不匹配
- 候选评论过长、过模板化、过像调研
- 同一 thread 已经发过
- 同一 subreddit 当天已达到上限

不满足自动发送条件时，当前优先改成：
- `reviewing`

如果后续要升级自动发送执行层，再逐步启用：
- `scheduled`
- `skipped`
- `needs_retry`

---

## Step 4：频控与熔断

### 早期阶段默认频控
- 每天只发 **3–5 条评论**
- 同一 subreddit 每天最多 **1–2 条**
- 两条评论之间保留明显间隔
- 同一 thread 永远只发一次
- 当前阶段 **评论优先，不发主帖**

### 熔断规则
以下任一触发，就暂停当天自动发送：

- 连续 2 条出现 `auto_removed` / `mod_removed`
- 当天删除率明显升高
- 出现 rate limit
- 登录态失效
- 浏览器执行失败率异常升高

触发熔断后：
- 当天只做发现、排队、回填
- 所有待发项停止自动执行
- `dashboard` 和 `daily_reports` 记录 blocker

---

## Step 5：执行评论

### 原则
`candidate_reply` 不是绝对最终原文，执行前允许做一次轻微自动微调。

### 执行前检查 5 项
1. 是否贴当前评论链
2. 是否太长
3. 是否重复楼里原话
4. 是否像真人
5. 是否符合该 subreddit 语气

### 允许的自动微调规则
- 同义词替换
- 缩短过长句子
- 去掉像调研/营销的语气
- 避免品牌推荐口吻
- 避免一模一样模板跨区重复

### 不允许的自动动作
- 自动加链接
- 自动加品牌/店铺名
- 自动变成推荐贴
- 自动把原本低风险回复改成导流回复

### 执行后更新 `reply_queue`
成功时，当前最少要回填：
- `posted_at`
- `result_status = pending`
- `engagement_signal = none`

如果你决定把自动发送状态机真正落到表里，再启用：
- `queue_status = posted`
- `queue_status = needs_retry`
- `queue_status = removed`

失败时，当前建议至少在 `notes` 里记清失败原因；
如果后续启用扩展状态，再写：
- `result_status = auto_removed / mod_removed / post_failed`

---

## Step 6：第一次回看（30–60 分钟）

检查：

- 评论是否还在
- 是否被自动删除
- 是否有人回复
- 是否有 upvote

### 更新
#### `reply_queue`
- `result_status`
- `engagement_signal`

当前表先只要求把结果写实：
- 还在 / 无反馈 → `result_status = pending` 或后续改成 `no_response`
- 有回复 / 有明显互动 → 更新 `engagement_signal`
- 被删 → 更新为 `auto_removed` 或 `mod_removed`

`check_1_done` / `check_2_pending` 这类状态，建议作为后续自动发送阶段的扩展状态，不要求当前立即启用。

#### `engagements`
新增一条记录，至少写：
- 社区
- 线程链接
- 评论主题
- 当前有没有反馈
- 要不要继续跟

---

## Step 7：第二次回看（12–24 小时）

检查：

- 有无新增回复
- 回复里是否出现明确痛点/偏好
- 是否值得继续跟进

### 更新 `engagements`
- `user_signal`
- `followup_needed`
- `followup_type`
- `lead_potential`
- `action_status`
- `notes`

### 更新 `reply_queue`
当前表先以 `result_status + engagement_signal + notes` 为主：
- 生命周期结束：在 `notes` 标明已完成观察
- 如果被删：更新 `result_status`
- 如果后续仍需观察：保留结果状态并写清原因

`completed` / `removed` 这类更细的 `queue_status`，建议作为后续自动发送阶段的扩展状态。

---

## Step 8：反馈反哺

### 反哺到 `keywords`
把用户真实语言沉淀进去，例如：
- lightweight
- easy access
- bad pockets
- awkward strap
- too heavy

### 反哺到 `content_drafts`
把高互动问题转成下一轮讨论题/回复题。

### 反哺到 landing page
将高频用户原话提炼成：
- pain points
- hero copy
- bullets
- CTA 文案

---

## Step 9：日报更新

### 更新 `daily_reports`
记录：
- 今日新线程数
- 今日入队数
- 今日评论执行数
- 今日删除数
- 高信号互动数
- 最有效 `thread_type`
- 下一步动作

### 更新 `dashboard`
记录：
- comments queued
- comments posted
- high-signal followups
- current blocker
- next priority

---

## 状态机定义

### `reply_queue.queue_status`（当前已使用）
- queued
- reviewing
- approved

### `reply_queue.queue_status`（预留扩展）
- scheduled
- posted
- completed
- skipped
- removed
- needs_retry
- cooldown

### `reply_queue.result_status`（当前已使用 / 建议逐步启用）
- pending
- auto_removed
- mod_removed
- no_response
- got_replies
- got_upvotes
- high_signal
- post_failed

### `reply_queue.engagement_signal`
- none
- low
- medium
- high

### `engagements.user_signal`
- question
- interest
- objection
- pain_point
- comparison
- purchase_intent
- casual

### `engagements.followup_needed`
- yes
- no

### `engagements.followup_type`
- reply
- dm
- content_idea
- product_feedback
- landing_page_input
- community_research

### `engagements.lead_potential`
- low
- medium
- high

### `engagements.action_status`
- new
- pending
- completed
- ignored

---

## 评论风格原则

评论要像：
- 真实使用者补一句
- 共鸣式经验分享
- 轻微补充，不抢话题

评论不要像：
- 市场调研
- 产品经理访谈
- 卖货引流
- 品牌推荐贴
- 模板机复读

---

## 异常处理

### 1. 登录态失效
- 暂停自动发送
- 标记当天 blocker
- 只保留发现、排队、回填

### 2. 页面结构异常 / 评论框不存在
- 该条任务标记 `needs_retry`
- 不强行重复提交
- 记录异常原因

### 3. Rate limit
- 当天立刻降频
- 必要时进入 `cooldown`
- 不再继续自动发下一条

### 4. 评论提交失败
- 标记 `post_failed`
- 进入 `needs_retry`
- 先查线程状态和登录态，再决定是否重试

### 5. 删除异常升高
- 触发熔断
- 暂停当天执行
- 只做发现、排队、复盘

---

## 当前最重要的现实判断

这套 SOP 目前适合：

- **低风险评论自动发送**
- **中高风险项人工审核**
- **自动发现 → 自动排队 → 自动发送低风险评论 → 自动回填**

当前还不适合：

- 大规模自动主帖
- 高频全自动评论
- 无风控的无人值守强执行

原因：
- Reddit 风控严格
- 当前账号信任度仍低
- 仍处于冷启动验证期

---

## 最终架构（一句话）

> `communities` 决定去哪，`posting_rules` 决定能不能做，`reply_playbooks` 决定怎么回，`reply_queue` 决定今天回哪条，自动发送准入条件决定哪些能直接发，`engagements` 记录回完后的价值，`daily_reports` / `dashboard` 负责复盘。

---

## 后续升级方向

### Phase 1（当前）
- 自动发现
- 自动排队
- 自动生成候选回复
- 只自动发送低风险评论

### Phase 2
- 自动回填互动结果
- 自动熔断与恢复
- 更细的相似度去重与语气控制

### Phase 3
- 持久登录态的独立浏览器执行器
- 自动发现 → 自动回复 → 自动回填 → 自动日报

---

## 注意

Relay 只适合作为调试或临时接管方案，不适合作为长期自动化底座。

长期自动化底座应为：

- **独立浏览器实例**
- **持久登录态**
- **表驱动执行**
- **风控优先**
