# 个人知识库

多领域卡片盒(Zettelkasten)个人知识库,纯 Markdown + git 可迁移。

## 结构

- 领域目录:`技术/`、`工作/`、`生活/`、`学习/`,各含 `inbox/ 卡片/ 文献/ MOC/`。
- 记录区:`记录/` 下 `日志/ 周报/ 收支/ 单词/`,时间流组织。
- 附件:`_assets/`,相对路径引用。
- 模板:`模板/`,7 个。

## 用法

1. 用 Obsidian 打开本目录(作为 vault)。
2. 新建卡片:用模板 `卡片`,标题写主题名,aliases 同填主题名。
3. 链接用 `[[标题]]`(靠 aliases 解析到时间戳 ID 文件名)。
4. 日志由 Daily Notes 自动建;周报/收支/单词用对应模板。

## 插件(5 个,需在 Obsidian 内安装)

Obsidian Git、Templater、Dataview、Calendar、Paste image rename。

## 换电脑恢复

```bash
git clone <remote-url> my-vault
```
然后用 Obsidian 打开 `my-vault` 目录即可;附件为相对路径,自动生效。

## 同步

用 Obsidian Git 插件自动 commit/push/pull,或手动 `git add -A && git commit && git push`。
