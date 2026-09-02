---
tags: [python, "30天python", "day26"]
天数: 26
标题: "Python网页"
source: "Chinese/26_python_web_cn.md"
---

# 第26天 - Python网页

- [第26天](#-第26天)
  - [Python网络编程](#python网络编程)
    - [Flask](#flask)
      - [文件夹结构](#文件夹结构)
    - [设置项目目录](#设置项目目录)
    - [创建路由](#创建路由)
    - [创建模板](#创建模板)
    - [Python脚本](#python脚本)
    - [导航](#导航)
    - [创建布局](#创建布局)
      - [提供静态文件](#提供静态文件)
    - [部署](#部署)
      - [创建Heroku账户](#创建heroku账户)
      - [登录Heroku](#登录heroku)
      - [创建requirements和Procfile](#创建requirements和procfile)
      - [将项目推送到Heroku](#将项目推送到heroku)
  - [练习：第26天](#练习第26天)

## 📘 第26天

## Python网络编程

Python是一种通用编程语言，可以用于多种场合。在本节中，我们将看到如何使用Python进行网络开发。Python有许多Web框架。Django和Flask是最流行的框架。今天，我们将学习如何使用Flask进行Web开发。

### Flask

Flask是用Python编写的Web开发框架。Flask使用Jinja2模板引擎。Flask也可以与其他现代前端库（如React）一起使用。

如果你还没有安装virtualenv包，请先安装它。虚拟环境将允许隔离项目依赖与本地机器依赖。

#### 文件夹结构

完成所有步骤后，你的项目文件结构应该如下所示：

```sh
├── Procfile
├── app.py
├── env
│   ├── bin
├── requirements.txt
├── static
│   └── css
│       └── main.css
└── templates
    ├── about.html
    ├── home.html
    ├── layout.html
    ├── post.html
    └── result.html
```

### 设置项目目录

按照以下步骤开始使用Flask。

步骤1：使用以下命令安装virtualenv。

```sh
pip install virtualenv
```

步骤2：

```sh
asabeneh@Asabeneh:~/Desktop$ mkdir python_for_web
asabeneh@Asabeneh:~/Desktop$ cd python_for_web/
asabeneh@Asabeneh:~/Desktop/python_for_web$ virtualenv venv
asabeneh@Asabeneh:~/Desktop/python_for_web$ source venv/bin/activate
(env) asabeneh@Asabeneh:~/Desktop/python_for_web$ pip freeze
(env) asabeneh@Asabeneh:~/Desktop/python_for_web$ pip install Flask
(env) asabeneh@Asabeneh:~/Desktop/python_for_web$ pip freeze
Click==7.0
Flask==1.1.1
itsdangerous==1.1.0
Jinja2==2.10.3
MarkupSafe==1.1.1
Werkzeug==0.16.0
(env) asabeneh@Asabeneh:~/Desktop/python_for_web$
```

我们创建了一个名为python_for_web的项目目录。在项目中，我们创建了一个虚拟环境*venv*，它可以是任何名称，但我喜欢称它为_venv_。然后我们激活了虚拟环境。我们使用pip freeze检查项目目录中安装的包。pip freeze的结果是空的，因为还没有安装包。

现在，让我们在项目目录中创建app.py文件并编写以下代码。app.py文件将是项目中的主文件。以下代码包含flask模块和os模块。

### 创建路由

主页路由。

```py
# 导入flask
from flask import Flask
import os # 导入操作系统模块

app = Flask(__name__)

@app.route('/') # 这个装饰器创建主页路由
def home ():
    return '<h1>欢迎</h1>'

@app.route('/about')
def about():
    return '<h1>关于我们</h1>'


if __name__ == '__main__':
    # 部署时我们使用环境变量
    # 使其同时适用于生产和开发环境
    port = int(os.environ.get("PORT", 5000))
    app.run(debug=True, host='0.0.0.0', port=port)
```

要运行flask应用程序，在主flask应用程序目录中输入python app.py。

运行_python app.py_后，检查本地主机5000端口。

让我们添加额外的路由。
创建关于路由

```py
# 导入flask
from flask import Flask
import os # 导入操作系统模块

app = Flask(__name__)

@app.route('/') # 这个装饰器创建主页路由
def home ():
    return '<h1>欢迎</h1>'

@app.route('/about')
def about():
    return '<h1>关于我们</h1>'

if __name__ == '__main__':
    # 部署时我们使用环境变量
    # 使其同时适用于生产和开发环境
    port = int(os.environ.get("PORT", 5000))
    app.run(debug=True, host='0.0.0.0', port=port)
```

现在，我们在上面的代码中添加了关于路由。如果我们想渲染HTML文件而不是字符串呢？使用*render_template*函数可以渲染HTML文件。让我们在项目目录中创建一个名为templates的文件夹，并在其中创建home.html和about.html。让我们也从flask导入*render_template*函数。

### 创建模板

在templates文件夹内创建HTML文件。

home.html

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>主页</title>
  </head>

  <body>
    <h1>欢迎回家</h1>
  </body>
</html>
```

about.html

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>关于</title>
  </head>

  <body>
    <h1>关于我们</h1>
  </body>
</html>
```

### Python脚本

app.py

```py
# 导入flask
from flask import Flask, render_template
import os # 导入操作系统模块

app = Flask(__name__)

@app.route('/') # 这个装饰器创建主页路由
def home ():
    return render_template('home.html')

@app.route('/about')
def about():
    return render_template('about.html')

if __name__ == '__main__':
    # 部署时我们使用环境变量
    # 使其同时适用于生产和开发环境
    port = int(os.environ.get("PORT", 5000))
    app.run(debug=True, host='0.0.0.0', port=port)
```

正如你所看到的，为了访问不同的页面或进行导航，我们需要一个导航。让我们为每个页面添加一个链接，或者创建一个我们用于每个页面的布局。

### 导航

```html
<ul>
  <li><a href="/">主页</a></li>
  <li><a href="/about">关于</a></li>
</ul>
```

现在，我们可以使用上面的链接在页面之间导航。让我们创建一个额外的页面来处理表单数据。你可以给它取任何名字，我喜欢称它为post.html。

我们可以使用Jinja2模板引擎向HTML文件注入数据。

```py
# 导入flask
from flask import Flask, render_template, request, redirect, url_for
import os # 导入操作系统模块

app = Flask(__name__)

@app.route('/') # 这个装饰器创建主页路由
def home ():
    techs = ['HTML', 'CSS', 'Flask', 'Python']
    name = '30天Python编程挑战'
    return render_template('home.html', techs=techs, name=name, title='主页')

@app.route('/about')
def about():
    name = '30天Python编程挑战'
    return render_template('about.html', name=name, title='关于我们')

@app.route('/post')
def post():
    name = '编程语言文章'
    path = request.path
    return render_template('post.html', name=name, path=path, title='文章')

if __name__ == '__main__':
    # 部署时我们使用环境变量
    # 使其同时适用于生产和开发环境
    port = int(os.environ.get("PORT", 5000))
    app.run(debug=True, host='0.0.0.0', port=port)
```

home.html

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>{{title}}</title>
  </head>

  <body>
    <ul>
      <li><a href="/">主页</a></li>
      <li><a href="/about">关于</a></li>
      <li><a href="/post">文章</a></li>
    </ul>
    <h1>欢迎回到{{name}}</h1>
    <ul>
      {% for tech in techs %}
      <li>{{tech}}</li>
      {% endfor %}
    </ul>
  </body>
</html>
```

about.html

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>{{title}}</title>
  </head>

  <body>
    <ul>
      <li><a href="/">主页</a></li>
      <li><a href="/about">关于</a></li>
      <li><a href="/post">文章</a></li>
    </ul>
    <h1>关于{{name}}</h1>
  </body>
</html>
```

post.html

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>{{title}}</title>
  </head>

  <body>
    <ul>
      <li><a href="/">主页</a></li>
      <li><a href="/about">关于</a></li>
      <li><a href="/post">文章</a></li>
    </ul>
    <h1>{{name}}</h1>
    <p>当前路径: {{path}}</p>
    <form action="/result" method="POST">
      <div>
        <input
          type="text"
          name="first_name"
          placeholder="第一名字"
          required
        />
      </div>
      <div>
        <input
          type="text"
          name="last_name"
          placeholder="姓氏"
          required
        />
      </div>
      <div>
        <input type="text" name="old_job" placeholder="旧工作" />
      </div>
      <div>
        <input type="text" name="current_job" placeholder="当前工作" />
      </div>
      <div>
        <input type="text" name="country" placeholder="国家" />
      </div>
      <div>
        <button type="submit">提交</button>
      </div>
    </form>
  </body>
</html>
```

现在，让我们添加一个接收表单数据的路由。我们使用POST方法，因为我们将收到表单数据。

```py
# 导入flask
from flask import Flask, render_template, request, redirect, url_for
import os # 导入操作系统模块

app = Flask(__name__)

@app.route('/') # 这个装饰器创建主页路由
def home():
    techs = ['HTML', 'CSS', 'Flask', 'Python']
    name = '30天Python编程挑战'
    return render_template('home.html', techs=techs, name=name, title='主页')

@app.route('/about')
def about():
    name = '30天Python编程挑战'
    return render_template('about.html', name=name, title='关于我们')

@app.route('/post')
def post():
    name = '文章'
    return render_template('post.html', name=name, title='文章')


@app.route('/result', methods=['POST'])
def result():
    first_name = request.form['first_name']
    last_name = request.form['last_name']
    old_job = request.form['old_job']
    current_job = request.form['current_job']
    country = request.form['country']
    print(first_name, last_name, old_job, current_job, country)
    result_data = {
        'first_name':first_name,
        'last_name':last_name,
        'old_job': old_job,
        'current_job': current_job,
        'country':country
    }
    return render_template('result.html', result_data = result_data, title= '结果')

if __name__ == '__main__':
    # 部署时我们使用环境变量
    # 使其同时适用于生产和开发环境
    port = int(os.environ.get("PORT", 5000))
    app.run(debug=True, host='0.0.0.0', port=port)
```

result.html

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>{{title}}</title>
  </head>

  <body>
    <ul>
      <li><a href="/">主页</a></li>
      <li><a href="/about">关于</a></li>
      <li><a href="/post">文章</a></li>
    </ul>
    <h1>表单数据</h1>

    <ul>
      <li>第一名字: {{result_data.first_name}}</li>
      <li>姓氏: {{result_data.last_name}}</li>
      <li>旧工作: {{result_data.old_job}}</li>
      <li>当前工作: {{result_data.current_job}}</li>
      <li>国家: {{result_data.country}}</li>
    </ul>
  </body>
</html>
```

在现实世界中，我们不会在所有页面中重复HTML代码。而是创建一个布局并将其继承到其他文件中。让我们使用继承（模板）。现在，我们需要创建的不是三个不同的文件，而是一个名为layout.html的布局文件。然后其他文件将继承它。

### 创建布局

layout.html

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <link
      href="https://fonts.googleapis.com/css?family=Lato:300,400|Nunito:300,400|Raleway:300,400&display=swap"
      rel="stylesheet"
    />
    <link
      rel="stylesheet"
      href="{{ url_for('static', filename='css/main.css') }}"
    />
    {% if title %}
    <title>30天Python - {{ title}}</title>
    {% else %}
    <title>30天Python</title>
    {% endif %}
  </head>

  <body>
    <header>
      <div class="menu-container">
        <div>
          <a class="brand-name nav-link" href="/">30天Python</a>
        </div>
        <ul class="nav-lists">
          <li class="nav-list">
            <a class="nav-link active" href="{{ url_for('home') }}">主页</a>
          </li>
          <li class="nav-list">
            <a class="nav-link active" href="{{ url_for('about') }}">关于</a>
          </li>
          <li class="nav-list">
            <a class="nav-link active" href="{{ url_for('post') }}">文章</a>
          </li>
        </ul>
      </div>
    </header>
    <main>
      {% block content %} {% endblock %}
    </main>
  </body>
</html>
```

在上面的布局中，我们创建了一个可以被所有继承布局的页面使用的公共布局。在布局内部，我们可以看到导航链接。我们使用{% block content %}{% endblock %}标记以允许子模板添加内容。

home.html

```html
{% extends 'layout.html' %} {% block content %}
<div class="container">
  <h1>欢迎回到{{name}}</h1>
  <p>
    这个项目是通过使用以下技术构建的:
    <span class="tech">Flask</span>, <span class="tech">Python</span>
    and <span class="tech">HTML</span> CSS</span>
  </p>
  <ul>
    {% for tech in techs %}
    <li class="tech">{{tech}}</li>

    {% endfor %}
  </ul>
</div>

{% endblock %}
```

about.html

```html
{% extends 'layout.html' %} {% block content %}
<div class="container">
  <h1>关于{{name}}</h1>
  <p>
    这个挑战是一个30天编程挑战，旨在帮助你学习Python编程语言，通过每天解决一个Python问题。
  </p>
</div>
{% endblock %}
```

post.html

```html
{% extends 'layout.html' %} {% block content %}
<div class="container">
  <h1>{{name}}</h1>
  <p>{{path}}</p>
  <form action="/result" method="POST">
    <div>
      <input type="text" name="first_name" placeholder="第一名字" required />
    </div>
    <div>
      <input type="text" name="last_name" placeholder="姓氏" required />
    </div>
    <div>
      <input type="text" name="old_job" placeholder="旧工作" />
    </div>
    <div>
      <input type="text" name="current_job" placeholder="当前工作" />
    </div>
    <div>
      <input type="text" name="country" placeholder="国家" />
    </div>
    <div>
      <button type="submit">提交</button>
    </div>
  </form>
</div>

{% endblock %}
```

result.html

```html
{% extends 'layout.html' %} {% block content %}
<div class="container">
  <h1>表单数据</h1>
  <ul>
    <li>第一名字: {{result_data.first_name}}</li>
    <li>姓氏: {{result_data.last_name}}</li>
    <li>旧工作: {{result_data.old_job}}</li>
    <li>当前工作: {{result_data.current_job}}</li>
    <li>国家: {{result_data.country}}</li>
  </ul>
</div>

{% endblock %}
```

#### 提供静态文件

以下是main.css文件，我们将把它放在static/css目录中：

```css
/* === GENERAL === */
body {
  margin: 0;
  padding: 0;
  font-family: "Lato", sans-serif;
  background-color: #f0f8ea;
}

.container {
  max-width: 80%;
  margin: auto;
  padding: 30px;
}

ul {
  list-style-type: none;
  padding: 0;
}

.tech {
  color: #5bbc2e;
}

/* === HEADER === */
header {
  background-color: #5bbc2e;
}

.menu-container {
  display: flex;
  justify-content: space-between;
  padding: 20px 30px;
}

.brand-name {
  color: white;
  font-weight: 800;
  font-size: 24px;
}

.nav-lists {
  display: flex;
}

.nav-list {
  margin-right: 15px;
}

.nav-link {
  text-decoration: none;
  color: white;
  font-weight: 300;
}

/* === FORM === */

form {
  margin: 30px 0;
  border: 1px solid #ddd;
  padding: 30px;
  border-radius: 10px;
}

form > div {
  margin-bottom: 15px;
}

input {
  width: 100%;
  padding: 10px;
  border: 1px solid #ddd;
  border-radius: 5px;
  outline: 0;
  font-size: 16px;
  box-sizing: border-box;
  margin-top: 5px;
}

button {
  padding: 12px 24px;
  border: 0;
  background-color: #5bbc2e;
  color: white;
  border-radius: 10px;
  font-size: 16px;
  outline: 0;
  cursor: pointer;
}

button:hover {
  background-color: #4b9c25;
}
```

### 部署

#### 创建Heroku账户

Heroku提供了一个免费的托管服务。如果你想要部署一个应用程序，你应该有一个Heroku账户。

#### 登录Heroku

```sh
asabeneh@Asabeneh:~/Desktop/python_for_web$ heroku login
heroku: Press any key to open up the browser to login or q to exit:
Opening browser to https://cli-auth.heroku.com/auth/cli/browser/ec0972d5-d8c6-4adf-b004-a42a22dd09a8
Logging in... done
Logged in as asabeneh@gmail.com
asabeneh@Asabeneh:~/Desktop/python_for_web$
```

#### 创建requirements和Procfile

在部署应用程序之前，我们需要告诉Heroku哪些依赖包需要安装，以及如何运行应用程序。Heroku使用requirements.txt文件获取应用程序依赖包的信息。使用pip freeze命令列出所有依赖包及其版本，并将其写入requirements.txt。

```sh
asabeneh@Asabeneh:~/Desktop/python_for_web$ pip freeze
Click==7.0
Flask==1.1.1
itsdangerous==1.1.0
Jinja2==2.10.3
MarkupSafe==1.1.1
Werkzeug==0.16.0
asabeneh@Asabeneh:~/Desktop/python_for_web$ pip freeze > requirements.txt
```

Procfile告诉Heroku如何运行应用程序。在本例中，我们使用Gunicorn作为WSGI HTTP服务器，用于运行Python Web应用程序。我们需要将Gunicorn添加到我们的依赖包中。

```sh
asabeneh@Asabeneh:~/Desktop/python_for_web$ pip install gunicorn
asabeneh@Asabeneh:~/Desktop/python_for_web$ pip freeze > requirements.txt
```

现在，让我们创建一个Procfile，并添加以下内容：

```sh
web: gunicorn app:app
```

#### 将项目推送到Heroku

```sh
asabeneh@Asabeneh:~/Desktop/python_for_web$ heroku create 30-days-of-python-app
Creating ⬢ 30-days-of-python-app... done
https://30-days-of-python-app.herokuapp.com/ | https://git.heroku.com/30-days-of-python-app.git
asabeneh@Asabeneh:~/Desktop/python_for_web$ git init
Initialized empty Git repository in /home/asabeneh/Desktop/python_for_web/.git/
asabeneh@Asabeneh:~/Desktop/python_for_web$ heroku git:remote -a 30-days-of-python-app
set git remote heroku to https://git.heroku.com/30-days-of-python-app.git
asabeneh@Asabeneh:~/Desktop/python_for_web$ echo -e "venv\n.vscode" > .gitignore
asabeneh@Asabeneh:~/Desktop/python_for_web$ git add .
asabeneh@Asabeneh:~/Desktop/python_for_web$ git commit -m "first python web app"
[master (root-commit) 9dfcc6a] first python web app
 9 files changed, 403 insertions(+)
 create mode 100644 .gitignore
 create mode 100644 Procfile
 create mode 100644 app.py
 create mode 100644 requirements.txt
 create mode 100644 static/css/main.css
 create mode 100644 templates/about.html
 create mode 100644 templates/home.html
 create mode 100644 templates/layout.html
 create mode 100644 templates/post.html
 create mode 100644 templates/result.html
asabeneh@Asabeneh:~/Desktop/python_for_web$ git push heroku master
Enumerating objects: 14, done.
Counting objects: 100% (14/14), done.
Delta compression using up to 2 threads
Compressing objects: 100% (12/12), done.
Writing objects: 100% (14/14), 6.08 KiB | 1.52 MiB/s, done.
Total 14 (delta 2), reused 0 (delta 0)
remote: Compressing source files... done.
remote: Building source:
remote:
remote: -----> Python app detected
remote: -----> Installing python-3.6.10
remote: -----> Installing pip
remote: -----> Installing dependencies with Pipenv 2018.5.18…
remote:        Installing dependencies from Pipfile.lock (872ae5)…
remote: -----> Installing SQLite3
remote: -----> $ python manage.py collectstatic --noinput
remote:        Traceback (most recent call last):
remote:          File "manage.py", line 10, in <module>
remote:            from app import app
remote:        ModuleNotFoundError: No module named 'app'
remote:
remote:  !     Error while running '$ python manage.py collectstatic --noinput'.
remote:        See traceback above for details.
remote:
remote:        You may need to update application code to resolve this error.
remote:        Or, you can disable collectstatic for this application:
remote:
remote:           $ heroku config:set DISABLE_COLLECTSTATIC=1
remote:
remote:        https://devcenter.heroku.com/articles/django-assets
remote: -----> Discovering process types
remote:        Procfile declares types -> web
remote:
remote: -----> Compressing...
remote:        Done: 55.7M
remote: -----> Launching...
remote:        Released v3
remote:        https://30-days-of-python-app.herokuapp.com/ deployed to Heroku
remote:
remote: Verifying deploy... done.
To https://git.heroku.com/30-days-of-python-app.git
 * [new branch]      master -> master
asabeneh@Asabeneh:~/Desktop/python_for_web$
```

如你所见，我们已经成功地创建了第一个网络应用程序、对其进行部署，并将其托管在Heroku上。您可以使用此[链接](https://30-days-of-python-app.herokuapp.com/)尝试本应用程序。

事不宜迟，让我们做一些练习，巩固所学到的知识。

## 练习：第26天

1. 创建一个名为"成绩计算器"的Flask应用程序。用户可以输入他们的分数、科目名称，然后应用程序应根据分数显示不同的消息：
   - 如果分数≥90，显示"优秀！你的[科目]成绩是[分数]"。
   - 如果80≤分数<90，显示"很好！你的[科目]成绩是[分数]"。
   - 如果70≤分数<80，显示"一般！你的[科目]成绩是[分数]"。
   - 如果60≤分数<70，显示"及格！你的[科目]成绩是[分数]"。
   - 如果分数<60，显示"你需要更加努力！你的[科目]成绩是[分数]"。

2. 创建一个"体重指数计算器"应用程序，计算体重指数(BMI)，公式为体重(kg)/(身高(m)²)。根据BMI值显示不同的健康状态：
   - BMI<18.5："体重过轻"
   - 18.5≤BMI<24.9："健康体重"
   - 25≤BMI<29.9："超重"
   - BMI≥30："肥胖"

3. 创建一个博客应用程序，用户可以添加、编辑和删除博客文章。

4. 创建一个"任务管理器"应用程序，用户可以添加、查看和删除任务。

🎉 恭喜！🎉

---

## 🔗 相关链接

- 上一篇：[[第25天-Pandas]]
- 下一篇：[[第27天-Python与MongoDB]]
- 返回主页：[[00-索引]]
