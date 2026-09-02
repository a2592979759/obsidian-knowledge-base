# 日志 App（Demo）设计文档

> 日期：2026-09-02
> 状态：定稿，待用户审
> 类型：Flutter 安卓端个人日记 App（**纯本地存储 Demo**）
> 目标：最快能跑通的验证版本，不做备份、不做同步、不做日历图

## 1. 目标与范围

一个安卓手机端的个人生活日记 App，**只存本地**。Demo 版唯一目标：**能写一篇日记（文字+心情+一张照片），存下来，能再看到**。

**v1 Demo 范围：**

- ✅ 写日记：日期 + 标题 + 正文 + 心情（emoji）+ 标签（逗号分隔）+ 一张照片
- ✅ 日记列表：按日期倒序显示已有日记
- ✅ 查看/编辑：点开一篇看详情、可编辑
- ✅ 删除：能删一篇
- ❌ 不做日历视图
- ❌ 不做标签筛选 / 搜索
- ❌ 不做照片多选、不做相册访问（只拍照或选文件夹）
- ❌ 不做备份 / 导出 / 同步
- ❌ 不做数据库复杂关联（标签、照片全用逗号分隔字符串单表存）

## 2. 技术选型

| 项 | 选择 | 理由 |
|----|------|------|
| 框架 | Flutter | 单套跑安卓，UI 快，热重载 |
| 本地存储 | **sqflite** | 轻量，单表足够；不引入重型 drift ORM |
| 状态管理 | setState | Demo 无需 provider/riverpod |
| 照片存储 | 应用私有目录 `[documents]/photos/` | 不依赖系统相册权限，自己选/拍即存 |

## 3. 数据模型（单表）

日记 = 一行记录，标签和照片用逗号分隔字符串存（**牺牲规范化换简单，demo 可接受**）。

```dart
// demo.db 表 entries
class DiaryEntry {
  int id;            // 主键自增
  String date;       // 'yyyy-MM-dd'，唯一（一天一篇）
  String title;
  String content;
  String mood;       // 'happy' | 'calm' | 'sad'
  String tags;       // '工作,生活' —— 逗号分隔字符串
  String photo;      // 照片文件名（存 photos/ 目录）或空
}
```

**表结构：**

```sql
CREATE TABLE entries (
  id      INTEGER PRIMARY KEY AUTOINCREMENT,
  date    TEXT UNIQUE NOT NULL,
  title   TEXT,
  content TEXT,
  mood    TEXT,
  tags    TEXT,
  photo   TEXT
);
```

**依赖：**
- `sqflite`：SQLite
- `path_provider`：拿应用文档目录
- `image_picker`：拍照选择照片

## 4. 架构（极简分层）

```
UI (setState)
   │  只调 Repository
   ▼
DiaryRepository  (业务编排：组装数据、读写照片)
   │        │
   ▼        ▼
 DiaryDao   PhotoStore
 (SQLite)   (文件读写)
```

- **DiaryDao**：`insert/getAll/update/delete` 操作 entries 表
- **PhotoStore**：把选中的照片拷贝到 `photos/`，文件名 = `日期_时间戳.jpg`；删除日记时删对应文件（仅 demo 简单处理）
- **Repository**：调 DAO 拿数据 + 调 PhotoStore 管照片，返回给 UI

## 5. 页面与数据流

### 页面（3 个）

1. **首页列表**：展示所有日记卡片（日期 + 标题 + 心情 emoji 预览），倒序。点「+」进写日记。长按或按钮删除。
2. **写/编辑日记页**：日期（默认今天）、标题、正文、心情（3 选 1 的 emoji）、标签（逗号输入）、照片（拍照/选图）。点保存。
3. **详情页**（可选，demo 可合并到编辑页）：直接点卡片进编辑即可，详情用编辑页承载。

### 数据流

```
打开 App → Repository.getAll() → 列表页展示 (倒序)
点「+」→ 写日记页 → 填好点保存
        → Repository.add(entry)
             ├─ photo? → PhotoStore.save() → 返回文件名
             └─ DiaryDao.insert(entry)
        → 回列表页刷新
```

## 6. 错误处理（demo 够用）

- **拍照/选图失败**：捕获异常，提示"照片选取失败"，正文仍可保存（照片为空）。
- **保存失败**（如当天已存在，date 唯一冲突）：catch，提示用户，不崩溃。
- **删除照片文件失败**：忽略，不阻塞（demo 留着垃圾不管）。
- 日期唯一用 `UNIQUE` 约束兜底；编辑时若改到另一天也做冲突提示。

## 7. 测试与验证

Demo 验证 = 能跑起来 + 手动走通主流程：

- [ ] `flutter run` 启动 App
- [ ] 写一篇：填标题/正文/心情/标签，拍一张照，保存
- [ ] 再写一篇，确认列表倒序出现两篇
- [ ] 点开某一篇编辑，改正文，保存，确认更新
- [ ] 删一篇，确认列表消失、photos/ 里关联图片一并删除
- [ ] 杀掉 App 重开，确认数据还在（本地持久化）

不写自动化测试（demo 阶段 YAGNI）。以上用手动核对替代。

## 8. 交付清单

- [ ] Flutter 项目骨架（`flutter create`）
- [ ] `pubspec.yaml` 加 sqflite / path_provider / image_picker
- [ ] 数据库 helper 类（建表、CRUD）
- [ ] PhotoStore（拷贝、删除照片文件）
- [ ] Repository
- [ ] 3 个页面 + 路由
- [ ] 跑通主流程

## 9. 范围边界 / 明确的"不做"

- 不做多照片、不做图片压缩。
- 不做 mood 趋势图。
- 不做搜索、筛选、日历。
- 不做统计。
- **不做备份/导出/同步** —— 数据仅存手机本地，刷机/卸载/App 数据清除会永久丢失，用户需自行知晓。

## 10. 骨架程序（伪码）

```dart
// DiaryDao
Future<List<DiaryEntry>> getAll() async {
  final rows = await db.query('entries', orderBy: 'date DESC');
  return rows.map(fromRow).toList();
}
Future<int> insert(DiaryEntry e) async {
  try { return await db.insert('entries', toMap(e)); }
  on DatabaseException { throw DayConflictException(); }
}

// PhotoStore
Future<String> savePhoto(XFile photo, String date) async {
  final dir = await getAppDocumentsDirectory();
  final name = '${date}_${DateTime.now().millisecondsSinceEpoch}.jpg';
  await File(photo.path).copy('${dir.path}/photos/$name');
  return name;
}

// Repository.add
Future<void> add(entry) async {
  if (entry.photo != null) entry.photo = await photoStore.savePhoto(...);
  await dao.insert(entry);
}
```
