

## 自定义Session组件（公共组件，Tornado为例）


Tornado有cookie功能，但缺少session功能，这里我们自己开发实现一个Tornado 的session功能

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

