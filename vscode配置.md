---
date: 2026-08-26
author: lqq
tags:
  - vscode
---

**工欲善其事必先利其器，所有好的插件可以大幅度提升效率**
## vscode官网

~~~http
https://code.visualstudio.com/
~~~



## vscode中`settings.json`其中的一些基础配置的基础配置

~~~json
{
  "files.autoSave": "afterDelay",
  "files.autoGuessEncoding": true,
  "workbench.list.smoothScrolling": true,
  "editor.cursorSmoothCaretAnimation": "on",
  "editor.smoothScrolling": true,
  "editor.cursorBlinking": "smooth",
  "editor.mouseWheelZoom": true,
  "editor.formatOnPaste": true,
  "editor.formatOnType": true,
  "editor.formatOnSave": true,
  "editor.wordWrap": "on",
  "editor.guides.bracketPairs": true,
  //"editor.bracketPairColorization.enabled": true, (此设置vscode在较新版本已默认开启)
  "editor.suggest.snippetsPreventQuickSuggestions": false,
  "editor.acceptSuggestionOnEnter": "smart",
  "editor.suggestSelection": "recentlyUsed",
  "window.dialogStyle": "custom",
  "debug.showBreakpointsInOverviewRuler": true,
}

~~~

在图示位置打开settings.json文件复制即可

![[_assets/vscode-settings.json配置.png]]



## vscode插件

~~~
插件种类繁多，对于各式各样的插件，我将我推荐的插件分成了四类，其中基础功能类是我认为在各种地方都很有帮助的插件，几乎可以是必备，其余的插件在很多情况下也很有用，可以按需安装
~~~

#### 外观类

##### 主题

One Dark Pro

GitHub Theme

Dracula Official

##### 图标主题

Material Icon Theme

vscode-icons

#### 背景

Vibrancy Continued

~~~
使vscode背景高斯模糊，非常吃性能（慎用

使用方法：按F1或者ctrl+shift+p，键入Reload Vibrancy，然后重启vscode

取消方法：按F1或者ctrl+shift+p，键入Disable Vibrancy，然后重启vscode
~~~

#### 基础功能类

Chinese (Simplified) (简体中文)

Error Lens

Path Intellisense

Image preview

#### 拓展功能类

CodeSnap

Prettier - Code formatter

GBK to UTF8 for vscode

Hex Editor

Doxygen Documentation Generator

Remote - SSH

Hungry Delete

#### 算法练习类

Code Runner

【代码运行工具】支持多种语言，语言运行环境需自己配置

推荐修改配置：

~~~
{
  "code-runner.runInTerminal": true,
  "code-runner.saveAllFilesBeforeRun": true,
  "code-runner.saveFileBeforeRun": true
}

~~~

Competitive Programming Helper (cph)

刷算法题时很好用，可以自己设置样例，一键全部运行