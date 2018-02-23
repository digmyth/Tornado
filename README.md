# Tornado
Tornado是一个MVC web框架,可以工作在阻塞模式或异步非阻塞模式，因为异步非阻塞工作模式Tornado非常出名，速度相当快，同时可以当作异步非阻塞IO模块来用.
值得花时间来学习它

[参考博客](https://www.cnblogs.com/wupeiqi/articles/5702910.html)

## Tornado 基础知识学习


### 基本应用

a  Django/Tornado比较
Django:
```
  socket:     无，  wsgiref
   中间件：    有
 路由系统：    有
视图函数：     有
    ORM：     有
模板引擎：     有
simple_tag:   有
   cookie:    有  
  session:    有
    csrf:     有
    xss:      有
    其它：缓存，信号，Form, ModelForm,Admin
```    
 Tornado:
```
     socket:    有， 有wsgiref    (Tornado就是用了自有socket实现异步非阻塞和websocket,Tornado如果用了wsgiref就没有异步非阻塞和websocket功能)
     中间件：    没有（自己写）
    路由系统:    有
    视图函数：   有
    uimethod:   有（相当于simple_tag）
    uimodule:   有（相当于simple_tag）
     cookie:    有  
    session:    无
       csrf:    有
        xss:    有
        其它：  没有
```

总结： 一个web框架必须有路由系统，视图函数，模板引警，cookies/csrf_token/xss


b tornado安装：
```
pip3 install tornado
```
快速上手示例
```
import tornado.web
import tornado.ioloop
from tornado.web import RequestHandler

class IndexHandler(RequestHandler):
    def get(self):
        self.write("index page...")

application = tornado.web.Application([
    (r"/index.html", IndexHandler),
])

if __name__ == "__main__":
    application.listen(8888)
    tornado.ioloop.IOLoop.instance().start()
```

c 基本使用
基于URL正则的快速上手示例,随便建个文件就可以工作起来,注意里面传参
Tornado_dir/server.py
```
import tornado.ioloop
import tornado.web

class HomeHandler(tornado.web.RequestHandler):
    def get(self,nid):
        self.write("Home,%s world"%nid)

application = tornado.web.Application([
    (r"/home/(\d+)", HomeHandler),
])

if __name__ == "__main__":
    application.listen(8888)
    tornado.ioloop.IOLoop.instance().start()
```

如果有域名匹配，则域名匹配优先
```
class XxHandler(tornado.web.RequestHandler):
    def get(self):
        self.write("Hello, www.xx.com")
        
application.add_handlers("www.xx.com",[
    (r"/index", XxHandler),
])
```

根据别名反向生成URL，Tornado没有namespace
```
class MainHandler(tornado.web.RequestHandler):
    def get(self):
        url=self.application.reverse_url('n1')
        print("n1", url)

        self.write("Hello, world")

class HomeHandler(tornado.web.RequestHandler):
    def get(self,nid):
        url=self.application.reverse_url('n2',nid)  # 根据别名反向生成URL
        print("n2", url)
        self.write("Hello, home page")

application = tornado.web.Application([
    (r"/index", MainHandler,{},'n1'),               # 第4个参数是别名
    (r"/home/(\d+)", HomeHandler,{},'n2'),
])
```

d  简单写一个登录页面来驱动我们的学习，涉及到目录结构分类
E:\pycharm_project\tornado_t1  项目目录
    tornado_dir
        app.py:  路由关系映射
 
 
 HTML文件位置定义
 ```
 settings = {'template_path': 'views',}
 #views/tpl/login.html
  
 class LoginHandler(tornado.web.RequestHandler):   #后端
    def get(self):
        self.render("tpl/login.html",msg="")
 ```
 
静态文件定义
```
settings = {  # 添加配置项
    'static_path': 'wxq',
    # 'static_url_prefix':'/wxq/',
    }
<link rel="stylesheet" href="/static/css/commons.css">   # 或
<link rel="stylesheet" href="{{ static_url('css/commons.css') }}">  # 推荐用法，内部实现的md5避免缓存
```

后端取值
```
        user=self.get_argument('user')      # get或post取单个值 
        user=self.get_arguments()           # get或post取多个值
        self.get_query_argument()           #  GET 取值
        self.get_query_arguments()          #  多个值
        self.get_body_argument()            #  POST 取值  
        self.get_body_arguments()           #  多个值
        self.request                        #  封装了所有请求信息
        self.request.files("xx")            #  取文件信息
```
xsrf 使用
```
settings = {'set_xsrf': True,} # 添加配置项
<form method="post">     
        {% raw xsrf_form_html() %}
</form>
```

由于没有sesssion,只能用cookie做登录验证，有普通cookie和加密cookie,过期时间这里是时间戳
当设加密cookie时，需要添加配置项
settings = {'cookie_secret':'asdf',}  #  类似于加盐
设cookie
```
    if user == 'wxq' and pwd == '123':
        v = time.time() + 10  # second
        # self.set_cookie('xx', user, expires=v)
        self.set_secure_cookie('xx', user, expires=v)   
        self.redirect("seed.html")
    else:
        self.render("login.html",msg="用户名或密码错误")

```

取cookie
```
class SeedHandler(tornado.web.RequestHandler):
    def get(self):
        # if not self.get_cookie('xx',None):
        if not self.get_secure_cookie('xx',None):
            self.redirect('login.html')
            return None    #  注意这里要有返回，不然代码还会向下执行

        self.write("seed list")

```

模板语言
Tornado与Django略有不同：
```
1  xsrf  {% raw xsrf_form_html() %}
2  值的渲染不在是点，而是python的语法相同 {{ item[0]  }} {{  item.get("title") }}
3  语句endfor endif 变是end 
  <ul>
      {% for item in list_info %}
          <li>{{item}}</li>
      {% end %}
  </ul>

{% if len(items) > 2 %} {% end %}
{% block content %}{% end %}
```


extends include 引入对html的重用性与Django 一样，此处就略过了
```
{%  extends 'layout.html' %}
{% block content %}{% end %}
```

Tornado内置渲染方法
```
在模板中默认提供了一些函数、字段、类以供模板使用：

escape: tornado.escape.xhtml_escape 的別名
xhtml_escape: tornado.escape.xhtml_escape 的別名
url_escape: tornado.escape.url_escape 的別名
json_encode: tornado.escape.json_encode 的別名
squeeze: tornado.escape.squeeze 的別名
linkify: tornado.escape.linkify 的別名
datetime: Python 的 datetime 模组
handler: 当前的 RequestHandler 对象
request: handler.request 的別名
current_user: handler.current_user 的別名
locale: handler.locale 的別名
_: handler.locale.translate 的別名
static_url: for handler.static_url 的別名
xsrf_form_html: handler.xsrf_form_html 的別名
```
当这些渲染方法不能满足我们需求时需要我们自定义类似simple_tag方法:ui_methods/ui_modules
ui_methods
定义
```
tornado_dir/uimethod_test.py
def tab(self):
    return "<a href='http://www.baidu.com'>百度</a>"

```

注册
```
 from tornado_t import uimethod_test as mt
settings = {
    'set_xsrf': True,
    'template_path': 'views',
    'static_path': 'wxq',
    # 'static_url_prefix':'/wxq/',
    'cookie_secret':'asdf',
    'login_url': '/login.html',
    'ui_methods': mt,
   }
```

使用
```
{{ tab() }}     # 不渲染 html标签 
{% raw  tab() %}
```
 
 
 




### 异步非阻塞



### 原生WebSocket
Tornado原生支持Websocket


## 自定义Session组件（公共组件，Tornado为例）

tornado执行到类的get()或post()时己经是对象调用了，那么这个类一定实例化在前，也就是先执行了`__init__()`构造方法,我们在源码里可以看到构造函数最后还有一个勾子函数`self.initialize(**kwargs)`,也就是说我们Handler类可以定义一个initialize方法，不必自己定义重写__init__()构造方法

同时还有一个知识点,`{'k1':'v1'}`参数会传给`initialize(**kwargs)`方法的**kwargs参数，'n1'是URL别名

```
class IndexHandler(RequestHandler):
    def initialize(self,*args,**kwargs):
        print(kwargs)

    def get(self):
        self.write("index page...")

application = tornado.web.Application([
    (r"/index.html", IndexHandler,{'k1':'v1'},'n1'),
])
```

cookies/session流程

1 生成一段随机字符串作为value写入浏览器，key一般为'session_id',称为cookies
2 服务端session中保存敏感信息,key为这个随机字符串，value={'user':'wxq','k1':'v1'}自定义session属性

session_code.py示例代码
```
def gen_random_str():
    md5 = hashlib.md5()
    md5.update(str(time.time()).encode("utf-8"))
    return md5.hexdigest()

class CacheSession():
    container = {}
    def __init__(self,handler):
        self.handler = handler
        self.session_id = settings.SESSION_ID
        self.expires = settings.EXPIRES
        self.initial()

    def initial(self):
        client_random_str=self.handler.get_cookie(self.session_id)
        if client_random_str and client_random_str in self.container:
            self._id = client_random_str
        else:
            self._id = gen_random_str()
            self.container[self._id] = {}  # session 保存
        expires = time.time() + self.expires
        self.handler.set_cookie(self.session_id, self._id, expires=expires)  # set_cookie

    def __getitem__(self, item):
        return self.container[self._id].get(item)

    def __setitem__(self, key, value):
        self.container[self._id][key] = value

    def __delitem__(self, key):
        if key in self.container[self._id]:
            del self.container[self._id][key]
            
class RedisSession():
    def __init__(self,handler):
        self.handler = handler
        self.session_id = settings.SESSION_ID
        self.expires = settings.EXPIRES
        self.initial()

    @property
    def conn(self):
        import redis
        conn = redis.Redis(host='192.168.1.8',port=6379)
        return conn

    def initial(self):
        client_random_str=self.handler.get_cookie(self.session_id)
        if client_random_str and self.conn.exists(client_random_str):
            self._id = client_random_str
        else:
            self._id = gen_random_str()

        expires = time.time() + self.expires
        self.handler.set_cookie(self.session_id, self._id, expires=expires)  # set_cookie
        self.conn.expire(self._id, self.expires)

    def __getitem__(self, item):
        val = self.conn.hget(self._id,item)
        if  val:
            return  json.loads(val)
        else:
            return None

    def __setitem__(self, key, value):
        self.conn.hset(self._id,key,json.dumps(value))

    def __delitem__(self, key):
        self.conn.hdel(self._id,key)
```

Tornado/app.py示例代码

```
import tornado.web
import tornado.ioloop
from tornado.web import RequestHandler
from session_code import SessionFactory

sett = {
    'template_path': 'views',
}

class SessionHandler():
    def initialize(self,**kwargs):
        cls = SessionFactory.getsession()
        self.session = cls(self)    # RedisSession对象、CacheSession对象

class IndexHandler(SessionHandler,RequestHandler):
    def get(self):
        user=self.session['user']
        print(user)
        if user:
            self.write("index page...")
        else:
            self.redirect('/login.html')

class LoginHandler(SessionHandler,RequestHandler):
    def get(self):
        self.render('login.html')

    def post(self, *args, **kwargs):
        user = self.get_argument('user')
        pwd = self.get_argument('pwd')
        if user == 'wxq' and pwd == '123':
            self.session['user'] = user
            self.redirect('/index.html')
            return None
        self.render('login.html')

application = tornado.web.Application([
    (r"/login.html", LoginHandler),
    (r"/index.html", IndexHandler,{'k1':'v1'},'n1'),
],**sett)

if __name__ == "__main__":
    application.listen(8888)
    tornado.ioloop.IOLoop.instance().start()
```

settings.py示例代码
```
ENGINE = 'session_code.RedisSession'
SESSION_ID = '__session_id'
EXPIRES = 300
```

## 自定义Form验证组件（公共组件，Tornado为例）



