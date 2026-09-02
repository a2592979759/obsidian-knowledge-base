---
tags: [python, "30天python", "day29"]
天数: 29
标题: "构建API"
source: "Chinese/29_building_API_cn.md"
---

# 第29天 - 构建API

- [第29天](#第29天)
- [构建API](#构建api)
  - [API的结构](#api的结构)
  - [使用GET获取数据](#使用get获取数据)
  - [通过ID获取文档](#通过id获取文档)
  - [使用POST创建数据](#使用post创建数据)
  - [使用PUT更新](#使用put更新)
  - [使用Delete删除文档](#使用delete删除文档)
- [💻 练习：第29天](#-练习第29天)

## 第29天

## 构建API

在本节中，我们将介绍一个RESTful API，它使用HTTP请求方法来GET、PUT、POST和DELETE数据。

RESTful API是一种应用程序编程接口(API)，使用HTTP请求来GET、PUT、POST和DELETE数据。在前几节中，我们学习了Python、Flask和MongoDB。我们将利用我们获得的知识，使用Python Flask和MongoDB开发一个RESTful API。每个具有CRUD(创建、读取、更新、删除)操作的应用程序都有一个API，用于从数据库创建数据、获取数据、更新数据或删除数据。

浏览器只能处理GET请求。因此，我们必须有一个工具，可以帮助我们处理所有请求方法(GET、POST、PUT、DELETE)。

API示例：

- 国家API：https://restcountries.eu/rest/v2/all
- 猫品种API：https://api.thecatapi.com/v1/breeds

[Postman](https://www.getpostman.com/)是API开发领域非常流行的工具。所以，如果你想完成本节内容，你需要[下载Postman](https://www.getpostman.com/)。Postman的替代品是[Insomnia](https://insomnia.rest/download)。

![[_assets/postman.png]]

### API的结构

API端点是一个可以帮助检索、创建、更新或删除资源的URL。结构如下所示：
示例：
https://api.twitter.com/1.1/lists/members.json
返回指定列表的成员。只有当经过身份验证的用户拥有指定列表时，才会显示私人列表成员。
公司名称后跟版本，然后是API的目的。
方法：
HTTP方法和URL

API使用以下HTTP方法进行对象操作：

```sh
GET        用于对象检索
POST       用于对象创建和对象操作
PUT        用于对象更新
DELETE     用于对象删除
```

让我们构建一个收集关于30DaysOfPython学生信息的API。我们将收集姓名、国家、城市、出生日期、技能和个人简介。

为了实现这个API，我们将使用：

- Postman
- Python
- Flask
- MongoDB

### 使用GET获取数据

在这一步中，让我们使用虚拟数据并将其作为json返回。为了将其作为json返回，我们将使用json模块和Response模块。

```py
# 导入flask

from flask import Flask,  Response
import json

app = Flask(__name__)

@app.route('/api/v1.0/students', methods = ['GET'])
def students ():
    student_list = [
        {
            'name':'Asabeneh',
            'country':'Finland',
            'city':'Helsinki',
            'skills':['HTML', 'CSS','JavaScript','Python']
        },
        {
            'name':'David',
            'country':'UK',
            'city':'London',
            'skills':['Python','MongoDB']
        },
        {
            'name':'John',
            'country':'Sweden',
            'city':'Stockholm',
            'skills':['Java','C#']
        }
    ]
    return Response(json.dumps(student_list), mimetype='application/json')


if __name__ == '__main__':
    # 部署时使用
    # 使其在生产和开发环境中都能工作
    port = int(os.environ.get("PORT", 5000))
    app.run(debug=True, host='0.0.0.0', port=port)
```

当你在浏览器上请求http://localhost:5000/api/v1.0/students URL时，你将获得以下内容：

![[_assets/get_on_browser.png]]

当你在Postman上请求http://localhost:5000/api/v1.0/students URL时，你将获得以下内容：

![[_assets/get_on_postman.png]]

我们不再显示虚拟数据，而是将Flask应用程序与MongoDB连接，并从MongoDB数据库获取数据。

```py
# 导入flask

from flask import Flask,  Response
import json
import pymongo


app = Flask(__name__)

#
MONGODB_URI='mongodb+srv://asabeneh:your_password@30daysofpython-twxkr.mongodb.net/test?retryWrites=true&w=majority'
client = pymongo.MongoClient(MONGODB_URI)
db = client['thirty_days_of_python'] # 访问数据库

@app.route('/api/v1.0/students', methods = ['GET'])
def students ():

    return Response(json.dumps(student), mimetype='application/json')


if __name__ == '__main__':
    # 部署时使用
    # 使其在生产和开发环境中都能工作
    port = int(os.environ.get("PORT", 5000))
    app.run(debug=True, host='0.0.0.0', port=port)
```

通过连接Flask，我们可以从thirty_days_of_python数据库中获取学生集合数据。

```sh
[
    {
        "_id": {
            "$oid": "5df68a21f106fe2d315bbc8b"
        },
        "name": "Asabeneh",
        "country": "Finland",
        "city": "Helsinki",
        "age": 38
    },
    {
        "_id": {
            "$oid": "5df68a23f106fe2d315bbc8c"
        },
        "name": "David",
        "country": "UK",
        "city": "London",
        "age": 34
    },
    {
        "_id": {
            "$oid": "5df68a23f106fe2d315bbc8e"
        },
        "name": "Sami",
        "country": "Finland",
        "city": "Helsinki",
        "age": 25
    }
]
```

### 通过ID获取文档

我们可以使用ID访问单个文档，让我们使用ID访问Asabeneh。
http://localhost:5000/api/v1.0/students/5df68a21f106fe2d315bbc8b

```py
# 导入flask

from flask import Flask,  Response
import json
from bson.objectid import ObjectId
import json
from bson.json_util import dumps
import pymongo


app = Flask(__name__)

#
MONGODB_URI='mongodb+srv://asabeneh:your_password@30daysofpython-twxkr.mongodb.net/test?retryWrites=true&w=majority'
client = pymongo.MongoClient(MONGODB_URI)
db = client['thirty_days_of_python'] # 访问数据库

@app.route('/api/v1.0/students', methods = ['GET'])
def students ():

    return Response(json.dumps(student), mimetype='application/json')
@app.route('/api/v1.0/students/<id>', methods = ['GET'])
def single_student (id):
    student = db.students.find({'_id':ObjectId(id)})
    return Response(dumps(student), mimetype='application/json')

if __name__ == '__main__':
    # 部署时使用
    # 使其在生产和开发环境中都能工作
    port = int(os.environ.get("PORT", 5000))
    app.run(debug=True, host='0.0.0.0', port=port)
```

```sh
[
    {
        "_id": {
            "$oid": "5df68a21f106fe2d315bbc8b"
        },
        "name": "Asabeneh",
        "country": "Finland",
        "city": "Helsinki",
        "age": 38
    }
]
```

### 使用POST创建数据

我们使用POST请求方法创建数据

```py
# 导入flask

from flask import Flask,  Response
import json
from bson.objectid import ObjectId
import json
from bson.json_util import dumps
import pymongo
from datetime import datetime


app = Flask(__name__)

#
MONGODB_URI='mongodb+srv://asabeneh:your_password@30daysofpython-twxkr.mongodb.net/test?retryWrites=true&w=majority'
client = pymongo.MongoClient(MONGODB_URI)
db = client['thirty_days_of_python'] # 访问数据库

@app.route('/api/v1.0/students', methods = ['GET'])
def students ():

    return Response(json.dumps(student), mimetype='application/json')
@app.route('/api/v1.0/students/<id>', methods = ['GET'])
def single_student (id):
    student = db.students.find({'_id':ObjectId(id)})
    return Response(dumps(student), mimetype='application/json')
@app.route('/api/v1.0/students', methods = ['POST'])
def create_student ():
    name = request.form['name']
    country = request.form['country']
    city = request.form['city']
    skills = request.form['skills'].split(', ')
    bio = request.form['bio']
    birthyear = request.form['birthyear']
    created_at = datetime.now()
    student = {
        'name': name,
        'country': country,
        'city': city,
        'birthyear': birthyear,
        'skills': skills,
        'bio': bio,
        'created_at': created_at

    }
    db.students.insert_one(student)
    return ;
def update_student (id):
if __name__ == '__main__':
    # 部署时使用
    # 使其在生产和开发环境中都能工作
    port = int(os.environ.get("PORT", 5000))
    app.run(debug=True, host='0.0.0.0', port=port)
```

### 使用PUT更新

```py
# 导入flask

from flask import Flask,  Response
import json
from bson.objectid import ObjectId
import json
from bson.json_util import dumps
import pymongo
from datetime import datetime


app = Flask(__name__)

#
MONGODB_URI='mongodb+srv://asabeneh:your_password@30daysofpython-twxkr.mongodb.net/test?retryWrites=true&w=majority'
client = pymongo.MongoClient(MONGODB_URI)
db = client['thirty_days_of_python'] # 访问数据库

@app.route('/api/v1.0/students', methods = ['GET'])
def students ():

    return Response(json.dumps(student), mimetype='application/json')
@app.route('/api/v1.0/students/<id>', methods = ['GET'])
def single_student (id):
    student = db.students.find({'_id':ObjectId(id)})
    return Response(dumps(student), mimetype='application/json')
@app.route('/api/v1.0/students', methods = ['POST'])
def create_student ():
    name = request.form['name']
    country = request.form['country']
    city = request.form['city']
    skills = request.form['skills'].split(', ')
    bio = request.form['bio']
    birthyear = request.form['birthyear']
    created_at = datetime.now()
    student = {
        'name': name,
        'country': country,
        'city': city,
        'birthyear': birthyear,
        'skills': skills,
        'bio': bio,
        'created_at': created_at

    }
    db.students.insert_one(student)
    return
@app.route('/api/v1.0/students/<id>', methods = ['PUT']) # 这个装饰器创建主页路由
def update_student (id):
    query = {"_id":ObjectId(id)}
    name = request.form['name']
    country = request.form['country']
    city = request.form['city']
    skills = request.form['skills'].split(', ')
    bio = request.form['bio']
    birthyear = request.form['birthyear']
    created_at = datetime.now()
    student = {
        'name': name,
        'country': country,
        'city': city,
        'birthyear': birthyear,
        'skills': skills,
        'bio': bio,
        'created_at': created_at

    }
    db.students.update_one(query, student)
    # return Response(dumps({"result":"a new student has been created"}), mimetype='application/json')
    return
def update_student (id):
if __name__ == '__main__':
    # 部署时使用
    # 使其在生产和开发环境中都能工作
    port = int(os.environ.get("PORT", 5000))
    app.run(debug=True, host='0.0.0.0', port=port)
```

### 使用Delete删除文档

```py
# 导入flask

from flask import Flask,  Response
import json
from bson.objectid import ObjectId
import json
from bson.json_util import dumps
import pymongo
from datetime import datetime


app = Flask(__name__)

#
MONGODB_URI='mongodb+srv://asabeneh:your_password@30daysofpython-twxkr.mongodb.net/test?retryWrites=true&w=majority'
client = pymongo.MongoClient(MONGODB_URI)
db = client['thirty_days_of_python'] # 访问数据库

@app.route('/api/v1.0/students', methods = ['GET'])
def students ():

    return Response(json.dumps(student), mimetype='application/json')
@app.route('/api/v1.0/students/<id>', methods = ['GET'])
def single_student (id):
    student = db.students.find({'_id':ObjectId(id)})
    return Response(dumps(student), mimetype='application/json')
@app.route('/api/v1.0/students', methods = ['POST'])
def create_student ():
    name = request.form['name']
    country = request.form['country']
    city = request.form['city']
    skills = request.form['skills'].split(', ')
    bio = request.form['bio']
    birthyear = request.form['birthyear']
    created_at = datetime.now()
    student = {
        'name': name,
        'country': country,
        'city': city,
        'birthyear': birthyear,
        'skills': skills,
        'bio': bio,
        'created_at': created_at

    }
    db.students.insert_one(student)
    return
@app.route('/api/v1.0/students/<id>', methods = ['PUT']) # 这个装饰器创建主页路由
def update_student (id):
    query = {"_id":ObjectId(id)}
    name = request.form['name']
    country = request.form['country']
    city = request.form['city']
    skills = request.form['skills'].split(', ')
    bio = request.form['bio']
    birthyear = request.form['birthyear']
    created_at = datetime.now()
    student = {
        'name': name,
        'country': country,
        'city': city,
        'birthyear': birthyear,
        'skills': skills,
        'bio': bio,
        'created_at': created_at

    }
    db.students.update_one(query, student)
    # return Response(dumps({"result":"a new student has been created"}), mimetype='application/json')
    return ;
@app.route('/api/v1.0/students/<id>', methods = ['DELETE'])
def delete_student (id):
    db.students.delete_one({"_id":ObjectId(id)})
    return
if __name__ == '__main__':
    # 部署时使用
    # 使其在生产和开发环境中都能工作
    port = int(os.environ.get("PORT", 5000))
    app.run(debug=True, host='0.0.0.0', port=port)
```

## 💻 练习：第29天

1. 实现上述示例并开发[这个](https://thirtydayofpython-api.herokuapp.com/)

🎉 恭喜！🎉

---

## 🔗 相关链接

- 上一篇：[[第28天-API]]
- 下一篇：[[第30天-结论]]
- 返回主页：[[00-索引]]
