# 1 claude常用的快捷键和指令
Claude Code 正在**帮你找 D 盘里名字带 `claude` 的文件夹**

它执行的命令是：

plaintext

```
find "D:/" -maxdepth 2 -type d -iname "*claude*"
```

翻译：

**在 D 盘里搜索名字包含 "claude" 的文件夹**

| 快捷键 / 指令               | 功能说明                     |     |
| ---------------------- | ------------------------ | --- |
| `!`                    | 进入 bash 模式执行命令           |     |
| `/`                    | 唤起 Claude 内置命令行          |     |
| `@`                    | 快速引用文件路径                 |     |
| `&`                    | 将执行的操作置于后台运行             |     |
| `/btw`                 | 提出附带的侧边问题（不中断主任务）        |     |
| `\\` + `⏎(回车)`         | 输入时插入换行符（不触发立即提交）        |     |
| 双击 `esc`               | 清空当前输入框内容                |     |
| `shift` + `tab`        | 自动接受编辑建议 / 补全内容          |     |
| `ctrl` + `shift` + `-` | 撤销上一步操作                  |     |
| `ctrl` + `o`           | 开启详细输出模式（展示更多执行信息）       |     |
| `alt` + `v`            | 粘贴图片到输入框                 |     |
| `alt` + `p`            | 切换使用的 AI 模型              |     |
| `ctrl` + `t`           | 切换任务状态（如暂停 / 继续）         |     |
| `ctrl` + `s`           | 暂存当前输入的提示词（不提交）          |     |
| `ctrl` + `g`           | 在系统默认编辑器（$EDITOR）中编辑输入内容 |     |
| `/keybindings`         | 打开自定义快捷键的配置界面            |     |
|                        |                          |     |
# 2 claude 在执行任务时候可以并行的回答一些小问题
**原文**：Tip: Use /btw to ask a quick side question without interrupting Claude's current work

**译文**：小提示：使用 /btw 指令可提出简短的附带问题，且不会中断 Claude 当前的工作进程

### 关键细节拆解

1. **/btw**：Claude Code 专属内置指令，核心作用是 **「分轨提问」**，和主任务互不干扰；
2. **quick side question**：特指**简短、非核心的附带问题**（比如主任务是写代码，顺带问某个函数用法、命令含义，都适合用 /btw）；
3. **without interrupting...current work**：Claude 会优先完成当前正在处理的核心任务（如代码编写、脚本运行、文档解析），同时单独响应 /btw 的问题，不会因新问题暂停主任务，也不会让主任务的上下文被附带问题覆盖。
 命令在做什么

bash

运行

```
cd "C:\Users\13411" && git status 2>/dev/null || echo "Not a git repository"
```

**作用：**

1. 进入你的用户目录
2. 检查**当前文件夹是不是 Git 仓库**
3. 如果是 → 显示状态
4. 如果不是 → 显示 `Not a git repository`
# 3
