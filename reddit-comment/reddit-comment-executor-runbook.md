# Reddit 自动发送执行器 Runbook

## 目标

这份 runbook 只负责一件事：

**把 `reply_queue` 里符合条件的 Reddit 评论任务，安全地执行出去，并把结果回填。**

它不是策略 SOP，不负责定义评论方向；它负责把已经筛好的任务真正发出去。

---

## 适用范围

适用于：
- Reddit 评论自动发送
- 低风险、低频、已审核通过的评论任务
- 基于 `reply_queue` 的表驱动执行

不适用于：
- Reddit 主帖自动发布
- 高频批量发送
- 无审核的高风险评论
- 无登录态的临时执行

---

## 执行器前提

开始前必须满足：

1. **独立浏览器实例可用**
2. **Reddit 登录态有效**
3. **账号未处于明显风控/异常状态**
4. **`reply_queue` 中已有 `approved + low risk` 项**
5. **当天发送频率未超标**

只要有一项不满足：
- 停止自动发送
- 只做排队、检查、回填

---

## 输入表

主要输入：`reply_queue`

关键字段：
- `platform`
- `community_name`
- `thread_url`
- `thread_title`
- `thread_type`
- `target_comment_summary`
- `reply_goal`
- `playbook_ref`
- `candidate_reply`
- `priority`
- `risk_level`
- `queue_status`
- `posted_at`
- `result_status`
- `engagement_signal`
- `notes`

辅助输入：
- `posting_rules`
- `reply_playbooks`
- `dashboard`
- `daily_reports`

---

## 输出表

主要回填：
- `reply_queue`
- `engagements`
- `daily_reports`
- `dashboard`

---

## 执行总流程

1. 做 browser preflight / warm-up
2. 读取可执行任务
3. 做执行前准入检查
4. 将目标线程转换为 **old.reddit 优先** 的执行 URL
5. 打开目标线程
6. 找到正确评论链 / 评论框
7. 对 `candidate_reply` 做最后微调
8. 提交评论
9. 记录执行结果
10. 安排第一次回看
11. 回填日报与 dashboard

## Browser preflight / warm-up

### 目的
在真正发送前，先验证 OpenClaw 浏览器执行器本身是可用的，避免把正式发送窗口浪费在启动故障上。

### 标准检查项
1. browser status 正常
2. browser 进程处于 running
3. CDP / DevTools ready
4. 持久登录态浏览器可连接
5. old.reddit 页面可打开

### 标准恢复顺序
如果 preflight 失败，恢复顺序固定为：
1. **重启 OpenClaw gateway**
2. 再检查 browser status
3. 如果浏览器未运行，执行 browser start
4. 再验证 CDP / DevTools ready
5. 通过后才继续 comment send

### 遇到以下信号时，不要硬发
- `Could not connect to Chrome`
- `DevToolsActivePort` 缺失
- browser running = false
- CDP ready = false
- browser 可执行器不可连接

命中这些 blocker 时，执行器应：
- 先做恢复，不直接发评论
- 恢复失败则停止本轮发送
- 在 `daily_reports` / `dashboard` 记录 blocker

## 执行路径策略

### 默认策略
- **默认先尝试 old.reddit**
- 不把新版 Reddit 作为主执行器
- 新版 Reddit 仅作为：
  - 辅助查看上下文
  - 无法切到 old.reddit 时的有限 fallback

### old.reddit 转换规则
如果 `thread_url` 是：
- `https://www.reddit.com/...`

优先转换成：
- `https://old.reddit.com/...`

执行器应先在 old.reddit 检查：
- 页面是否存在
- 是否已登录
- 是否可见评论表单
- 是否可直接提交

### fallback 规则
只有在以下情况下才允许尝试新版 Reddit：
- old.reddit 页面不可达
- old.reddit 无评论入口
- old.reddit 登录态异常但新页面可用

如果新版 Reddit 出现以下信号，直接停止本条，不再硬提：
- 文本写入成功但提交后未实际发布
- `Comment` / `Reply` 点击后页面无状态变化
- 无报错、无 toast，但评论未出现
- 评论框 ref / selector 持续歧义

---

## Step 1：读取可执行任务

从 `reply_queue` 中只读取满足以下条件的记录：

- `platform = reddit`
- `queue_status = approved`
- `risk_level = low`
- `result_status = pending` 或为空
- `posted_at` 为空

### 排序建议
优先顺序：
1. `priority = high`
2. 更新鲜的 thread
3. 来自更安全的 subreddit
4. 语义更贴当前评论链的项

### 当天数量上限
早期阶段建议：
- 每天最多执行 **3–5 条**
- 同一 subreddit 每天最多 **1–2 条**
- 同一 thread 永远只发一次

---

## Step 2：执行前准入检查

对每条候选任务，逐条检查：

### 账号与环境检查
- Reddit 是否已登录
- 页面是否能正常访问
- 当前是否出现 rate limit
- 最近 24 小时删除率是否异常

### 任务本身检查
- `candidate_reply` 不为空
- `candidate_reply` 不含链接
- `candidate_reply` 不含品牌推荐
- `candidate_reply` 不像广告或调研
- `thread_url` 可访问
- 线程未锁定
- 线程未归档
- 线程没有明显大规模删评/折叠异常

### 社区规则检查
结合 `posting_rules` 确认：
- `action_type = comment` 是否允许
- 当前社区是否仍适合评论
- 是否存在明确禁区（例如 sales/authenticity/deal 贴）

