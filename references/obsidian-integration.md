# Obsidian 集成指南

> 核心原则：`novel-craft` 本体纯 Markdown、无插件依赖。Obsidian 是增强层，不是运行依赖。
>
> 触发条件：项目目录或其父目录存在 `.obsidian/` 目录时，自动启用 Obsidian 模式。

---

## 双模式架构

`novel-craft` 支持两种运行模式，init 时自动检测：

| 维度 | 标准模式 | Obsidian 模式 |
|------|---------|--------------|
| 触发条件 | 任意目录 | 目录或其父目录存在 `.obsidian/` |
| 章节文件 | 纯 Markdown | 带 YAML frontmatter |
| 线索管理 | 总表（Markdown 表格） | 总表 + `线索/*.md` 独立笔记（Dataview） |
| 链接 | 无 | `[[双链]]` |
| 模板 | 基础模板 | Templater 兼容模板 |
| 状态追踪 | `.novel-craft/state.json` | `state.json` + 章节 frontmatter |
| 模式切换 | 手动编辑 `state.json` | 手动编辑 `state.json` |

**关键设计**：两种模式共享同一套核心流程（init→plan→write→review→polish→maintain），差异仅在文件格式和可选增强功能。标准模式的用户随时可以把项目目录拖进 Obsidian，变成 Obsidian 模式，无需重建。

---

## 1. Vault 推荐结构

两种模式的目录结构相同，Obsidian 模式额外使用 frontmatter 和独立笔记。

```
{书名}/
├── .obsidian/               ← Obsidian 配置（可选）
├── .novel-craft/
│   └── state.json           ← 状态机（Claude Code 读取）
├── CLAUDE.md                ← 项目规范（Claude Code 读取，Obsidian 当普通笔记）
├── 00-设定/
│   └── 世界观.md
├── 01-角色/
│   └── 人物语音锚点.md
├── 02-大纲/
│   ├── 不可变核心.md
│   ├── 总纲.md
│   └── 第1卷-章纲.md
├── 03-正文/
│   ├── 第1章-标题.md        ← Obsidian 模式：带 frontmatter
│   └── 第2章-标题.md
├── 04-台账/
│   ├── 线索轨道表.md         ← 核心总表（Agent 读写）
│   ├── 线索/                ← Obsidian 模式：每条线索独立笔记（可选）
│   │   ├── C-001-敲墙声.md
│   │   └── C-002-名单.md
│   ├── 检查单回溯.md
│   └── 事件时间线.md
├── 05-文风/
│   ├── 写作红线.md
│   └── 去AI味规则.md
└── 06-管线/
    └── 进度看板.md
```

**说明**：
- `CLAUDE.md` 放在 vault 根目录，方便 Claude Code / Agent 在该目录工作时读取项目规范。Obsidian 只是把它当普通 Markdown 笔记打开。
- `04-台账/线索轨道表.md` 是 Agent 的核心读写对象，保持为 Markdown 表格。
- `04-台账/线索/` 是 Obsidian 增强层：每条重要线索另建独立笔记，带 frontmatter，供 Dataview 查询。
- `03-正文/` 下的章节文件在 Obsidian 模式下统一带 frontmatter。

---

## 2. 双链命名规范

在章纲和正文中使用 `[[...]]` 链接相关实体，Obsidian 会自动建立图谱关系。

| 实体类型 | 双链写法 | 目标位置 |
|---------|---------|---------|
| 角色 | `[[李太平]]` | `01-角色/李太平.md`（如存在） |
| 线索 | `[[C-001-敲墙声]]` | `04-台账/线索/C-001-敲墙声.md` |
| 地点 | `[[南天门]]` | `00-设定/南天门.md`（如存在） |
| 卷纲 | `[[第1卷-章纲]]` | `02-大纲/第1卷-章纲.md` |
| 前章 | `[[第5章-标题]]` | `03-正文/第5章-标题.md` |

**规范**：
- 角色名直接用本名，不加前缀。
- 线索笔记加 `C-` 前缀 + 线索ID，避免与人名冲突。
- 地点/势力如未建独立笔记，不强制建，避免空链泛滥。
- 章纲中写 `[[角色名]]` 时，Agent 读到后应去 `01-角色/` 查对应文件。

---

## 3. Dataview 可选增强

**核心总表（线索轨道表.md）仍是 Agent 的主读写对象**。Dataview 查询基于 `04-台账/线索/` 下的独立线索笔记。

### 线索笔记 frontmatter 规范

每条线索独立一篇笔记，文件名：`{线索ID}-{线索名}.md`

```markdown
---
type: clue
clue_id: C-001
clue_name: 敲墙声
clue_type: clue
current_form: "墙内部是黑色骨粉，第八天会爆开"
tension: 8
last_chapter: 35
next_chapter: 36
payoff_target: "第一卷末"
visibility: 暗示
status: active
related_chars: ["李太平", "老张"]
related_factions: ["天工司"]
---

# C-001 · 敲墙声

## 当前形态
{current_form}

## 历史出现
- ch2: 首次出现，三短一长
- ch16: 更近，频率加快
- ch35: 墙内部是黑色骨粉

## 计划
- ch36-40: 继续升温，与门缝白光交汇
```

### Dataview 查询示例

**写第12章前，查该出现的线索：**

```dataview
TABLE clue_id, tension, next_chapter, visibility
FROM "04-台账/线索"
WHERE type = "clue" AND status = "active" AND next_chapter <= 12 AND tension >= 3
SORT tension DESC
```

**查 overdue 线索：**

```dataview
TABLE clue_id, tension, last_chapter, next_chapter
FROM "04-台账/线索"
WHERE type = "clue" AND status = "active" AND next_chapter < 12
SORT next_chapter ASC
```

