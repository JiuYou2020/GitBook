# Nginx动态缓存与过滤

软件负载均衡位于系统的入口，流入分布式系统的请求都会经过这里，换句话说，相对整个系统而言，软件负载均衡是离客户端更近的地方，所以可以将一些不经常变化的数据放到这里作为缓存，降低用户请求访问应用服务器的频率。例如一些用户的基本信息，修改频率就不是很高，并且使用比较频繁，这些信息就可以放到缓存服务器 `Redis` 中，当用户请求这部分信息时通过软件负载均衡器直接返回给用户，这样就省去了调用应用服务器的环节，从而能够更快地响应用户。

如图 3-8 所示，从接入层来的用户请求，通过 `Nginx` 代理层，按照如下几个步骤对数据进行访问。

1. 用户请求首先通过 `F5`访问 `Nginx`，如果请求需要获取的数据在 `Nginx` 本地缓存中有，就直接返回给用户。
2. 如果没有命中 `Nginx` 缓存，则可以通过 `Nginx` 直接调用缓存服务器获取数据并返回。
3. 如果缓存服务器中也不存在用户需要的数据，则需要回源到上游应用服务器并返回数据。

<figure><img src="https://files.gitbook.com/v0/b/gitbook-x-prod.appspot.com/o/spaces%2FddwIPrCChH9wzK5lX95z%2Fuploads%2FhwBHH5jB54QgxLW7iZ1C%2Fimage.png?alt=media&#x26;token=c3e7e6ad-9c94-49f9-94a5-b055bac63037" alt=""><figcaption></figcaption></figure>

使用上图第 (3) 步的方案，无非可以减少调用步骤，因为应用服务有可能存在其他的调用、数据转换、网络传输成本，同时还包含一些业务逻辑和访问数据库操作，影响响应时间。从代理层调用的缓存数据有如下特点：

1. 这类数据变化不是很频繁，例如用户信息；
2. 业务逻辑简单，例如判断用户是否有资格参加秒杀活动、判断商品是否在秒杀活动范围内；
3. 此类缓存数据需要专门的进程对其进行刷新，如果无法命中数据还是需要请求应用服务器。

实现这种方案一般需要加入少许的代码脚本。以`Nginx`为例，需要加入 `Lua` 脚本协助实现，针对 `OpenResty Lua` 的具体开发，这里不做展开。如图 3-9 所示的例子中描述的是从客户端传输 `userId`（用户 ID）到系统中查询用户信息，这些`userId` 已经事先放到了`Redis`缓存中，表示这部分用户可以参与秒杀活动。在`Nginx` 中对比用户请求的 `userId` 和缓存中存放的 `userId` 是否一致，如果一致就让其进行后续的访问，否则拒绝请求。这只是一个例子，在实际操作中这个 `userId` 可以是商品 ID，相应的操作是判断该商品是否为秒杀商品；也可以是用户访问的`IP` 地址，通过看该地址是否在黑名单上来判断用户请求是否为恶意请求。

<figure><img src="https://files.gitbook.com/v0/b/gitbook-x-prod.appspot.com/o/spaces%2FddwIPrCChH9wzK5lX95z%2Fuploads%2FtpKneDQULveZnrWJV9C1%2Fimage.png?alt=media&#x26;token=eb5a2eff-45f1-4c87-9464-2bea7d819f0f" alt=""><figcaption></figcaption></figure>

具体步骤如下。

1. 用户请求秒杀服务时，会附带 `userId` 信息。
2. 系统需要确认用户的身份和权限。`Redis` 事先缓存了用户的鉴权信息，于是先通过`Nginx` 上的 `Lua` 脚本查询`Redis` 缓存，如果能够获得`userId` 的相关信息，就直接返回用户拥有的权限，进行后面的操作。
3. 如果从缓存中获取不到`userId` 的相关信息，就去请求上游的用户鉴权服务，之后再进行后面的业务流程。

现在用代码帮助大家理解上面的例子，最重要的是打开思路。上面提到了 `Lua` 脚本，这是一种轻量小巧的脚本语言，由标准 C 语言编写并以源代码形式开放，嵌入到 Nginx 中为其提供灵活的扩展和定制功能，负责调用 `Redis` 缓存和上游服务器的业务。接下来看一下 `Lua` 的具体实现和 `Nginx` 的配置。

先建立`Lua` 脚本，其包括如下功能。

