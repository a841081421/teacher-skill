# Teacher Skill 设计文档

> 日期: 2026-06-02
> 状态: 已通过设计评审

## 概述

一套通用的 AI 学习助手 skill，支持 Claude Code 和 OpenCode。用户输入 `/learn` 后，系统自动引导：目标拆解 → 计划制定 → 每日教学 → 总结归档。

## 架构

### Skill 列表

| Skill | 职责 | 触发条件 |
|-------|------|---------|
| `learn` | 入口路由，判断当前阶段 | 用户输入 `/learn` |
| `curriculum-builder` | 目标拆解 + 计划制定 | `_progress.md` 不存在时 |
| `teacher-core` | 一问一答教学 + 检验理解 | 用户说"开始学习"/"继续" |
| `progress-tracker` | 进度管理 + 计划调整 | 用户说"看看进度"/"调整计划" |
| `note-generator` | 总结 + 生成 .md 笔记 | 用户说"今天学完了"/"总结" |

### 目录结构

```
skills/
  learn/
    SKILL.md
  curriculum-builder/
    SKILL.md
  teacher-core/
    SKILL.md
  progress-tracker/
    SKILL.md
  note-generator/
    SKILL.md
```

### 数据流

所有 skill 通过一个核心文件协调：用户指定笔记目录下的 `_progress.md`。

```
用户输入 /learn
      ↓
   learn (入口路由)
      ↓ 读取 _progress.md
      ↓
  ┌─────────────────────────────────┐
  │ _progress.md 不存在？            │
  │   → curriculum-builder          │
  │                                   │
  │ _progress.md 存在，今天有计划？    │
  │   → teacher-core                 │
  │                                   │
  │ 教学完成，用户说"结束"？           │
  │   → note-generator               │
  │                                   │
  │ 需要查看/调整计划？               │
  │   → progress-tracker             │
  └─────────────────────────────────┘
```

## 核心数据文件

### `_progress.md` 格式

```markdown
# 学习进度

## 基本信息
- 目标: [用户输入的目标]
- 笔记目录: [路径]
- 每日学习时间: [X 小时]
- 总周长: [X 周]
- 创建日期: [日期]

## 课程计划
### 阶段1: [名称] (第1-2周)
- [ ] 知识点A — 精通
- [ ] 知识点B — 熟练
### 阶段2: [名称] (第3-4周)
- [ ] 知识点C — 了解

## 学习记录
### 2026-06-02
- 完成: 知识点A (部分)
- 笔记: 2026-06-02-知识点A.md
- 下次继续: 知识点A 的后半部分
```

## 各 Skill 详细设计

### 1. learn/SKILL.md

**YAML frontmatter:**
```yaml
---
name: learn
description: Use when the user wants to learn something new, continue daily learning, check learning progress, or wrap up a study session
---
```

**路由逻辑 (Quick Reference Table):**

| 条件 | 动作 |
|------|------|
| 笔记目录下没有 `_progress.md` | → 调用 `curriculum-builder` |
| `_progress.md` 存在，用户说"开始学习"/"继续" | → 调用 `teacher-core` |
| 用户说"今天学完了"/"总结" | → 调用 `note-generator` |
| 用户说"看看进度"/"调整计划" | → 调用 `progress-tracker` |
| 用户直接说了新目标 | → 确认是否放弃旧计划，是则调用 `curriculum-builder` |

**Iron Law:**
```
NEVER START TEACHING WITHOUT CHECKING _progress.md FIRST
```

### 2. curriculum-builder/SKILL.md

**交互流程（3 步，每步等用户确认）：**

**步骤 1：目标拆解**
- 用户说目标（如"Java 后端开发岗"、"过六级"）
- LLM 输出：技能树 + 每个技能的掌握程度要求（精通/熟练/了解）
- 等用户确认或调整

**步骤 2：学习参数**
- 询问：每天预期学习时间（小时）、预期总周长
- 询问：笔记保存到哪个目录（默认 `./notes`）

