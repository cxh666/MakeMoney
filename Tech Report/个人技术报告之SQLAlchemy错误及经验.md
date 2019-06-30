- [SQL的错误与经验](#SQL的错误与经验)
  * [数据库表的建立](#数据库表的建立)
    + [sqlalchemy中default值的坑](#sqlalchemy中default值的坑)
    + [sqlalchemy中声明类型的坑](#sqlalchemy中声明类型的坑)
    + [sqlalchemy中PickleType的坑](#sqlalchemy中PickleType的坑)
    + [sqlalchemy中关联的坑](#sqlalchemy中关联的坑)
  * [多对多查询](#多对多查询)
  * [Like查询](#Like查询)
  * [参考链接](#参考链接)

# SQL的错误与经验

搭建完数据库的感想就是：不要使用flask_sqlalchemy！如果要使用sqlite，也请使用原生的sqlalchemy，不过最好搭建完整的sql吧。

## 数据库表的建立

### sqlalchemy中default值的坑

**这个模块对于default关键字是没有直接加载的！**但是你又想给这些数据一个初始化的值怎么办。

1. 如果只是想要一个默认初始化值，而不能给用户初始化其他值，那么直接在init函数中初始化即可

```python
	def __init__(self, **kwargs):
		self.exMoney = 0
		self.income = 0
		self.expend = 0
        super(User, self).__init__(**kwargs)
```

2. 如果是实现，如果用户提供初始化值则使用用户的初始化值，否则则默认初始化。

```python
	#初始化
	def __init__(self, **kwargs):
		if 'start_time' not in kwargs:
			kwargs['start_time'] = self.__table__.c.start_time.default.arg
		if 'end_time' not in kwargs:
			kwargs['end_time'] = self.__table__.c.end_time.default.arg
		if 'type' not in kwargs:
			kwargs['type'] = self.__table__.c.type.default.arg
		super(Task, self).__init__(**kwargs)
```

### sqlalchemy中声明类型的坑

因为python中类型是可以直接赋值并改变的。

也就是说`a = 1`的时候a是一个int类型，但是直接`a = 'str'`，a就直接变成了string类型。

这样子问题就出现了，**sqlalchemy是没有对类型进行审核的！**

所以就需要在后端自己对类型进行转换，增加了相当多的工作量。

### sqlalchemy中PickleType的坑

sqlalchemy中，PickleType是个非常好用的东西，可以减少很多类的使用，也可以减少很多关系，对于数据库的搭建来说可以说简单很多。

**但是呢！PickleType值的替换是一个非常坑的事情！**

也就是说修改了一个值`a = [1,2,3]`，接下来要对a进行修改直接赋值并`session.commit()`是不对成功的。只能将a删除，然后重新赋值一个新的a，再重新提交才能成功。

### sqlalchemy中关联的坑

sqlAlchemy中的关联对一个关联关系都没有使用，只是用relationship，所以使用起来需要自己注意一下数据库的关系搭建，`特别是数据库中的对象的关联强弱问题`。

## 多对多查询

在数据库建立多对多关系之后，当要从一个表查询后得到另一个表的内容时，如Task和User，查询Task中User的username的时候，不能从Task出发，应该查到User再用User的backref拿到所有tasks。

## Like查询

/search/title_key_word

```
#正确的like查询
task_list = db.session.query(Task).filter(Task.title.like(key)).all()
#错误的like查询
task_list = Task.query.filter_by(Task.title.like(key))
```

## 参考链接

[Default value doesn't work](<https://github.com/aio-libs/aiopg/issues/49>)

[Column Insert/Update **Default**s — SQLAlchemy 1.3 Documentation](https://docs.sqlalchemy.org/en/13/core/defaults.html?highlight=schema column#sqlalchemy.schema.ColumnDefault)

[python - flask-sqlalchemy append to **pickle**type doesn't update - Stack Overflow](https://stackoverflow.com/questions/47998500/flask-sqlalchemy-append-to-pickletype-doesnt-update)

[day12-sqlalchemy 多对多关联](https://www.cnblogs.com/zhangqigao/articles/7722418.html)

[Flask-sqlalchemy多对多关系](<https://blog.csdn.net/Co_zy/article/details/78147903>)
