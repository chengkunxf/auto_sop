# Reddit 自动看回复 / 跟进回复 Runbook

## 目标

这份 runbook 只负责一件事：

**当我们已经发出一条 Reddit 评论后，如何检查有没有人回复我们，并把值得继续跟进的回复真正处理掉。**

它不定义策略边界；策略边界看：
- `reddit-reply-followup-sop.md`

这份文档只定义执行流程。

---

## 输入

主要输入：
- `reply_queue`
- `engagements`

辅助输入：
- `posting_rules`
- `daily_reports`
- `dashboard`

---

## 输出

主要回填：
- `engagements`
- 必要时更新 `reply_queue`
- `daily_reports`
- `dashboard`

---

## 执行总流程

1. 找到已发送的评论
2. 检查是否有新回复
3. 读取新回复内容
4. 把新回复分成 A / B / C 类
5. 对 A 类生成 follow-up candidate
6. 做风险检查
7. 对 low risk 项自动继续回复
8. 回填 `engagements`
9. 更新 `daily_reports` / `dashboard`

---

## Step 1：找到已发送评论

从 `reply_queue` 里优先读取满足这些条件的记录：

- `platform = reddit`
- `posted_at` 不为空
- `result_status = pending / got_replies / got_upvotes / high_signal`
- 最近 24–72 小时内发出的评论优先

### 优先级建议
优先看：
1. 最近刚发出的评论
2. `engagement_signal` 非 `none` 的评论
3. 已经观察到有人互动的评论

---

## Step 2：检查有没有新回复

打开对应 thread 后，检查：

- 我们自己的评论是否仍存在
- 有没有人直接回复我们
- 是否出现新增互动

### 三种情况
#### 情况 A：没有新回复
- 记录“暂无 follow-up”
- 不继续动作

#### 情况 B：有回复，但价值一般
- 更新 `engagements`
- 通常不继续回

#### 情况 C：有回复，而且值得继续聊
- 进入 follow-up 候选流程

---

## Step 3：读取新回复内容

对每条新回复，至少记录：

- 对方大意在说什么
- 语气如何
- 有没有新信息
- 有没有追问
- 有没有真实痛点 / 偏好 / 比较逻辑

### 建议记录格式
先写成一句短摘要，例如：
- `User asked whether weight or strap comfort mattered more in daily use.`
- `User agreed and added that outside pockets are the main reason a bag feels usable.`
- `User replied casually with no real follow-up angle.`

---

## Step 4：A / B / C 分类

按 `reddit-reply-followup-sop.md` 的规则分：

### A 类：值得继续回
例如：
- 明确追问
- 明确痛点
- 真实比较逻辑
- 高价值使用经验

### B 类：只记录，不回
例如：
- 轻认同
- 普通附和
- 有互动但没延展空间

### C 类：不回
例如：
- 敌对
- 引战
- 跑题
- 高风险

### 当前执行原则
- A 类：进入 follow-up candidate
- B 类：只回填，不自动回复
- C 类：只回填，不自动回复

---

## Step 5：对 A 类生成 follow-up candidate

### 目标
跟着对方的话自然接一句，而不是重新开题。

### 推荐 follow-up 类型
- 共鸣式补充
- 轻追问
- 轻澄清
- 补一句使用经验
- 自然收尾式回应

### 生成原则
follow-up candidate 必须满足：
- 简短
- 贴题
- 像真人
- 不像市场调研
- 不带链接
- 不带品牌推荐
- 不带 CTA

### 示例方向
#### 对方补充痛点
- 顺着痛点补一句自己的经验

#### 对方追问
- 回答问题，但只答到自然为止

#### 对方比较选择逻辑
- 补一句你自己的取舍感受

---

## Step 6：风险检查

生成 follow-up candidate 后，再做一次执行前检查。

### 必须通过
- 当前回复属于 A 类
- 语气自然
- 没有品牌/链接/CTA
- 当前社区仍允许这种互动
- 没有引战迹象
- 当前对话轮数不深

### 任何一项不通过
- 不自动回复
- 只记录到 `engagements`
- 必要时标记人工看

### 当前策略
- 只自动执行 `low risk` follow-up
- `medium / high risk` 一律不自动发

---

## Step 7：执行 follow-up

### 执行动作
- 打开对应 thread
- 定位到别人回复我们的那一层
- 点击 reply
- 填入 follow-up candidate
- 提交
- 检查评论是否出现

### 提交后立即检查
- 评论是否可见
- 是否报错
- 是否被吞
- 是否有 rate limit

如果失败：
- 不继续追发
- 记录失败原因

---

## Step 8：回填 `engagements`

这是这条线里最重要的回填表。

每次发现新回复后，至少要补：
- 对方回复摘要
- `user_signal`
- `followup_needed`
- `followup_type`
- `lead_potential`
- `action_status`
- `notes`

### 建议写法
#### A 类并已跟进
- `followup_needed = yes`
- `action_status = pending` 或 `completed`
- `notes` 写：对方说了什么、我们怎么跟进的

#### B 类只记录
- `followup_needed = no`
- `action_status = completed`
- `notes` 写：有互动，但不值得继续回

#### C 类不回
- `followup_needed = no`
- `action_status = ignored`
- `notes` 写：风险/不回原因

---

## Step 9：必要时更新 `reply_queue`

当前阶段不一定要给 follow-up 单独起一整套状态机。

但至少可以在原记录或对应 notes 里补：
- 有无新回复
- 是否已跟进
- follow-up 是否成功
- 是否需要再次观察

如果后续你要把 follow-up 也正式流程化，可以再引入单独字段或状态。

---

## Step 10：更新日报和总览

### `daily_reports`
记录：
- 今日发现了多少条新回复
- 其中多少条值得继续回
- 实际跟进了多少条
- 有没有高信号表达
- 有没有异常

### `dashboard`
记录：
- follow-up pending 数
- high-signal replies 数
- next priority
- blocker

---

## 推荐执行节奏

### 检查频率
优先在这些时间点看：
- 发出后 30–60 分钟
- 发出后 12–24 小时
- 之后只看高价值线程

### 跟进节奏
- 不要求秒回
- 低频、自然
- 每条线程最多只自动跟进 1 轮

---

## 异常处理

### 1. 我们的原评论被删
- 不继续 follow-up
- 更新 `result_status`
- 记录异常

### 2. 对方回复敌对/引战
- 不自动回
- 记录为 C 类

### 3. 页面结构异常
- 暂停该条 follow-up
- 记录失败原因

### 4. 提交 follow-up 失败
- 不强重试
- 记录失败原因

### 5. 社区风险升高
- 暂停该 subreddit 的自动 follow-up
- 只记录不继续发

---

## 最小成功标准

一轮自动看回复 / 跟进回复，只有同时满足以下条件，才算执行合格：

1. 成功检查到至少一条已发评论的后续状态
2. 成功把新回复分成 A / B / C 类
3. 对 A 类生成了 follow-up candidate
4. 只对 low risk 项做自动跟进
5. `engagements` 已回填

---

## 一句话版本

> 找到我们已经发出的 Reddit 评论，检查有没有人回复；把新回复分成 A / B / C 类；对 A 类生成自然、低风险的 follow-up candidate，并只在 low risk 条件下自动继续回一层，最后把结果回填到 `engagements`、必要时补 `reply_queue`，并更新 `daily_reports` 与 `dashboard`。
