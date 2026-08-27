---
tags: [python, "30天python", "day27"]
天数: 27
标题: "Python与MongoDB"
source: "Chinese/27_python_with_mongodb_cn.md"
---

# 第27天 - Python与MongoDB

- [第27天](#-第27天)
- [Python与MongoDB](#python与mongodb)
  - [MongoDB](#mongodb)
    - [SQL与NoSQL的比较](#sql与nosql的比较)
    - [获取连接字符串(MongoDB URI)](#获取连接字符串mongodb-uri)
    - [将Flask应用程序连接到MongoDB集群](#将flask应用程序连接到mongodb集群)
    - [创建数据库和集合](#创建数据库和集合)
    - [向集合中插入多个文档](#向集合中插入多个文档)
    - [MongoDB查找](#mongodb查找)
    - [使用查询进行查找](#使用查询进行查找)
    - [使用修饰符的查询查找](#使用修饰符的查询查找)
    - [限制文档数量](#限制文档数量)
    - [使用排序进行查找](#使用排序进行查找)
    - [使用查询进行更新](#使用查询进行更新)
    - [删除文档](#删除文档)
    - [删除集合](#删除集合)
  - [💻 练习：第27天](#-练习第27天)

## 📘 第27天

## Python与MongoDB

Python是一种后端技术，可以与不同的数据库应用程序连接。它可以连接到SQL和NoSQL数据库。在本节中，我们将Python与MongoDB数据库连接，MongoDB是一种NoSQL数据库。

## MongoDB

MongoDB是一种NoSQL数据库。MongoDB以类似JSON的文档形式存储数据，这使得MongoDB非常灵活和可扩展。让我们看看SQL和NoSQL数据库的不同术语。下表将展示SQL与NoSQL数据库之间的区别。

### SQL与NoSQL的比较

![[_assets/sql-vs-nosql.png]]

在本节中，我们将重点关注NoSQL数据库MongoDB。通过点击注册按钮，然后在下一页点击注册，让我们在[mongoDB](https://www.mongodb.com/)上注册。

![[_assets/mongodb-signup-page.png]]

完成表格并点击继续

![[_assets/mongodb-register.png]]

选择免费计划

![[_assets/mongodb-free.png]]

选择最近的免费区域，并为你的集群命名。

![[_assets/mongodb-cluster-name.png]]

现在，一个免费的沙箱已创建

![[_assets/mongodb-sandbox.png]]

允许所有本地主机访问

![[_assets/mongodb-allow-ip-access.png]]

添加用户和密码

![[_assets/mongodb-add-user.png]]

创建MongoDB URI链接

![[_assets/mongodb-create-uri.png]]

选择Python 3.6或更高版本的驱动程序

![[_assets/mongodb-python-driver.png]]

### 获取连接字符串(MongoDB URI)

复制连接字符串链接，你将获得类似这样的内容：

```sh
mongodb+srv://asabeneh:<password>@30daysofpython-twxkr.mongodb.net/test?retryWrites=true&w=majority
```

不要担心这个URL，它是连接你的应用程序与mongoDB的一种方式。
让我们用你添加用户时使用的密码替换密码占位符。

**示例：**

```sh
mongodb+srv://asabeneh:123123123@30daysofpython-twxkr.mongodb.net/test?retryWrites=true&w=majority
```

现在，我已经替换了所有内容，密码是123123123，数据库名称是*thirty_days_python*。这只是一个示例，你的密码必须比示例密码更强。

Python需要mongoDB驱动程序才能访问mongoDB数据库。我们将使用_pymongo_和_dnspython_来连接我们的应用程序与mongoDB基础。在你的项目目录中安装pymongo和dnspython。

```sh
pip install pymongo dnspython
```

要使用mongodb+srv://URI，必须安装"dnspython"模块。dnspython是用于Python的DNS工具包。它支持几乎所有记录类型。

### 将Flask应用程序连接到MongoDB集群

```py
# 导入flask
from flask import Flask, render_template
import os # 导入操作系统模块
MONGODB_URI = 'mongodb+srv://asabeneh:your_password_goes_here@30daysofpython-twxkr.mongodb.net/test?retryWrites=true&w=majority'
client = pymongo.MongoClient(MONGODB_URI)
print(client.list_database_names())

app = Flask(__name__)
if __name__ == '__main__':
    # 部署时我们使用环境变量
    # 使其同时适用于生产和开发环境
    port = int(os.environ.get("PORT", 5000))
    app.run(debug=True, host='0.0.0.0', port=port)
```

当我们运行上面的代码时，我们会得到默认的mongoDB数据库。

```sh
['admin', 'local']
```

### 创建数据库和集合

让我们创建一个数据库，如果数据库和集合在mongoDB中不存在，它们将被创建。让我们创建一个名为*thirty_days_of_python*的数据库和*students*集合。

创建数据库的方法：

```sh
db = client.name_of_databse # 我们可以这样创建数据库，或者使用第二种方式
db = client['name_of_database']
```

```py
# 导入flask
from flask import Flask, render_template
import os # 导入操作系统模块
MONGODB_URI = 'mongodb+srv://asabeneh:your_password_goes_here@30daysofpython-twxkr.mongodb.net/test?retryWrites=true&w=majority'
client = pymongo.MongoClient(MONGODB_URI)
# 创建数据库
db = client.thirty_days_of_python
# 创建students集合并插入文档
db.students.insert_one({'name': 'Asabeneh', 'country': 'Finland', 'city': 'Helsinki', 'age': 250})
print(client.list_database_names())

app = Flask(__name__)
if __name__ == '__main__':
    # 部署时我们使用环境变量
    # 使其同时适用于生产和开发环境
    port = int(os.environ.get("PORT", 5000))
    app.run(debug=True, host='0.0.0.0', port=port)
```

在创建数据库后，我们还创建了一个students集合，并使用*insert_one()*方法插入了一个文档。
现在，数据库*thirty_days_of_python*和*students*集合已经创建，并且文档已经插入。
检查你的mongoDB集群，你将看到数据库和集合。在集合内部，将有一个文档。

```sh
['thirty_days_of_python', 'admin', 'local']
```

如果你在mongoDB集群上看到这个，这意味着你已经成功创建了一个数据库和一个集合。

![[_assets/mongodb-creating_database.png]]

如果你看到上图，文档已经创建，并带有一个长ID作为主键。每次我们创建一个文档，mongoDB都会为它创建一个唯一的ID。

### 向集合中插入多个文档

*insert_one()*方法一次插入一个项目，如果我们想一次插入多个文档，我们可以使用*insert_many()*方法或for循环。
我们可以使用for循环一次插入多个文档。

```py
# 导入flask
from flask import Flask, render_template
import os # 导入操作系统模块
MONGODB_URI = 'mongodb+srv://asabeneh:your_password_goes_here@30daysofpython-twxkr.mongodb.net/test?retryWrites=true&w=majority'
client = pymongo.MongoClient(MONGODB_URI)

students = [
        {'name':'David','country':'UK','city':'London','age':34},
        {'name':'John','country':'Sweden','city':'Stockholm','age':28},
        {'name':'Sami','country':'Finland','city':'Helsinki','age':25},
    ]
for student in students:
    db.students.insert_one(student)

app = Flask(__name__)
if __name__ == '__main__':
    # 部署时我们使用环境变量
    # 使其同时适用于生产和开发环境
    port = int(os.environ.get("PORT", 5000))
    app.run(debug=True, host='0.0.0.0', port=port)
```

### MongoDB查找

*find()*和*findOne()*方法是在mongoDB数据库集合中查找数据的常用方法。它类似于MySQL数据库中的SELECT语句。
让我们使用_find_one()_方法来获取数据库集合中的一个文档。

- \*find_one({"\_id": ObjectId("id"}): 如果没有提供id，则获取第一次出现的文档

```py
# 导入flask
from flask import Flask, render_template
import os # 导入操作系统模块
MONGODB_URI = 'mongodb+srv://asabeneh:your_password_goes_here@30daysofpython-twxkr.mongodb.net/test?retryWrites=true&w=majority'
client = pymongo.MongoClient(MONGODB_URI)
db = client['thirty_days_of_python'] # 访问数据库
student = db.students.find_one()
print(student)

app = Flask(__name__)
if __name__ == '__main__':
    # 部署时我们使用环境变量
    # 使其同时适用于生产和开发环境
    port = int(os.environ.get("PORT", 5000))
    app.run(debug=True, host='0.0.0.0', port=port)
```

```sh
{'_id': ObjectId('5df68a21f106fe2d315bbc8b'), 'name': 'Asabeneh', 'country': 'Helsinki', 'city': 'Helsinki', 'age': 250}
```

上面的查询返回第一个条目，但我们可以使用特定的\_id来定位特定的文档。让我们做一个例子，使用David的id来获取David对象。
'\_id':ObjectId('5df68a23f106fe2d315bbc8c')

```py
# 导入flask
from flask import Flask, render_template
import os # 导入操作系统模块
from bson.objectid import ObjectId # id对象
MONGODB_URI = 'mongodb+srv://asabeneh:your_password_goes_here@30daysofpython-twxkr.mongodb.net/test?retryWrites=true&w=majority'
client = pymongo.MongoClient(MONGODB_URI)
db = client['thirty_days_of_python'] # 访问数据库
student = db.students.find_one({'_id':ObjectId('5df68a23f106fe2d315bbc8c')})
print(student)

app = Flask(__name__)
if __name__ == '__main__':
    # 部署时我们使用环境变量
    # 使其同时适用于生产和开发环境
    port = int(os.environ.get("PORT", 5000))
    app.run(debug=True, host='0.0.0.0', port=port)
```

```sh
{'_id': ObjectId('5df68a23f106fe2d315bbc8c'), 'name': 'David', 'country': 'UK', 'city': 'London', 'age': 34}
```

我们已经看到如何使用上面的例子来使用_find_one()_。让我们进一步了解_find()_

- _find()_: 如果我们不传递查询对象，返回集合中的所有出现。该对象是pymongo.cursor对象。

```py
# 导入flask
from flask import Flask, render_template
import os # 导入操作系统模块

MONGODB_URI = 'mongodb+srv://asabeneh:your_password_goes_here@30daysofpython-twxkr.mongodb.net/test?retryWrites=true&w=majority'
client = pymongo.MongoClient(MONGODB_URI)
db = client['thirty_days_of_python'] # 访问数据库
students = db.students.find()
for student in students:
    print(student)

app = Flask(__name__)
if __name__ == '__main__':
    # 部署时我们使用环境变量
    # 使其同时适用于生产和开发环境
    port = int(os.environ.get("PORT", 5000))
    app.run(debug=True, host='0.0.0.0', port=port)
```

```sh
{'_id': ObjectId('5df68a21f106fe2d315bbc8b'), 'name': 'Asabeneh', 'country': 'Finland', 'city': 'Helsinki', 'age': 250}
{'_id': ObjectId('5df68a23f106fe2d315bbc8c'), 'name': 'David', 'country': 'UK', 'city': 'London', 'age': 34}
{'_id': ObjectId('5df68a23f106fe2d315bbc8d'), 'name': 'John', 'country': 'Sweden', 'city': 'Stockholm', 'age': 28}
{'_id': ObjectId('5df68a23f106fe2d315bbc8e'), 'name': 'Sami', 'country': 'Finland', 'city': 'Helsinki', 'age': 25}
```

我们可以通过在_find({}, {})_中传递第二个对象来指定要返回的字段。0表示不包含，1表示包含，但我们不能混合使用0和1，除了\_id。

```py
# 导入flask
from flask import Flask, render_template
import os # 导入操作系统模块

MONGODB_URI = 'mongodb+srv://asabeneh:your_password_goes_here@30daysofpython-twxkr.mongodb.net/test?retryWrites=true&w=majority'
client = pymongo.MongoClient(MONGODB_URI)
db = client['thirty_days_of_python'] # 访问数据库
students = db.students.find({}, {"_id":0,  "name": 1, "country":1}) # 0表示不包含，1表示包含
for student in students:
    print(student)

app = Flask(__name__)
if __name__ == '__main__':
    # 部署时我们使用环境变量
    # 使其同时适用于生产和开发环境
    port = int(os.environ.get("PORT", 5000))
    app.run(debug=True, host='0.0.0.0', port=port)
```

```sh
{'name': 'Asabeneh', 'country': 'Finland'}
{'name': 'David', 'country': 'UK'}
{'name': 'John', 'country': 'Sweden'}
{'name': 'Sami', 'country': 'Finland'}
```

### 使用查询进行查找

在mongoDB中，find接受一个查询对象。我们可以传递一个查询对象，我们可以过滤我们想要过滤的文档。

```py
# 导入flask
from flask import Flask, render_template
import os # 导入操作系统模块

MONGODB_URI = 'mongodb+srv://asabeneh:your_password_goes_here@30daysofpython-twxkr.mongodb.net/test?retryWrites=true&w=majority'
client = pymongo.MongoClient(MONGODB_URI)
db = client['thirty_days_of_python'] # 访问数据库

query = {
    "country":"Finland"
}
students = db.students.find(query)

for student in students:
    print(student)

app = Flask(__name__)
if __name__ == '__main__':
    # 部署时我们使用环境变量
    # 使其同时适用于生产和开发环境
    port = int(os.environ.get("PORT", 5000))
    app.run(debug=True, host='0.0.0.0', port=port)
```

```sh
{'_id': ObjectId('5df68a21f106fe2d315bbc8b'), 'name': 'Asabeneh', 'country': 'Finland', 'city': 'Helsinki', 'age': 250}
{'_id': ObjectId('5df68a23f106fe2d315bbc8e'), 'name': 'Sami', 'country': 'Finland', 'city': 'Helsinki', 'age': 25}
```

带有修饰符的查询

```py
# 导入flask
from flask import Flask, render_template
import os # 导入操作系统模块
import pymongo

MONGODB_URI = 'mongodb+srv://asabeneh:your_password_goes_here@30daysofpython-twxkr.mongodb.net/test?retryWrites=true&w=majority'
client = pymongo.MongoClient(MONGODB_URI)
db = client['thirty_days_of_python'] # 访问数据库

query = {
    "city":"Helsinki"
}
students = db.students.find(query)
for student in students:
    print(student)

app = Flask(__name__)
if __name__ == '__main__':
    # 部署时我们使用环境变量
    # 使其同时适用于生产和开发环境
    port = int(os.environ.get("PORT", 5000))
    app.run(debug=True, host='0.0.0.0', port=port)
```

```sh
{'_id': ObjectId('5df68a21f106fe2d315bbc8b'), 'name': 'Asabeneh', 'country': 'Finland', 'city': 'Helsinki', 'age': 250}
{'_id': ObjectId('5df68a23f106fe2d315bbc8e'), 'name': 'Sami', 'country': 'Finland', 'city': 'Helsinki', 'age': 25}
```

### 使用修饰符的查询查找

```py
# 导入flask
from flask import Flask, render_template
import os # 导入操作系统模块
import pymongo

MONGODB_URI = 'mongodb+srv://asabeneh:your_password_goes_here@30daysofpython-twxkr.mongodb.net/test?retryWrites=true&w=majority'
client = pymongo.MongoClient(MONGODB_URI)
db = client['thirty_days_of_python'] # 访问数据库
query = {
    "country":"Finland",
    "city":"Helsinki"
}
students = db.students.find(query)
for student in students:
    print(student)

app = Flask(__name__)
if __name__ == '__main__':
    # 部署时我们使用环境变量
    # 使其同时适用于生产和开发环境
    port = int(os.environ.get("PORT", 5000))
    app.run(debug=True, host='0.0.0.0', port=port)
```

```sh
{'_id': ObjectId('5df68a21f106fe2d315bbc8b'), 'name': 'Asabeneh', 'country': 'Finland', 'city': 'Helsinki', 'age': 250}
{'_id': ObjectId('5df68a23f106fe2d315bbc8e'), 'name': 'Sami', 'country': 'Finland', 'city': 'Helsinki', 'age': 25}
```

带有修饰符的查询

```py
# 导入flask
from flask import Flask, render_template
import os # 导入操作系统模块
import pymongo

MONGODB_URI = 'mongodb+srv://asabeneh:your_password_goes_here@30daysofpython-twxkr.mongodb.net/test?retryWrites=true&w=majority'
client = pymongo.MongoClient(MONGODB_URI)
db = client['thirty_days_of_python'] # 访问数据库
query = {"age":{"$gt":30}}
students = db.students.find(query)
for student in students:
    print(student)

app = Flask(__name__)
if __name__ == '__main__':
    # 部署时我们使用环境变量
    # 使其同时适用于生产和开发环境
    port = int(os.environ.get("PORT", 5000))
    app.run(debug=True, host='0.0.0.0', port=port)
```

```sh
{'_id': ObjectId('5df68a21f106fe2d315bbc8b'), 'name': 'Asabeneh', 'country': 'Finland', 'city': 'Helsinki', 'age': 250}
{'_id': ObjectId('5df68a23f106fe2d315bbc8c'), 'name': 'David', 'country': 'UK', 'city': 'London', 'age': 34}
```

```py
# 导入flask
from flask import Flask, render_template
import os # 导入操作系统模块
import pymongo

MONGODB_URI = 'mongodb+srv://asabeneh:your_password_goes_here@30daysofpython-twxkr.mongodb.net/test?retryWrites=true&w=majority'
client = pymongo.MongoClient(MONGODB_URI)
db = client['thirty_days_of_python'] # 访问数据库
query = {"age":{"$lt":30}}
students = db.students.find(query)
for student in students:
    print(student)
```

```sh
{'_id': ObjectId('5df68a23f106fe2d315bbc8d'), 'name': 'John', 'country': 'Sweden', 'city': 'Stockholm', 'age': 28}
{'_id': ObjectId('5df68a23f106fe2d315bbc8e'), 'name': 'Sami', 'country': 'Finland', 'city': 'Helsinki', 'age': 25}
```

### 限制文档数量

我们可以使用_limit()_方法来限制我们返回的文档数量。

```py
# 导入flask
from flask import Flask, render_template
import os # 导入操作系统模块
import pymongo

MONGODB_URI = 'mongodb+srv://asabeneh:your_password_goes_here@30daysofpython-twxkr.mongodb.net/test?retryWrites=true&w=majority'
client = pymongo.MongoClient(MONGODB_URI)
db = client['thirty_days_of_python'] # 访问数据库
db.students.find().limit(3)
```

### 使用排序进行查找

默认情况下，排序是升序的。我们可以通过添加-1参数将排序更改为降序。

```py
# 导入flask
from flask import Flask, render_template
import os # 导入操作系统模块
import pymongo

MONGODB_URI = 'mongodb+srv://asabeneh:your_password_goes_here@30daysofpython-twxkr.mongodb.net/test?retryWrites=true&w=majority'
client = pymongo.MongoClient(MONGODB_URI)
db = client['thirty_days_of_python'] # 访问数据库
students = db.students.find().sort('name')
for student in students:
    print(student)

students = db.students.find().sort('name',-1)
for student in students:
    print(student)

students = db.students.find().sort('age')
for student in students:
    print(student)

students = db.students.find().sort('age',-1)
for student in students:
    print(student)

app = Flask(__name__)
if __name__ == '__main__':
    # 部署时我们使用环境变量
    # 使其同时适用于生产和开发环境
    port = int(os.environ.get("PORT", 5000))
    app.run(debug=True, host='0.0.0.0', port=port)
```

升序

```sh
{'_id': ObjectId('5df68a21f106fe2d315bbc8b'), 'name': 'Asabeneh', 'country': 'Finland', 'city': 'Helsinki', 'age': 250}
{'_id': ObjectId('5df68a23f106fe2d315bbc8c'), 'name': 'David', 'country': 'UK', 'city': 'London', 'age': 34}
{'_id': ObjectId('5df68a23f106fe2d315bbc8d'), 'name': 'John', 'country': 'Sweden', 'city': 'Stockholm', 'age': 28}
{'_id': ObjectId('5df68a23f106fe2d315bbc8e'), 'name': 'Sami', 'country': 'Finland', 'city': 'Helsinki', 'age': 25}
```

降序

```sh
{'_id': ObjectId('5df68a23f106fe2d315bbc8e'), 'name': 'Sami', 'country': 'Finland', 'city': 'Helsinki', 'age': 25}
{'_id': ObjectId('5df68a23f106fe2d315bbc8d'), 'name': 'John', 'country': 'Sweden', 'city': 'Stockholm', 'age': 28}
{'_id': ObjectId('5df68a23f106fe2d315bbc8c'), 'name': 'David', 'country': 'UK', 'city': 'London', 'age': 34}
{'_id': ObjectId('5df68a21f106fe2d315bbc8b'), 'name': 'Asabeneh', 'country': 'Finland', 'city': 'Helsinki', 'age': 250}
```

### 使用查询进行更新

我们将使用*update_one()*方法来更新一个项目。它接受两个对象，一个是查询，另一个是新对象。
第一个人，Asabeneh得到了一个非常不合理的年龄。让我们更新Asabeneh的年龄。

```py
# 导入flask
from flask import Flask, render_template
import os # 导入操作系统模块
import pymongo

MONGODB_URI = 'mongodb+srv://asabeneh:your_password_goes_here@30daysofpython-twxkr.mongodb.net/test?retryWrites=true&w=majority'
client = pymongo.MongoClient(MONGODB_URI)
db = client['thirty_days_of_python'] # 访问数据库

query = {'age':250}
new_value = {'$set':{'age':38}}

db.students.update_one(query, new_value)
# 让我们检查结果，看看年龄是否被修改
for student in db.students.find():
    print(student)

app = Flask(__name__)
if __name__ == '__main__':
    # 部署时我们使用环境变量
    # 使其同时适用于生产和开发环境
    port = int(os.environ.get("PORT", 5000))
    app.run(debug=True, host='0.0.0.0', port=port)
```

```sh
{'_id': ObjectId('5df68a21f106fe2d315bbc8b'), 'name': 'Asabeneh', 'country': 'Finland', 'city': 'Helsinki', 'age': 38}
{'_id': ObjectId('5df68a23f106fe2d315bbc8c'), 'name': 'David', 'country': 'UK', 'city': 'London', 'age': 34}
{'_id': ObjectId('5df68a23f106fe2d315bbc8d'), 'name': 'John', 'country': 'Sweden', 'city': 'Stockholm', 'age': 28}
{'_id': ObjectId('5df68a23f106fe2d315bbc8e'), 'name': 'Sami', 'country': 'Finland', 'city': 'Helsinki', 'age': 25}
```

当我们想要一次更新多个文档时，我们使用*upate_many()*方法。

### 删除文档

方法*delete_one()*删除一个文档。*delete_one()*方法接受一个查询对象参数。它只删除第一次出现的文档。
让我们从集合中删除一个John。

```py
# 导入flask
from flask import Flask, render_template
import os # 导入操作系统模块
import pymongo

MONGODB_URI = 'mongodb+srv://asabeneh:your_password_goes_here@30daysofpython-twxkr.mongodb.net/test?retryWrites=true&w=majority'
client = pymongo.MongoClient(MONGODB_URI)
db = client['thirty_days_of_python'] # 访问数据库

query = {'name':'John'}
db.students.delete_one(query)

for student in db.students.find():
    print(student)
# 让我们检查结果，看看年龄是否被修改
for student in db.students.find():
    print(student)

app = Flask(__name__)
if __name__ == '__main__':
    # 部署时我们使用环境变量
    # 使其同时适用于生产和开发环境
    port = int(os.environ.get("PORT", 5000))
    app.run(debug=True, host='0.0.0.0', port=port)
```

```sh
{'_id': ObjectId('5df68a21f106fe2d315bbc8b'), 'name': 'Asabeneh', 'country': 'Finland', 'city': 'Helsinki', 'age': 38}
{'_id': ObjectId('5df68a23f106fe2d315bbc8c'), 'name': 'David', 'country': 'UK', 'city': 'London', 'age': 34}
{'_id': ObjectId('5df68a23f106fe2d315bbc8e'), 'name': 'Sami', 'country': 'Finland', 'city': 'Helsinki', 'age': 25}
```

如你所见，John已经从集合中删除。

当我们想要删除多个文档时，我们使用*delete_many()*方法，它接受一个查询对象。如果我们将一个空的查询对象传递给*delete_many({})*，它将删除集合中的所有文档。

### 删除集合

使用_drop()_方法，我们可以从数据库中删除一个集合。

```py
# 导入flask
from flask import Flask, render_template
import os # 导入操作系统模块
import pymongo

MONGODB_URI = 'mongodb+srv://asabeneh:your_password_goes_here@30daysofpython-twxkr.mongodb.net/test?retryWrites=true&w=majority'
client = pymongo.MongoClient(MONGODB_URI)
db = client['thirty_days_of_python'] # 访问数据库
db.students.drop()
```

现在，我们已经从数据库中删除了students集合。

## 💻 练习：第27天

🎉 恭喜！🎉

---

## 🔗 相关链接

- 上一篇：[[第26天-Python网页]]
- 下一篇：[[第28天-API]]
- 返回主页：[[00-索引]]
