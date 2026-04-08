# project-analyzer-skill

一个面向本地代码仓库分析的通用 Skill 仓库。

当前仓库提供一个可安装 Skill：

- `project-analyzer`

这个 Skill 的目标很明确：

- 分析当前项目或指定路径下的项目
- 梳理项目入口、模块关系、执行链路
- 生成程序架构图和代码逻辑图
- 生成核心流程各阶段的数据结构说明与 mock 数据示例
- 将结果直接写入目标项目的 `docs/` 目录，而不只是临时在对话里回答

它是一个通用型 Skill，不绑定某个具体业务系统。

虽然这是通用 Skill，但它不应该忽略目标仓库已有的文档风格。如果目标项目里已经有架构说明、历史分析文档、纯文本图示样例，这些文件应该优先作为“表现形式”的参考，再结合源码核实技术事实。

## Skill 能产出什么

执行 `project-analyzer` 后，默认会在目标项目根目录下生成 `docs/` 目录，并写入一个或多个 Markdown 文档。

默认输出文件如下：

```text
docs/project-overview.md
docs/architecture-analysis.md
docs/mock-data-stages.md
```

这些文档适合用于：

- 新同学快速了解项目结构
- 给同事解释系统架构与代码逻辑
- 给 AI 工具准备稳定、可复用的项目文档
- 分析某个流程在不同阶段的数据长什么样

## Skill 适用场景

`project-analyzer` 适合做这些事：

- 分析一个陌生代码仓库
- 生成程序架构图
- 生成代码逻辑图
- 梳理主流程调用链
- 说明模块职责
- 输出某个核心流程每个阶段的数据结构
- 生成 mock 数据 demo
- 把分析结果写进项目文档目录

它不适合替代：

- 面向缺陷发现的代码 review
- 性能压测或 profiling
- 正式的架构决策记录
- 自动化 schema 采集工具

## Skill 的工作方式

Skill 的核心流程大致是：

1. 确定目标项目目录
2. 检查仓库结构
3. 查找入口文件和核心模块
4. 识别并梳理一条或多条主执行链路
5. 推断各阶段的数据形态
6. 将结果写入目标项目的 `docs/` 目录

默认规则：

- 如果用户没有传路径，就分析当前工作目录
- 如果用户传了路径，就分析指定目录

语言规则：

- 如果用户明确要求“用中文”或“用英文”，Skill 必须按指定语言输出
- 如果用户没有明确说语言，Skill 会优先参考目标仓库里已有文档的主语言
- 语言只影响标题、说明文字和注释，不影响源码标识符、文件名、函数名、队列名等真实技术名词

## 仓库结构

```text
project-analyzer-skill/
├── README.md
├── README.zh-CN.md
└── skills/
    └── project-analyzer/
        ├── SKILL.md
        ├── agents/
        │   └── openai.yaml
        └── references/
            ├── analysis-checklist.md
            ├── examples.md
            └── output-spec.md
```

## 安装方式

这个仓库已经按“后续推送到 GitHub 并通过 `npx` 安装”的思路组织好了。

你推送到 GitHub 后，一般可以按下面这种方式安装：

安装整个仓库：

```bash
npx skills add <你的GitHub用户名或组织>/project-analyzer-skill
```

如果安装器支持直接安装单个 skill 子目录，也可以这样：

```bash
npx skills add <你的GitHub用户名或组织>/project-analyzer-skill/skills/project-analyzer
```

请把 `<你的GitHub用户名或组织>` 替换成你自己的地址。

## 使用方式

这个 Skill 的设计目标就是“简单、好记、容易输入”。

分析当前项目：

```text
/project-analyzer
```

显式分析当前目录：

```text
/project-analyzer .
```

分析指定目录下的项目：

```text
/project-analyzer /path/to/repo
```

指定用中文输出：

```text
/project-analyzer . 用中文生成文档
```

指定用英文输出：

```text
/project-analyzer . generate docs in English
```

## 预期输出风格

输出文档应尽量做到：

- 用真实文件名和目录名
- 有真实入口文件和真实调用链
- 模块职责说明清晰
- 架构图和逻辑图可直接复制到 Markdown
- mock 数据结构贴近代码实际
- 对不确定部分明确标注“推断”或“示意”
- 如果目标仓库已有分析文档，新的 `docs/` 输出应尽量继承原有图风格和术语风格

如果目标仓库中已经存在下面这类文件：

- `architecture_analysis.md`
- `mock_data_stages.md`
- `.txt` 图示草稿

那么这些文件应优先作为以下方面的风格参考：

- 文档语言是中文还是英文
- 图是简洁风还是高注释密度风格
- 是否展示函数名、队列名、配置文件
- 是否使用分层块、泳道、宽分隔线
- 标题命名和章节组织方式

但如果你在调用时已经明确要求“用中文”或“用英文”，那么语言以你的显式指令为准，仓库已有文档只继续作为版式和图风格参考。

`mock-data-stages.md` 尤其建议区分：

- 代码中可以确认的数据结构
- 用于说明的 mock 示例值

## 主要文件说明

### `skills/project-analyzer/SKILL.md`

Skill 主体定义文件，包含：

- 触发条件
- 工作流程
- 输出要求
- 文档写入规则

### `skills/project-analyzer/references/output-spec.md`

定义输出文档的结构规范，包括：

- `project-overview.md`
- `architecture-analysis.md`
- `mock-data-stages.md`

### `skills/project-analyzer/references/analysis-checklist.md`

提供仓库分析时的检查清单，帮助避免遗漏：

- 项目结构
- 入口文件
- 核心逻辑
- 基础设施
- 输出质量检查

### `skills/project-analyzer/references/examples.md`

提供示例风格，帮助 Skill 生成更统一的：

- 程序架构图
- 代码逻辑图
- mock 数据阶段示例

## 本地维护建议

如果你后面想继续完善这个 Skill，建议优先修改这些文件：

- `skills/project-analyzer/SKILL.md`
- `skills/project-analyzer/references/output-spec.md`
- `skills/project-analyzer/references/analysis-checklist.md`
- `skills/project-analyzer/references/examples.md`

如果后面想加更丰富的 UI 元信息，可以再扩展：

- `skills/project-analyzer/agents/openai.yaml`

## 推送到 GitHub 的建议步骤

你准备好之后，可以按下面步骤推送：

```bash
cd ~/Untitled/devspace/project-analyzer-skill
git init
git add .
git commit -m "Initial project-analyzer skill"
git branch -M main
git remote add origin git@github.com:<你的GitHub用户名>/project-analyzer-skill.git
git push -u origin main
```

## 后续可增强方向

如果后面要继续打磨，这几个方向会比较有价值：

- 增加脚本，自动在目标项目里初始化 `docs/` 输出文件
- 增加时间戳命名，避免覆盖已有分析文档
- 增加针对常见技术栈的专项参考文档
- 增加更多示例输出，覆盖后端、前端、Worker 型项目
- 增加数据库、消息队列、第三方 API 的专门说明模板

## 设计思路说明

这个仓库刻意做了职责拆分：

- `SKILL.md` 负责触发和核心流程
- `references/` 放更长的补充说明
- `README` 面向人类维护者，方便发布和使用

这样做的好处是：

- Skill 本体保持简洁
- 仓库又足够好维护
- 后面发到 GitHub 后，别人看 README 也能很快理解怎么安装和使用
