git和typora同步

https://github.com/yusenyi123/notebook/blob/master/Typora%E4%BD%BF%E7%94%A8/Typora%2Bgithub-%E4%BA%91%E7%AC%94%E8%AE%B0%E6%9C%AC.md

git的使用方法，包括上传和拉取代码，打通本地上传的路径

[视频同步笔记：狂神聊Git](https://mp.weixin.qq.com/s/Bf7uVhGiu47uOELjmC5uXQ)

# 视频同步笔记：狂神聊 Git

## 版本控制

### 什么是版本控制

版本控制（Revision control）是管理文件、目录或工程修改历史的软件工程技术，作用包括：

- 实现跨区域多人协同开发

- 追踪文件历史记录

- 保护源代码和文档

- 统计工作量

- 支持并行开发，提高效率

- 跟踪软件研发过程，降低人为错误

**缺乏版本控制的问题**：代码一致性问题、内容冗余、并发性冲突、安全性风险等。

### 常见版本控制工具

- **主流工具**：Git、SVN、CVS、VSS、TFS、Visual Studio Online

- **影响力最大的工具**：Git（分布式）与 SVN（集中式）

### 版本控制分类

#### 1. 本地版本控制

- **特点**：记录文件更新快照或补丁，适合个人使用（如 RCS）。

#### 2. 集中版本控制（如 SVN）

- **特点**：

- 所有版本数据存于中央服务器，用户本地仅同步版本

- 需联网查看历史版本或切换分支

- 风险：服务器损坏可能丢失全部数据

- **代表产品**：SVN、CVS、VSS

#### 3. 分布式版本控制（如 Git）

- **特点**：

- 每个用户本地保存完整版本库，可离线提交

- 联网时推送修改到其他服务器或用户

- 数据安全性高：任意用户设备可恢复全部数据

- **优势**：不受服务器损坏或网络问题影响

## Git 与 SVN 的主要区别

| **对比项**       | **SVN（集中式）**       | **Git（分布式）**          |
| ---------------- | ----------------------- | -------------------------- |
| **版本库位置**   | 中央服务器              | 每个用户本地都是完整版本库 |
| **联网需求**     | 必须联网工作            | 可离线工作，联网时同步修改 |
| **协同方式**     | 从服务器拉取 / 推送修改 | 本地修改后推送给其他用户   |
| **代码更新查看** | 需服务器支持            | 本地直接查看更新内容       |

## 聊聊 Git 的历史

- **诞生背景**：

- 2002 年，Linux 内核项目使用 BitKeeper 管理代码

- 2005 年，BitKeeper 收回免费使用权，Linus Torvalds 用 2 周开发 Git 替代

- **核心优势**：免费、开源，专为 Linux 内核开发，目前世界上最先进的分布式版本控制系统

- **创始人**：李纳斯・托沃兹（Linus Torvalds，1969 年出生于芬兰）

## Git 环境配置

### 软件下载

- **官网**：[git 官网](https://git-scm.com/)

- **国内镜像**：[淘宝镜像（Windows）](http://npm.taobao.org/mirrors/git-for-windows/)
- [CNPM Binaries Mirror](https://registry.npmmirror.com/binary.html?path=git-for-windows/v2.50.0.windows.1/)

### 安装与启动

- **安装**：无脑下一步即可

- **启动程序**：

- **Git Bash**：Unix/Linux 风格命令行（推荐）

- **Git CMD**：Windows 风格命令行（已 Deprecated）

- **Git GUI**：图形界面（不建议初学者优先使用）

### 常用 Linux 命令（在 Git Bash 中使用）

1. cd：改变目录

1. cd ..：回退到上一目录，cd直接进入默认目录

1. pwd：显示当前目录路径

1. ls/ll：列出当前目录文件（ll显示更详细）

1. touch：新建文件（如touch index.js）

1. rm：删除文件（如rm index.js）

1. mkdir：新建目录

1. rm -r：删除文件夹（如rm -r src）

**警告**：rm -rf / 会删除电脑全部文件，切勿在 Linux 中尝试！

1. mv：移动文件（如mv index.html src）

1. reset/clear：清屏

1. history：查看命令历史

1. help：获取帮助

1. exit：退出

## Git 配置

### 查看配置

- 查看所有配置：git config -l

- 查看系统配置：git config --system --list

- 查看当前用户配置：git config --global --list

### 配置文件位置

1. **系统级**：Git\etc\gitconfig

1. **用户级**：C:\Users\Administrator\.gitconfig（全局配置）

### 设置用户名与邮箱（必做）

```
git config --global user.name "用户名"  # 全局用户名
git config --global user.email "邮箱"    # 全局邮箱
```

**说明**：--global为全局配置，若针对特定项目，去掉该参数即可。

## Git 基本理论：三个区域与工作流程

### 四个工作区域（含远程）

1. **工作目录（Workspace）**：存放项目代码的本地目录

1. **暂存区（Stage/Index）**：临时存放改动的文件列表

1. **本地仓库（Repository）**：安全存放所有版本数据，HEAD 指向最新版本

1. **远程仓库（Remote）**：托管代码的服务器

### 工作流程

1. 在工作目录中添加 / 修改文件

1. 将文件放入暂存区（git add）

1. 提交暂存区内容到本地仓库（git commit）

1. 推送至远程仓库（git push）

### 文件三种状态

- **已修改（Modified）**：文件被修改但未暂存

- **已暂存（Staged）**：文件已放入暂存区

- **已提交（Committed）**：文件已存入本地仓库

## Git 项目搭建

### 创建工作目录

- 建议：目录不含中文，可直接为项目根目录

### 本地仓库搭建

#### 1. 新建仓库

```
git init  # 在项目根目录初始化Git仓库
```

初始化后会生成.git目录，存储版本控制信息。

#### 2. 克隆远程仓库

```
git clone [远程仓库URL]  # 示例：git clone https://gitee.com/kuangstudy/openclass.git
```

## Git 文件操作

### 文件四种状态

- **Untracked（未跟踪）**：文件在目录中但未加入 Git 库，git add后变为 Staged

- **Unmodified（未修改）**：文件已入库且未改动，修改后变为 Modified，git rm后变为 Untracked

- **Modified（已修改）**：文件已修改但未暂存，git add后变为 Staged，git checkout可恢复为 Unmodified

- **Staged（已暂存）**：文件在暂存区，git commit后变为 Unmodified，git reset后变为 Modified

### 查看文件状态

```
git status [filename]  # 查看指定文件状态
git status             # 查看所有文件状态
git add .              # 添加所有文件到暂存区
git commit -m "提交信息"  # 提交暂存区内容到本地仓库
```

### 忽略文件（.gitignore）

在项目根目录创建.gitignore文件，规则如下：

```
# 注释
*.txt        # 忽略所有.txt文件
!lib.txt     # 但lib.txt除外
/temp        # 仅忽略根目录下的temp文件
build/       # 忽略build目录下所有文件
doc/*.txt    # 忽略doc目录下的txt文件，但不包括doc/server/arch.txt
```

## 使用码云（Gitee）

### 1. 注册与配置

- 注册码云账号，完善个人信息

- 官网：[码云 Gitee](https://gitee.com/)

### 2. 设置 SSH 公钥（免密码登录）

```
# 进入SSH目录
cd ~/.ssh
# 生成公钥
ssh-keygen -t rsa
```

生成后将id_rsa.pub中的内容添加到码云账户的 SSH 公钥设置中。

### 3. 创建仓库

- 新建仓库时可设置：

- 仓库名称、描述、开源权限

- 添加.gitignore 和开源许可证（如 GPL-3.0）

- 初始化 README 文件

### 4. 克隆仓库到本地

```
git clone [仓库SSH地址]
```

## IDEA 中集成 Git

### 1. 新建项目并绑定 Git

- 将远程仓库代码拷贝到 IDEA 项目目录，IDEA 会自动识别 Git 仓库

### 2. 常用操作

- **添加到暂存区**：选中文件后右键 -> Git -> Add

- **提交到本地仓库**：Git -> Commit

- **推送到远程仓库**：Git -> Push

### 3. 冲突解决

当合并分支时文件被同时修改，需手动修改冲突文件，然后重新提交。

## Git 分支操作

### 分支概念

- 分支类似 “平行宇宙”，可独立开发功能，完成后合并到主分支

- **主分支（master）**：稳定版本，用于发布

- **开发分支（如 dev）**：日常开发使用

### 常用分支指令

```
git branch                # 列出所有本地分支
git branch -r             # 列出所有远程分支
git branch [分支名]       # 新建分支但不切换
git checkout -b [分支名]  # 新建并切换到分支
git merge [分支名]        # 合并指定分支到当前分支
git branch -d [分支名]    # 删除本地分支
git push origin --delete [分支名]  # 删除远程分支
git branch -dr [remote/分支名]  # 同步远程分支删除
```

### IDEA 中分支操作

- 在 IDEA 的 Git Branches 窗口中可：

- 新建、切换、合并分支

- 解决冲突文件

### 分支最佳实践

- 开发工作在 dev 分支进行，稳定后合并到 master

- master 分支仅用于发布新版本，不直接修改

## 作业练习

1. 与小伙伴协作搭建远程仓库，练习 Git 协同开发

1. 多实践常用命令，总结学习笔记到博客

> “有道无术，术尚可求；有术无道，止于术。”—— 狂神说

赠人玫瑰，手有余香 ❤️



以下是使用 Git 将本地文件更新到远程仓库的标准流程，按步骤说明：

### **1. 确认当前状态**

```
git status
```

- 查看工作区和暂存区的文件状态

- 确认要提交的文件已修改

### **2. 添加文件到暂存区**

```
git add <文件名>        # 添加单个文件
# 或
git add .               # 添加所有修改的文件（推荐）
```

### **3. 本地提交**

```
git commit -m "更新说明"
```

- **提交信息**应清晰描述修改内容（如："修复登录验证问题"、"添加用户列表页面"）

### **4. 拉取远程最新代码（重要！）**

```
git pull origin <分支名>
```

- **目的**：避免与远程仓库代码冲突

- 示例：拉取远程 main 分支

```
git pull origin main
```

- 若有冲突，需手动解决后再次提交

### **5. 推送本地提交到远程仓库**

```
git push origin <分支名>
```

- 示例：推送到远程 main 分支

```
git push origin main
```

### **首次推送新分支**

若本地分支是新建的，需额外关联远程分支：

```
git push -u origin <分支名>
```

- -u 参数会将本地分支与远程分支关联，之后可直接用 git push

### **完整示例**

假设你修改了项目中的 index.html 和 style.css 文件：

```
# 1. 查看状态
git status

# 2. 添加所有修改
git add .

# 3. 本地提交
git commit -m "优化首页样式和布局"

# 4. 拉取远程更新（假设在main分支）
git pull origin main

# 5. 推送本地提交
git push origin main
```

### **常见问题**

1. **冲突解决**：

- 拉取时若提示冲突，手动编辑冲突文件

- 标记冲突位置 <<<<<<<、=======、>>>>>>>

- 修改后 git add 和 git commit

1. **忘记关联远程分支**：

```
git branch --set-upstream-to=origin/<分支名>
```

1. **强制推送（慎用）**：

```
git push -f origin <分支名>  # 覆盖远程分支（可能丢失他人提交）
```

掌握以上步骤，即可完成本地文件到远程仓库的更新。核心是 **先拉后推**，避免冲突！



### 可以通过如下方法进行git中文乱码的解决

~~~
git config --global core.quotepath false
~~~

### 针对已有乱码文件的处理

若你不想重新提交这些文件，而是希望在已有的提交中修正显示问题，可使用以下命令：

~~~
git config --global i18n.logoutputencoding utf-8
git config --global i18n.commitencoding utf-8
~~~

或者

~~~
git mv "\344\270\200\345\221\250\351\245\256\351\243\237.md" 嵌入式系统学习.md
~~~

## 查看提交历史

~~~
git log --online
~~~

### 图形化显示分支和提交

~~~
git log --graph --online --all
~~~

### **使用标签管理版本**

git tag 是 Git 中用来给某个提交（commit）打标签的命令，通常用于标记重要的发布版本。

标签分为两种：轻量标签（lightweight tag） 和 附注标签（annotated tag）。

* 创建轻量标签

	~~~
	git tag v1.0
	~~~

* 推送标签到远程仓库

	~~~
	git push origin v1.0
	~~~

	**小贴士：** 使用语义化版本号（如 v1.0.0）可以更清晰地管理项目版本。

	如果想给某个特定的提交打标签，可以指定提交的哈希值：

	~~~
	git tag v1.0.0 <commit-hash>
	~~~

	指定某个提交并推送

	~~~
	git tag -a v1.0.0 <commit-hash> -m "Release version 1.0.0"
	git push origin v1.0.0
	~~~

​	一次性推送所有标签：

~~~
git push origin --tags
~~~

删除标签，它需要先删除本地标签，再推送删除操作：

~~~
git tag -d v1.0.0
git push origin --delete v1.0.0
~~~



通过 `.gitignore` 文件指定 Git 应忽略的文件或文件夹：

~~~
# 忽略 node_modules 文件夹
node_modules/
 
# 忽略环境配置文件
.env
~~~

将 `.gitignore` 文件添加到版本控制中：

~~~
git add .gitignore
git commit -m "添加 .gitignore 文件"
~~~

**小贴士：** 在项目初始化时就配置好 `.gitignore`，避免不必要的文件被提交。

当你需要频繁切换分支时，可以使用以下命令返回上一个分支：

~~~
git checkout -
~~~

搜索包含特定关键词的提交：

~~~
git log -S "关键词"
~~~

随着项目的推进，可能会产生许多不再需要的分支。清理这些分支可以让仓库更整洁：

- 删除本地分支：

	~~~
	git branch -d branch-name
	~~~

git branch -d：安全删除，当你分支的更改没有被合并到其他分支再删除，更常用。

git branch -D：强制删除，适合清理无用分支时使用。

- 删除远程分支：

	~~~
	git push origin --delete branch-name
	~~~

	**小贴士：** 定期清理无用分支，保持仓库的整洁和可维护性。

	最后，Git 是一门实践性很强的工具，只有多用、多练，才能真正掌握它的精髓。

**拉取远程更新并合并**

~~~
git pull origin master --allow-unrelated-histories
~~~

**处理合并冲突**
若拉取后出现冲突，Git 会标记冲突文件。编辑这些文件，解决冲突后提交

~~~
git add .
git commit -m "合并远程更新"
~~~

**再次推送**

~~~
git push origin master
~~~

### **更安全的工作流（避免冲突）**

建议在推送前先拉取远程更新：

~~~
# 1. 拉取远程更新（自动合并）
git pull --rebase origin master

# 2. 解决可能的冲突（若有）
git rebase --continue

# 3. 推送
git push origin master
~~~

### **补充命令**

**查看远程分支与本地差异**

~~~
git diff master origin/master

~~~

**查看远程仓库信息**

~~~
git remote -v
~~~

**强制推送（谨慎使用）**
若确定要覆盖远程历史：

~~~
git push -f origin master
~~~

通过上述步骤，你可以解决推送冲突并保持代码同步。
