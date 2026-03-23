# Reddit Comment Daily Operator Checklist

## 目标

这是一份**每天值班只看一页**的清单。

它不展开讲原理，只回答：

**今天该检查什么、发什么、回填什么。**

如果时间紧，优先看这份。

---

## 一、开工前检查（先看 2 分钟）

### 1. 看 `dashboard`
确认：
- 今天的 `next_priority`
- 有没有 `blocker`
- 昨天评论执行数
- 当前高信号 followup 数

### 2. 看 `daily_reports`
确认：
- 昨天哪些 thread_type 表现最好
- 有没有删除异常
- 今天最应该继续哪条线

### 3. 看 Reddit 执行条件
确认：
- 登录态是否稳定
- 当前是否适合自动发送
- 最近 24 小时有没有明显删除/风控异常
- browser preflight 是否通过

### 4. 做 browser preflight
固定先检查：
- browser status
- browser running
- CDP / DevTools ready
- old.reddit 是否可打开

如果 preflight 失败：
1. 先重启 OpenClaw gateway
2. 再次检查 browser status
3. 如果浏览器未运行，执行 browser start
4. 恢复成功后再进入发送
5. 恢复失败则今天只排队、不发送

### 开工判断
#### 可以发
满足：
- 登录态正常
- 无明显删除异常
- `reply_queue` 里有 `approved + low risk`

#### 不发，只排队
满足任一：
- 登录失效
- rate limit
- 删除异常升高
- 今天账号状态不稳

---

## 二、今天先做什么

### 默认优先顺序
1. 看 `reply_queue`
2. 看 `engagements`
3. 补 `reply_queue`
4. 执行低风险评论
5. 回填结果
6. 更新 `daily_reports` 和 `dashboard`

---

## 三、`reply_queue` 快速检查

先确认这 4 个数：

1. `approved` 有多少条
2. `high + low risk` 有多少条
3. 今天已经发了多少条
4. 当前总队列是否 `>= 20`

### 判断标准
- 如果 `reply_queue < 20`
  - 先补队列，不急着发
- 如果 `approved + low risk` 很少
  - 先补线程/补候选回复
- 如果队列够、且有安全项
  - 可以进入执行

---

## 四、今天可发评论的挑选规则

今天优先发：
- `queue_status = approved`
- `risk_level = low`
- `priority = high`
- 来自更安全的 subreddit
- thread 更新鲜、语境更明确的项

今天不优先发：
- `reviewing`
- `medium risk`
- 旧 thread
- 语境不明确
- 支持型 / 敏感社区

---

## 五、发送前最后检查

每发 1 条前，都过一遍：

1. 当前 thread 还活着吗
2. 没锁帖、没归档吗
3. 目标评论链还贴题吗
4. `candidate_reply` 还自然吗
5. 今天这个 subreddit 发过没有

如果有一项不对：
- 今天这条先不发
- 改回 `reviewing` 或写清原因

---

## 六、今天的执行节奏

### 早期默认节奏
- 一次只发 **1 条**
- 发完先观察
- 稳定后再发下一条
- 每天总量控制在 **3–5 条**

### 不要这样发
- 连续轰炸同一个 subreddit
- 短时间连发多条
- 不看上下文直接贴模板
- 有异常还继续硬发

---

## 七、发完立刻做什么

每发完 1 条，立刻：

### 回填 `reply_queue`
至少写：
- `posted_at`
- `result_status`
- `engagement_signal`
- `notes`

### 记录第一次回看任务
- 30–60 分钟后回来检查

如果失败：
- 把失败原因写清楚
- 不要直接跳到下一条假装没事

---

## 八、第一次回看（30–60 分钟）

看：
- 评论还在不在
- 有没有被删
- 有没有回复
- 有没有 upvote / 互动

### 更新
- `reply_queue`
- `engagements`

如果出现明确痛点/偏好：
- 顺手记进 `engagements.notes`
- 后面反哺 `keywords` / `content_drafts`

---

## 九、第二次回看（12–24 小时）

看：
- 有没有新增回复
- 有没有高价值用户语言
- 要不要继续跟进

### 更新
- `engagements.user_signal`
- `engagements.followup_needed`
- `engagements.followup_type`
- `engagements.lead_potential`
- `engagements.action_status`

---

## 十、当天收工前必须做的事

### 1. 更新 `daily_reports`
至少写：
- 今天发了几条
- 删除了几条
- 哪类 thread 最有效
- 明天先做什么
- blocker 是什么

### 2. 更新 `dashboard`
至少写：
- `comments_posted_today`
- `high_intent_followups`
- `next_priority`
- `blocker`
- `notes`

### 3. 看一眼明天还剩什么
确认：
- `reply_queue` 是否仍然够用
- 有没有高信号 followup 没处理
- 明天是该继续执行，还是先补队列

---

## 十一、熔断清单

出现下面任一情况，当天停止自动发送：
- 连续 2 条删除
- rate limit
- 登录失效
- 评论框异常
- 页面结构异常

停止后只做：
- 发现
- 排队
- 回填
- 复盘

不要继续硬发。

---

## 十二、今天算完成的最低标准

当天至少满足：

### 队列侧
- `reply_queue >= 20`

### 执行侧
二选一：
- 至少成功执行 1 条低风险评论
- 或明确记录今天为什么不宜执行

### 回填侧
- `reply_queue` 已回填
- `engagements` 至少补了必要记录
- `daily_reports` 和 `dashboard` 已更新

---

## 一句话版本

> 每天先看 `dashboard` 和 `daily_reports`，确认今天能不能发；再从 `reply_queue` 里挑 `approved + low risk` 的项低频执行；每发一条立刻回填，之后做两次回看，最后更新 `daily_reports` 和 `dashboard`。
