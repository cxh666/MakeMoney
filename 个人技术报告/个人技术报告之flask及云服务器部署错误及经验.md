[TOC]

## flask的错误与经验

### flask_login

1. 出现一个耗时特别久的事情！

   在使用flask_login的时候，没有在 init.py中加入：

   ```
   login = LoginManager(app)
   login.login_view = 'login'
   ```

   在model.py中加入：

   ```python
   #重点就是这个，没有加！而且可以使用print试一下有没有输出6，原来的程序就是一直没有输出6，经过排查，知道了是python程序是没有和服务器持续建立链接的，所以每次登录之后，就直接退出了，导致了系统自动退出，如果在login之后redirect到logout就可以得到正确的结果了。
   @login.user_loader
   def load_user(id):
   	print(6)
   	return User.query.get(int(id))
   ```

2. 上面那个问题！关键在与测试程序！谁能想到？

   使用的原来的测试程序：

   ```python
   #给本地服务器传递post数据，并接受post数据，测试本地服务器的正常运行
   import json
   import requests
   import pdb
   
   url = 'http://localhost:5050/login'
   s = json.dumps({'id': 16340000, 'username': 'xiaotong', 'password': 'xiaozhu'})
   r = requests.post(url, data=s)
   print(r)
   pdb.set_trace()
   url = 'http://localhost:5050/logout'
   r = requests.get(url)
   print(r)
   
   pdb.set_trace()
   
   print('end-----------')
   ```

   后来的测试程序（问题就在requests请求的不连续性！）：

   ```python
   #给本地服务器传递post数据，并接受post数据，测试本地服务器的正常运行
   import json
   import requests
   import pdb
   
   
   url = 'http://localhost:5050/login'
   s = json.dumps({'id': 16340000, 'username': 'xiaotong', 'password': 'xiaozhu'})
   r = requests.post(url, data=s)
   print(r)
   pdb.set_trace()
   url = 'http://localhost:5050/logout'
   r = requests.get(url)
   print(r)
   
   pdb.set_trace()
   
   print('end-----------')
   ```

### 时间

在使用任务的过程中有使用到时间，python中时间的一种经典的模块就是datetime。

1. 基本使用方法

```python
from datetime import datetime, timedelta
#创建
time = datetime.now()
print ("当前的日期和时间是 %s" % time)
print ("ISO格式的日期和时间是 %s" % time.isoformat() )
print ("当前的年份是 %s" %time.year)
print ("当前的月份是 %s" %time.month)
print ("当前的日期是  %s" %time.day)
print ("dd/mm/yyyy 格式是  %s/%s/%s" % (time.day, time.month, time.year) )
print ("当前小时是 %s" %time.hour)
print ("当前分钟是 %s" %time.minute)
print ("当前秒是  %s" %time.second)

#加减
#构造方法：datetime.timedelta(days=0, seconds=0, microseconds=0, milliseconds=0, minutes=0, hours=0, weeks=0)
delta = timedelta(days=1)
time2 = time + delta

#输出
print(time)
print(time2)
```

2. 与str的转换，小程序前端的时间存在.000z，也可以进行转换

```python
#将str转换为datetime形式
from datetime import datetime
format = '%Y-%m-%d %H:%M:%S'
datetime.strptime(time1, format)

#如果有.000Z
format = '%Y-%m-%d %H:%M:%S.%fZ'
```


## 其他

### 新浪云容器部署

本以为简单的上传运行就可以解决的事情，结果弄了特别长的时间才终于解决。总之就是，容器耗费更少更好管理，但是使用起来确实比较麻烦，如果只是简单的使用，还是直接使用服务器吧。



1. 首先是注册和登录新浪云，这个就不赘述，[网址](<https://www.sinacloud.com/>)
2. 然后使用云容器和云应用都可以，如果只是搭建后端的话，使用云容器即可，花费便宜一些。
3. 云容器创建示例

![1559321464075](/home/xh/.config/Typora/typora-user-images/1559321464075.png)

4. 创建成功之后，一般会使用到 运行环境管理--容器管理和应用设置和应用总览

5. 使用git上传代码

   ```git
   git init
   git remote rm sinacloud
   git remote add sinacloud https://git.sinacloud.com/mytest2
   git add .
   git commit -m "mytest2"
   git push sinacloud master
   ```

6. 上传之后，就可以在容器的终端进行操作并运行了（记得先修改代码为监听5050端口）

7. 然后就可以通过 <https://mytest2.applinzi.com/> 来访问服务器了

### 腾讯云服务器部署

​	在服务器上搭建比较方便，直接运行0.0.0.0即可，注意不要使用到127.0.0.1就行，然后就可以直接通过浏览器ip:port访问了，相比起容器方便很多。



## 参考链接

[新浪云SAE使用入门，教你如何发布自己的网站](https://www.cnblogs.com/jhmydear/p/3926478.html)

[新浪云SAE做小程序后台](<https://developers.weixin.qq.com/community/develop/doc/026342388cfe6778f250809b3ef553d2>)

[新浪云]([https://www.sinacloud.com/doc/search.html?q=%E5%AE%B9%E5%99%A8&check_keywords=yes&area=default](https://www.sinacloud.com/doc/search.html?q=容器&check_keywords=yes&area=default))

[[flask如何记录登录状态](https://segmentfault.com/q/1010000007156283)](<https://segmentfault.com/q/1010000007156283>)

[Flask-Login 让实现登录功能变简单](<https://blog.csdn.net/stone0823/article/details/85163369>)

[Python 日期和时间](<https://www.runoob.com/python/python-date-time.html>)

[数据库中的Date,DateTime和TimeStamp类型](<https://blog.csdn.net/shendl/article/details/1790933>)

[python中date、datetime、string的相互转换](https://www.cnblogs.com/cathouse/archive/2012/11/19/2777678.html)