1. 创建 `Redis` 连接，以 `userId` 为键读取 `Redis` 缓存中的信息，传入`userId`看其是否存在于 `Redis`中。在 `get_redis` 函数中输入参数 `userId`，返回值 `response` 若不为空，则说明`userId` 在 `Redis` 中。
2. 关闭 `Redis` 连接。在 `close_redis` 函数中输入参数 redis 即可。
3. 连接上游的应用服务器。在 `get_http` 函数中输入参数 `userId`，获取对应用户的访问权限。

具体的实现代码如下所示：

```lua
local redis = require("resty.redis")  ①
local cjson = require("cjson")  ②
local ngx_var = ngx.var  ③
local function get_redis(userId)  ④
local red = redis:new()  ⑤
red:set_timeout(2000)  ⑥
local ip = "192.168.1.1"  ⑦
local port = 8888  ⑧
local ok, err = red:connect(ip, port)  ⑨
local response, err = red:get(userId)  ⑩
close_redis(red)  ⑪
return response  ⑫
end
local userId = ngx_var.userId  ⑬
local content = get_redis(userId)  ⑭
if not content then
    content = get_http(userId)  ⑮
end
    return ngx_exit(404)
end
```

调用上述代码时，先由 `Nginx` 获取用户的 `URL` 请求，截取 `userId` 参数传给 `Lua` 脚本。然后`Lua`脚本建立与 `Redis` 的连接，查询`userId`是否在`Redis`缓存中。如果存在就直接返回结果，并继续后面的操作；如果不存在，则调用`get_http`函数请求上游的用户鉴权服务，最后调用`close_redis`函数关闭 Redis 的连接。

这里对上述代码做简单介绍。

1. 引用 `Lua` 的 `redis` 模块。
2. 引用`Lua` 的`cjson`模块，用来实现 `Lua` 值与 `Json` 值之间的相互转换。
3. 定义 `ngx_var`，获取 `Nginx` 传入的请求参数。
4. 在 `get_redis` 函数中输入参数 `userId`，返回值为 `response`，其不为空说明`userId` 对应的用户在`Redis` 缓存中。
5. 新建 `redis` 对象。
6. 设置超时时间。
7. 设置 `Redis` 的`IP` 地址。
8. 设置 `Redis` 的端口号。
9. 打开 `Redis` 连接。
10. 输入`userId`获取对应的值，并存放到`response`和`err`变量中。在`response`为空的情况下，可以返回 `null`，并且关闭 `Redis` 连接，此处代码并未详细给出。
11. 调用完毕`redis` 对象后关闭 `Redis` 连接。
12. 返回 `response`。
13. 程序开始执行，首先从请求的参数变量中获取 `userId`，放到`Lua` 本地变量中。
14. 调用 `get_redis` 函数，传入 `userId`，把返回的结果放到 `content` 中。
15. 如果 `Redis` 缓存中不存在 `userId`，就调用 `get_http` 函数请求上游的用户鉴权服务。

以上 `Lua` 脚本主要是通过 `get_redis` 函数传入`userId`，再从`Redis` 缓存中获取信息。将此脚本保存在`/usr/checkuserid.lua` 下面。现在思考一个问题，这个`Lua`脚本是如何与`Nginx`合作的。假设用户通过一个 URL（[http://XXX.XXX.XXX.XXX/userid/123](http://xxx.xxx.xxx.xxx/userid/123)）访问应用系统，其中传入的 `userid 为123`。Nginx 配置代码如下所示，通过 location 配置的正则表达式`location ~^/userid(\d+)$`实现与`Lua`命令 `content_by_lua_file` 的绑定，此命令后面的参数是`Lua`脚本存放的地址。这样传入 `get_redis` 函数的 `userId` 参数就是 123，于是`Lua`脚本中的 `get_redis(userId)` 就会执行之后的查找操作了。

```lua
location ~ ^/userid(\d+)$ {
    default_type 'text/html';
    charset utf-8;
    lua_code_cache on;
        set $userId $1;
    content_by_lua_file /usr/checkuserid.lua;
}
```

动态缓存方式除了缓存用户信息，还能起到过滤功效，可以过滤一些不满足条件的用户。注意，这里提到的用户信息的过滤和缓存只是一个例子。主要想表达的意思是可以将一些变化不频繁的数据提到代理层来缓存，提高响应效率，同时可以根据风控系统返回的信息，过滤疑似机器人的代码或者恶意请求，例如从固定`IP`发送来的、频率过高的请求。特别在秒杀场景中代理层可以识别来自秒杀系统的请求，如果请求中带有秒杀系统的参数，就要把此请求路由至秒杀系统的服务器集群，这样才能将其和正常的业务请求分割开。
