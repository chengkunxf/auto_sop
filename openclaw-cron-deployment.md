# OpenClaw Cron Deployment Guide

## 目标

这份文档专门解决一件事：

**当 `auto_sop` 迁移到另一台机器时，如何重新创建对应的 OpenClaw cron。**

重点不是复制当前机器的绝对路径，而是：
- clone 仓库
- 设置本机路径
- 替换变量
- 按模板建 cron

---

## 一、迁移时要区分两类路径

### 1. SOP / 文档路径
这是规则源路径。

例如当前机器是：
- `/Users/chengkun/Documents/personal/shop_sop/auto_sop`

但在新机器上，它可以是任意路径，例如：
- `/opt/shop_sop/auto_sop`
- `/srv/auto_sop`
- `/home/ubuntu/auto_sop`

### 2. 活数据路径
这是运行中的 workbook / 数据文件路径。

例如当前机器是：
- `/Users/chengkun/.openclaw/workspace/bag-community-acquisition.xlsx`

在新机器上也可以不同，例如：
- `/srv/openclaw/data/bag-community-acquisition.xlsx`
- `/opt/openclaw/workspace/bag-community-acquisition.xlsx`

**原则：**
- `auto_sop` 负责规则
- workbook 负责活数据
- 迁移时两者路径都要重新指定

---

## 二、部署前先准备 4 个变量

在新机器上，先确定这 4 个值：

### 1. `SOP_ROOT`
`auto_sop` 仓库在新机器上的本地路径。

示例：
```text
/opt/shop_sop/auto_sop
```

### 2. `DATA_XLSX`
运行中的 Excel 文件路径。

示例：
```text
/srv/openclaw/data/bag-community-acquisition.xlsx
```

### 3. `TZ`
cron 使用的时区。

示例：
```text
Asia/Shanghai
```

### 4. `DELIVERY_TARGET`
任务结果要投递到哪里。

例如：
- Discord channel id
- 其他目标 channel / recipient

---

## 三、先 clone 仓库

在新机器上：

```bash
git clone git@github.com:chengkunxf/auto_sop.git <SOP_ROOT>
```

例如：

```bash
git clone git@github.com:chengkunxf/auto_sop.git /opt/shop_sop/auto_sop
```

---

## 四、当前推荐的 job 结构

当前这套 Reddit / community 自动化，推荐 5 个 job：

1. Daily community collection
2. Daily reddit browser preflight
3. Daily reddit comment send
4. Daily reddit first review
5. Daily reddit second review

---

## 五、job 模板（机器无关版）

下面所有路径都要用你新机器上的变量替换：
- `<SOP_ROOT>`
- `<DATA_XLSX>`
- `<TZ>`
- `<DELIVERY_TARGET>`

---

## 1) Daily community collection

### 时间建议
- `00:05`

### prompt 模板

```text
按 <SOP_ROOT>/community-collection/daily-community-collection-sop.md 跑一轮每日社群与关键词更新。优先更新 keywords、communities、content_drafts、reply_queue，并按当前 SOP 的最低门槛与 fallback 规则执行。当前活数据仍使用 <DATA_XLSX>。完成后给出简短结果摘要：新增了什么、数量多少、是否达标、下一步建议。
```

---

## 2) Daily reddit browser preflight

### 时间建议
- `00:25`

### prompt 模板

```text
执行 Reddit comment send 前的 browser preflight / warm-up。按 <SOP_ROOT>/reddit-comment/reddit-comment-sop.md、<SOP_ROOT>/reddit-comment/reddit-comment-executor-runbook.md、<SOP_ROOT>/reddit-comment/reddit-comment-daily-operator-checklist.md 中的 browser preflight 机制执行：先检查 browser status、browser running、CDP/DevTools ready、持久登录态浏览器可连接、old.reddit 可打开。若发现 browser 执行器不可用，按标准恢复顺序处理：先重启 OpenClaw gateway，再检查 browser status，如未运行则执行 browser start，再次验证 CDP/DevTools ready。仅做预热与恢复，不发送评论。若恢复失败，汇报 blocker 到当前投递目标。
```

