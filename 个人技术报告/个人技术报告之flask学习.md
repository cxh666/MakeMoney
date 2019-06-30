- [教程汇总](#教程汇总)
- [python3教程学习笔记](#python3教程学习笔记)
  * [配置](#配置)
  * [hello world](#hello world)
  * [模板](#模板)
  * [表单](#表单)
  * [数据库](#数据库)
  * [shell上下文](#shell上下文)
  * [flask 热加载](#flask 热加载)
  * [用户登录](#用户登录)
- [其他参考链接](#其他参考链接)

## 教程汇总

[python2的flask教程](<http://docs.jinkan.org/docs/flask/>)

[python3的flask教程](<https://juejin.im/entry/5af569076fb9a07aa34a5077>)

[python3 Flask-SQLAlechemy文档](<https://flask-sqlalchemy.palletsprojects.com/en/2.x/>)

[flask-login文档](<https://flask-login.readthedocs.io/en/latest/>)

## python3教程学习笔记

home/xh_files/app_flask

### 配置

环境：python3.7.3 , flask 1.0.3 , flask-wtf

1. flask 更新到最新版本

   flask的更新命令：pip install -U flask

2. 虚拟环境的使用

   在使用flask的时候（以及其他所以依赖的时候），经常遇到不同的程序需要不同版本的三方包，所以可以使用虚拟环境来解决这个问题，python3.5以上可以直接使用`python3 -m venv venv`来创建，然后使用` source venv/bin/activate`来激活并进入venv（如果是低于3.4版本，则安装称为[virtualenv](https://virtualenv.pypa.io/)的第三方工具，然后`virtualenv venv`


### hello world

```python
#app/__init__.py
from flask import Flask

app = Flask(__name__)

from app import routes
```

```python
#app/routers.py
from app import app

@app.route('/')
@app.route('/index')
def index():
    return "Hello, World!"
```

```python
#strat.py
from app import app
```

```python
#.flaskenv
FLASK_APP=start.py
```

运行：`flask run`

### 模板

1. {{...}}是动态的，只有运行时才知道内容。

2. 条件语句：{%...%}。如

   ```python
   {% if title %}
   <title>{{ title }} - Microblog</title>
   {% else %}
   <title>Welcome to Microblog!</title>
   {% endif %}
   ```

3. 循环语句：{%...%}。如

   ```python
   {% for post in posts %}
   <div><p>{{ post.author.username }} says: <b>{{ post.body }}</b></p></div>
   {% endfor %}
   ```

4. 模板的继承

   base.html中用如下代码代替：

   ```
   {% block content %}{% endblock %}
   ```

   index.html中用如下代码集成上述的网页：

   ```
   {% extends "base.html" %}
   
   {% block content %}
       <h1>Hi, {{ user.username }}!</h1>
       {% for post in posts %}
       <div><p>{{ post.author.username }} says: <b>{{ post.body }}</b></p></div>
       {% endfor %}
   {% endblock %}
   ```


### 表单

使用Flask-WTF插件来处理应用中的Web表单

1. 安装

   `pip install flask-wtf`

2. 使用

   ①直接使用app.config对象（不推荐）

   `app.config['SECRET_KEY'] = 'you-will-never-guess'`

   ②使用config.py配置类。

   在顶级目录下面，新建config.py文件：

   ```python
   import os
   
   class Config(object):
       SECRET_KEY = os.environ.get('SECRET_KEY') or 'you-will-never-guess'
   ```

   ```python
   #__init__.py
   from flask import Flask
   from config import Config
   
   app = Flask(__name__)
   app.config.from_object(Config)
   
   from app import routes
   ```

   重新运行strat.py，然后就可以在python的交互式界面中运行：

   ```python
   >>> from microblog import app
   >>> app.config['SECRET_KEY']
   'you-will-never-guess'
   ```

   

3. SECERT_KEY的加密是必要的

4. **整个表单的搭建**

   ①用户登录表单，以FlaskForm为框架，使用wtforms中的表单字段的类来辅助搭建。

   ```python
   #app/forms.py
   from flask_wtf import FlaskForm
   from wtforms import StringField, PasswordField, BooleanField, SubmitField
   from wtforms.validators import DataRequired
   
   class LoginForm(FlaskForm):
       username = StringField('Username', validators=[DataRequired()])
       password = PasswordField('Password', validators=[DataRequired()])
       remember_me = BooleanField('Remember Me')
       submit = SubmitField('Sign In')
   ```

   ②表单模板，新增login.html文件并extends来继承base.html基础模板

   ```python
   #app/templates/login.html
   from flask_wtf import FlaskForm
   from wtforms import StringField, PasswordField, BooleanField, SubmitField
   from wtforms.validators import DataRequired
   
   class LoginForm(FlaskForm):
       username = StringField('Username', validators=[DataRequired()])
       password = PasswordField('Password', validators=[DataRequired()])
       remember_me = BooleanField('Remember Me')
       submit = SubmitField('Sign In')
   ```

   需要注意的是

   > `form.hidden_tag()`模板参数生成了一个隐藏字段，其中包含一个用于保护表单免受CSRF攻击的`token`。 对于保护表单，你需要做的所有事情就是在模板中包括这个隐藏的字段，并在Flask配置中定义`SECRET_KEY`变量，Flask-WTF会完成剩下的工作。

   ③表单视图

   ```python
   #app/routes.py
   from flask import render_template
   from app import app
   from app.forms import LoginForm
   
   # ...
   
   @app.route('/login')
   def login():
       form = LoginForm()
       return render_template('login.html', title='Sign In', form=form)
   ```

   ④优化

   在base.html中可以增加进入login.html的便携链接；

   到此没有说明**接受表单数据**之后如何处理，可以增加处理逻辑，详见教程 接受表单数据 进行修改：增加字段验证的error显示并处理flash的数据进行span显示；

   对/index和/login这类链接的优化：可以使用flask的url_for 来进行批量管理。
   
   5. 总结
   
   文件目录如下：

![1559033140321](/home/xh/.config/Typora/typora-user-images/1559033140321.png)

	先从flaskenv开始，配置了开始为start.py，所以运行flask run开始：
	
	**.flaskenv -> start.py -> app/__init__.py -> config.py & routes.py**	
	
	至此就服务器就开始完毕，等待客户访问，并从routes.py引导客户进入网页，而forms.py则存储表单类，以备routes.py 处理表单。

### 数据库

flask数据库拓展：Flask-SQLAlchemy，属于ORM（object Relational Mapper），ORM允许使用高级实体而不是表和SQL来管理数据库，ORM的工作就是将高级操作转换成数据库命令。

flask数据库迁移框架：flask-migrate

1. 安装

   `pip install flask-sqlalchemy`

   `pip install flask-migrate`

2. 代码

   具体代码编写见[教程]([https://github.com/luhuisicnu/The-Flask-Mega-Tutorial-zh/blob/master/docs/%E7%AC%AC%E5%9B%9B%E7%AB%A0%EF%BC%9A%E6%95%B0%E6%8D%AE%E5%BA%93.md](https://github.com/luhuisicnu/The-Flask-Mega-Tutorial-zh/blob/master/docs/第四章：数据库.md))

3. 使用

   根目录下：`flask db init`初始化数据库，会产生一个migrations的文件夹

   `flask db migrate`自动迁移

   `flask db upgrate`应用迁移（第一次会产生一个app.py 文件）

   > 通过数据库迁移机制的支持，在你修改应用中的模型之后，将生成一个新的迁移脚本（`flask db migrate`），你可能会审查它以确保自动生成的正确性，然后将更改应用到你的开发数据库（`flask db upgrade`）。 测试无误后，将迁移脚本添加到源代码管理并提交。

### shell上下文

可以在start.py中实现一个函数：

```python
from app import app, db
from app.models import User, Post

@app.shell_context_processor
def make_shell_context():
    return {'db': db, 'User': User, 'Post': Post}
```

然后就可以预先导入app。

接着，每次在进入python的shell中进行测试等，就不用每次都输入from...import..了，直接命令行执行flask shell即可。

### flask 热加载

先配置好config.py文件（详见笔记4），在Config函数中加上`TEMPLATES_AUTO_RELOAD = True`，然后在start.py中加上

```python
app.jinja_env.auto_reload = True
app.run(debug=True, host='0.0.0.0')
```

### 用户登录

1. 存储密码使用密码哈希

   ```python
   from werkzeug.security import generate_password_hash
   hash = generate_password_hash('foobar')
   ```

   验证密码正确性：

   ```python
   >>> from werkzeug.security import check_password_hash
   >>> check_password_hash(hash, 'foobar')
   True
   >>> check_password_hash(hash, 'barfoo')
   False
   ```

2. flask-login插件（详细代码见教程第五章）

   1. 安装

      ```python
      pip install flask-login
      ```

   2. 实例在init.py中创建和初始化，并在model.py中加载好模块
   3. 用户登入（意外情况：检查格式，重复登入，密码错误）
   4. 用户登出（新建登出按钮）
   5. 限制部分网页，需要登入才可查看
   6. 显示已经注册的用户
   7. 用户注册（注册名重复，邮件重复）

## 其他参考链接

1. 热加载： <https://zhuanlan.zhihu.com/p/34202182>

   
