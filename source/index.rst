django-redis 中文文档
=====================

Andrey Antukh, niwi@niwi.be 4.7.0

翻译: `RaPoSpectre`_

.. _RaPoSpectre: https://www.rapospectre.com

1. 介绍
-------

django-redis 基于 BSD 许可, 是一个使 Django 支持 Redis cache/session
后端的全功能组件.

1.1 为何要用 django-redis ?
~~~~~~~~~~~~~~~~~~~~~~~~~~~

因为:

-  持续更新
-  本地化的 redis-py URL 符号连接字符串
-  可扩展客户端
-  可扩展解析器
-  可扩展序列器
-  默认客户端主/从支持
-  完善的测试
-  已在一些项目的生产环境中作为 cache 和 session 使用
-  支持永不超时设置
-  原生进入 redis 客户端/连接池支持
-  高可配置 ( 例如仿真缓存的异常行为 )
-  默认支持 unix 套接字
-  支持 Python 2.7, 3.4, 3.5 以及 3.6

1.2 可用的 django-redis 版本
~~~~~~~~~~~~~~~~~~~~~~~~~~~~

-  稳定版本: 4.7.0
-  稳定版本: 3.8.4

1.3 我该使用哪个版本
~~~~~~~~~~~~~~~~~~~~

版本号像 3.6, 3.7 … 等的是主要发行版本, 会包含向后不兼容的内容.
跟多信息请在升级前阅读升级日志.

版本号像 3.7.0, 3.7.1… 等的是小更新或者 bug 修复版本, 一般只会包含 bug
修复, 没有功能更新.

1.4 依赖
~~~~~~~~

1.4.1 Django 版本支持
^^^^^^^^^^^^^^^^^^^^^

-  django-redis 3.8.x 支持 django 1.4, 1.5, 1.6, 1.7 (或许会有 1.8)
-  django-redis 4.4.x 支持 django 1.6, 1.7, 1.8, 1.9 和 1.10

1.4.2 Redis Server 支持
^^^^^^^^^^^^^^^^^^^^^^^

-  django-redis 3.x.y 支持 redis-server 2.6.x 或更高
-  django-redis 4.x.y 支持 redis-server 2.8.x 或更高

1.4.3 其他依赖
^^^^^^^^^^^^^^

所有版本的 django-redis 基于 redis-py >= 2.10.0.

2. 用户指南
-----------

2.1 安装
~~~~~~~~

安装 django-redis 最简单的方法就是用 pip :

::

    pip install django-redis

2.2 作为 cache backend 使用配置
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

为了使用 django-redis , 你应该将你的 django cache setting 改成这样:

::

    CACHES = {
        "default": {
            "BACKEND": "django_redis.cache.RedisCache",
            "LOCATION": "redis://127.0.0.1:6379/1",
            "OPTIONS": {
                "CLIENT_CLASS": "django_redis.client.DefaultClient",
            }
        }
    }

为了更好的互操作性并使连接字符串更加 “标准”, 从 3.8.0 开始 django-redis
使用 redis-py native url notation 作为连接字符串.

*URL 格式举例*

::

    redis://[:password]@localhost:6379/0
    rediss://[:password]@localhost:6379/0
    unix://[:password]@/path/to/socket.sock?db=0

支持三种 URL scheme :

-  redis://: 普通的 TCP 套接字连接
-  rediss://: SSL 包裹的 TCP 套接字连接
-  unix://: Unix 域套接字连接

指定数据库数字的方法:

-  db 查询参数, 例如: redis://localhost?db=0
-  如果使用 redis:// scheme, 可以直接将数字写在路径中, 例如:
   redis://localhost/0

在某些环境下连接密码不是 url 安全的, 这时你可以忽略密码或者使用方便的
OPTIONS 设置:

::

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

注意, 这样配置不会覆盖 uri 中的密码, 所以如果你已经在 uri 中设置了密码,
此设置将被忽略.

2.3 作为 session backend 使用配置
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Django 默认可以使用任何 cache backend 作为 session backend, 将
django-redis 作为 session 储存后端不用安装任何额外的 backend

::

    SESSION_ENGINE = "django.contrib.sessions.backends.cache"
    SESSION_CACHE_ALIAS = "default"


2.4 使用 django-redis 进行测试
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

