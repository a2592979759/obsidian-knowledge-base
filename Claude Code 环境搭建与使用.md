---
title: Claude Code 环境搭建与使用
tags: [工具链, claude, 环境]
---

# Claude Code 环境搭建与使用

## 零、搭建环境
1. 在 GitHub 下载 win32-x64 的 `claude.exe`
2. 下载 **ccswitch** 并接入 API，添加路由
3. 在打开 ccswitch 的前提下打开 Claude Code

## 一、环境变量配置
- 在 GitHub 找到 Claude 的根目录，添加到用户 PATH 环境变量（开始菜单搜索「环境变量」）
- 或在新建项目文件夹路径栏输入 `cmd`，再输入 `claude`，可直接在当前路径打开

## 二、在 VSCode / CMD 中使用
- VSCode：`claude code for vs code` 插件，`new session`；**优点是可以看到代码变化**。注意其功能可能受限
- CMD：网页安装 `npx skill add XXXX --skill pptx`（空格选择、回车确定）
- VSCode 中输 `/` 查看已装 skill；Claude 中输 `/skills` 查看
- `!` 切换终端模式，可执行终端命令
- 打开插件：Claude 输 `/plugin` 弹出可视化插件；或 `ctrl + ,` 打开设置配置

## 三、Claude Code 使用核心
- **模式**：自动 / 询问 / 先讨论（计划模式，`shift + tab` 切换）；跳过审批用三种模式
- `yes` 允许 or 其他
- `/rewind` 回滚
- `/effort` 修改思考强度
- `/resume` 查看最近历史；`claude --continue` 直接打开上次会话
- `/exit` 后还可在 cmd / powershell 用 `claude`
- `/init` 生成 `CLAUDE.md`（`@CLAUDE.md` 可让 Claude 直接生成；可加 `## 注意事项`）
- `/context` 查看上下文；`/compact` 压缩上下文；仍不行则 `/clear`
- `/agents` 管理 Agent（library / create new agent / continue 选模型 / alt+回车清权限 / esc 退出）
- `/plugins` 打开插件相关；**plugins = skills + agents + mcp + Hooks**
- 换行：`shift + 回车` 或 `ctrl + j`；`ctrl + g` 在文本编辑器编辑
- 在 terminal 中可选择对应模型

## 四、多工具协同（VSCode + Claude + IDE）
- 用 VSCode 打开工程项目，配合 Qt 或其他 IDE 调试验证，Claude 直接修改
- 两边同时改同一文件时注意保存与刷新，避免覆盖
- 可直接询问 Claude 添加插件、git 操作、代码仓库管理与迭代
