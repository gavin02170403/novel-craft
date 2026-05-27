---
name: novel-craft
description: 长篇小说项目操作系统。规划、写作、审查、润色与台账维护工作流。不是代写工具，是持续维护世界观、人物、线索、节奏、风格、进度的小说生产管线。
---

# 写书工坊 · novel-craft

> 定位：长篇小说项目操作系统。
> 核心价值不是替用户写一章，而是持续维护：世界观、人物、线索、节奏、风格、台账、进度。

---

## 六阶段 + 隐藏路由

```
route/status（隐藏，每次触发先执行）
    ↓
init → plan → write → review → polish → maintain
```

| 阶段 | 核心任务 | 用户触发词 |
|------|---------|-----------|
| route | 诊断项目状态，判断该进哪个阶段 | 所有请求的隐式前置 |
| init | 生成项目骨架 + 最小可写信息 | "开新书" / "初始化" |
| plan | 总纲 / 卷纲 / 章纲（轻量默认，严格可选） | "规划第X卷" |
| write | 写前加载 → 白金流程起草 → 冷改 → 写后登记 | "写第X章" |
| review | 结构审查 / 硬红线 / 人物语音 / 检查单回溯 | "审第X章" |
| polish | 去AI味 / 衔接检查 / 节奏微调 | "润色第X章" |
| maintain | 线索轨道表 / 伏笔台账 / 时间线 / 人物状态 | "维护" / 异常触发 |

---

## 全局原则：最小读取

**每个阶段只读必要文件，禁止全项目扫描。**

| 阶段 | 必读文件（≤5个） | 不读（除非遇到具体问题） |
|------|----------------|------------------------|
| init | `.novel-craft/state.json`（如存在）、用户输入 | 无 |
| plan | `state.json`、`02-大纲/总纲.md`、本书不可变核心 | 正文全文、设定全文 |
| write | `state.json`、上一章末尾、本章章纲、线索轨道表命中行、出场人物语音锚点 | 细纲全文、伏笔总台账全文、设定全文 |
| review | `state.json`、待审正文、本章章纲、上一章末尾、人物语音锚点（出场角色） | 世界观全文、总纲全文 |
| polish | `state.json`、待润色正文、上一章末尾、下一章开头 | 章纲、轨道表 |
| maintain | `state.json`、线索轨道表、检查单回溯（最近10章）、事件时间线 | 正文全文 |

**写第X章时的精确读取规则：**
1. `state.json` —— 确认当前卷/章/模式
2. `03-正文/第{X-1}章` 最后一段 —— 语感
3. `02-大纲/第{V}卷-章纲.md` 中第X章条目 —— 方向
4. `04-台账/线索轨道表.md` —— 只读满足 `下次该出现 <= X` 且 `状态=active` 且 `紧张度>=3` 的行
5. `01-角色/人物语音锚点.md` —— 只读本章出场角色

---

## 阶段0：route/status（隐藏路由）

每次用户请求触发时，先执行状态诊断。不暴露给用户，只用于决定进入哪个阶段。

### 诊断清单

```
□ 当前目录是否有 .novel-craft/state.json？
  → 无 → 建议 init
□ state.json 中 book_title 是否非空？
  → 无 → 建议 init（骨架未完整）
□ 检测运行模式：
  → 项目目录或其父目录存在 .obsidian/ → 启用 Obsidian 模式
  → 无 .obsidian/ → 标准模式
□ state.json 中 next_required_action 是什么？
  → 直接按此字段路由（优先级最高）
□ 02-大纲/总纲.md 是否存在？
  → 无 → 建议 plan（总纲）
□ 目标卷纲是否存在？
  → 无 → 建议 plan（卷纲）
□ 目标章是否有章纲？
  → 无 → 建议 plan（章纲）
□ state.json chapter_status[X] 是什么？
  → 未写/null → 进入 write
  → draft → 进入 review
  → reviewed → 进入 polish
  → polished → 询问用户意图
□ 是否有 overdue 线索？（线索轨道表中 下次该出现 < 当前章 且 状态=active）
  → 有 → 警告并建议 maintain
□ 是否触发异常 maintain？见下方"异常触发条件"
  → 是 → 建议 maintain（优先级高于正常流程）
```

### 路由决策表

| 用户输入 | 诊断结果 | 进入阶段 |
|---------|---------|---------|
| "开新书" / "初始化" | 任意 | init |
| "规划" / "大纲" / "plan" | 缺总纲→plan总纲；缺卷纲→plan卷纲 | plan |
| "写第X章" / "draft X" | 有章纲→write；无章纲→先plan | write |
| "审第X章" / "review X" | 有草稿→review；无草稿→阻断 | review |
| "润色第X章" / "polish X" | 已审→polish；未审→建议先review | polish |
| "维护" / "maintain" / "台账" | 任意 | maintain |
| 模糊输入（如"继续"） | 按 next_required_action 推断 | 动态 |

