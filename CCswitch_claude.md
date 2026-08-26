---
tags:
  - ccswitch
  - claude
  - setting
author: lqq
date: 2026-08-26
---
## CCswitch and claude setting

~~~json
{
  "enabledPlugins": {
    "handoff@handoff": true,
    "ponytail@ponytail": true,
    "superpowers@claude-plugins-official": true
  },
  "env": {
    "ANTHROPIC_AUTH_TOKEN": "sk-b082aa5af5284d228cf957884c5b21d5",
    "ANTHROPIC_BASE_URL": "https://api.deepseek.com/anthropic",
    "ANTHROPIC_DEFAULT_FABLE_MODEL": "deepseek-v4-flash",
    "ANTHROPIC_DEFAULT_FABLE_MODEL_NAME": "deepseek-v4-flash",
    "ANTHROPIC_DEFAULT_HAIKU_MODEL": "deepseek-v4-flash",
    "ANTHROPIC_DEFAULT_OPUS_MODEL": "deepseek-v4-flash",
    "ANTHROPIC_DEFAULT_OPUS_MODEL_NAME": "deepseek-v4-flash",
    "ANTHROPIC_DEFAULT_SONNET_MODEL": "deepseek-v4-flash",
    "ANTHROPIC_DEFAULT_SONNET_MODEL_NAME": "deepseek-v4-flash",
    "ANTHROPIC_MODEL": "deepseek-v4-flash",
    "CLAUDE_CODE_SUBAGENT_MODEL": "deepseek-v4-flash"
  },
  "includeCoAuthoredBy": false
}
~~~

更换了供应商添加了deepseek有关的多模态api
~~~json
{
  "enabledPlugins": {
    "handoff@handoff": true,
    "ponytail@ponytail": true,
    "superpowers@claude-plugins-official": true
  },
  "env": {
    "ANTHROPIC_AUTH_TOKEN": "sk-b082aa5af5284d228cf957884c5b21d5",
    "ANTHROPIC_BASE_URL": "https://api.deepseek.com/anthropic",
    "ANTHROPIC_DEFAULT_FABLE_MODEL": "deepseek-v4-flash",
    "ANTHROPIC_DEFAULT_FABLE_MODEL_NAME": "deepseek-v4-flash",
    "ANTHROPIC_DEFAULT_HAIKU_MODEL": "deepseek-v4-flash",
    "ANTHROPIC_DEFAULT_OPUS_MODEL": "deepseek-v4-flash-vision-exp",
    "ANTHROPIC_DEFAULT_OPUS_MODEL_NAME": "deepseek-v4-flash-vision-exp",
    "ANTHROPIC_DEFAULT_SONNET_MODEL": "deepseek-v4-flash",
    "ANTHROPIC_DEFAULT_SONNET_MODEL_NAME": "deepseek-v4-flash",
    "ANTHROPIC_MODEL": "deepseek-v4-flash-vision-exp",
    "CLAUDE_CODE_SUBAGENT_MODEL": "deepseek-v4-flash"
  },
  "includeCoAuthoredBy": false
}
~~~
