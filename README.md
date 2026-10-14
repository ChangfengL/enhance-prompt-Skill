# enhance-prompt

把口语草稿改写成一段可直接发送的提示词。对标 WorkBuddy `llm:enhancePrompt`：只增强、不执行，可接到 Claude Code 和其他 Agent。

效果是一段完整指令，不是带【问题】【目标】标题的工单，也不会开始做任务。

## 它做什么

- 补清任务、范围、约束和期望产出
- 保留原文语言（中文 / 英文 / 中英混合）
- 不编造原文没有的技术栈、文件名、产品名或数据
- 写 **WHAT**（要什么结果），不写 HOW（实现步骤）
- 不回答原问题，只把问题写清楚

草稿：`帮我写本周周报`

增强结果方向：

```text
请汇总本周工作进展，按事项写成可直接发送的周报：每条写清结果（不要只写「跟进中」），标出风险和需要谁拍板，并列出下周计划。全文一页内。
```

## 在 Claude Code 里用

把 `enhance-prompt/` 拷到个人或项目的 skills 目录：

```bash
# 个人
cp -r enhance-prompt ~/.claude/skills/

# 当前项目
cp -r enhance-prompt .claude/skills/
```

显式调用（不会自动触发）：

```text
/enhance-prompt 帮我写本周周报
```

回复分两块：可整段复制的「增强后的提示词」，以及不超过 3 条的「改了什么」。确认后再把增强结果发给执行 Agent。

## 接到其他 Agent / 按钮 / API

绑定到显式动作（按钮、slash、`POST { text }`），不要对每条用户消息自动增强。一次调用只返回增强正文。

- 能配独立 system + user 时，优先用 [`enhance-prompt/references/workbuddy-prompts.md`](enhance-prompt/references/workbuddy-prompts.md) 的 Rn/zn 原文
- 否则把 [`enhance-prompt/references/portable-prompt.md`](enhance-prompt/references/portable-prompt.md) 整段设为系统提示，用户消息只放草稿原文

不要把聊天记录、工具结果或仓库摘要拼进输入。

## 仓库结构

```text
enhance-prompt/
  SKILL.md                          Skill 正文：流程、算法、硬约束、宿主适配
  references/
    workbuddy-prompts.md            WorkBuddy 原版 Rn/zn 提示词
    portable-prompt.md              给其他 Agent / API 的可移植系统提示
    examples.md                     校准样本（周报、解释代码、中英混合等）
```

改写质量由 `SKILL.md` 里的算法和硬约束决定。宿主只能改包装（怎么取稿、怎么交稿），不能改文风、长度策略和硬约束。

## 硬约束（摘要）

1. 语言最高优先，禁止输出语言分析
2. 约 800 字以内，写成完整句子
3. 不要向用户索要 how-to 或代码片段
4. 不要建议原文没提到的技术栈 / 文件名 / 产品名
5. 不要加人设、XML、CoT、评分
6. 缺具体对象时用泛指，不要提问拦住改写
7. 改完即停：不读仓库、不改文件、不开始做任务