---

## 阶段1：init（开书）

### 目标
生成项目骨架。只创建目录结构和最小可写信息，不要求用户第一天填满所有文件。

### 充分性闸门（init完成前必须满足）

1. 书名确定（可工作名）
2. 题材确定
3. 一句话故事
4. 结局锚点确定（不可偏离）
5. 主角姓名 + 核心欲望

### 执行步骤

**Step 1：信息采集（分2-3轮）**

| 轮次 | 采集项 | 必要性 |
|------|--------|--------|
| 1 | 书名、题材、一句话故事、结局锚点 | 必须 |
| 2 | 主角姓名、核心欲望、核心缺陷 | 必须 |
| 3 | 目标体量（卷数/章数）、目标平台/读者 | 可选 |

**Step 2：检测运行模式并生成项目骨架**

检测：项目目录或其父目录是否存在 `.obsidian/` 目录？
- **是** → `mode: "obsidian"`，启用增强约定
- **否** → `mode: "standard"`，纯 Markdown

两种模式共享核心流程，差异仅在文件格式和增强功能：

| 差异点 | 标准模式 | Obsidian 模式 |
|--------|---------|--------------|
| 章节文件 | 纯 Markdown | 带 YAML frontmatter |
| 线索管理 | 总表（表格） | 总表 + 独立笔记（Dataview） |
| 链接 | 无 | `[[双链]]` |
| 模板 | 基础模板 | Templater 兼容模板 |
| 状态追踪 | state.json | state.json + frontmatter |

创建7大目录（见 templates/init/ 模板）。关键文件：
- `CLAUDE.md` —— 项目级写作规范（Claude Code 读取，Obsidian 当普通笔记）
- `.novel-craft/state.json` —— 状态机
- `02-大纲/总纲.md` —— 仅填结局锚点、核心冲突、卷数划分
- `02-大纲/不可变核心.md` —— 比世界观更重要的锚点
- `04-台账/线索轨道表.md` —— 空表（plan阶段填入第一批）
- `04-台账/检查单回溯.md` —— 空模板

**Obsidian 模式额外生成**（可选，用户确认后）：
- `04-台账/线索/` 目录 —— 供 Dataview 查询的独立线索笔记
- 章节 frontmatter 模板 —— 写入 `templates/obsidian/章节笔记.md`
- 角色 frontmatter 模板 —— 写入 `templates/obsidian/角色笔记.md`

**Step 3：生成"本书不可变核心"**

写入 `02-大纲/不可变核心.md`（比世界观更重要，长篇写歪时最该回看）：

```markdown
## 本书不可变核心

- 一句话故事：
- 主角底层欲望：
- 主角错误信念：
- 最终结局锚点（不可偏离）：
- 核心爽点：
- 核心禁忌（绝不能出现的情节/设定）：
- 读者期待（读者打开本书时默认会得到什么）：
```

**Step 4：生成 state.json**

按模板填充（见 templates/init/.novel-craft/state.json）。

**Step 5：验证 + 登记 next_required_action**

验证骨架存在后：
```json
{"next_required_action": "plan"}
```

---

## 阶段2：plan（规划）

### 目标
产出可直接进入写作的章纲。先锁定卷级节奏，再批量拆章。

### 最小读取

- `state.json`（确认 outline_mode、当前卷）
- `02-大纲/不可变核心.md`
- `02-大纲/总纲.md`
- 已规划卷的章纲（如需跨卷一致性检查）

### 章纲模式

默认轻量细纲。满足任一条件时建议切 `strict`：
- 多主角或群像
- 悬疑/权谋/多线并行
- 单卷超过30章
- 伏笔线超过12条 active
- 时间线跨地域、跨阵营、跨年代
- 用户明确要求工程化大纲

**模式切换**：`state.json` 中 `"outline_mode": "lightweight" | "strict"`

格式见 `references/outline-modes.md`。

### 执行步骤

**Step 1：确认卷范围**
- 卷名、章范围、核心冲突、卷末高潮

**Step 2：生成卷节拍表**
- 中段反转（必须有）、危机链3次递增、卷末钩子
- 输出：`02-大纲/第{V}卷-节拍表.md`

**Step 3：生成卷时间线**
- 时间体系、本卷跨度、倒计时事件（如有）
- 输出：`02-大纲/第{V}卷-时间线.md`

**Step 4：批量拆章**
- 批次：默认10章/批，复杂题材8章/批
- 输出：`02-大纲/第{V}卷-章纲.md`
- 同时更新线索轨道表（投下本卷第一批伏笔）

**Step 5：登记状态**

```json
{
  "progress": {"current_volume": V},
  "chapter_status": {"N": "planned", ...},
  "next_required_action": "write"
}
```

