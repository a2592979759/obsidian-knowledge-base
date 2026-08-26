---
title: VSCode 环境与插件问题
tags: [vscode, 插件, 工具链]
---

# VSCode 环境与插件问题

## 一、C/C++ 与 clang 插件冲突
- 在 VSCode 中遇到 c/c++ 插件与 clang 插件冲突的问题，无法进行高亮跳转
- 但是 `alt + 左右`、`ctrl + 滑轮`、`ctrl + /` 功能正常，可暂时当一个文本编辑器用
- 解决思路：与其他的调试 IDE 同时打开，而 Claude 使用一个空的目录打开
- 后面如果需要其阅读文件啥的，可直接放置在此文件夹下
- 另外注意：能否引入 mcp 或其他 plugin/skill 使它增加类似解析 pdf 的功能
- VSCode 中似乎 claude 的功能是受限制的

## 二、VSCode 插件市场
- VSCode 的插件市场很广泛，几乎所有的功能都能搜到
- 可以抽空研究一下，但是要注意各插件之间可能存在的冲突问题

## 三、与 Claude 协同
- 在 VSCode 中通过可视化管理它的插件
- `ctrl + ,` 打开设置配置
- 在 vscode 中输入 `/` 查看安装的 skill
- 如果两边同时改了同一个文件，注意保存和刷新，避免覆盖

## 四、相关工具
- VSCode 内搜索 `claude code for vs code` 插件 → new session，可以看到代码的变化
- 写代码装 VSCode，找到 codex 扩展，装本地代理和 ccswitch
