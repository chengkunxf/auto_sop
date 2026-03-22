# Reddit 评论执行后 · Excel 回填模板

## 目标

把 Reddit 评论执行后的回填动作标准化，避免每次临时想：
- `reply_queue` 怎么写
- `engagements` 怎么补
- `daily_reports` / `dashboard` 怎么记

这份模板只解决：**发完之后怎么回填表。**

---

## 一、优先回填顺序

执行完一条评论后，按这个顺序回填：

1. `reply_queue`
2. `engagements`
3. `daily_reports`
4. `dashboard`

如果时间紧，至少先把：
- `reply_queue`
- `engagements`

写完整。

---

## 二、`reply_queue` 回填模板

### 场景 A：发送成功，刚发出

**最少更新字段：**
- `posted_at` = 实际发送时间
- `result_status` = `pending`
- `engagement_signal` = `none`
- `notes` += `已执行，待 30–60 分钟回看`

**示例：**
- `posted_at`: `2026-03-21 17:58`
- `result_status`: `pending`
- `engagement_signal`: `none`
- `notes`: `Executed successfully. Pending first review in 30–60 min.`

---

### 场景 B：发送失败

**最少更新字段：**
- `result_status` = `post_failed`
- `notes` += 失败原因

**常见失败原因写法：**
- `Post failed: login expired.`
- `Post failed: comment box not available.`
- `Post failed: Reddit rate limit triggered.`
- `Post failed: page structure changed / could not locate reply target.`

---

### 场景 C：发送后立即怀疑被吞/被删

**最少更新字段：**
- `result_status` = `pending` 或 `auto_removed`
- `notes` += `提交后状态异常，优先在第一次回看确认`

**示例：**
- `result_status`: `pending`
- `notes`: `Comment submitted but visibility uncertain; verify in first review.`

如果已经明确被删：
- `result_status` = `auto_removed` 或 `mod_removed`

---

### 场景 D：第一次回看后无互动

**最少更新字段：**
- `result_status` = `no_response`（如果你决定启用）
- `engagement_signal` = `low` 或 `none`
- `notes` += `Still live, no visible engagement in first review.`

如果当前还不想启用 `no_response`：
- 也可以先保留 `pending`
- 但 `notes` 必须写清当前状态

---

### 场景 E：第一次回看后有互动

**最少更新字段：**
- `result_status` = `got_replies` 或 `got_upvotes`
- `engagement_signal` = `medium` / `high`
- `notes` += 简短互动说明

**示例：**
- `result_status`: `got_replies`
- `engagement_signal`: `medium`
- `notes`: `Got 2 replies in first review; users discussed compartments and quick access.`

---

### 场景 F：确认高信号

**最少更新字段：**
- `result_status` = `high_signal`
- `engagement_signal` = `high`
- `notes` += 为什么是高信号

**高信号通常指：**
- 明确痛点表达
- 强偏好表达
- 真实比较与选择逻辑
- 可反哺 landing page / content / keywords 的语言

---

## 三、`engagements` 新增模板

通常在第一次回看后新增一条 `engagements` 记录。

### 标准字段模板
- `date`: 当天日期
- `community_name`: subreddit 名称
- `platform`: `reddit`
- `post_link`: 线程链接
- `comment_summary`: 这条评论大概讲了什么
- `user_signal`: `question / interest / objection / pain_point / comparison / purchase_intent / casual`
- `followup_needed`: `yes / no`
- `followup_type`: `reply / content_idea / product_feedback / landing_page_input / community_research`
- `lead_potential`: `low / medium / high`
- `action_status`: `new / pending / completed / ignored`
- `owner`: `openclaw`
- `notes`: 当前互动情况 + 后续判断

---

### 场景 A：有回复但价值一般

**示例：**
- `user_signal`: `casual`
- `followup_needed`: `no`
- `followup_type`: `community_research`
- `lead_potential`: `low`
- `action_status`: `completed`
- `notes`: `Comment stayed live; mild engagement only; no strong reusable language.`

---

### 场景 B：有明显痛点表达

**示例：**
- `user_signal`: `pain_point`
- `followup_needed`: `yes`
- `followup_type`: `content_idea`
- `lead_potential`: `medium`
- `action_status`: `pending`
- `notes`: `User emphasized bad pockets and constant rummaging; useful for next content angle.`

---

### 场景 C：有明确比较/选择逻辑

**示例：**
- `user_signal`: `comparison`
- `followup_needed`: `yes`
- `followup_type`: `landing_page_input`
- `lead_potential`: `high`
- `action_status`: `pending`
- `notes`: `User compared shoulder vs crossbody in terms of comfort and convenience; strong landing-page language.`

---

### 场景 D：完全没信号

**示例：**
- `user_signal`: `casual`
- `followup_needed`: `no`
- `followup_type`: `community_research`
- `lead_potential`: `low`
- `action_status`: `completed`
- `notes`: `No replies or useful signal after observation window.`

---

## 四、`daily_reports` 回填模板

每天至少补这些结果：

- 今日评论执行数
- 今日删除数
- 高信号互动数
- 最有效 `thread_type`
- 下一步动作
- blocker

### 简写模板

**如果今天执行正常：**
- `top_insights`: `Organization/pockets language continues to outperform generic bag talk.`
- `next_actions`: `Continue low-risk comments in r/handbags and review high-signal replies for keyword feedback.`
- `blockers`: `No major blocker; maintain low frequency.`

**如果今天触发异常：**
- `top_insights`: `Execution quality fine, but Reddit account sensitivity remains high.`
- `next_actions`: `Pause auto-send and continue discovery/queue building only.`
- `blockers`: `Rate limit / removals / login instability.`

---

## 五、`dashboard` 回填模板

至少更新：
- `comments_posted_today`
- `high_intent_followups`
- `next_priority`
- `blocker`
- `notes`

### 简写模板

**正常日：**
- `next_priority`: `Review first-wave comment results and expand safest approved queue items.`
- `blocker`: `None` 或简短风险描述
- `notes`: `Low-risk Reddit comment execution remains stable.`

**异常日：**
- `next_priority`: `Pause execution and continue queue building only.`
- `blocker`: `Auto removals / login issue / rate limit`
- `notes`: `Execution paused pending safer conditions.`

---

## 六、最小回填标准

每发完 1 条评论，至少要做到：

### 必填
#### `reply_queue`
- `posted_at`
- `result_status`
- `engagement_signal`
- `notes`

### 至少补 1 条
#### `engagements`
- 新增一条对应记录

如果这两步没做，就不算执行完成。

---

## 七、一句话版本

> 发完 Reddit 评论后，先回填 `reply_queue` 的发送结果，再补 `engagements` 的信号判断，最后把当天整体情况写进 `daily_reports` 和 `dashboard`。
