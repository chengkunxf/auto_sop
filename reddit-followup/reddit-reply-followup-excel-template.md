# Reddit 自动看回复 / 跟进回复 · Excel 回填模板

## 目标

这份模板专门解决：

**当别人回复了我们，或者我们又继续回了一层之后，Excel 应该怎么回填。**

它服务的对象是：
- `engagements`
- 必要时的 `reply_queue`
- `daily_reports`
- `dashboard`

---

## 一、优先回填顺序

follow-up 这条线里，回填优先顺序建议是：

1. `engagements`
2. `reply_queue`
3. `daily_reports`
4. `dashboard`

如果时间紧，至少先把：
- `engagements`
- `reply_queue`

写完整。

---

## 二、`engagements` 回填模板

这是 follow-up 线里最核心的表。

每次发现别人回复了我们，至少要补一条或更新一条 `engagements`。

### 标准字段模板
- `date`
- `community_name`
- `platform`
- `post_link`
- `comment_summary`
- `user_signal`
- `followup_needed`
- `followup_type`
- `lead_potential`
- `action_status`
- `owner`
- `notes`

---

## 三、按场景回填 `engagements`

### 场景 A：有人回复，但只是轻认同，不继续回

**推荐写法：**
- `user_signal = casual`
- `followup_needed = no`
- `followup_type = community_research`
- `lead_potential = low`
- `action_status = completed`
- `notes = User replied with light agreement only; recorded but no follow-up needed.`

---

### 场景 B：有人回复，并补充了真实痛点

**推荐写法：**
- `user_signal = pain_point`
- `followup_needed = yes`
- `followup_type = content_idea` 或 `product_feedback`
- `lead_potential = medium`
- `action_status = pending`
- `notes = User added a real pain point around weight / pockets / organization; follow-up candidate generated.`

如果已经真的继续回了一层：
- `action_status = completed`
- `notes` 改成：`Follow-up posted successfully after user added pain-point detail.`

---

### 场景 C：有人回复，并在做选择比较

**推荐写法：**
- `user_signal = comparison`
- `followup_needed = yes`
- `followup_type = landing_page_input` 或 `content_idea`
- `lead_potential = high`
- `action_status = pending` 或 `completed`
- `notes = User compared options based on comfort / capacity / style; strong reusable comparison language.`

---

### 场景 D：有人明确追问我们

**推荐写法：**
- `user_signal = question`
- `followup_needed = yes`
- `followup_type = reply`
- `lead_potential = medium`
- `action_status = pending`
- `notes = User directly asked a follow-up question; safe candidate reply should answer briefly and naturally.`

如果已成功回复：
- `action_status = completed`
- `notes = User asked a direct follow-up; reply posted successfully.`

---

### 场景 E：敌对 / 风险高 / 不回

**推荐写法：**
- `user_signal = objection` 或 `casual`
- `followup_needed = no`
- `followup_type = community_research`
- `lead_potential = low`
- `action_status = ignored`
- `notes = Reply classified as high-risk / hostile / not worth continuing.`

---

### 场景 F：我们继续回了一层，但失败

**推荐写法：**
- `user_signal` 保留原本分类
- `followup_needed = yes`
- `followup_type = reply`
- `lead_potential` 按原本判断
- `action_status = pending`
- `notes = Follow-up candidate prepared, but posting failed due to browser / login / rate-limit issue.`

---

## 四、`reply_queue` 回填模板

当前阶段 follow-up 不一定单独起完整状态机，所以 `reply_queue` 回填建议尽量轻量。

### 最少更新方式
优先在原记录的 `notes` 里补清楚：
- 有人回复了没有
- 这条回复属于 A / B / C 哪类
- 是否生成 follow-up candidate
- 是否已经继续回复
- follow-up 是否成功

---

### 场景 A：发现新回复，但不继续回

**`reply_queue.notes` 示例：**
- `New reply observed; classified as B-type; recorded only, no follow-up sent.`

---

### 场景 B：发现新回复，并生成了 follow-up candidate

**`reply_queue.notes` 示例：**
- `New reply observed; classified as A-type; follow-up candidate prepared.`

---

### 场景 C：follow-up 已成功发送

**`reply_queue.notes` 示例：**
- `A-type reply detected; low-risk follow-up executed successfully.`

如果你后面决定正式给 follow-up 建状态，也可以再扩。

---

### 场景 D：follow-up 发送失败

**`reply_queue.notes` 示例：**
- `A-type reply detected; follow-up attempt failed due to rate limit / browser issue.`

---

## 五、`daily_reports` 回填模板

每天如果发生了 follow-up 互动，至少补这些：

- 今日发现了多少条新回复
- 其中多少条属于 A 类
- 实际跟进了多少条
- 有没有高信号表达
- 今天 follow-up 的 blocker 是什么

### 正常日示例
- `top_insights = Users continue to reveal stronger pain-point language in follow-up replies than in first-touch comments.`
- `next_actions = Review high-signal replies and convert the best phrases into keywords and content angles.`
- `blockers = None; keep follow-up low-frequency and low-risk.`

### 异常日示例
- `top_insights = Follow-up quality is fine, but execution risk increased.`
- `next_actions = Pause auto follow-up and record replies only.`
- `blockers = Rate limit / login instability / rising deletion sensitivity.`

---

## 六、`dashboard` 回填模板

至少补：
- `high_intent_followups`
- `next_priority`
- `blocker`
- `notes`

### 示例
#### 正常日
- `next_priority = Review latest A-type replies and continue only the safest follow-ups.`
- `blocker = None`
- `notes = Follow-up layer is generating useful user language without needing aggressive engagement.`

#### 异常日
- `next_priority = Pause follow-up execution and continue observation only.`
- `blocker = Follow-up risk too high / browser issue / rate limit`
- `notes = Replies are being logged, but auto follow-up is temporarily paused.`

---

## 七、最小回填标准

每次发现一条值得处理的回复后，至少要做到：

### 必填
#### `engagements`
- `user_signal`
- `followup_needed`
- `followup_type`
- `lead_potential`
- `action_status`
- `notes`

### 至少补 1 处说明
#### `reply_queue`
- 在 `notes` 里写明 follow-up 的判断或结果

如果这两步没做，就不算 follow-up 处理完成。

---

## 八、一句话版本

> 当别人回复了我们后，优先把这条互动写进 `engagements`，明确它属于什么用户信号、要不要继续跟、有没有高价值；然后再在 `reply_queue.notes` 里补清这次 follow-up 的判断和结果，最后把整体情况反映到 `daily_reports` 和 `dashboard`。
