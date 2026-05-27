# novel-craft · 长篇小说项目操作系统

> 不是"又一个写作提示词包"，而是一套可复用、可迭代、防漂移的长篇小说生产管线。
> 核心价值不是替你写一章，而是持续维护：世界观、人物、线索、节奏、风格、台账、进度。

---

## 定位

基于五卷 530 章长篇网文《天庭包工头》的实际写作经验提炼而成。

- **用户**：网文作者、类型小说创作者、长篇连载写手
- **场景**：多章节、多卷册、长周期写作项目
- **痛点**：Agent 用久了忘状态、误路由、模板填得不一致、设定漂移

---

## 核心特性

| 特性 | 说明 |
|------|------|
| **六阶段工作流** | `init → plan → write → review → polish → maintain`，每步有明确的进入/退出条件 |
| **最小读取原则** | 每个阶段只读 ≤5 个必要文件，禁止全项目扫描，防止上下文膨胀 |
| **线索轨道表** | 12 字段动态调度工具，控制伏笔的紧张度、出现频率、读者可见度 |
| **检查单回溯** | 每章回答 4 个问题，分类为累加/清零/转向/冗余，预警节奏堆积 |
| **双模式架构** | 标准模式（纯 Markdown）+ Obsidian 模式（frontmatter + 双链 + Dataview） |
| **状态机分离** | `chapter_status`（流程位置）与 `review_result`（质量判定）独立存储 |
| **去 AI 味四规则** | 标点限额、动作链条压缩、感官留白、斩断空镜余音 |

---

## 快速开始

### 1. 克隆仓库

```bash
git clone https://github.com/gavin02170403/novel-craft.git
```

### 2. 复制模板到你的写作目录

```bash
# 创建新书目录
mkdir 我的新书
cd 我的新书

# 复制初始化模板
cp -r ../novel-craft/templates/init/* .
```

### 3. 开始第一本书

编辑 `.novel-craft/state.json` 填入书名、题材，然后在支持 Claude Code 的编辑器中输入：

> "开新书"

Agent 会自动进入 `init` → `plan` → `write` 流程。

---

## 六阶段工作流

```
route/status（隐藏诊断，每次触发先执行）
    ↓
init → plan → write → review → polish → maintain
```

| 阶段 | 核心任务 | 触发词 |
|------|---------|--------|
| **init** | 生成项目骨架 + 最小可写信息 | "开新书" / "初始化" |
| **plan** | 总纲 / 卷纲 / 章纲（轻量默认，严格可选） | "规划第 X 卷" |
| **write** | 写前加载 → 白金流程起草 → 冷改 → 写后登记 | "写第 X 章" |
| **review** | 结构审查 / 硬红线 / 人物语音 / 检查单回溯 | "审第 X 章" |
| **polish** | 去 AI 味 / 衔接检查 / 节奏微调 | "润色第 X 章" |
| **maintain** | 线索轨道表 / 伏笔台账 / 时间线 / 人物状态 | "维护" |

---

## 目录结构

```
{书名}/
├── .novel-craft/
│   └── state.json              # 状态机（路由主状态源）
├── CLAUDE.md                   # 项目级写作规范
├── 00-设定/
│   └── 世界观.md
├── 01-角色/
│   └── 人物语音锚点.md
├── 02-大纲/
│   ├── 不可变核心.md            # 比世界观更重要的锚点
│   ├── 总纲.md
│   └── 第{V}卷-章纲.md
├── 03-正文/
│   └── 第X章-标题.md
├── 04-台账/
│   ├── 线索轨道表.md            # 剧情事实源
│   ├── 检查单回溯.md
│   └── 事件时间线.md
├── 05-文风/
│   ├── 写作红线.md
│   └── 去AI味规则.md
└── 06-管线/
    └── 进度看板.md
```

---

## 双模式架构

| 维度 | 标准模式 | Obsidian 模式 |
|------|---------|--------------|
| 触发条件 | 任意目录 | 目录或其父目录存在 `.obsidian/` |
| 章节文件 | 纯 Markdown | 带 YAML frontmatter |
| 链接 | 无 | `[[双链]]` |
| 模板 | 基础模板 | Templater 兼容模板 |
| 状态追踪 | `state.json` | `state.json` + 章节 frontmatter |

**关键**：两种模式共享同一套核心流程，差异仅在文件格式和可选增强功能。标准模式随时可拖进 Obsidian 升级为 Obsidian 模式，无需重建。

详见 [`references/obsidian-integration.md`](references/obsidian-integration.md)。

---

## 状态机

```json
{
  "book_title": "",
  "genre": "",
  "current_volume": 1,
  "current_chapter": 0,
  "outline_mode": "lightweight",
  "chapter_status": {"1": "draft|reviewed|polished|locked"},
  "review_result": {"1": "pass|pass_with_minor_revisions|fail"},
  "next_required_action": "init|plan|write|review|polish|maintain",
  "next_chapter_directive": "",
  "locked_decisions": [],
  "open_questions": [],
  "mode": "standard|obsidian"
}
```

---

## 参考文档

| 文档 | 内容 |
|------|------|
| [`SKILL.md`](SKILL.md) | 主控流程文件，六阶段完整定义 |
| [`references/obsidian-integration.md`](references/obsidian-integration.md) | 双模式架构与 Obsidian 增强指南 |
| [`references/outline-modes.md`](references/outline-modes.md) | 章纲轻量/严格双模式说明 |
| [`references/review-rubric.md`](references/review-rubric.md) | 14 条硬红线 + 7 条软提醒 |
| [`references/anti-ai-polish.md`](references/anti-ai-polish.md) | 去 AI 味四规则与改写公式 |
| [`references/acceptance-test.md`](references/acceptance-test.md) | v1.0 验收标准与回归测试清单 |

---

## 使用前提

- 支持 Claude Code 的编辑器（VS Code / JetBrains / Terminal）
- 无需额外插件、无需 Python 脚本、无需 RAG 依赖
- 纯 Markdown + JSON，任何文本编辑器都能打开

---

## 版本

当前版本：**v1.0**

基于《调色师》小样书完成验收测试，全部通过。

---

## License

MIT
