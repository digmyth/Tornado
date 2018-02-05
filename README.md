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

c 基本使用
快速上手,随便建个文件就可以工作起来,注意里面传参
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


### 异步非阻塞



### 原生WebSocket
Tornado原生支持Websocket


## 自定义Session组件（公共组件，Tornado为例）


## 自定义Form验证组件（公共组件，Tornado为例）