---

## 阶段3：write（写作）

### 目标
产出可发布章节。完整执行：写前加载 → 白金流程起草 → 冷改 → 写后登记。

### 最小读取

- `state.json`
- `03-正文/第{X-1}章` 最后一段
- `02-大纲/第{V}卷-章纲.md` 第X章条目
- `04-台账/线索轨道表.md` —— 只读命中行
- `01-角色/人物语音锚点.md` —— 只读出场角色

### 执行步骤

**Step 1：白金流程起草**
- 从欲望开始：本章POV角色想要什么？
- 不中断、不修改、不回头看
- 卡住就写对话
- 写到"底"就停

**Step 2：冷改**
- 先砍："写得真好"的段落、重复解释、「他明白了/意识到/觉得」
- 再查：结尾在画面/对话上？人物串味？多余解释？
- 读出声：拗口改顺

**Step 3：检查单4问 + 章节分类**

写入 `04-台账/检查单回溯.md`：

| # | 问题 | 回答格式 |
|---|------|---------|
| 1 | 推动了哪条已有线索？ | 线索名：从 X → Y |
| 2 | 引入新线索了吗？ | 是/否。若"是"：预计N章内再次出现 |
| 3 | 章末读者最想知道什么？ | 1-3个问题 |
| 4 | 删掉本章哪条线会断？ | 线索名 / "无"（警惕） |

分类：累加 / 清零 / 转向 / 冗余

**Step 4：更新线索轨道表**
- 紧张度、上次出现、最近一次形态、下次该出现

**Step 5：登记状态**

```json
{
  "progress": {"current_chapter": X, "last_written": X},
  "chapter_status": {"X": "draft"},
  "next_required_action": "review"
}
```

---

## 阶段4：review（审查）

### 目标
AI逐条审查，输出固定格式报告。用户只做最终判断。

### 最小读取

- `state.json`
- 待审正文
- 本章章纲
- 上一章最后一段
- 人物语音锚点（出场角色）

### 执行步骤

**Step 1：结构审查**
- 章纲是否覆盖？
- 章节类型：累加/清零/转向/冗余？
- 时间连续性？
- 因果跳跃？

**Step 2：硬红线审查（逐条打勾）**

从 `references/review-rubric.md` 读取并执行：

```
□ 结尾是否总结式抒情？
□ 是否使用空泛大词？
□ 是否连续解释人物心理？
□ 是否有因果跳跃？
□ 人物口吻是否互换？
□ 结尾是否强行升华？
□ 场景转换是否无承接？
□ 主角是否无行动只思考？
□ 主角是否没有做选择？
□ 场景是否没有冲突或压力变化？
□ 情绪结论是否有身体/动作/场景承托？
□ 章末钩子是否和本章内容无因果关系？
□ 重要转折是否有前置信息？
□ 润色是否删除或模糊了因果链？（针对已润色章节复核）
```

**Step 3：人物语音审查**
- 出场角色逐一对照语音锚点
- 标记串味

**Step 4：输出固定格式报告**

```text
结论：通过 / 需小修 / 打回重写

必须改：
- [问题描述] | [原文证据] | [修复建议]

建议改：
- [问题描述] | [修复建议]

可保留：
- [问题描述] | [理由]

写入台账：
- [需要更新线索轨道表/伏笔台账/时间线的条目]
```

同时写入 `04-台账/检查单回溯.md` 的审查记录区。

**修改不覆盖原则**：review 和 polish 阶段默认不直接覆盖正文。处理方式：
- 必须改：生成 `第X章-{标题}.polished.md`，在副本上修改，原文保留
- 建议改：在审查报告中列出，由用户决定是否应用
- 用户明确说"直接改"时，才覆盖原文

**章节状态分离原则**：
```json
{
  "chapter_status": {"3": "reviewed"},
  "review_result": {"3": "pass_with_minor_revisions"}
}
```
- `chapter_status`：流程位置（draft/reviewed/polished/locked）
- `review_result`：质量判定（pass/pass_with_minor_revisions/fail）
- 两者独立，避免"通过但还在改"的状态混淆

**Step 5：登记状态**

```json
{
  "progress": {"last_reviewed": X},
  "chapter_status": {"X": "reviewed"},
  "next_required_action": "polish"
}
```

---

## 阶段5：polish（润色）

### 目标
去AI味 + 衔接检查 + 节奏微调。只改表达不改事实。

### 最小读取

- `state.json`
- 待润色正文
- 上一章末尾 + 下一章开头（衔接检查）

### 执行步骤

**Step 1：去AI味四规则**

从 `references/anti-ai-polish.md` 读取并执行。

**Step 2：衔接检查（3秒检查法）**
- 动机：他为什么突然做这个？
- 线索：上一章发现的东西还记得吗？
- 情绪：上一章那么强烈，这一章怎么延续？

