# Notion Batch Publishing and Blog Structure Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Publish the 15 approved Notion pages and reorganize the Hugo blog around six permanent categories.

**Architecture:** Keep all public articles in `content/post`, use Hugo categories as the single source of truth, and expose the six categories through the theme's home-category cards plus a compact top navigation. Existing articles are updated in place; empty Notion pages become minimal public stubs; images are copied to per-article directories under `static/images`.

**Tech Stack:** Hugo 0.157, Reimu theme, Markdown, TOML, Notion connector, Git.

**Spec:** `docs/superpowers/specs/2026-08-25-notion-batch-publishing-and-blog-structure-design.md`

## Global Constraints

- Main categories are exactly: `计算机基础`, `游戏客户端开发`, `技术美术`, `AI`, `个人作品开发文档`, `游戏美术`.
- `Git与P4V` and `设计模式` belong to `游戏客户端开发`.
- Every public article contains the exact AI workflow disclaimer once.
- Empty Notion pages receive only the disclaimer and the approved long-term-learning status sentence.
- Do not add `date`, `lastmod`, or `publishDate` fields.
- Preserve approved original titles and update existing articles in place.
- Store retained images locally; do not publish temporary Notion URLs.
- Do not modify the Unreal project or unrelated working-tree files.
- Push directly to `main`; do not create a pull request.

---

### Task 1: Configure the six-category site structure

**Files:**
- Modify: `hugo.toml`
- Verify: `public/index.html`, `public/categories/index.html`

**Interfaces:**
- Consumes: Reimu `params.menu` and `params.home_categories` configuration.
- Produces: compact navigation and six stable category cards.

- [ ] **Step 1: Record the failing structure check**

Run a read-only check that searches the generated homepage for all six category names and the header for 首页、分类、归档、关于. The current build is expected to miss the six category cards.

- [ ] **Step 2: Add exact Hugo parameters**

Add menu entries for `home`, `category`, `archives`, and `about`. Enable `params.home_categories` and add one content entry for each of the six approved category names. Set home and taxonomy sorting to `weight` so publication dates are not used for ordering.

- [ ] **Step 3: Build and verify navigation**

Run `hugo --minify`, then verify `public/index.html` contains each category name and `public/categories/index.html` exists.

- [ ] **Step 4: Commit**

Commit with message `建立博客六栏目结构`.

### Task 2: Publish game-client-development pages

**Files:**
- Modify: `content/post/design-pattern.md`
- Create: `content/post/Git与P4V.md`
- Create: `content/post/RPG实战.md`
- Create: `static/images/git-p4v/` retained image files

**Interfaces:**
- Consumes: Notion pages `3c579c19-f6f8-806a-a1bc-f0c71ead08db`, `3c579c19-f6f8-803a-b8e0-e7e8af98ccb2`, and `3c579c19-f6f8-80b1-89f1-fc09f6718f5b`.
- Produces: three public articles categorized as `游戏客户端开发`.

- [ ] **Step 1: Fetch current Notion contents**

Fetch all three pages immediately before editing. For `RPG实战`, record the six child-page titles and URLs for the project index.

- [ ] **Step 2: Update design patterns in place**

Keep the title `摇摇晃晃的设计模式`; merge the user's Notion additions with the existing blog article; remove textbook-style repetition and keep personal examples and conclusions.

- [ ] **Step 3: Create Git and RPG articles**

Create `Git与P4V` from the user's workflow notes and retain its relevant image locally. Create `RPG实战` as a project-learning index listing the six Notion child themes without inventing their missing content.

- [ ] **Step 4: Verify and commit**

Check exact categories, disclaimer count, local image existence, absence of date fields, and a successful Hugo build. Commit with message `发布游戏客户端开发笔记`.

### Task 3: Publish personal project documents

**Files:**
- Create: `content/post/2025.6FPS Demo开发文档.md`
- Modify: `content/post/雪景环境Demo开发文档.md`
- Create: `static/images/fps-demo/` retained image files

**Interfaces:**
- Consumes: Notion page `3c579c19-f6f8-803e-bcba-d5427e37e48d` and the already approved snow article.
- Produces: two public articles categorized as `个人作品开发文档`.

- [ ] **Step 1: Fetch and structure the FPS notes**

Organize only the recorded design, network synchronization, character, enemy AI, and gameplay-loop material. Mark incomplete systems as current goals rather than completed features.

- [ ] **Step 2: Localize the three FPS images**

Download meaningful project images into `static/images/fps-demo/`, give them descriptive names, and insert them beside the matching sections.

- [ ] **Step 3: Normalize project front matter**

Assign the FPS and snow articles to `个人作品开发文档`, add explicit weights, keep no date fields, and leave the approved snow body unchanged.

- [ ] **Step 4: Verify and commit**

Run the image-reference check and Hugo build. Commit with message `发布个人作品开发文档`.

### Task 4: Publish technical-art pages

