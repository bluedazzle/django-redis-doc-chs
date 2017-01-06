# django-redis 官方文档中文版

## 1. 介绍 

django-redis 基于 BSD 许可, 是一个使 Django 支持 Redis cache/session 后端的全功能组件. 

### 1.1 为何要用 django-redis ? 

因为: 

* 持续更新 
* 本地化的 redis-py URL 符号连接字符串 
* 可扩展客户端 
* 可扩展解析器 
* 可扩展序列器 
* 默认客户端主/从支持 
* 完善的测试 
* 已在一些项目的生产环境中作为 cache 和 session 使用 
* 支持无限长超时设置 
* 原生进入 redis 客户端/连接池支持 
* 高可配置 ( 例如仿真缓存的异常行为 ) 
* 默认支持 unix 套接字 
* 支持 Python 2.7, 3.4, 3.5 以及 3.6 

### 1.2 可用的 django-redis 版本 

* 稳定版本: 4.7.0 
* 稳定版本: 3.8.4 

### 1.3 我该使用哪个版本 

版本号像 3.6, 3.7 ... 等的是主要发行版本, 会包含向后不兼容的内容. 跟多信息请在升级前阅读升级日志. 

版本号像 3.7.0, 3.7.1... 等的是小更新或者 bug 修复版本, 一般只会包含 bug 修复, 没有功能更新.

### 1.4 依赖 

#### 1.4.1 Django 版本支持

* django-redis 3.8.x 支持 django 1.4, 1.5, 1.6, 1.7 (或许会有 1.8) 
* django-redis 4.4.x 支持 django 1.6, 1.7, 1.8, 1.9 和 1.10 

#### 1.4.2 Redis Server 支持 

* django-redis 3.x.y 支持 redis-server 2.6.x 或更高 
* django-redis 4.x.y 支持 redis-server 2.8.x 或更高 

#### 1.4.3 其他依赖 

所有版本的 django-redis 基于 redis-py >= 2.10.0. 

## 2. 用户指南 

### 2.1 安装

安装 django-redis 最简单的方法就是用 pip : 

```
pip install django-redis
```

### 2.2 作为 cache backend 使用配置 

为了使用 django-redis , 你应该将你的 django cache setting 改成这样: 

```
CACHES = {
    "default": {
        "BACKEND": "django_redis.cache.RedisCache",
        "LOCATION": "redis://127.0.0.1:6379/1",
        "OPTIONS": {
            "CLIENT_CLASS": "django_redis.client.DefaultClient",
        }
    }
}
```

为了更好的互操作性并使连接字符串更加 "标准", 从 3.8.0 开始 django-redis 使用 redis-py native url notation 作为连接字符串.

*URL 格式举例* 

```
redis://[:password]@localhost:6379/0
rediss://[:password]@localhost:6379/0
unix://[:password]@/path/to/socket.sock?db=0
```

支持三种 URL scheme : 

* redis://: 普通的 TCP 套接字连接 
* rediss://: SSL 包裹的 TCP 套接字连接 
* unix://:  Unix 域套接字连接 

指定数据库数字的方法: 

* db 查询参数, 例如: redis://localhost?db=0 
* 如果使用 redis:// scheme, 可以直接将数字写在路径中, 例如: redis://localhost/0 

在某些环境下连接密码不是 url 安全的, 这时你可以忽略密码或者使用方便的 OPTIONS 设置: 

```
CACHES = {
    "default": {
        "BACKEND": "django_redis.cache.RedisCache",
        "LOCATION": "redis://127.0.0.1:6379/1",
        "OPTIONS": {
            "CLIENT_CLASS": "django_redis.client.DefaultClient",
            "PASSWORD": "mysecret"
        }
    }
}
```

注意, 这样配置不会覆盖 uri 中的密码, 所以如果你已经在 uri 中设置了密码, 此设置将被忽略. 

### 2.3 作为 session backend 使用配置 

Django 默认可以使用任何 cache backend 作为 session backend, 将 django-redis 作为 session 储存后端不用安装任何额外的 backend 

```
SESSION_ENGINE = "django.contrib.sessions.backends.cache"
SESSION_CACHE_ALIAS = "default"
```

### 2.4 使用 django-redis 进行测试 