任何一项不通过：
- 不执行发送
- 在 `notes` 记录原因
- 必要时把 `queue_status` 改回 `reviewing`

---

## Step 3：打开目标线程

执行器打开 `thread_url` 后，先检查：

- 页面是否成功加载
- 帖子是否存在
- 是否需要重新登录
- 是否能看到评论区
- 是否能找到回复入口

### 如果失败
- 不继续提交
- 在 `notes` 记录：
  - 登录失效
  - 页面加载失败
  - 线程不存在
  - 评论框不可用

---

## Step 4：定位正确回复位置

### 默认原则
优先回复：
- 与 `target_comment_summary` 最匹配的评论链
- 如果没有明确目标评论链，再回复主贴下评论框

### 检查点
- 当前链是否真的在讨论对应主题
- 附近已有回复是否与 `candidate_reply` 太相似
- 该楼层语气是否适合当前回复

### 不要发的情况
- 楼里已经有人用几乎同样的话说过
- 当前链已经偏题
- 对方在争论、阴阳怪气或明显敌对
- 该楼层是规则敏感内容

---

## Step 5：发送前最后微调

`candidate_reply` 允许做一次轻微调整，但只能做低风险微调。

### 允许做的事
- 替换一两个重复词
- 缩短过长句子
- 去掉生硬连接词
- 把语气调得更像真人
- 避免和楼里原话重复

### 不允许做的事
- 加链接
- 加品牌名 / 店铺名
- 改成推荐帖
- 改成导流帖
- 改成问卷/调研口吻

### 发送前最后确认 5 项
1. 贴题
2. 简洁
3. 自然
4. 不像模板
5. 不像营销

---

## Step 6：提交评论

### 提交动作
- 点击评论框
- 粘贴最终回复
- 再检查一次内容
- 提交

### 提交后立即检查
- 评论是否出现在页面上
- 是否被立即折叠/消失
- 是否出现错误提示
- 是否提示频率限制

---

## Step 7：提交后立即回填

### 如果发送成功
在 `reply_queue` 里至少更新：
- `posted_at`
- `result_status = pending`
- `engagement_signal = none`
- `notes` 增补“已执行，待回看”

### 如果发送失败
在 `reply_queue` 里至少更新：
- `notes` 写明失败原因
- `result_status` 视情况写：
  - `post_failed`
  - `auto_removed`
  - `mod_removed`

如果当前表还没正式启用更细状态，先不要乱扩 `queue_status`，优先把失败原因写进 `notes`。

---

## Step 8：第一次回看（30–60 分钟）

检查：
- 评论是否还在
- 是否被自动删掉
- 是否有人回复
- 是否有 upvote / 明显互动

### 更新 `reply_queue`
- `result_status`
- `engagement_signal`
- `notes`

### 更新 `engagements`
新增一条记录：
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

## Step 9：第二次回看（12–24 小时）

检查：
- 是否出现新增回复
- 是否有高价值用户语言
- 是否值得继续跟进
- 是否值得反哺 `keywords` / `content_drafts`

### 回填重点
- `engagements.user_signal`
- `engagements.followup_needed`
- `engagements.followup_type`
- `engagements.lead_potential`
- `engagements.action_status`
- `engagements.notes`

---

## Step 10：频控与熔断执行规则

### 频控
- 每天最多 **3–5 条**
- 同一 subreddit 每天最多 **1–2 条**
- 两条评论之间保留明显时间间隔
- 支持型社区默认不自动发

### 熔断条件
出现以下任一情况，立即停止当天自动发送：
- 连续 2 条删除
- rate limit
- 登录态失效
- 浏览器执行异常
- 评论刚发出就明显消失

### 熔断后动作
- 停止发送
- 更新 `dashboard.blocker`
- 更新 `daily_reports.blockers`
- 当天只保留发现、排队、复盘

---

## Step 11：日报回填

### `daily_reports`
至少更新：
- 今日评论执行数
- 今日删除数
- 高信号互动数
- 最有效 `thread_type`
- 下一步动作
- blocker

### `dashboard`
至少更新：
- comments posted
- high-signal followups
- next priority
- blocker
- notes

---

## 异常处理清单

### A. 登录失效
- 停止自动发送
- 不尝试盲发
- 记录 blocker

### B. 页面打不开
- 记录失败
- 不重复暴力刷新
- 保留任务待后续复查

### C. 评论框不存在
- 不执行提交
- 记录页面状态

### D. 提交报错
- 写入 `post_failed`
- 记录报错提示

### E. 立即被删
- 写入 `auto_removed` 或 `mod_removed`
- 触发风险检查

### F. Rate limit
- 进入当天 cooldown
- 停止后续自动执行

---

## 最小成功标准

一次执行器运行，只有同时满足以下条件，才算成功：

1. 成功读取到可执行任务
2. 至少执行了 1 条有效发送，或明确判定当天不宜发送
3. `reply_queue` 已回填结果
4. `engagements` / `daily_reports` / `dashboard` 至少更新到必要程度
5. 没有出现未记录的异常

---

## 一句话版本

> 读取 `approved + low risk` 的 Reddit 评论任务，做执行前风控检查，用持久登录态浏览器打开目标线程，微调并提交评论，随后回填 `reply_queue`、`engagements`、`daily_reports` 和 `dashboard`，同时严格执行频控和熔断。
