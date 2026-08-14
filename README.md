# 个人知识库

多领域卡片盒(Zettelkasten)个人知识库,纯 Markdown + git 可迁移。

## 结构

- 领域目录:`技术/`、`工作/`、`生活/`、`学习/`,各含 `inbox/ 卡片/ 文献/ MOC/`。
- 记录区:`记录/` 下 `日志/ 周报/ 收支/ 单词/`,时间流组织。
- 附件:`_assets/`,相对路径引用。
- 模板:`模板/`,7 个。

## 用法

1. 用 Obsidian 打开本目录(作为 vault)。
2. 新建卡片:用 **Templater** 的 Insert template 插入 `卡片` 模板(核心 Templates 不展开 `<% tp. %>`),标题写主题名,aliases 同填主题名。
3. 链接用 `[[标题]]`(靠 aliases 解析到时间戳 ID 文件名)。
4. 日志由 Daily Notes 自动建;周报/收支/单词用对应模板。

## 插件(5 个,需在 Obsidian 内安装)

Obsidian Git、Templater、Dataview、Calendar、Paste image rename。

## 首次配置(仅第一次打开)

1. 安装 5 个社区插件(设置 → 第三方插件 → 浏览,搜 ID 安装并启用)。
2. Templater:设置 → Templater → Template folder location 填 `模板`。
3. 日历/日志:设置 → 核心插件 → Daily Notes → 笔记文件夹填 `记录/日志`(已通过 daily-notes.json 预配,核对即可)。
4. 模板文件夹(核心 Templates)已预配为 `模板`,核对即可。

## 换电脑恢复

```bash
git clone <remote-url> my-vault
```
然后用 Obsidian 打开 `my-vault` 目录即可;附件为相对路径,自动生效。首次打开须按「首次配置」安装 5 个插件。

## 同步

用 Obsidian Git 插件自动 commit/push/pull,或手动 `git add -A && git commit && git push`。