django-redis 支持定制基于 Redis 的客户端 ( 参考[可扩展 redis 客户端][] )
可以用来测试, 例如: 替换默认的客户端为 fakerdis
(https://github.com/jamesls/fakeredis) 或者 mockredis
(https://github.com/locationlabs/mockredis). 这样做可以不用依赖真的
redis server 做集成测试.

*使用 fakeredis 举例:*

::

    import fakeredis
    CACHES = {
        "default": {
            "OPTIONS": {
                "REDIS_CLIENT_CLASS": "fakeredis.FakeStrictRedis",
            }
        }
    }

如果在测试完毕后想清理所有数据, 在你的 TestCase 中加入如下代码:

::

    def tearDown(self):
        from django_redis import get_redis_connection
        get_redis_connection("default").flushall()

3. 进阶使用
-----------

3.1 Pickle 版本
~~~~~~~~~~~~~~~

django-redis 使用 pickle 序列化几乎所有数据.

默认使用最新的 pickle. 如果你想设置其他版本, 使用 PICKLE\_VERSION 参数:

::

    CACHES = {
        "default": {
            # ...
            "OPTIONS": {
                "PICKLE_VERSION": -1  # Use the latest protocol version
            }
        }
    }

3.2 套接字超时
~~~~~~~~~~~~~~

套接字超时设置使用 SOCKET\_TIMEOUT 和 SOCKET\_CONNECT\_TIMEOUT 参数:

::

    CACHES = {
        "default": {
            # ...
            "OPTIONS": {
                "SOCKET_CONNECT_TIMEOUT": 5,  # in seconds
                "SOCKET_TIMEOUT": 5,  # in seconds
            }
        }
    }

SOCKET\_CONNECT\_TIMEOUT : socket 建立连接超时设置

SOCKET\_TIMEOUT : 连接建立后的读写操作超时设置

3.3 压缩支持
~~~~~~~~~~~~

django-redis 支持压缩, 但默认是关闭的. 你可以激活它:

::

    CACHES = {
        "default": {
            # ...
            "OPTIONS": {
                "COMPRESSOR": "django_redis.compressors.zlib.ZlibCompressor",
            }
        }
    }

使用 lzma 压缩的例子:

::

    import lzma

    CACHES = {
        "default": {
            # ...
            "OPTIONS": {
                "COMPRESSOR": "django_redis.compressors.lzma.LzmaCompressor",
            }
        }
    }

3.4 memcached 异常行为
~~~~~~~~~~~~~~~~~~~~~~

在某些情况下, redis 只作为缓存使用, 当它关闭时如果你不希望触发异常. 这是
memcached backend 的默认行为, 你可以使用 django-redis 模拟这种情况.

为了设置这种类似memcached 的行为 ( 忽略连接异常 ), 使用
IGNORE\_EXCEPTIONS 参数:

::

    CACHES = {
        "default": {
            # ...
            "OPTIONS": {
                "IGNORE_EXCEPTIONS": True,
            }
        }
    }

Also, you can apply the same settings to all configured caches, you can
set the global flag in your settings:

当然,你也可以给所有缓存配置相同的忽略行为:

::

    DJANGO_REDIS_IGNORE_EXCEPTIONS = True

3.5 日志忽略异常
~~~~~~~~~~~~~~~~

当使用 IGNORE\_EXCEPTIONS 或者 DJANGO\_REDIS\_IGNORE\_EXCEPTIONS
参数忽略异常时, 你也许会用到 DJANGO\_REDIS\_LOG\_IGNORED\_EXCEPTIONS
参数来配置日志异常:

::

    DJANGO_REDIS_LOG_IGNORED_EXCEPTIONS = True

如果你想设置指定的 logger 输出异常, 只需要设置全局变量
DJANGO\_REDIS\_LOGGER 为 logger 的名称或其路径即可. 如果没有 logger
被设置并且 DJANGO\_REDIS\_LOG\_IGNORED\_EXCEPTIONS=True 时此参数将取
**name** :

::

    DJANGO_REDIS_LOGGER = 'some.specified.logger'

3.6 永不超时设置
~~~~~~~~~~~~~~~~

django-redis comes with infinite timeouts support out of the box. And it
behaves in same way as django backend contract specifies:

django-redis 支持永不超时设置. 其表现和 django backend 指定的相同:

-  timeout=0 立即过期
-  timeout=None 永不超时

::

    cache.set("key", "value", timeout=None)


3.7 通过值 (value) 获取 ttl (time to live)
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

With redis, you can access to ttl of any stored key, for it,
django-redis exposes ttl function.

It returns:

在 redis 中, 你可以获取任何 key 的 ttl, django-redis 也支持获取 ttl
的函数:

它返回:

-  0 key 不存在 (或已过期).
-  None key 存在但没有设置过期.
-  ttl 任何有超时设置的 key 的超时值.

以 keys 搜索过期:

::

    >>> from django.core.cache import cache
    >>> cache.set("foo", "value", timeout=25)
    >>> cache.ttl("foo")
    25
    >>> cache.ttl("not-existent")
    0

3.8 expire & persist
~~~~~~~~~~~~~~~~~~~~

除了简单的 ttl 查询, 你可以使用 persist 或者 expire
方法让一个值永久存在或者指定一个新的过期时间:

使用 persist 的例子:

::

    >>> cache.set("foo", "bar", timeout=22)
    >>> cache.ttl("foo")
    22
    >>> cache.persist("foo")
    >>> cache.ttl("foo")
    None

使用 expire 的例子:

::

    >>> cache.set("foo", "bar", timeout=22)
    >>> cache.expire("foo", timeout=5)
    >>> cache.ttl("foo")
    5

3.9 locks
~~~~~~~~~

django-redis 支持 redis 分布式锁. 锁的线程接口是相同的,
因此你可以使用它作为替代.

*使用 python 上下文管理器分配锁的例子*:

::

    with cache.lock("somekey"):
        do_some_thing()


3.10 扫描 & 删除键 (keys)
~~~~~~~~~~~~~~~~~~~~~~~~~

django-redis 支持使用全局通配符的方式来检索或者删除键.

*使用通配符搜索的例子*

::

    >>> from django.core.cache import cache
    >>> cache.keys("foo_*")
    ["foo_1", "foo_2"]

这个简单的写法将返回所有匹配的值,
但在拥有很大数据量的数据库中这样做并不合适. 在 redis 的 server side
cursors 2.8 版及以上, 你可以使用 ``iter_keys`` 取代 ``keys`` 方法,
``iter_keys`` 将返回匹配值的迭代器, 你可以使用迭代器高效的进行遍历.

*使用 server side cursors 搜索*

::

    >>> from django.core.cache import cache
    >>> cache.iter_keys("foo_*")
    <generator object algo at 0x7ffa9c2713a8>
    >>> next(cache.iter_keys("foo_*"))
    "foo_1"

如果要删除键, 使用 ``delete_pattern`` 方法, 它和 ``keys``
方法一样也支持全局通配符, 此函数将会返回删掉的键的数量

*使用 delete\_pattern 的例子*

::

    >>> from django.core.cache import cache
    >>> cache.delete_pattern("foo_*")

3.11 Redis 本地命令
~~~~~~~~~~~~~~~~~~~

django-redis 有限制的支持一些 Redis 原子操作, 例如 ``SETNX`` 和 ``INCR``
命令.

你可以在 set() 方法中加上 ``nx`` 参数使用来使用 ``SETNX`` 命令

*例子:*

::

    >>> from django.core.cache import cache
    >>> cache.set("key", "value1", nx=True)
    True
    >>> cache.set("key", "value2", nx=True)
    False
    >>> cache.get("key")
    "value1"

当值 (value) 有合适的键 (key) 时, ``incr`` 和 ``decr`` 也可以使用 Redis
原子操作

3.12 原生客户端使用
~~~~~~~~~~~~~~~~~~~

在某些情况下你的应用需要进入原生 Redis 客户端使用一些 django cache
接口没有暴露出来的进阶特性. 为了避免储存新的原生连接所产生的另一份设置,
django-redis 提供了方法 ``get_redis_connection(alias)``
使你获得可重用的连接字符串.

::

    >>> from django_redis import get_redis_connection
    >>> con = get_redis_connection("default")
    >>> con
    <redis.client.StrictRedis object at 0x2dc4510>

**警告 不是所有的扩展客户端都支持这个特性.**

3.13 连接池
~~~~~~~~~~~

django-redis 使用 redis-py 的连接池接口, 并提供了简单的配置方式.
除此之外, 你可以为 backend 定制化连接池的产生.

redis-py 默认不会关闭连接, 尽可能重用连接

3.13.1 配置默认连接池
^^^^^^^^^^^^^^^^^^^^^

配置默认连接池很简单, 你只需要在 ``CACHES`` 中使用
``CONNECTION_POOL_KWARGS`` 设置连接池的最大连接数量即可:

::

    CACHES = {
        "default": {
            "BACKEND": "django_redis.cache.RedisCache",
            ...
            "OPTIONS": {
                "CONNECTION_POOL_KWARGS": {"max_connections": 100}
            }
        }
    }

你可以得知连接池已经打开多少连接:

::

    from django.core.cache import get_cache
    from django_redis import get_redis_connection

    r = get_redis_connection("default")  # Use the name you have defined for Redis in settings.CACHES
    connection_pool = r.connection_pool
    print("Created connections so far: %d" % connection_pool._created_connections)

3.13.2 使用你自己的连接池子类
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

有时你想使用自己的连接池子类. django-redis 提供了
``CONNECTION_POOL_CLASS`` 来配置连接池子类

*myproj/mypool.py*

::

    from redis.connection import ConnectionPool

    class MyOwnPool(ConnectionPool):
        # Just doing nothing, only for example purpose
        pass

*setting.py*

::

    # Omitting all backend declaration boilerplate code.

    "OPTIONS": {
        "CONNECTION_POOL_CLASS": "myproj.mypool.MyOwnPool",
    }

3.13.3 定制化的 connection factory
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

如果之前的方法都不合适, 你可以定制 django-redis 的 connection factory
过程甚至完全重写.

django-redis 默认使用Django setting 中
``DJANGO_REDIS_CONNECTION_FACTORY`` 参数指定的
``django_redis.pool.ConnectionFactory`` 类产生连接.

*ConnectionFactory 类的部分接口*

::

    # Note: Using Python 3 notation for code documentation ;)

    class ConnectionFactory(object):
        def get_connection_pool(self, params:dict):
            # Given connection parameters in the `params` argument,
            # return new connection pool.
            # It should be overwritten if you want do something
            # before/after creating the connection pool, or return your
            # own connection pool.
            pass

        def get_connection(self, params:dict):
            # Given connection parameters in the `params` argument,
            # return a new connection.
            # It should be overwritten if you want to do something
            # before/after creating a new connection.
            # The default implementation uses `get_connection_pool`
            # to obtain a pool and create a new connection in the
            # newly obtained pool.
            pass

        def get_or_create_connection_pool(self, params:dict):
            # This is a high layer on top of `get_connection_pool` for
            # implementing a cache of created connection pools.
            # It should be overwritten if you want change the default
            # behavior.
            pass

        def make_connection_params(self, url:str) -> dict:
            # The responsibility of this method is to convert basic connection
            # parameters and other settings to fully connection pool ready
            # connection parameters.
            pass

        def connect(self, url:str):
            # This is really a public API and entry point for this
            # factory class. This encapsulates the main logic of creating
            # the previously mentioned `params` using `make_connection_params`
            # and creating a new connection using the `get_connection` method.
            pass

3.14 可扩展解析器
~~~~~~~~~~~~~~~~~

redis-py (django-redis 使用的 Redis 客户端) 支持的纯净 Python Redis
解析器可以满足大部分普通任务, 但如果你想要性能更好, 可以使用 ``hiredis``

hiredis 是一个用 C 写的 Redis 客户端, 并且他的解析器可以用在
django-redis 中:

::

    "OPTIONS": {
        "PARSER_CLASS": "redis.connection.HiredisParser",
    }

3.15 可扩展客户端
~~~~~~~~~~~~~~~~~

django\_redis
设计的非常灵活和可配置。它提供了可扩展的后端，拥有易扩展的特性.

3.15.1 默认客户端
^^^^^^^^^^^^^^^^^

我们已经说明了默认客户端几乎所有的特点, 但有一个例外:
默认客户端支持主从配置.

如果需要主从设置, 你需要更改 ``LOCATION`` 参数:

::

    "LOCATION": [
        "redis://127.0.0.1:6379/1",
        "redis://127.0.0.1:6378/1",
    ]

第一个字段代表 master 服务器, 第二个字段代表 slave 服务器.

**警告 主从设置没有在生产环境中经过大量测试**

3.15.2 分片客户端
^^^^^^^^^^^^^^^^^

此可扩展客户端实现了客户端分片, 它几乎继承了默认客户端的全部功能.
如果需要使用, 请将配置改成这样

::

    CACHES = {
        "default": {
            "BACKEND": "django_redis.cache.RedisCache",
            "LOCATION": [
                "redis://127.0.0.1:6379/1",
                "redis://127.0.0.1:6379/2",
            ],
            "OPTIONS": {
                "CLIENT_CLASS": "django_redis.client.ShardClient",
            }
        }
    }

**警告 分片客户端仍处于试验阶段, 请在生产环境中谨慎使用**

3.15.3 集群客户端
^^^^^^^^^^^^^^^^^

我们同时也在尝试解决惊群问题, 更多信息请阅读\ `Wikipedia`_

.. _Wikipedia: https://en.wikipedia.org/wiki/Thundering_herd_problem

和上文讲的一样, 客户端基本继承了默认客户端所有功能,
增加额外的方法以获取/设置键 (keys)

*设置举例*

::

    CACHES = {
        "default": {
            "BACKEND": "django_redis.cache.RedisCache",
            "LOCATION": "redis://127.0.0.1:6379/1",
            "OPTIONS": {
                "CLIENT_CLASS": "django_redis.client.HerdClient",
            }
        }
    }

一些其他的设置:

-  CACHE\_HERD\_TIMEOUT: 设置集群超时 (默认值为: 60s)

3.16 可扩展序列器
~~~~~~~~~~~~~~~~~

客户端在将数据发给服务器之前先会序列化数据. django-redis 默认使用 Python
pickle 序列化数据.

如果需要使用 json 序列化数据, 使用 JSONSerializer

*设置举例*

::

    CACHES = {
        "default": {
            "BACKEND": "django_redis.cache.RedisCache",
            "LOCATION": "redis://127.0.0.1:6379/1",
            "OPTIONS": {
                "CLIENT_CLASS": "django_redis.client.DefaultClient",
                "SERIALIZER": "django_redis.serializers.json.JSONSerializer",
            }
        }
    }

使用 MsgPack http://msgpack.org/ 进行序列化 (需要 msgpack-python 库支持)

*设置举例*

::

    CACHES = {
        "default": {
            "BACKEND": "django_redis.cache.RedisCache",
            "LOCATION": "redis://127.0.0.1:6379/1",
            "OPTIONS": {
                "CLIENT_CLASS": "django_redis.client.DefaultClient",
                "SERIALIZER": "django_redis.serializers.msgpack.MSGPackSerializer",
            }
        }
    }

3.17 可扩展 Redis 客户端
~~~~~~~~~~~~~~~~~~~~~~~~

django-redis 默认使用 ``redis.client.StrictClient`` 作为 Redis 客户端,
你可以使用其他客户端替代, 比如之前在讲测试时我们用 fakeredis
代替真实客户端.

使用 ``REDIS_CLIENT_CLASS in the CACHES`` 来配置你的客户端, 使用
``REDIS_CLIENT_KWARGS`` 提供配置客户端的参数 (可选).

*设置举例*

::

    CACHES = {
        "default": {
            "OPTIONS": {
                "REDIS_CLIENT_CLASS": "my.module.ClientClass",
                "REDIS_CLIENT_KWARGS": {"some_setting": True},
            }
        }
    }

4. 许可
-------

::

    Copyright (c) 2011-2015 Andrey Antukh <niwi@niwi.nz>
    Copyright (c) 2011 Sean Bleier

    All rights reserved.

    Redistribution and use in source and binary forms, with or without
    modification, are permitted provided that the following conditions
    are met:
    1. Redistributions of source code must retain the above copyright
       notice, this list of conditions and the following disclaimer.
    2. Redistributions in binary form must reproduce the above copyright
       notice, this list of conditions and the following disclaimer in the
       documentation and/or other materials provided with the distribution.
    3. The name of the author may not be used to endorse or promote products
       derived from this software without specific prior written permission.

    THIS SOFTWARE IS PROVIDED BY THE AUTHOR ``AS IS'' AND ANY EXPRESS OR
    IMPLIED WARRANTIES, INCLUDING, BUT NOT LIMITED TO, THE IMPLIED WARRANTIES
    OF MERCHANTABILITY AND FITNESS FOR A PARTICULAR PURPOSE ARE DISCLAIMED.
    IN NO EVENT SHALL THE AUTHOR BE LIABLE FOR ANY DIRECT, INDIRECT,
    INCIDENTAL, SPECIAL, EXEMPLARY, OR CONSEQUENTIAL DAMAGES (INCLUDING, BUT
    NOT LIMITED TO, PROCUREMENT OF SUBSTITUTE GOODS OR SERVICES; LOSS OF USE,
    DATA, OR PROFITS; OR BUSINESS INTERRUPTION) HOWEVER CAUSED AND ON ANY
    THEORY OF LIABILITY, WHETHER IN CONTRACT, STRICT LIABILITY, OR TORT
    (INCLUDING NEGLIGENCE OR OTHERWISE) ARISING IN ANY WAY OUT OF THE USE OF
    THIS SOFTWARE, EVEN IF ADVISED OF THE POSSIBILITY OF SUCH DAMAGE.