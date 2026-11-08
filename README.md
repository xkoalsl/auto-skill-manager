# Auto Skill Manager

让 AI Agent 具备"自我进化"能力：缺什么能力就自己去 GitHub 找靠谱的装上，旧了就自动清理。

## 这是什么？

这是一个可以安装到任何 AI Agent 工作台的 **skill 包**。装上之后，你的 AI 会：

1. **自动搜索** — 发现现有能力不够用时，主动去 GitHub 搜靠谱的 skill
2. **自动安装** — 筛选靠谱的、格式正确的 skill，装到本地
3. **自动追踪** — 记录每个 skill 的使用频率
4. **自动清理** — 90 天不用的自动标记，问你要不要删
5. **自动替换** — 发现更好的上位替代时，先装新的，问你要不要删旧的

## 支持哪些 AI 工作台？

任何支持 `SKILL.md` 格式（YAML frontmatter + Markdown 指令）的 AI Agent 工作台都能用，包括但不限于：

- Claude / Anthropic Agent
- Doubao Work / 豆包办公
- 其他自定义 Agent 框架

## 怎么安装？

### 方式一：手动安装

1. 把这个仓库 clone 或下载下来
2. 把 `SKILL.md` 放到你的 skills 目录下（通常是 `.user_skills/auto-skill-manager/SKILL.md`）
3. 把 `installed-skills.template.json` 复制一份，重命名为 `installed-skills.json`，放到同一个目录下
4. 让你的 AI 读一遍 `SKILL.md`，告诉它"以后按这个流程管理技能"

### 方式二：告诉你的 AI 直接读

直接对你的 AI 说：

> "去 https://github.com/xkoalsl/auto-skill-manager 这个仓库，读一下 SKILL.md，以后按里面的流程帮我管理技能。"

它会自己下载、安装、初始化。

## 文件结构

```
auto-skill-manager/
├── SKILL.md                          # 核心操作指南（AI 读这个）
├── installed-skills.template.json    # 已安装技能清单模板
└── README.md                         # 本文件（人类看的说明）
```

## 工作原理

```
用户任务 → AI 发现能力不够 → 去 GitHub 搜索 → 筛选靠谱的
    ↓
安装到本地 → 记录清单 → 告诉用户"我装了个新技能"
    ↓
每次使用 → 更新使用频率
    ↓
定期检查 → 90天没用的标记不活跃 → 问用户要不要删
    ↓
发现更好的替代 → 先装新的 → 标记旧的为已替代 → 问用户要不要删旧的
```

## 为什么需要这个？

AI 的能力不是一成不变的。随着任务越来越复杂，你会发现：

- 有些能力它不会，需要教
- 教了一次又一次，不如装个 skill 一劳永逸
- 有些 skill 装了之后再也没用过，占地方
- 有些老 skill 后来有了更好的版本

Auto Skill Manager 就是帮你自动处理这些事，不用你手动管。

## 安全吗？

- 只安装 star > 100、有维护、格式规范的 skill
- 安装前会先预览 SKILL.md 内容
- 删除任何东西之前都会先问用户
- 替换旧 skill 之前也会先问用户

它不会自作主张删东西，也不会装来路不明的东西。

## License

MIT