**查按读者可见度分布：**

```dataview
TABLE length(rows) as count
FROM "04-台账/线索"
WHERE type = "clue"
GROUP BY visibility
```

> 注意：Dataview 查询是只读的观察工具。Agent 仍应读写 `线索轨道表.md` 作为权威状态源。`线索/` 目录下的笔记由用户在 maintain 阶段手动同步，或作为可视化辅助。

---

## 4. Templater 可选模板

### 章节笔记模板

`templates/obsidian/章节笔记.md`

```markdown
---
type: chapter
volume: <% tp.file.title.split("第")[1].split("章")[0] %>
chapter: <% tp.file.title.split("第")[1].split("章")[0] %>
title: ""
status: draft
pov: ""
outline: "第<% tp.file.title.split("第")[1].split("章")[0] %>卷-章纲"
word_count: 0
reviewed: false
polished: false
---

# <% tp.file.title %>

## 写前（完成后删除）
- [ ] 读上一章最后一段
- [ ] 扫线索轨道表"下次该出现"
- [ ] 确认POV角色的欲望

## 正文

（在此处写正文）

## 写后检查单

1. 推动了哪条线索？
2. 引入新线索了吗？
3. 章末读者最想知道什么？
4. 删掉本章哪条线会断？

##  verdict
- [ ] 累加章
- [ ] 清零章
- [ ] 转向章
- [ ] 冗余章
```

### 角色笔记模板

`templates/obsidian/角色笔记.md`

```markdown
---
type: character
name: ""
role: protagonist|deuteragonist|antagonist|supporting
first_appearance: ""
status: active
---

# {角色名}

## 基础信息
- 姓名：
- 身份：
- 首次出场：

## 核心驱动
- 欲望：
- 缺陷：
- 错误信念：

## 语音锚点
- 说话特征：
- 紧张时的变化：
- 常见偏差（审查时对照）：

## 按卷变化
| 卷 | 语音特征 | 状态 |
|----|---------|------|
| 第一卷 | | |
| 第三卷 | | |
| 第五卷 | | |

## 关系网
- 与 [[主角]]：
- 与 [[反派]]：
```

### 线索笔记模板

`templates/obsidian/线索笔记.md`

```markdown
---
type: clue
clue_id: ""
clue_name: ""
clue_type: clue|char|item|place|power
current_form: ""
tension: 0
last_chapter: 0
next_chapter: 0
payoff_target: ""
visibility: 隐藏|暗示|半明|明牌|已兑现
status: active|dormant|resolved|abandoned
related_chars: []
related_factions: []
---

# {clue_id} · {clue_name}

## 当前形态
{current_form}

## 历史出现
- chX: {描述}

## 计划
- chX-X: {计划}
```

---

## 与 `novel-craft` 本体的关系

| 层面 | 本体（无Obsidian） | Obsidian增强 |
|------|-------------------|-------------|
| 状态机 | `.novel-craft/state.json` | 同左 |
| 线索状态 | `04-台账/线索轨道表.md`（总表） | 总表 + `04-台账/线索/*.md`（可视化） |
| 章节状态 | `state.json` 中 `chapter_status` | 章节文件 frontmatter 中 `status` |
| 项目规范 | `CLAUDE.md` | 同左 |
| 审查记录 | `04-台账/检查单回溯.md` | 同左 |
| 章纲 | `02-大纲/第V卷-章纲.md` | 同左（可加双链） |

### 状态源冲突规则

当 `state.json`、Obsidian frontmatter、`线索轨道表.md`、检查单回溯之间出现冲突时：

| 冲突场景 | 主状态源 | 处理方式 |
|---------|---------|---------|
| 章节状态不一致 | `state.json` 的 `chapter_status` | 以 state.json 为准，提示用户核对 |
| 线索紧张度不一致 | `04-台账/线索轨道表.md` | 以轨道表为准，state.json 不存线索细节 |
| 剧情事实不一致 | `04-台账/检查单回溯.md` + 正文 | 以正文和检查单为准，state.json 只存指针 |
| 设定冲突 | `02-大纲/不可变核心.md` | 以不可变核心为准，其他文件服从 |

**核心原则**：
1. `state.json` 是**路由主状态源**——决定下一步该做什么。
2. `线索轨道表.md` 是**剧情事实源**——记录线索的权威状态。
3. 正文 frontmatter 是**章节局部状态**——只影响该章的读写。
4. **冲突时先提示用户，不自动覆盖。** Agent 发现冲突后列出差异，等用户裁决。

**同步规则**：
1. Agent 以 `.novel-craft/state.json` 和 `线索轨道表.md` 为权威状态源。
2. Obsidian frontmatter 和 `线索/*.md` 是人类观察层，不反向写入状态机。
3. 用户在 maintain 阶段可以手动同步两边，但 Agent 不自动同步，避免冲突。

---

## 模式切换

**标准模式 → Obsidian 模式**：
1. 把项目目录拖进 Obsidian 作为 vault（或移入已有 vault）。
2. Obsidian 会自动创建 `.obsidian/` 目录。
3. 手动编辑 `state.json`，把 `"mode": "standard"` 改成 `"mode": "obsidian"`。
4. 可选：用 `templates/obsidian/` 下的模板生成 frontmatter 和独立线索笔记。

**Obsidian 模式 → 标准模式**：
1. 手动编辑 `state.json`，把 `"mode": "obsidian"` 改成 `"mode": "standard"`。
2. frontmatter 和双链不会破坏标准模式的运行（Agent 会忽略它们）。
3. 如需彻底清理，可批量删除 frontmatter 和 `线索/` 目录。

**关键**：模式切换不会丢失数据。frontmatter 和双链只是标准 Markdown 的兼容扩展，标准模式下 Agent 会跳过它们。
