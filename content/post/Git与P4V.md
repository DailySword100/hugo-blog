---
title: "Git与P4V"
categories: [游戏客户端开发]
tags: [Git, P4V, 版本控制]
weight: 30
draft: false
---

# Git与P4V

此博客为AI工作流自动管理，由博主的个人笔迹与GPT对话习惯自动整理，为博主复习整理查阅，必有错误，请注意鉴别是否AI幻觉。

-
	![版本控制工作流](/images/git-p4v/workflow.jpeg)
- 使用的代码仓库为gitee
- 下载代码仓库到本地，使用git clone xxxx地址xxxxxxxxxxxx
- 使用 git add. 将添加的新文件或者代码提交到本地
- 使用 git commit -am"提示内容" 也叫做修改代码的日志
- 使用git push 提交代码到远程仓库
- 使用git pull 拉取最新的代码内容
- 分支管理：
- 需要添加一个分支 在开发时只在自己的分支上进行开发 防止出现问题时正在开发难以处理 影响团队进度
- 那么如何创建一个新的分支 并且切换这个分支呢 ： git checkout -b 'name'
- 两个分支之间互不影响
- 要先切换到分支下 再拉代码
- 分支合并 ：先拿到分支 拉出来再切换回自己 再合并 git merge
- 注意分支冲突 是否保留双方更改 沟通
- 解决： 把前面的符号删掉就好了
- commit 后面的是版本号 git reset --hard 04619（前几个版本号）
-
- 强制上传方法
- 这个错误还是Gitee不支持Git LFS导致的（即使你只传核心文件，部分大资源仍被标记为LFS跟踪），解决方法是取消Git LFS的所有跟踪，改用普通Git上传：
- 步骤1：取消所有LFS跟踪
	- 复制以下命令到Git Bash执行：
	- bash
- 取消所有文件的LFS跟踪
	- git lfs untrack "\*.uasset"
	- git lfs untrack "\*.umap"
	- git lfs untrack "Content/\*"
	- git lfs untrack "Source/\*"
- （覆盖之前所有track的规则）
	- 步骤2：重新暂存并提交
		- bash
- 重新暂存文件（清除LFS标记）
	- git add .
	- git commit --amend --no-edit
	- 步骤3：强制推送（跳过LFS）
		- bash
- 强制推送，忽略LFS相关错误
	- git push -u origin main --no-verify
	- 说明
		- Gitee免费版不支持LFS，取消LFS跟踪后，文件会以普通Git方式上传。若仍提示文件过大，需在.gitignore中再过滤Content/里的超大资源（如单个超过100MB的.uasset），只保留必要的小资源文件。
-
- 成功上传
- 那这次咱们简化步骤，直接从本地Git推送到刚初始化好的Gitee仓库：
- 步骤1：回到本地项目的Git Bash
	- bash
- 1. 确保在项目根目录
	- cd D:/RPG/RPG
- 2. 初始化Git（若之前删了.git）
	- git init
- 3. 添加你要上传的文件
	- git add Source/
	- git add RPG.uproject # 替换成你的项目入口文件名
- 4. 提交
	- git commit -m "上传Source代码和项目入口"
	- 步骤2：关联Gitee仓库（用仓库的HTTPS地址）
		- bash
		- git remote add origin https://gitee.com/initially77/rpg.git
	- 步骤3：推送（因为远程有README，用强制推送覆盖）
		- bash
		- git push -u origin main --force
		- 这次远程仓库已经初始化，本地也明确只加了核心文件，应该能成功。要不要我帮你核对下命令里的仓库地址？
