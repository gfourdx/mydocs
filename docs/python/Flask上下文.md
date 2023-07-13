# Flask上下文

---

###  应用程序上下文

参考: https://flask.palletsprojects.com/en/latest/appcontext/

即The Application Context, 具体表现为current_app和g

##### current_app

在工厂函数中实例化的Flask对象app时有个重要的属性config

在其他模块(比如view、model等)中要访问 config 中的配置需要导入 app

但app所处的工厂函数有诸如注册蓝图等操作, 这样会就可能会导致循环导入

然而, Flask处理请求期间会自动推送应用程序上下文

在请求期间运行的视图函数、错误处理程序和其他函数等均可以访问current_app

current_app指向处理当前活动的应用程序(即Flask的实例app)

此时用current_app即可解决致循环导入问题

##### g

g变量是一个应用程序级别的全局变量, 与current_app有相同的生命周期

但它被设计为在处理单个请求时存储临时数据, 即不同请求之间的 g 变量是不同的

当请求处理结束时，请求上下文环境将被销毁，并且 g 对象也将被删除

一个简单的例子是使用钩子函数在每个请求之前检查token并将token存储到g变量

后续真正的请求就可以拿到g变量中存储的token

### 请求上下文

参考: https://flask.palletsprojects.com/en/latest/reqcontext/

即The Request Context, 具体表现为request和session

##### request

处理请求时Flask会自动创建推送请求上下文(RequestContext )

其中包含一个新的 request 对象, 该对象表示当前HTTP请求

处理结束后请求上下文(包括request对象)会随之被删除

在请求期间运行的视图函数、错误处理程序和其他函数等

均可以访问请求上下文(都指向同一个request对象)

请求上下文对于每一个线程都是唯一的, 不同线程有不同的上下文空间, 并且不知道父线程指向的请求

##### session

每个请求都会有自己独立的session对象

因此在多个并发请求之间不会出现数据竞争的问题

可以使用session对象方便地存储和访问与客户端会话相关的数据

> **current_app、g、request和session都是线程安全的**

### 上下文推送与弹出过程

简单来说就是:

1、Flask应用程序开始处理请求

2、推送应用程序上下文

3、推送请求上下文

4、请求结束

5、弹出请求上下文

6、弹出应用程序上下文

---

### 请求与响应过程

上面提到的[上下文推送与弹出过程](#上下文推送与弹出过程)是从客户端发起请求到服务端返回响应的其中一部分

整体流程参考: https://flask.palletsprojects.com/en/latest/lifecycle/#serving-the-application

具体来说就是:

1、使用浏览器或其他工具发送一个HTTP请求(request)

2、WSGI服务器收到请求

3、WSGI服务器将HTTP data(数据)转换为WSGI environ dict

4、WSGI服务器携带上面environ调用WSGI application(即Flask)

5、Flask进行所有内部处理以将请求路由到视图函数，处理错误等

6、Flask将视图函数返回转换为WSGI响应数据，并将其传递给WSGI服务器

7、WSGI服务器创建并发送HTTP响应(response)

8、客户端收到HTTP响应

[上下文推送与弹出过程](#上下文推送与弹出过程)发生在步骤4中, 更为具体过程请参考:

https://flask.palletsprojects.com/en/latest/lifecycle/#how-a-request-is-handled

---

### 扩展

`current_app._get_current_object()`和`current_app`的区别

current_app._get_current_object() 返回当前应用对象的非代理版本, 即真实的Flask实例对象

current_app 是一个通过LocalProxy返回的线程本地变量(Thread Local Variable)

LocalProxy 的作用就是可以根据线程/协程返回对应当前协程/线程的对象，也就是说

- 线程 A 往 LocalProxy 中塞入 A
- 线程 B 往 LocalProxy 中塞入 B

无论在是什么地方，

- 线程 A 永远取到得是 A，线程 B 取到得永远是 B

代码案例:

```
from threading import Thread
from flask import current_app, render_template
from flask.ext.mail import Message
from . import mail


def send_async_email(app, msg):
    with app.app_context():
    	mail.send(msg)


def send_email(to, subject, template, **kwargs):
	app = current_app._get_current_object()
	msg = Message(app.config['FLASKY_MAIL_SUBJECT_PREFIX'] + ' ' + subject, sender=app.config['FLASKY_MAIL_SENDER'], recipients=[to])
    msg.body = render_template(template + '.txt', **kwargs)
    msg.html = render_template(template + '.html', **kwargs)
    thr = Thread(target=send_async_email, args=[app, msg])
    thr.start()
    return thr
```

这里使用current_app._get_current_object()不使用current_app

就是应为通过Thread线程开了另一个线程, 因为再另一个线程中是获取不到current_app

为了在线程中使用Flask的上下文就必须使用current_app._get_current_object()

