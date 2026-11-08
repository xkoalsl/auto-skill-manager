---
name: auto-skill-manager
description: 自动技能管理器——当 AI 发现现有能力不够用时，自动去 GitHub 搜索、评估、安装靠谱的 agent skill；同时追踪使用频率，长期不用的自动清理，有上位替代的自动替换。适用于任何支持 SKILL.md 格式的 AI Agent 工作台。
---

# Auto Skill Manager

> 让 AI Agent 具备"自我进化"能力：缺什么能力就自己去 GitHub 找靠谱的装上，旧了就自动清理。

---

## 一、触发条件

当满足以下任一条件时，启动自动搜索流程：

1. 用户请求的任务，现有 skill 列表中没有匹配的
2. 现有 skill 能做但做起来明显低效/重复（比如同一个脚本写了三遍）
3. 用户明确说"这个能力你没有/你不会"

---

## 二、搜索策略

### 第一步：确认缺口

先盘点现有 skill 列表，确认确实没有能覆盖该任务的 skill。不要重复安装已有功能。

### 第二步：搜索 GitHub

使用 GitHub 搜索工具搜索：

```
query: "<关键词> SKILL.md in:readme stars:>50"
sort: stars
order: desc
perPage: 10
```

**优先检查的高可信仓库**（直接看这些仓库里有没有现成的）：

| 仓库 | 说明 | 可信度 |
|------|------|--------|
| `anthropics/skills` | Anthropic 官方 skill 仓库 | 官方 |
| `obra/superpowers` | 高 star agentic skills 框架 | 高 |
| `addyosmani/agent-skills` | 生产级工程技能 | 高 |

### 第三步：筛选靠谱仓库

按以下标准筛选（满足越多越靠谱）：

| 标准 | 最低要求 |
|------|----------|
| star 数 | > 100（官方仓库除外） |
| 是否 archived | 必须不是 archived |
| 最近更新 | 3 个月内有提交 |
| 格式验证 | 有标准 `SKILL.md` 文件，frontmatter 含 `name` + `description` |
| 文档质量 | 有 README 说明用法 |

**一票否决**：没有 SKILL.md 文件、格式不符合规范、代码明显不完整的，直接跳过。

---

## 三、安装流程

### 1. 下载单个 skill 文件夹

用 git sparse-checkout 只拉取需要的 skill 文件夹，不要整个克隆仓库：

```bash
# 在你的 .user_skills 目录下操作
cd .user_skills

# 方式一：单个独立仓库
git clone --depth 1 https://github.com/<owner>/<repo>.git temp-<skill-name>
# 然后把需要的 skill 文件夹移出来，删掉 temp

# 方式二：monorepo（多个 skill 在一个仓库里）
git clone --depth 1 --filter=blob:none --sparse https://github.com/<owner>/<repo>.git temp-<repo>
cd temp-<repo>
git sparse-checkout set skills/<skill-name>
# 然后把 skills/<skill-name> 移到 .user_skills 根目录
```

### 2. 验证格式

安装后必须验证：

- [ ] 文件夹根目录有 `SKILL.md` 文件
- [ ] SKILL.md 开头有 YAML frontmatter，包含 `name` 和 `description` 字段
- [ ] name 字段和文件夹名一致
- [ ] 没有多余的 README.md、CHANGELOG.md 等无关文件

验证不通过的，删除并告诉用户原因。

### 3. 记录安装

安装成功后，更新 `installed-skills.json`（从 template 复制一份），追加一条记录：

```json
{
  "name": "<skill-name>",
  "source_repo": "<owner/repo>",
  "source_path": "<repo内路径>",
  "installed_date": "YYYY-MM-DD",
  "last_updated": "YYYY-MM-DD",
  "last_used": "YYYY-MM-DD",
  "use_count": 0,
  "status": "active",
  "replaced_by": null,
  "description": "<一句话描述>"
}
```

### 4. 告诉用户

安装完成后，用一句话告诉用户：
"我发现缺 XX 能力，从 GitHub <仓库名> 装了一个 <skill名>，以后遇到同类任务会自动用上。"

---

## 四、使用频率追踪

每次**实际使用**某个自定义 skill（`.user_skills` 下的 skill）时，必须更新 `installed-skills.json`：

- `last_used` 更新为当天日期
- `use_count` 加 1

**注意**：只是"考虑用"但没真正触发的，不算使用。只有当你真正加载了该 skill 的 SKILL.md 并按它执行了任务，才算一次使用。

---

## 五、自我迭代：自动清理与替换

### 5.1 长期不用自动清理

**规则**：

- 当某个 skill 的 `last_used` 距今超过 90 天，且 `status: "active"`
- 自动将其 `status` 标记为 `"inactive"`
- 下次和用户对话时，主动提一句："我注意到 <skill名> 已经 90 天没用了，要不要删掉？"

**删除操作**：

- 用户确认后，删除 `.user_skills/<skill-name>/` 整个文件夹
- 从 `installed-skills.json` 中移除该条目
- 告诉用户已清理

### 5.2 有上位替代自动替换

**触发时机**：

- 当你因为某个任务去 GitHub 搜索新 skill 时，发现搜到的 skill 比已安装的同类 skill 更好

**"更好"的判断标准**（满足任意两条即可）：

- star 数是现有 skill 的 2 倍以上（且 > 500）
- 最近 3 个月有更新，而现有 skill 半年以上没动
- 功能覆盖更全面（现有 skill 只覆盖了部分场景）
- 社区评价明显更好（fork 数、issue 处理情况等）

**替换流程**：

1. 先安装新的替代 skill（按上面的安装流程）
2. 在 `installed-skills.json` 中，把旧 skill 的 `status` 标记为 `"superseded"`，`replaced_by` 填新 skill 的名字
3. 告诉用户："我发现 <新skill名> 比 <旧skill名> 更好（原因：XXX），已经装了新的。旧的那个要不要删掉？"
4. 用户确认后，删除旧 skill 文件夹和对应记录

**注意**：不要自动删除旧 skill，一定要先问用户。替换是"装新的 + 标记旧的"，删除需要用户确认。

### 5.3 损坏自动标记

当更新或使用某个 skill 时发现格式损坏（SKILL.md 缺失、frontmatter 错误等）：

- 将 `status` 标记为 `"broken"`
- 告诉用户："<skill名> 损坏了，原因：XXX。要不要我重新装一遍？"

---

## 六、定期自动维护

建议配置一个每周定时任务，执行以下操作：

1. **更新已安装的 skill**：对所有 `status: "active"` 的 skill，有 git 来源的执行 `git pull`
2. **检查使用频率**：遍历所有 skill，把超过 90 天没使用的标记为 `inactive`
3. **验证格式**：检查所有 active skill 的 SKILL.md 是否完好
4. **输出报告**：简洁列出本次更新了哪些、哪些标记为不活跃、哪些损坏了

---

## 七、注意事项

- **不要安装来路不明的 skill**：star 太低、没有维护、格式混乱的，宁愿自己写一个也不要装
- **不要重复安装**：装之前先查 `installed-skills.json` 和现有 skill 列表
- **安装前先预览**：先读一下 SKILL.md 内容，确认真的能解决问题，不要盲装
- **尊重许可证**：注意 LICENSE 文件，不要安装有传染性许可证的 skill
- **清理要谨慎**：长期不用不等于没用，一定要问用户再删
- **替换要透明**：装了新的替代旧的，一定要告诉用户原因，让用户决定删不删旧的
