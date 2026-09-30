# 问卷评分引擎（Questionnaire Scoring）

一句话：**把一份选择题问卷，变成一份带雷达图的评估报告。**

你提问、对方选答案，它负责按答案计分、按能力域分层汇总、算出总分和成熟度等级，
最后输出一份可以直接交付给客户的 HTML 报告（含雷达图、能力域明细、改进建议）。

内置 NIST CSF 网络安全成熟度评估题库：**158 道题 / 24 个能力域 / 4 大域**
（识别、保护、检测、响应与恢复），与原版网页评估工具**逐分对齐**。
也支持把你自己的问卷做成题库放进去用。

---

## 一、安装（三选一，挑最简单的）

### 方式一：直接拖进对话框（推荐，30 秒）

打开 WorkBuddy，把本项目的 **zip 直接拖进输入框**，说一句：

> 帮我安装这个技能

**不用解压，不用管目录。** 推荐先试这个。

### 方式二：双击安装脚本

下载解压后，进入 `questionnaire-scoring` 文件夹，双击 **`INSTALL.bat`**。
看到 `[OK] Installed successfully` 就装好了，重启一下 WorkBuddy。

### 方式三：手工复制 / clone

```bash
git clone https://github.com/Robert-Zhang-X/questionnaire-scoring.git
```

把 `questionnaire-scoring` 整个文件夹放到：

| 环境 | 路径 |
|---|---|
| WorkBuddy / CodeBuddy | `~/.workbuddy/skills/questionnaire-scoring` |
| Claude Code | `~/.claude/skills/questionnaire-scoring` |

Windows 上即 `C:\Users\<你的用户名>\.workbuddy\skills\`

---

## 二、装好之后怎么用

不用记命令，直接用大白话跟 AI 说：

| 你想做的事 | 就这么说 |
|---|---|
| 看一眼报告长什么样 | 「随机填一份 NIST 评估，出个报告给我看看」 |
| 正式做评估 | 「帮我做一次 NIST CSF 安全成熟度评估」 |
| 中途看进度 | 「还剩哪些题没答」 |
| 只评估某几块 | 「只做检测和响应这两部分」 |
| 换成自己的问卷 | 「我有一份问卷，帮我做成题库并评分」 |

AI 会一组一组把题目问出来，你按题号回字母就行（比如 `1=C 2=B 3=A`），
也可以直接说人话（「第1题我们有 Excel 台账但覆盖不全」），它会自己映射。

---

## 三、验证装好了没

跟 AI 说：**「列出可用的问卷题库」**

看到 `nist-csf-v1    158 题  24 组  4 域` 就说明装好了。

---

## 四、先看看产出效果

用浏览器打开 **`examples/sample_report.html`** ——这是一份完整报告的样例，
单文件、断网也能看。

> 注意：样例里的答案是随机生成的，**不代表任何真实组织的安全水平**。

---

## 五、系统要求

- **Python 3.8 以上**（WorkBuddy 自带，不用另外装）
- **不依赖任何第三方库**，纯标准库实现
- **可以离线用**：题库和报告都在本地，不联网也能跑

---

## 六、计分规则（想知道分数怎么来的看这里）

1. **单题分** = 所选选项自带的分值。**不能假设 A=1 / B=2**——内置题库有 16 道题是跳级映射（如 A=1、B=4、C=5）。
2. **N/A 是有效作答**，按 1 分计入（与原版网页一致），会拉低得分；只有留空才跳过不计。
3. **能力域分** = 该域内已答题的算术平均。
4. **领域分** = 该领域下各能力域的**等权平均**（题多的组不额外加权）。
5. **总分** = 4 个领域的**等权平均**（不是 158 题直接平均）。
6. 等级：<1.5 初始级 → <2.5 受管级 → <3.5 已定义级 → <4.5 量化管理级 → ≥4.5 优化级。

分数全由脚本算，**不让模型心算**——158 题的加权平均，模型必错。

---

## 七、已知的数据问题

原始问卷表本身有几处缺陷，**引擎按原样计分以保持与网页版一致，不擅自修正**
（改了分数就和源系统对不上），但报告末尾会自动提示：

| 题目 | 问题 |
|---|---|
| Detect-13 / Detect-14 | 是非题，但 A、B 两个选项**都是 0 分**，作答不影响得分 |
| Protect-64 | 只有 1 个计分选项（A=4） |

实际评估时，这几题**建议口头向受访方确认**，不能直接采信得分。

---

## 八、进阶：换成自己的问卷

引擎与题库解耦。照 `references/bank-schema.md` 造题库 JSON，
可从 `assets/bank_template.json`（3 题最小骨架）改起，然后跟 AI 说：

> 用我这个题库跑一遍评估

命令行方式：

```bash
python scripts/qscore.py validate my_bank.json
python scripts/qscore.py score answers.json --bank my_bank.json --format html --out r.html
```

## 九、命令行速查

想直接跑命令而不是跟 AI 对话的话：

```bash
QS=scripts/qscore.py

python $QS banks                                    # 列出内置题库
python $QS template --bank nist-csf-v1 --out a.json # 生成空白答案文件
python $QS show --bank nist-csf-v1 --group "Identity::Asset Management"   # 打印题目
python $QS next a.json                              # 看还剩哪些没答
python $QS score a.json --format html --out r.html  # 出报告
python $QS fill a.json --strategy random --score    # 随机填充并直接出分（测试用）
```

报告三种格式：`html`（给人看，含雷达图）/ `md`（便于二次编辑）/ `json`（供程序消费）。

## 十、开发者：回归测试

改过 `scripts/engine.py` 之后必跑：

```bash
python tests/test_engine.py
```

期望值固化在 `tests/nist_expected_scores.json`，**不是手算的**，是用 Node 跑网页原版 JS
评分函数实际得出的真值，覆盖总分、4 个域、24 个能力域共 29 项，另含自定义题库与边界情况。
全绿才算没改坏。

---

## 许可

MIT
