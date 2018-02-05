# Tornado
Tornado是一个web框架,可以工作在阻塞模式或异步非阻塞模式，因为异步非阻塞工作模式Tornado非常出名，速度相当快，同时可以当作异步非阻塞IO模块来用.
值得花时间来学习它

[参考博客](https://www.cnblogs.com/wupeiqi/articles/5702910.html)

## Tornado 基础知识学


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
    session:    有
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




### 异步非阻塞



### 原生WebSocket
Tornado原生支持Websocket


## 自定义Session组件（公共组件，Tornado为例）


## 自定义Form验证组件（公共组件，Tornado为例）