django-redis 支持定制基于 Redis 的客户端 ( 参考[可扩展 redis 客户端][] ) 可以用来测试, 例如: 替换默认的客户端为 fakerdis (https://github.com/jamesls/fakeredis) 或者 mockredis (https://github.com/locationlabs/mockredis). 这样做可以不用依赖真的 redis server 做集成测试. 

*使用 fakeredis 举例:* 

```
import fakeredis
CACHES = {
    "default": {
        "OPTIONS": {
            "REDIS_CLIENT_CLASS": "fakeredis.FakeStrictRedis",
        }
    }
}
```

如果在测试完毕后想清理所有数据, 在你的 TestCase 中加入如下代码: 

```
def tearDown(self):
    from django_redis import get_redis_connection
    get_redis_connection("default").flushall()
```

## 3. 进阶使用 

### 3.1 Pickle 版本 

django-redis 使用 pickle 序列化几乎所有数据. 

默认使用最新的 pickle. 如果你想设置其他版本, 使用 PICKLE_VERSION 参数: 

```
CACHES = {
    "default": {
        # ...
        "OPTIONS": {
            "PICKLE_VERSION": -1  # Use the latest protocol version
        }
    }
}
```

### 3.2 套接字超时 

套接字超时设置使用 SOCKET_TIMEOUT 和 SOCKET_CONNECT_TIMEOUT 参数: 

```
CACHES = {
    "default": {
        # ...
        "OPTIONS": {
            "SOCKET_CONNECT_TIMEOUT": 5,  # in seconds
            "SOCKET_TIMEOUT": 5,  # in seconds
        }
    }
}
```

SOCKET_CONNECT_TIMEOUT : socket 建立连接超时设置  

SOCKET_TIMEOUT : 连接建立后的读写操作超时设置 

### 3.3 压缩支持 

django-redis 支持压缩, 但默认是关闭的. 你可以激活它: 

```
CACHES = {
    "default": {
        # ...
        "OPTIONS": {
            "COMPRESSOR": "django_redis.compressors.zlib.ZlibCompressor",
        }
    }
}
``` 

使用 lzma 压缩的例子: 

```
import lzma

CACHES = {
    "default": {
        # ...
        "OPTIONS": {
            "COMPRESSOR": "django_redis.compressors.lzma.LzmaCompressor",
        }
    }
}
```

### 3.4 memcached 异常行为 

在某些情况下, redis 只作为缓存使用, 当它关闭时如果你不希望触发异常. 这是 memcached backend 的默认行为, 你可以使用 django-redis 模拟这种情况. 

为了设置这种类似memcached 的行为 ( 忽略连接异常 ), 使用 IGNORE_EXCEPTIONS 参数:

```
CACHES = {
    "default": {
        # ...
        "OPTIONS": {
            "IGNORE_EXCEPTIONS": True,
        }
    }
}
```

Also, you can apply the same settings to all configured caches, you can set the global flag in your settings:

当然,你也可以给所有缓存配置相同的忽略行为: 

```
DJANGO_REDIS_IGNORE_EXCEPTIONS = True

```  

### 3.5 日志忽略异常 

当使用 IGNORE_EXCEPTIONS 或者 DJANGO_REDIS_IGNORE_EXCEPTIONS 参数忽略异常时, 你也许会用到 DJANGO_REDIS_LOG_IGNORED_EXCEPTIONS 参数来配置日志异常: 

```
DJANGO_REDIS_LOG_IGNORED_EXCEPTIONS = True
```

如果你想设置指定的 logger 输出异常, 只需要设置全局变量 DJANGO_REDIS_LOGGER 为 logger 的名称或其路径即可. 如果没有 logger 被设置并且 DJANGO_REDIS_LOG_IGNORED_EXCEPTIONS=True 时此参数将取 __name__ :

```
DJANGO_REDIS_LOGGER = 'some.specified.logger'
```

### 3.6 永不超时设置 

django-redis comes with infinite timeouts support out of the box. And it behaves in same way as django backend contract specifies:

django-redis 支持永不超时设置. 其表现和 django backend 指定的相同:

* timeout=0 立即过期 
* timeout=None 永不超时 

```
cache.set("key", "value", timeout=None)
```

### 3.7 通过值 (value) 获取 ttl (time to live)

With redis, you can access to ttl of any stored key, for it, django-redis exposes ttl function.

It returns:

在 redis 中, 你可以获取任何 key 的 ttl, django-redis 也支持获取 ttl 的函数:

它返回: 

* 0 key 不存在 (或已过期). 
* None key 存在但没有设置过期. 
* ttl 任何有超时设置的 key 的超时值. 

以 keys 搜索过期: 

```
>>> from django.core.cache import cache
>>> cache.set("foo", "value", timeout=25)
>>> cache.ttl("foo")
25
>>> cache.ttl("not-existent")
0
```

### 3.8 expire & persist

除了简单的 ttl 查询, 你可以使用 persist 或者 expire 方法让一个值永久存在或者指定一个新的过期时间: 

使用 persist 的例子:

```
>>> cache.set("foo", "bar", timeout=22)
>>> cache.ttl("foo")
22
>>> cache.persist("foo")
>>> cache.ttl("foo")
None
```

使用 expire 的例子:

```
>>> cache.set("foo", "bar", timeout=22)
>>> cache.expire("foo", timeout=5)
>>> cache.ttl("foo")
5
```

### 3.9 locks 

django-redis 支持 redis 分布式锁. 锁的线程接口是相同的, 因此你可以使用它作为替代. 

*使用 python 上下文管理器分配锁的例子*: 

```
with cache.lock("somekey"):
    do_some_thing()
```