**步骤 3：生成计划**
- 根据技能总量 + 学习时间，自动分配阶段
- 每个阶段包含：名称、时间跨度、知识点列表、掌握程度
- 输出计划让用户确认
- 确认后写入 `_progress.md`

**Iron Law:**
```
NEVER SKIP THE USER CONFIRMATION STEP FOR SKILL TREE AND PLAN
```

**Red Flags:**
- 用户还没确认技能树就开始生成计划 → 停下来等确认
- 用户说"随便"或"你定" → 给出推荐方案但仍然要求确认

### 3. teacher-core/SKILL.md

**从 `_progress.md` 读取：**
- 当前阶段、今天该学的知识点、上次的"下次继续"位置

**教学流程：**
```
1. 告诉用户今天学什么（知识点名称 + 一句话预告）
2. 从直觉/类比引入概念（不堆术语）
3. 讲一个知识点 → 提一个检验问题 → 等用户回答
4. 回答正确：肯定，继续下一个知识点
   回答有误：温和纠正，用不同角度重新解释
5. 当天知识点讲完 → 一次性给出所有检验问题
6. 全部答完 → 更新 _progress.md 的"学习记录"和"下次继续"
```

**Iron Law:**
```
NEVER MOVE TO THE NEXT TOPIC WITHOUT A VERIFIED UNDERSTANDING CHECK
NEVER DUMP A WALL OF TEXT — ONE CONCEPT, ONE QUESTION AT A TIME
```

**Rationalization Table:**

| 借口 | 现实 |
|------|------|
| "这个知识点太简单，不用检验" | 简单的概念理解错会更隐蔽，检验只需 10 秒 |
| "用户说懂了，不用再问" | 说懂了不等于真懂，必须用具体问题验证 |
| "一次多讲几个知识点效率更高" | 堆砌信息不等于学习，一问一答才能发现盲区 |

**语气规则：**
- 语言：跟随用户输入的语言
- 风格：简洁直接，类比优先，具体数字举例
- 禁止：空洞描述、术语堆砌、一次性输出超过 300 字

### 4. progress-tracker/SKILL.md

**功能列表 (Quick Reference Table):**

| 用户说 | 动作 |
|--------|------|
| "看看进度" | 输出：已完成 X/Y 个知识点、当前阶段、学习天数 |
| "跳过这个知识点" | 标记跳过，记录原因，更新"下次继续" |
| "这个太难了" | 拆分当前知识点为更小的子知识点 |
| "调整计划" | 重新询问时间/周长，重新分配阶段（保留已完成记录） |
| "我几天没学了" | 提醒上次断点，建议从断点继续 |

**Iron Law:**
```
NEVER DELETE COMPLETED LEARNING RECORDS WHEN ADJUSTING PLANS
```

**异常处理 (Quick Reference Table):**

| 情况 | 处理 |
|------|------|
| `_progress.md` 损坏 | 告诉用户哪里坏了，提供修复建议 |
| 计划已全部完成 | 祝贺 + 建议回顾薄弱项或设新目标 |
| 中途换目标 | 保留旧文件为 `_progress_old.md`，重新走 curriculum-builder |
| 长时间没学习（>7天） | 温和提醒，建议复习上次内容 |

### 5. note-generator/SKILL.md

**触发条件：** 用户说"今天学完了"、"总结吧"、"结束"

**流程：**
```
1. 回顾本次教学的所有知识点（从对话历史提取）
2. 生成结构化笔记
3. 确认文件名和路径
4. 写入文件
5. 更新 _progress.md 的学习记录
```

**笔记模板：**
```markdown
# [主题名称]

> 日期: YYYY-MM-DD
> 阶段: [当前阶段名称]

## 核心知识点

### 知识点1: [名称]
- [要点]
- 关键公式/代码（如有）

## 纠正过的错误
- [错误理解 + 正确解释]

## 下一步学习
- [ ] [下一个知识点]
```

**文件命名：** `YYYY-MM-DD-[主题简称].md`
**路径：** `_progress.md` 里记录的笔记目录

**Iron Law:**
```
NEVER GENERATE NOTES WITHOUT INCLUDING CORRECTED MISTAKES
```
