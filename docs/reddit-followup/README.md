# Reddit Reply Follow-up README

## 这是什么

这是 Reddit **自动看回复 / 跟进回复** 这条线的总入口文档。

它和 `reddit-comment-automation-README.md` 是并列关系，不是它的附属章节。

两条线分工明确：
- `reddit-comment-automation-README.md`：负责**自动发第一条评论**
- `reddit-reply-followup-README.md`：负责**自动看别人有没有回复我们，并决定要不要继续跟进回复**

---

## 这条线要解决什么问题

当我们已经发出一条 Reddit 评论之后，后面会出现 3 种情况：

1. **没人回复**
   - 只需要记录结果，不继续动作

2. **有人回复，但只是轻互动**
   - 记录信号，通常不继续回

3. **有人回复，而且值得继续聊**
   - 需要判断是否继续回一层
   - 必要时自动生成并发送 follow-up

这条线就是为第 2、3 种情况准备的。

---

## 这条线的目标

自动完成以下动作：

1. 检查我们发出的评论有没有新回复
2. 判断新回复属于哪一类
3. 决定：
   - 不回
   - 只记录
   - 值得继续回
4. 生成 follow-up candidate
5. 对低风险项自动继续回复
6. 把结果回填到 Excel

---

## 它和 comment automation 的边界

### `reddit-comment-automation`
负责：
- 找 thread
- 生成第一条候选评论
- 自动发第一条评论
- 做第一次回看

### `reddit-reply-followup`
负责：
- 看别人有没有回复我们
- 判断回复质量和风险
- 决定要不要继续接话
- 自动发送第二层/后续回复
- 回填 follow-up 结果

一句话：

- **comment automation = 起第一句**
- **reply follow-up = 接后面的对话**

---

## 推荐文档结构

这条线建议拆成 5 份文档：

1. `reddit-reply-followup-README.md`
2. `reddit-reply-followup-sop.md`
3. `reddit-reply-followup-runbook.md`
4. `reddit-reply-followup-browser-runbook.md`
5. `reddit-reply-followup-excel-template.md`

---

## 1. `reddit-reply-followup-README.md`

### 作用
总入口。

### 回答的问题
- 这条线是干什么的
- 和 comment automation 怎么分工
- 应该先看哪份文档

---

## 2. `reddit-reply-followup-sop.md`

### 作用
策略层 SOP。

### 回答的问题
- 什么样的回复值得继续跟
- 什么样的回复只记录不回
- 什么样的回复绝对不回
- 自动跟进的风险边界是什么

---

## 3. `reddit-reply-followup-runbook.md`

### 作用
执行流程文档。

### 回答的问题
- 发现新回复后怎么检查
- 如何生成 follow-up candidate
- 如何决定是否发送
- 发送后怎么回填

---

## 4. `reddit-reply-followup-browser-runbook.md`

### 作用
浏览器动作文档。

### 回答的问题
- 怎么打开我们自己的评论
- 怎么定位别人给我们的回复
- 怎么点 reply
- 怎么填 follow-up
- 怎么确认发送结果

---

## 5. `reddit-reply-followup-excel-template.md`

### 作用
Excel 回填模板。

### 回答的问题
- `engagements` 怎么更新
- `reply_queue` 需不需要补 follow-up 状态
- `daily_reports` / `dashboard` 怎么记

---

## 这条线最关键的判断

不是“有人回复了就继续回”。

而是先判断这条回复属于哪一类：

### A. 值得继续回
例如：
- 对方明确追问
- 对方补充真实痛点
- 对方在做选择比较
- 对方提供了高价值真实语言

### B. 只记录，不回
例如：
- 简单认同
- 轻微附和
- 没有继续聊的空间

### C. 不回
例如：
- 敌对语气
- 引战
- 风险高
- 话题偏了
- 已经不值得继续投入

这一步是整条 follow-up 线的核心。

---

## 一条 follow-up 任务的标准流转

标准上应该这样走：

1. 我们先发出第一条评论
2. 第一次回看时发现有人回复
3. 判断这条回复属于 A/B/C 哪类
4. 如果属于 A：生成 follow-up candidate
5. 做风险检查
6. 低风险时自动继续回一层
7. 回填 `engagements`
8. 必要时更新 `daily_reports` / `dashboard`

---

## 当前阶段的现实定位

这条线当前应该定位成：

- **低频、低风险 follow-up 自动化**
- **只跟进值得继续聊的回复**
- **优先记录真实用户语言**
- **优先做自然交流，不做导流**

当前不适合：
- 高频连环回复
- 争论型 thread 自动接战
- 高敏感社区自动接话
- 把每一条互动都当销售线索追

---

## 推荐阅读顺序

如果你第一次搭这条线，按这个顺序看：

1. `reddit-reply-followup-README.md`
2. `reddit-reply-followup-sop.md`
3. `reddit-reply-followup-runbook.md`
4. `reddit-reply-followup-browser-runbook.md`
5. `reddit-reply-followup-excel-template.md`

---

## 后续最自然的扩展

下一步最自然的是继续补这 4 份：

- `reddit-reply-followup-sop.md`
- `reddit-reply-followup-runbook.md`
- `reddit-reply-followup-browser-runbook.md`
- `reddit-reply-followup-excel-template.md`

这样这条线就会和 comment automation 那条线结构一致。

---

## 一句话版本

> `reddit-comment-automation` 负责自动发第一条评论，`reddit-reply-followup` 负责自动看别人有没有回复我们、判断要不要继续接话，并把 follow-up 的结果回填到系统里；这份 README 是后者的总入口。
