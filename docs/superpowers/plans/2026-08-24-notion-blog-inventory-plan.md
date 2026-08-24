# Notion Blog Inventory Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** 清点 Notion 中的文章和集合，按六个固定栏目建立可追踪清单，并在博主逐篇审批后整理、提交和发布。

**Architecture:** Notion 是只读原始资料源，本地清单记录发现、归类、重复和发布状态，Hugo 博客保存经过筛选的公开版本。清点与发布分离：先建立全局索引，再一次处理一篇文章，每篇文章在内容审批前不得提交或推送。

**Tech Stack:** Notion connector、Hugo、Markdown、Git、Cloudflare Pages

**Spec:** `docs/superpowers/specs/2026-08-24-notion-blog-structure-design.md`

## Global Constraints

- 一级栏目名称和顺序固定为：计算机基础、游戏客户端开发、技术美术、AI、个人作品开发文档、游戏美术。
- Notion 仅作只读检索，不修改原始页面、数据库或属性。
- 每篇文章只能选择一个一级栏目，跨领域信息使用标签补充。
- 不发布没有学会、无法确认、明显照抄教程或不适合公开的内容。
- 不把文章标题改成模板化或过度 AI 化的标题，保留博主原有命名风格。
- 每篇公开文章注明：`此博客为AI工作流自动管理，由博主的个人笔迹与GPT对话习惯自动整理，为博主复习整理查阅，必有错误，请注意鉴别是否AI幻觉。`
- 一次只整理一篇文章；博主明确确认内容后，才允许提交和推送。
- 不创建 Git 审批 PR；内容审批由博主在当前任务中完成。
- Git 操作只暂存本篇文章及其图片，不包含 `.workbuddy/memory/MEMORY.md` 等无关改动。

---

### Task 1: 建立 Notion 全局文章清单

**Files:**
- Create: `docs/notion-blog-inventory.md`
- Read: `docs/superpowers/specs/2026-08-24-notion-blog-structure-design.md`

**Interfaces:**
- Consumes: Notion `search` 与 `fetch` 返回的页面 ID、标题、链接、父子关系和正文摘要。
- Produces: 一张按固定字段记录的 Markdown 清单，供后续去重、选篇和发布状态更新。

- [ ] **Step 1: 验证 Notion 连接**

调用 Notion `fetch`，参数为 `{"id":"self"}`。

Expected: 返回已连接工作区信息；如果不可访问，停止并要求博主重新连接 Notion。

- [ ] **Step 2: 分主题检索候选页面**

分别使用以下字面查询调用 Notion `search`，每次设置 `filters: {}`、`page_size: 25`，并合并去重结果：

```text
学习
游戏
美术
技术
AI
项目
笔记
```

Expected: 得到页面与数据库候选结果；相同页面 ID 只保留一次。

- [ ] **Step 3: 获取候选页面结构**

对每个 Notion 页面、数据库或数据源候选调用 `fetch`。记录页面标题、URL、父页面、子页面、数据库数据源和可见图片；外部连接源结果不传给 `fetch`。

Expected: 能区分独立文章、集合页、数据库和子文章。

- [ ] **Step 4: 写入统一清单**

创建 `docs/notion-blog-inventory.md`，每条记录严格使用以下字段：

```markdown
## 原始标题

- Notion：页面链接
- 类型：独立文章 / 集合页 / 子文章
- 建议栏目：六个固定栏目之一 / 待确认
- 完成状态：可整理 / 未学会 / 内容不足 / 待确认
- 博客对应：无 / 博客文章路径
- 重复关系：无 / 对应页面或文章
- 图片：无 / 有（数量）
- 公开建议：适合 / 整理后适合 / 暂不公开
- 处理状态：未处理
- 备注：一句话说明判断依据
```

Expected: 每个去重后的候选条目都有完整字段，没有空字段或虚构内容。

- [ ] **Step 5: 验证清单完整性**

运行：

```powershell
rg -n "^- (Notion|类型|建议栏目|完成状态|博客对应|重复关系|图片|公开建议|处理状态|备注)：" docs/notion-blog-inventory.md
```

Expected: 每篇条目都有 10 个字段；所有 Notion 链接均来自检索或抓取结果。

- [ ] **Step 6: 提交清单供博主审核**

只暂存 `docs/notion-blog-inventory.md`，检查暂存区后提交：

```powershell
git add -- docs/notion-blog-inventory.md
git diff --cached --check
git diff --cached --name-only
git commit -m "建立Notion文章清单"
```

Expected: 暂存区只包含清单文件；此提交不推送。

---

### Task 2: 对照现有博客并确定第一篇文章

**Files:**
- Modify: `docs/notion-blog-inventory.md`
- Read: `content/**/*.md`

**Interfaces:**
- Consumes: Task 1 的文章清单与现有 Hugo 文章标题、栏目、标签和正文。
- Produces: 标注重复关系的清单，以及一篇经过博主确认的首个处理对象。

- [ ] **Step 1: 提取现有博客文章信息**

运行：

```powershell
rg -n --glob "*.md" "^(title|categories|tags|draft):" content
```

Expected: 得到现有文章标题、栏目、标签和草稿状态；注意 `content/note` 与 `content/post` 可能存在重复文章。

- [ ] **Step 2: 对照标题和主题重复**

逐条比较 Notion 标题、主题摘要和博客正文，将清单中的“博客对应”和“重复关系”更新为真实路径或“无”。不因标题不同就自动判断为不同内容。

Expected: 已有博客内容、Notion 新增内容和可能的增量更新能够区分。

- [ ] **Step 3: 给出处理优先级**

优先顺序固定为：