**额外检查：润色是否删除或模糊因果链？**
- 对比润色前后，确认所有事件A→事件B的过渡仍然完整
- 如发现因果被砍，用"感官输入→身体反应→心理活动→行动决定"公式修复

**Step 3：节奏微调（软提醒）**
- 句式重复、"像"字密度、对话标签、信息解释长度

**Step 4：登记状态**

```json
{
  "chapter_status": {"X": "polished"},
  "next_required_action": "write"
}
```

---

## 阶段6：maintain（维护）

### 触发条件

**常规触发：**
- 当前章号 % 10 == 0
- 完成一卷时

**异常触发（优先级高于正常流程）：**
- 连续3章被判"冗余"
- 同一线索紧张度连续升高但没有形态变化
- 主要人物超过5章没有有效选择
- **主角连续2章没有主动选择或代价承担**
- 本卷目标连续3章没有推进
- 新增设定超过3条但没有写入台账
- 有 overdue 线索（下次该出现 < 当前章 且 active）

### 最小读取

- `state.json`
- `04-台账/线索轨道表.md`
- `04-台账/检查单回溯.md`（最近10章）
- `04-台账/事件时间线.md`

### 执行步骤

**Step 1：线索轨道表维护**
- 紧张度过低的线 → 安排激活
- 该交叉的线 → 制造交汇
- 休眠太久的线 → 标记盲区

**Step 2：伏笔台账维护**
- 新投伏笔写入总台账
- 该收的伏笔检查兑现窗口
- 被遗忘的伏笔（已投下但轨道表无记录）

**Step 3：事件时间线维护**
- 时间和章节对账
- 倒计时算术检查
- "时间真空"检查

**Step 4：人物状态维护**
- 语音是否按计划渐变？
- 有无"性格突变"无支撑？
- 关系变化是否连续？

**Step 5：交叉检查**
- 过去10章有无"悬空"章？
- 连续冗余章处理

**Step 6：生成维护报告**

写入 `06-管线/维护报告-第X卷-章Y-Z.md`。

**Step 7：登记状态**

```json
{
  "progress": {"last_maintained": X},
  "next_required_action": "write"
}
```

---

## 参考文件加载策略

| 阶段 | 读取的 reference | 路径 |
|------|-----------------|------|
| write | 白金流程提示 | 内嵌（短） |
| review | 硬红线清单 | `references/review-rubric.md` |
| polish | 去AI味手册 | `references/anti-ai-polish.md` |
| plan | 章纲模式 | `references/outline-modes.md` |
| init | 项目模板 | `templates/init/` |

---

## 状态机定义

```
initialized → planned → draft → reviewed → polished → locked
     ↑_________↓（返工路径）

maintain 是横向触发，不改变主线状态，但更新 last_maintained。
```

| 状态 | 含义 | 可进入的下一阶段 |
|------|------|----------------|
| initialized | 骨架已生成 | plan |
| planned | 已有章纲 | write |
| draft | 正文已起草 | review |
| reviewed | 审查完成 | write（返工） / polish |
| polished | 润色完成 | maintain |
| locked | 用户确认不再修改 | 只读 |

### state.json 核心字段

```json
{
  "book_title": "",
  "genre": "",
  "current_volume": 1,
  "current_chapter": 0,
  "outline_mode": "lightweight",
  "chapter_status": {
    "1": "draft|reviewed|polished|locked"
  },
  "review_result": {
    "1": "pass|pass_with_minor_revisions|fail"
  },
  "last_maintenance_chapter": 0,
  "next_required_action": "init|plan|write|review|polish|maintain",
  "next_chapter_directive": "",
  "locked_decisions": [],
  "open_questions": [],
  "mode": "standard"
}
```

- `locked_decisions`：已确定不可更改的设定（结局、主角底层欲望等）
- `open_questions`：尚未解决的创作问题
- `next_chapter_directive`：系统发现的创作建议，写入下一章的指导（如"避免连续累加"）
- `review_result`：独立于 `chapter_status` 的质量判定
- `mode`：`"standard"` 或 `"obsidian"`，init 时自动检测，后续可手动切换

---

## 失败恢复规则

| 失败场景 | 恢复动作 |
|---------|---------|
| 审查发现硬红线 | 生成 `.polished.md` 副本修改，原文保留；修改后重审 |
| 章纲缺失 | 阻断，建议先 plan |
| 线索轨道表损坏 | 从检查单回溯反向重建 |
| 连续3章被判"冗余" | 触发 maintain，强制调整卷节拍表 |
| 主角连续2章无主动选择 | 触发 maintain，下一章必须安排主动决策 |
| 人物语音系统性偏离 | 回到 maintain，修正语音锚点后重审 |
| 润色删除因果链 | 回到 polish 修复（副本），修复后不重审但需用户确认 |
