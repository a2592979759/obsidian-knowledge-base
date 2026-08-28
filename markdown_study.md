---
date: 2026-08-25
title: markdown 学习
author: lqq
tags:
  - markdown
---
## 链接
- 插入[[obsidian_edit_2]](输入两个中括号) 按住crtl就可以查看链接内容（鼠标还能实现跳转）
- [[obsidian_edit_2#heading 1]]
- [[obsidian_edit_2#^fbb806]]


![[obsidian_edit_2]]


![[obsidian_edit_2#heading 1]]

![[obsidian_edit_2#^902304]]

[goole](https://www.google.com)
（此处使用了AnuPpuccin的topic）

## 文本编辑

### heading3
#### heading4

（此处使用了AnuPpuccin的topic会有不同颜色的显示）
==两个等号是高亮==
**两个星号是加粗**
_一个下划线为斜体_
*一个星号为斜体*
~~两个波浪线是删除线

也可以通过ctrl + p来进行查找

## 列表

无序列表
减号空格
- 可以安装Outliner插件显示缩进
	- 快速调整节点

有序列表
1 + . 空格

1. 序号1
2. 序号2

任务清单
- [ ] task1
- [x]  task2
减号空格中括号空格空格
- [ ] 也可以直接ctrl + p找代办


## 引用(quote)
右小箭头
>this is somthing i said
>--by lqq
>层级

插入标注

> [!NOTE] lqq标注测试
> 标注内容测试
> >
basic
(note 、abstrace、summary、info、todo、tip、warning、failure、fail、missing、danger、error、bug、example、quote、cite)

插入注释
sdf[^3]
[^1]:这是关于插入注释的相关说明

[^2]: sdf 

[^3]: 教主


插入图片直接使用notepad++拖进来即可

![[_assets/test_png2.png]]

插入表格(或者装对应的插件)

|     |     |
| --- | --- |
|     |     |
|     |     |


插入代码
行内代码inline code
一对单引号
`test sdfljzxkjdf lsd flksjdlf jsldjf lskdjfkjgsdkjfasjkg`

三个引号
插入代码块

```c
unsigned int g_uw_test_flag = 0;



```