1. 内容相对完整、适合公开且尚未出现在博客的文章。
2. 已有博客文章，但 Notion 存在明显新增内容的文章。
3. 需要较多整理才能公开的文章。
4. 未学会或不适合公开的文章，只保留记录，不进入发布队列。

Expected: 向博主展示前三个候选及判断理由，不擅自选择。

- [ ] **Step 4: 获取第一篇处理对象的明确确认**

等待博主明确选择一篇文章。

Expected: 未收到选择前不读取无关页面全文、不修改 Hugo 内容、不提交发布文件。

- [ ] **Step 5: 保存清单状态**

将选中文章的“处理状态”改为“已选，待整理”，只提交清单变更，不推送。

---

### Task 3: 整理一篇 Notion 文章并生成博客预览

**Files:**
- Modify or Create: `content/<section>/<article>.md`
- Create when needed: `static/images/<article-slug>/...`
- Modify: `docs/notion-blog-inventory.md`

**Interfaces:**
- Consumes: 博主在 Task 2 选中的单个 Notion 页面、其必要子页面和图片。
- Produces: 一篇未提交的 Hugo 文章预览，以及清单中的“待审批”状态。

- [ ] **Step 1: 获取选中文章完整内容**

调用 Notion `fetch` 读取选中页面；只有当正文明确引用子页面时，才继续读取这些子页面。

Expected: 保存真实层级、图片来源和原始标题，不修改 Notion。

- [ ] **Step 2: 标记不可发布内容**

识别并列出明显教程转录、未理解内容、重复段落、隐私信息和无法核实的 AI 补充。遇到无法从上下文判断的内容时，在改稿前询问博主。

Expected: 不把不确定内容悄悄改写成确定结论。

- [ ] **Step 3: 生成 Hugo 文章**

文章 front matter 使用一个固定一级栏目，并按内容添加标签；正文开头加入全局约束中的 AI 工作流声明。标题沿用 Notion 原名或博主指定名称，不主动美化。

Expected: 正文保留博主表达，结构清楚，教程照抄部分已移除，图片与对应段落准确匹配。

- [ ] **Step 4: 下载并核对图片**

只保存文章正文实际引用的图片到独立目录 `static/images/<article-slug>/`，按原文出现顺序命名。逐张核对图片内容、段落位置和 Markdown 引用。

Expected: 无错图、漏图、重复图；不下载未使用附件。

- [ ] **Step 5: 构建并检查预览**

运行：

```powershell
hugo --destination G:\tmp\pblog-build-notion-preview --cleanDestinationDir
git diff --check
git status --short
```

Expected: Hugo 构建退出码为 0；改动仅包含本篇文章、必要图片和清单状态。

- [ ] **Step 6: 提交内容给博主审批**

向博主提供标题、栏目、目录结构、删除或保留说明、图片数量和本地文章文件链接。

Expected: 审批前不执行 `git add`、`git commit` 或 `git push`。

---

### Task 4: 发布已审批文章并验证线上结果

**Files:**
- Modify or Create: Task 3 已审批的单篇文章文件与必要图片
- Modify: `docs/notion-blog-inventory.md`

**Interfaces:**
- Consumes: 博主对 Task 3 内容的明确确认。
- Produces: GitHub `main` 上的单篇文章提交，以及可访问的 Cloudflare Pages 页面。

- [ ] **Step 1: 重新验证审批范围**

运行 Hugo 构建、`git diff --check`、`git status --short`，并确认没有 `.workbuddy/memory/MEMORY.md` 或其他无关文件进入发布范围。

Expected: 构建成功，范围只包含已审批文章、必要图片和清单状态。

- [ ] **Step 2: 精确暂存文件**

使用 `git add -- <已审批文章路径> <必要图片路径> docs/notion-blog-inventory.md`，然后运行：

```powershell
git diff --cached --check
git diff --cached --name-only
git diff --cached --stat
```

Expected: 暂存区只包含本篇已审批内容。

- [ ] **Step 3: 提交并推送**

提交信息使用 `添加<文章原始标题>` 或 `补充<文章原始标题>`；随后推送当前 `HEAD` 到 `origin/main`，不创建 PR。

Expected: 推送成功，远端 `main` 指向新提交。

- [ ] **Step 4: 验证线上文章**

等待 Cloudflare Pages 部署后，读取首页与文章直链，使用提交短 SHA 作为查询参数绕过浏览器缓存。

Expected: 首页出现文章入口；文章直链返回 200；标题、正文关键段落和图片均存在。

- [ ] **Step 5: 更新清单状态**

将对应条目的“处理状态”更新为“已发布”，记录博客路径；若部署失败，状态改为“已提交，部署待处理”，不得误报发布成功。

---

### Task 5: 逐篇循环与阶段性结构检查

**Files:**
- Modify: `docs/notion-blog-inventory.md`
- Modify or Create: 每次仅一篇 `content/**/*.md` 及其必要图片

**Interfaces:**
- Consumes: 尚未处理的清单条目。
- Produces: 逐篇审批和发布记录，以及稳定的六栏目博客结构。

- [ ] **Step 1: 每次只选择下一篇候选**

重复 Task 2 的优先级展示，由博主选择下一篇，禁止批量自动发布。

- [ ] **Step 2: 对每篇重复整理与发布门禁**

对每篇文章完整执行 Task 3 和 Task 4，不因前一篇已获批准而继承审批权限。

- [ ] **Step 3: 每完成五篇检查栏目分布**

统计六个栏目文章数量、重复主题和未分类条目，向博主报告结构是否失衡；只提出调整建议，不擅自移动已发布文章。

- [ ] **Step 4: 保留长期增量入口**

新的 Notion 页面进入清单时默认状态为“未处理”，按同一流程继续，不改变既有审批标准。