**Files:**
- Create: `content/post/计算机图形学.md`
- Modify: `content/post/Niagara与Material.md`
- Modify: `content/post/Unity-Shader入门精要学习笔记.md`
- Delete: `content/note/Unity-Shader入门精要学习笔记.md`
- Create: `content/post/Houdini VEX.md`
- Modify: `content/post/凌乱的美术流程资源标准.md`
- Create/modify: `static/images/niagara-material/`, `static/images/unity-shader/`, `static/images/art-pipeline/`

**Interfaces:**
- Consumes: Notion pages `3c579c19-f6f8-80a9-91d2-edcb2ff5d554`, `3c579c19-f6f8-8002-b7d1-c33e76aa007d`, `3c579c19-f6f8-80cf-ba84-db9018e0dfef`, `3c579c19-f6f8-80bd-ac12-c98604092fca`, and `3c579c19-f6f8-80e6-b883-e6b0aa6391e6`.
- Produces: five unique public articles categorized as `技术美术`.

- [ ] **Step 1: Create exact empty stubs**

Create `计算机图形学` and `Houdini VEX` with front matter, the exact disclaimer, and only the approved long-term-learning sentence.

- [ ] **Step 2: Update Niagara and Shader articles**

Sync the latest Notion text and retained images. Remove the duplicate Shader file under `content/note` so Hugo publishes only one article for the topic.

- [ ] **Step 3: Update the art-pipeline article**

Keep the title `凌乱的美术流程资源标准`, retain the external-reference acknowledgement, remove tutorial transcription, and preserve the user's resource-standard conclusions.

- [ ] **Step 4: Verify and commit**

Check that all five articles use `技术美术`, image references are local and present, only one Shader article is generated, and Hugo builds successfully. Commit with message `发布技术美术学习笔记`.

### Task 5: Publish computer-foundation pages

**Files:**
- Create: `content/post/计算机组成原理.md`
- Create: `content/post/计算机网络.md`
- Create: `content/post/计算机操作系统.md`
- Modify: `content/post/令我秃头的数据结构与算法.md`

**Interfaces:**
- Consumes: Notion pages `3c579c19-f6f8-804b-8d8f-fe237dc8d622`, `3c579c19-f6f8-8069-ac7f-e95391aa3420`, `3c579c19-f6f8-80c3-987b-df27fcb88e82`, and `3c579c19-f6f8-802d-bac1-cfd17db1c222`.
- Produces: four public articles categorized as `计算机基础`.

- [ ] **Step 1: Create three empty stubs**

Create the three empty pages with no generated knowledge content.

- [ ] **Step 2: Update data structures in place**

Keep the title `令我秃头的数据结构与算法` and append only the current Notion notes about learning scope and memory addresses.

- [ ] **Step 3: Verify and commit**

Verify the category, disclaimer, empty-stub sentence, date-field absence, and Hugo build. Commit with message `建立计算机基础学习入口`.

### Task 6: Update the AI article and normalize legacy categories

**Files:**
- Modify: `content/post/扣扣嗖嗖的AI使用技巧与心得.md`
- Modify: `content/post/懵逼的客户端开发(Gameplay).md`
- Modify: `content/post/一言难尽的重构学习笔记.md`
- Update: other public post front matter as required by the six-category rule
- Create/modify: `static/images/ai-workflow/`

**Interfaces:**
- Consumes: Notion page `3b779c19-f6f8-8079-8361-e7080ba9e229` and existing public posts.
- Produces: no remaining public `学习笔记` category and one updated `AI` article.

- [ ] **Step 1: Update the AI article**

Sync the latest personal AI workflow notes, remove generic promotional text, retain the user's token-saving practices and approval workflow, and localize its meaningful image.

- [ ] **Step 2: Reclassify legacy posts**

Place Gameplay and refactoring under `游戏客户端开发`; ensure every other public post uses one of the six approved categories.

- [ ] **Step 3: Verify and commit**

Search front matter for `categories: [学习笔记]` and require zero results. Build Hugo and commit with message `统一博客文章分类`.

### Task 7: Final inventory, regression checks, and publication

**Files:**
- Modify: `docs/notion-blog-inventory.md`
- Verify: all `content/post/*.md`, `static/images/**`, and generated `public/**`

**Interfaces:**
- Consumes: all earlier task outputs.
- Produces: verified remote `main` publication.

- [ ] **Step 1: Update the inventory**

Mark all 15 screenshot pages as published, set their exact blog paths and image counts, and record empty-stub status where applicable.

- [ ] **Step 2: Run complete validation**

Run `hugo --minify`, `git diff --check`, a script that asserts one disclaimer and one approved category per public article, a scan for forbidden date fields, a local-image existence check, and a generated-permalink duplicate check.

- [ ] **Step 3: Inspect the generated site**

Confirm the homepage contains six category cards, the top navigation contains four entries, the category pages list the expected articles, and all 15 approved topics have public output.

- [ ] **Step 4: Commit and push**

Commit remaining inventory or validation corrections with message `完成Notion文章批量发布`, push `HEAD:main`, and verify the remote `refs/heads/main` commit hash.
