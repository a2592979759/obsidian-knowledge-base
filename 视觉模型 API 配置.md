---
title: 视觉模型 API 配置（魔搭 / 阿里云百炼）
tags: [api, 视觉模型, 配置, 工具链]
---

# 视觉模型 API 配置（魔搭 / 阿里云百炼）

## 一、魔搭社区（ModelScope）
- 官网：https://modelscope.cn/home
- 步骤：
  1. 浏览器打开 `https://modelscope.cn` → 注册/登录
  2. 首次使用会提示绑定阿里云账号（必须，按页面引导完成）
  3. 右上角头像 → 个人中心 → 访问令牌（或直接访问 `https://modelscope.cn/my/myaccesstoken`）
  4. 点「新建访问令牌」→ 命名 → 生成 → 复制（格式 `ms-xxxxxxxxxxxx`）
  5. 把 Key 贴给 Claude（自动去掉 `ms-` 前缀写入配置）；或自己改 `.mcp.json` 里 `MODELSCOPE_API_KEY` 那行
- ⚠ 按 README 说明，令牌要去掉 `ms-` 前缀再用（ModelScope 是唯一后端）

## 二、阿里云百炼（DashScope）
- Key 是阿里云百炼（DashScope）的 API key —— 因为 `VISION_BASE_URL` 配的是 DashScope，模型是 `qwen-vl-max-latest`，要用阿里云的 key，不是 DeepSeek 或别家的
- 步骤：
  1. 打开阿里云百炼控制台：https://bailian.console.aliyun.com
  2. 用阿里云账号登录（没账号就先注册 + 实名认证）
  3. 左侧菜单找到「API-KEY 管理」（或「模型广场 / API-KEY」）
  4. 点「创建 API-KEY」
  5. 复制生成的 key（形如 `sk-xxxxxxxx`，只显示一次，先存好）
- ▎新账号通常有 `qwen-vl-max` 的免费额度（新用户赠送），够试用
- 填入 `C:\Users\Administrator\AppData\Local\agent-vision-toolkit\env`，把 `VISION_API_KEY=` 改成 `VISION_API_KEY=sk-你的key`
- 保存即可，改完不用重启 Claude Code（工具每次调用时读这个文件）
