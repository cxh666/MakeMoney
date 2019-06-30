[TOC]

## 错误整理

[查询方式](<https://juejin.im/post/5bf741886fb9a049fa0f671e#heading-4>)

SQLAlchemy作为一个轻量型的数据库插件，使用起来是相当方便的

### 外键的问题

在使用外键如`db.ForeignKey('task.id')`的时候，这个task直接对应所对应的类的名称，而且是class第一个首字母大小写不区分（注意task.id需要是小写）

### 建立关系的方式

①task.author.append(user)

②user.tasks = task

③user初始化时tasks = task



### 有一个心态爆炸的bug！default的问题

sqlachemy的default有特别多的问题！就算使用网上的server_default啊还是其他的初始化方法，都没有起作用，那个值就是None，啥办法都没，到现在也没有解决，准备在注册的时候，直接初始化

**虽然default不能用，但是nullable还是可以用的**

！！解决：新增一个init函数，在初始化的过程中，如果没有这个类目，则加入default值并新增一个类目，然后进行初始化，代码如下：

```python
class User(UserMixin, db.Model):
	def __init__(self, **kwargs):
		if 'email' not in kwargs:
			kwargs['email'] = self.__table__.c.email.default.arg
		super(User, self).__init__(**kwargs)
    
	#学号(账号)
	id = db.Column(db.Integer, primary_key=True)
	#用户名
	username = db.Column(db.String(20), nullable=False)
	#密码hash
	password_hash = db.Column(db.String(128))
	#邮箱
	email = db.Column(db.String(20), default='123')
```

### 表之间建立关系

需要建立不同的key来实现，主要是声明ForeignKey。


1. 一对一

   ```python
   	sponsor_id = db.Column(db.Integer, db.ForeignKey('user.id'))
   	sponsor = db.relationship('User', backref=db.backref('sponsor_tasks', lazy=True, uselist=False))
   ```

   

2. 一对多

   ```python
   	sponsor_id = db.Column(db.Integer, db.ForeignKey('user.id'))
   	sponsor = db.relationship('User', backref=db.backref('sponsor_tasks', lazy=True))
   ```

   

3. 多对多

   ```python
   tags = db.Table('tags',
       db.Column('tag_id', db.Integer, db.ForeignKey('tag.id'), primary_key=True),
       db.Column('page_id', db.Integer, db.ForeignKey('page.id'), primary_key=True)
   )
   
   class Page(db.Model):
       id = db.Column(db.Integer, primary_key=True)
       tags = db.relationship('Tag', secondary=tags, lazy='subquery',
           backref=db.backref('pages', lazy=True))
   
   class Tag(db.Model):
       id = db.Column(db.Integer, primary_key=True)
       def __repr__(self):
       	return '<Task {}>'.format(self.id)
   ```

### lazy的意义

### key的使用

在使用sql的时候，可以设置id为primary key ，但是在初始化这个class的时候不给id赋值，这时候的id是没有值的。但是当使用db提交之后，sql就会给这个class分配一个id（从1开始，跳过重复）

```python
class Test(db.Model):
	id = db.Column(db.Integer, primary_key=True)
	name = db.Column(db.String)
```

```shell
>>> from app.models import Test
>>> t = Test(name='test')
>>> t
<Test2 (transient 140623735572688)>
>>> t.id
>>> db.session.add(t)
>>> db.session.commit()
>>> test = Test2.query.all()
>>> test
[<Test2 1>]
>>> t2 = Test(name='test2')
>>> t2.id
>>> t.id
1
```

### 有一个db.PickleType的问题

就是在使用这个类型的时候，会使得整个数据库和编程方便很多，但是在修改数据的时候会比较麻烦，比如一个属性a = db.PickleType。给a赋值a=[obj1, obj2]，其中obj为类。这时候如果要修改a的值，直接a[0].xxx=???的方式，是没有办法修改的。

原因：因为obj存在了数据的其他位置，而且不会随着commit而修改。

解决：所以需要重新把a的所有值拿出来`data = test.a`然后修改data，再把data赋值回去。



后来发现还是不能解决，只能把data提取出来，然后新建一个新的，再进行修改并提交，把原来的删除即可。虽然很费事，但是实在没找到更好的解决办法。

[flask-sqlalchemy append to pickletype doesn't update(https://stackoverflow.com/questions/47998500/flask-sqlalchemy-append-to-pickletype-doesnt-update)](<https://stackoverflow.com/questions/47998500/flask-sqlalchemy-append-to-pickletype-doesnt-update>)

## 其他参考链接

[sqlalchemy常见数据类型及配置](<https://blog.csdn.net/qq_29730101/article/details/81566569>)

[python – 提供存储在SQLAlchemy LargeBinary列中的图像](https://codeday.me/bug/20180909/246604.html)

[python – SQLAlchemy在一个映射类中的多个外键到同一个主键(https://codeday.me/bug/20180820/222510.html)](<https://codeday.me/bug/20180820/222510.html>)

[一个多选且可以补充的属性，数据库表如何设计？](<https://www.zhihu.com/question/21601697>)

[Python sqlalchemy.Enum() Examples](<https://www.programcreek.com/python/example/97906/sqlalchemy.Enum>)