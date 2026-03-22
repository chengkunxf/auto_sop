# Reddit 评论自动发送 · OpenClaw 浏览器执行版操作清单

## 目标

把 `reddit-comment-executor-runbook.md` 再往下落一层，变成 **OpenClaw browser 工具可直接照着执行的操作清单**。

这份 runbook 关注的是：
- 浏览器怎么开
- 线程怎么进
- 评论框怎么找
- 评论怎么发
- 发完怎么检查
- 怎么回填结果

---

## 使用前提

开始前先确认：

1. Reddit 已登录
2. 用的是**持久登录态浏览器**
3. 当前只执行 `approved + low risk` 项
4. 当天未触发熔断
5. 当前评论量未超频

---

## 推荐浏览器使用方式

### 优先方案
使用 OpenClaw `browser` 工具，优先接到：
- 持久登录态浏览器
- 可直接访问 Reddit 的实例

### 不推荐长期使用
- 临时 relay attach
- 没登录态的隔离浏览器
- 每次重新登录的会话

---

## 执行输入

从 `reply_queue` 取 1 条记录，至少拿到这些字段：

- `thread_url`
- `community_name`
- `thread_title`
- `target_comment_summary`
- `candidate_reply`
- `priority`
- `risk_level`
- `queue_status`

只处理：
- `queue_status = approved`
- `risk_level = low`

---

## 标准执行流程

### Step 1：打开 Reddit 线程

浏览器动作：
1. 打开 `thread_url`
2. 等页面加载完成
3. 确认帖子存在
4. 确认评论区可见

检查点：
- 页面不是登录页
- 页面不是错误页
- 线程没被删
- 页面能滚动到评论区

如果失败：
- 不继续执行
- 记录失败原因

---

### Step 2：确认当前线程仍值得回复

进入线程后，先看：
- 是否锁帖
- 是否归档
- 评论区是否还活跃
- 目标评论链是否还贴 `target_comment_summary`
- 当前语气是否适合自然接话

### 必须停止的情况
- 贴子锁了
- 已归档
- 目标楼层没了
- 话题已经跑偏
- 楼里已经有人说了几乎一样的话
- 当前链明显敏感/争议/敌对

如果任一命中：
- 不发送
- 退出本条
- 记录原因

---

### Step 3：定位评论入口

优先级：
1. 优先回复与 `target_comment_summary` 最匹配的楼层
2. 如果没有明确匹配楼层，再考虑回复主贴

浏览器动作：
- 找目标评论块
- 找 reply 按钮
- 点击进入评论输入框

如果找不到目标楼层：
- 不强发
- 可以退回主贴层，但前提是回复仍自然
- 如果不自然，就放弃本条

---

### Step 4：发送前最后人工式检查

把 `candidate_reply` 再看一遍，只允许轻微调整。

检查 5 项：
1. 贴题吗
2. 像真人吗
3. 会不会太长
4. 会不会像模板
5. 会不会像营销/调研

允许微调：
- 改 1–3 个词
- 去掉重复表达
- 缩短第一句
- 让语气更像当前楼层

不允许：
- 加链接
- 加品牌
- 加导流
- 改成推荐贴

---

### Step 5：填入评论并提交

浏览器动作：
1. 点击评论输入框
2. 填入最终评论文本
3. 再检查一次可见文本
4. 点击 Comment / Reply

提交后立刻检查：
- 评论是否出现在页面中
- 是否有报错
- 是否提示频率限制
- 是否出现登录失效

---

### Step 6：提交后即时验证

提交后的 10–30 秒内，看：
- 评论是否仍可见
- 是否被立即吞掉
- 是否有“removed”迹象
- 页面是否刷新异常

### 结果判断
#### A. 明显成功
- 页面还能看到评论
- 没报错
- 没被即时吞掉

#### B. 明显失败
- 报错
- rate limit
- 登录失效
- 评论未出现
- 提交后立刻消失

#### C. 不确定
- 页面状态怪异
- 评论一闪而过
- 无法确认是否真的发出

不确定时不要当作成功，先记为待复核。

---

## 回填规则

### 成功时回填 `reply_queue`
至少更新：
- `posted_at`
- `result_status = pending`
- `engagement_signal = none`
- `notes += 已执行，待 30–60 分钟回看`

### 失败时回填 `reply_queue`
至少更新：
- `notes += 失败原因`
- `result_status = post_failed / auto_removed / mod_removed`

### 不确定时回填
- `result_status = pending`
- `notes += 提交后状态不确定，优先在第一次回看确认`

---

## 第一次回看（30–60 分钟）

浏览器动作：
1. 重新打开 `thread_url`
2. 定位已发送评论
3. 看是否仍存在
4. 看是否有回复 / upvote / 互动

回填：
- `result_status`
- `engagement_signal`
- `notes`

如果有明显互动：
- 同步新增到 `engagements`

---

## 第二次回看（12–24 小时）

浏览器动作：
1. 再打开一次线程
2. 看是否有新增回复
3. 看是否出现高价值用户语言
4. 判断是否要继续跟进

回填：
- `engagements.user_signal`
- `engagements.followup_needed`
- `engagements.followup_type`
- `engagements.lead_potential`
- `engagements.action_status`
- `engagements.notes`

---

## 频控操作清单

执行器每次运行前都要检查：

- 今天已发几条
- 当前 subreddit 今天已发几条
- 距离上一条发送过去多久
- 最近是否有删除异常

### 早期建议
- 每次只发 1 条，然后观察
- 稳定后再发下一条
- 同一批次不要连续轰炸同一个 subreddit

---

## 熔断操作清单

如果出现以下任一情况，立即停止当天自动发送：
- 连续 2 条删除
- rate limit
- 登录态失效
- 评论框异常
- 页面结构异常

停止后动作：
1. 不再发下一条
2. 回填当前失败记录
3. 更新 `dashboard.blocker`
4. 更新 `daily_reports.blockers`

---

## OpenClaw browser 工具建议动作映射

实际执行时，典型会用到这些 browser 动作：

- `open` / `navigate`：打开 `thread_url`
- `snapshot`：抓页面结构，确认评论区与 reply 按钮
- `act click`：点击 reply
- `act type` / `act fill`：填入评论
- `act click`：提交评论
- `snapshot`：提交后确认评论是否出现

### 推荐节奏
1. `navigate` 到目标线程
2. `snapshot` 确认页面结构
3. `act` 点击 reply
4. `act` 填入文本
5. `act` 提交
6. `snapshot` 再确认结果

---

## 最小成功标准

一次 browser 执行，只有满足以下条件才算成功：

1. 成功打开目标线程
2. 成功定位可回复位置
3. 成功提交评论或明确判断不应发送
4. 已回填 `reply_queue`
5. 已安排后续回看

---

## 一句话版本

> 从 `reply_queue` 取一条 `approved + low risk` 的 Reddit 评论任务，用持久登录态浏览器打开线程，确认上下文仍适合回复，微调 `candidate_reply` 后提交评论，再立即验证、回填结果，并按频控和熔断规则决定是否继续发下一条。