---

## 3) Daily reddit comment send

### 时间建议
- `00:35`

### prompt 模板

```text
按 <SOP_ROOT>/reddit-comment/README.md、<SOP_ROOT>/reddit-comment/reddit-comment-sop.md、<SOP_ROOT>/reddit-comment/reddit-comment-executor-runbook.md、<SOP_ROOT>/reddit-comment/reddit-comment-browser-runbook.md 执行一轮 Reddit 自动发评论。当前阶段按新号冷启动规则运行：未来一周内每天只发送 1–3 条评论，不得超过 3 条；只处理 approved + low risk 项，尽量分散到不同 subreddit，同一 subreddit 当天优先只发 1 条。严格遵守频控与熔断规则。当前活数据仍使用 <DATA_XLSX>。若登录态或执行器不可用，停止发送并只汇报 blocker。完成后输出简短结果：发了几条、哪些社区、是否有异常。
```

---

## 4) Daily reddit first review

### 时间建议
- `01:20`

### prompt 模板

```text
按 <SOP_ROOT>/reddit-followup/README.md、<SOP_ROOT>/reddit-followup/reddit-reply-followup-sop.md、<SOP_ROOT>/reddit-followup/reddit-reply-followup-runbook.md 检查最近已发送 Reddit 评论的第一次回看窗口。重点看评论是否仍存活、有没有新回复、是否值得 follow-up，并按模板回填 reply_queue 与 engagements。当前活数据仍使用 <DATA_XLSX>。完成后输出简短结果：存活数、回复数、是否有 A 类 follow-up 候选。
```

---

## 5) Daily reddit second review

### 时间建议
- `12:30`

### prompt 模板

```text
对最近 12–24 小时内已发送的 Reddit 评论做第二次回看。按 <SOP_ROOT>/reddit-followup/reddit-reply-followup-sop.md、<SOP_ROOT>/reddit-followup/reddit-reply-followup-runbook.md、<SOP_ROOT>/reddit-followup/reddit-reply-followup-excel-template.md 更新 engagements、daily_reports、dashboard，并总结高信号表达、是否需要继续跟进、下一步建议。当前活数据仍使用 <DATA_XLSX>。
```

---

## 六、迁移时最容易出错的地方

### 1. 直接照抄旧机器绝对路径
错法：
- `/Users/chengkun/...`

正确做法：
- 全部替换成新机器上的 `<SOP_ROOT>` 和 `<DATA_XLSX>`

### 2. 忘了区分“规则源”和“活数据”
不要把：
- `auto_sop` 仓库路径
和
- workbook 运行路径
混成一个概念。

### 3. 只建 comment send，不建 preflight
如果跳过 preflight，夜间执行时更容易把发送窗口浪费在 browser 启动故障上。

### 4. 新机器没验证浏览器链路就开跑
迁移后第一天，建议手动验证：
- browser status
- browser start
- old.reddit 可打开
- Reddit 登录态存在

---

## 七、新机器上线前的最小验收

至少确认：

1. `auto_sop` 已 clone 到本机
2. workbook 路径已确定
3. browser preflight job 已创建
4. comment send job 已创建
5. old.reddit 可打开
6. 持久登录态 Reddit 账号已登录
7. 手动强制跑一次 preflight
8. 手动强制跑一次 comment send（允许只做 1 条验证）

---

## 八、建议的迁移顺序

1. clone `auto_sop`
2. 确定 `<SOP_ROOT>`
3. 确定 `<DATA_XLSX>`
4. 验证 OpenClaw browser 可用
5. 登录 Reddit 持久会话
6. 创建 5 个 cron
7. 手动跑 preflight
8. 手动跑一次 send 验证
9. 再交给 nightly cron

---

## 九、一句话版本

> 迁移到新机器时，不要复制旧绝对路径；先 clone `auto_sop`，确定新机器的 `SOP_ROOT` 和 `DATA_XLSX`，再按这份文档的机器无关模板重建 5 个 cron。
