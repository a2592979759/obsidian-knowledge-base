---
title: Skill、Plugin 与 MCP
tags: [工具链, skill, mcp, plugin]
---

# Skill、Plugin 与 MCP

## 一、基础概念

### agent skill 与 mcp 的差异
- **MCP** 本质上是独立运行的程序，完成某个特定的功能
- **agent skill** 相当于一个说明脚本

### skill 与 plugin 的区别
- **skill** 是一个小模块
- **plugin** 是一个安装包，包含很多个 skill 和其他脚本，功能的集合（**plugins = skills + agents + mcp + Hooks**）

## 二、Skill 的安装
- **全局**：`用户/.claude/skills/skill_name/XXX.md`
- **项目局部**：`./skills/skill_name/XXX.md`
- 包管理工具：`npx / bunx / pnpm` 三种，自动下载防止手动反复操作
- 查找：GitHub 搜 `claude-skill` 或 `claude-plugins`；skill 社区网站看排行榜与安装量
- 自动化查找：`npx skill add https://github.com/vercel-labs/skills --skill find-skills`
- 建议安装：`npx skill add XXXX --skill pptx`（空格选择、回车确定）
- 手动安装：直接把 `/skill_name/XXX.md` 放置到全局或局部目录即可
- 可让 Claude 直接创建 skill（自己想到并实践过的方法）

## 三、Skill 的触发方式
1. **自动触发**
2. **手动触发** `/skill_name`；`/skills` 查看全部
3. `disable-model-invocation: true` 禁止自动触发，只能手动触发
4. **删除**：直接删除 skill 文件夹；plugin 安装的用 `/plugin` 管理命令
5. `/plugin`：查看 installed、discover（官方插件市场）

## 四、社区插件
- 官方市场：`/plugin install superpowers@claude-plugins-official`
- 第三方市场：`/plugin install superpowers@superpowers-marketplace`
- 已用插件：`superpowers`、`handoff`、`ponytail`（`grillme`、`caveman`、`mattpocock` 等）
- 已用 skill：`embedded-c-naming`、`embedded-soft-skills` 等

## 五、MCP 服务与 automation
- **认识**：MCP Server 是自己编写的服务，对外暴露「工具」（函数），Claude 通过 stdio/HTTP 连接并调用。协议标准，让 LLM 与外部能力解耦
- **MCP vs LangChain vs Workflow**：

|           | 是什么                     | 谁用             |
| --------- | ----------------------- | -------------- |
| MCP       | 协议标准，暴露工具给 LLM          | 你写服务，Claude 调用 |
| LangChain | 应用框架，构建 LLM 应用          | 你写应用，自己跑       |
| Workflow  | Claude Code 的多 Agent 编排 | Claude 自己用，你触发 |

- **RAG**（Retrieval-Augmented Generation，检索增强生成）：让 LLM 回答前先去查资料再回答。解决知识截止、幻觉两个硬伤。流程：用户问 → 搜知识库 → 内容+问题发给 LLM → 基于资料回答
- 理解三者后想扩展 Claude 能力（查数据库、调 API）就写 MCP Server；想要自己做 LLM 应用用 LangChain；Workflow 直接用即可

## 六、配置与持久化
- ccswitch 有会话内容配置、选择具体模型和 mcp 服务的配置
- 已用插件配置（`~/.claude/settings.json` 附近）：

```json
{
  "includeCoAuthoredBy": false,
  "enabledPlugins": {
    "superpowers@claude-plugins-official": true,
    "ponytail@ponytail": true,
    "handoff@handoff": true
  }
}
```

- 思考：怎么把这些存到 git？怎么让 agent 有存储概念？怎么自我学习（基于 plugin/skill）？怎么导出配置文件快速复用？
