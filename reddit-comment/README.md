# Reddit Comment Automation README

## 目录索引

### 总入口
- `reddit-comment-automation-README.md`

### 策略层
- `reddit-comment-sop.md`

### 执行层
- `reddit-comment-executor-runbook.md`

### 浏览器动作层
- `reddit-comment-browser-runbook.md`

### 回填层
- `reddit-comment-excel-update-template.md`

### 日常值班层
- `reddit-comment-daily-operator-checklist.md`

---

## 这是什么

这是 Reddit 评论自动化这一条线的**总入口文档**。

它的作用只有一个：

**告诉你这套系统的文档分别负责什么，以及应该按什么顺序看。**

如果以后只记得一个入口，就看这份。

---

## 文档结构总览

当前这条线拆成 4 份主文档：

1. `reddit-comment-sop.md`
2. `reddit-comment-executor-runbook.md`
3. `reddit-comment-browser-runbook.md`
4. `reddit-comment-excel-update-template.md`

---

## 1. `reddit-comment-sop.md`

### 作用
这是**策略层 SOP**。

它回答的是：
- 这套 Reddit 评论自动发送系统要做什么
- 不该做什么
- 每张表分别负责什么
- 评论自动发送的准入条件是什么
- 频控、熔断、异常处理怎么定义

### 适合什么时候看
当你需要：
- 理解整套系统
- 判断当前阶段该不该自动发
- 看自动发送的边界和风险控制

就先看这份。

### 一句话理解
**它定义规则。**

---

## 2. `reddit-comment-executor-runbook.md`

### 作用
这是**执行器流程文档**。

它回答的是：
- 真正执行一条评论任务时，步骤是什么
- 从 `reply_queue` 里取什么数据
- 怎么做发送前检查
- 发完后怎么回填
- 什么时候回看

### 适合什么时候看
当你已经确认：
- 当前可以发
- 有 `approved + low risk` 项

并准备真正执行时，就看这份。

### 一句话理解
**它定义执行流程。**

---

## 3. `reddit-comment-browser-runbook.md`

### 作用
这是**OpenClaw browser 工具操作清单**。

它回答的是：
- 浏览器怎么开线程
- 怎么定位评论框
- 怎么做发送前最后检查
- 怎么提交评论
- 提交后怎么立刻验证

### 适合什么时候看
当你已经拿到一条可执行任务，准备在浏览器里真正点下去时，看这份。

### 一句话理解
**它定义浏览器动作。**

---

## 4. `reddit-comment-excel-update-template.md`

### 作用
这是**Excel 回填模板**。

它回答的是：
- 评论发完后 `reply_queue` 怎么写
- `engagements` 怎么补
- `daily_reports` 怎么记
- `dashboard` 怎么记
- 各种成功 / 失败 / 无互动 / 高信号场景该怎么回填

### 适合什么时候看
执行完评论后，或者做第一次 / 第二次回看时，看这份。

### 一句话理解
**它定义发完后怎么写表。**

---

## 推荐阅读顺序

### 如果你是第一次接手这条线
按这个顺序：

1. `reddit-comment-sop.md`
2. `reddit-comment-executor-runbook.md`
3. `reddit-comment-browser-runbook.md`
4. `reddit-comment-excel-update-template.md`

---

### 如果你今天只是要执行评论
按这个顺序：

1. `reddit-comment-sop.md`（快速确认今天能不能发）
2. `reddit-comment-executor-runbook.md`
3. `reddit-comment-browser-runbook.md`
4. `reddit-comment-excel-update-template.md`

---

### 如果你今天只是要复盘/回填
直接看：

1. `reddit-comment-excel-update-template.md`
2. 必要时回看 `reddit-comment-executor-runbook.md`

---

## 四份文档之间的关系

可以把它们理解成四层：

### 第一层：策略层
- `reddit-comment-sop.md`

回答：
- 要不要做
- 能不能做
- 什么情况下做

### 第二层：执行层
- `reddit-comment-executor-runbook.md`

回答：
- 真正执行的步骤是什么

### 第三层：浏览器动作层
- `reddit-comment-browser-runbook.md`

回答：
- 浏览器里具体怎么点、怎么发

### 第四层：回填层
- `reddit-comment-excel-update-template.md`

回答：
- 发完以后怎么写回系统

---

## 一条任务的标准流转

一条 Reddit 评论任务，标准上应该这样流转：

1. 在 `reply_queue` 中被筛成 `approved + low risk`
2. 先按 `reddit-comment-sop.md` 判断今天是否适合发送
3. 按 `reddit-comment-executor-runbook.md` 做执行前检查
4. 按 `reddit-comment-browser-runbook.md` 在浏览器里真正发出
5. 按 `reddit-comment-excel-update-template.md` 回填 `reply_queue`
6. 在第一次 / 第二次回看后补 `engagements`
7. 最后补 `daily_reports` 和 `dashboard`

---

## 当前阶段的现实定位

这套文档当前服务的是：

- **低风险 Reddit 评论自动化 / 半自动化执行**
- **评论优先，不做主帖自动化**
- **风控优先，不追求发量**

当前还不适合：
- 高频无人值守评论系统
- 大规模自动发帖系统
- 弱审核条件下的强执行

---

## 如果后面还要继续扩

下一步最自然的扩展方向有 3 个：

1. **把浏览器执行器真的代码化/流程化**
   - 让 OpenClaw 能按固定步骤自动发评论

2. **把状态机真正落回 Excel**
   - 把 `scheduled / posted / needs_retry / completed` 等状态正式启用

3. **补一个 daily operator checklist**
   - 让每天值班时只看一页就知道今天该做什么

---

## 一句话版本

> `reddit-comment-sop.md` 定规则，`reddit-comment-executor-runbook.md` 定执行流程，`reddit-comment-browser-runbook.md` 定浏览器动作，`reddit-comment-excel-update-template.md` 定回填方式；这份 README 负责把它们串起来。
