# openresty

openresty是一款基于nginx和lua脚本的高性能web服务器，里面集成了精良的lua库、第三方模块和大多数依赖库。可以方便的搭建超高并发，扩展性极强的动态web应用和动态网关。



### 安装openresty

参考官网

http://openresty.org/cn/linux-packages.html



### lua开发指南

参考：https://www.cnblogs.com/yanzi-meng/p/9450991.html



#### 指令

| 指令名称                                      | 说明                                                         |
| --------------------------------------------- | ------------------------------------------------------------ |
| lua_use_default_type                          | 是否使用default_type指令定义的Content-Type默认值             |
| lua_code_cache                                | content_by_lua_file文件是否cache                             |
| lua_regex_cache_max_entries                   |                                                              |
| lua_regex_match_limit                         |                                                              |
| **lua_package_path**                          | 用Lua写的lua外部库路径（.lua文件）                           |
| lua_package_cpath                             | 用C写的lua外部库路径（.so文件）                              |
| **init_by_lua**                               | master进程启动时挂载的lua代码                                |
| init_by_lua_file                              |                                                              |
| **init_worker_by_lua**                        | worker进程启动时挂载的lua代码，常用来执行一些定时器任务      |
| init_worker_by_lua_file                       |                                                              |
| set_by_lua                                    | 设置变量                                                     |
| set_by_lua_file                               |                                                              |
| **content_by_lua**                            | handler模块                                                  |
| **content_by_lua_file**                       |                                                              |
| rewrite_by_lua                                |                                                              |
| rewrite_by_lua_file                           |                                                              |
| **access_by_lua**                             | 需要执行lua代码块                                            |
| **access_by_lua_file**                        | 需要执行的lua文件                                            |
| header_filter_by_lua                          | header filter模块                                            |
| header_filter_by_lua_file                     |                                                              |
| body_filter_by_lua                            | body filter模块，ngx.arg[1]代表输入的chunk，ngx.arg[2]代表当前chunk是否为last |
| body_filter_by_lua_file                       |                                                              |
| log_by_lua                                    |                                                              |
| log_by_lua_file                               |                                                              |
| lua_need_request_body                         | 是否读请求体，跟ngx.req.read_body()函数作用类似              |
| lua_shared_dict                               | 创建全局共享的table（多个worker进程共享）                    |
| lua_socket_connect_timeout                    | TCP/unix 域socket对象connect方法的超时时间                   |
| lua_socket_send_timeout                       | TCP/unix 域socket对象send方法的超时时间                      |
| lua_socket_send_lowat                         | 设置cosocket send buffer的low water值                        |
| lua_socket_read_timeout                       | TCP/unix 域socket对象receive方法的超时时间                   |
| lua_socket_buffer_size                        | cosocket读buffer大小                                         |
| lua_socket_pool_size                          | cosocket连接池大小                                           |
| lua_socket_keepalive_timeout                  | cosocket长连接超时时间                                       |
| lua_socket_log_errors                         | 是否打开cosocket错误日志                                     |
| lua_ssl_ciphers                               |                                                              |
| lua_ssl_crl                                   |                                                              |
| lua_ssl_protocols                             |                                                              |
| lua_ssl_trusted_certificate                   |                                                              |
| lua_ssl_verify_depth                          |                                                              |
| lua_http10_buffering                          |                                                              |
| rewrite_by_lua_no_postpone                    |                                                              |
| lua_transform_underscores_in_response_headers |                                                              |
| lua_check_client_abort                        | 是否监视client提前关闭请求的事件，如果打开监视，会调用ngx.on_abort()注册的回调 |
| lua_max_pending_timers                        |                                                              |
| lua_max_running_timers                        |                                                              |



#### ngx变量

| table      | 说明                                                         |
| ---------- | ------------------------------------------------------------ |
| ngx.arg    | 指令参数，如跟在content_by_lua_file后面的参数                |
| ngx.var    | 变量，ngx.var.VARIABLE引用某个变量。请求相关的变量只能获取，不能修改和增加 |
| ngx.ctx    | 请求的lua上下文。如果要保存变量，需要建议放在ngx.ctx         |
| ngx.header | 响应头，ngx.header.HEADER引用某个头                          |
| ngx.status | 响应码                                                       |

 

#### ngx方法

| API                            | 说明                                                 |
| ------------------------------ | ---------------------------------------------------- |
| **ngx.log**                    | 输出到error.log                                      |
| print                          | 等价于 ngx.log(ngx.NOTICE, ...)                      |
| ngx.send_headers               | 发送响应头                                           |
| ngx.headers_sent               | 响应头是否已发送                                     |
| ngx.resp.get_headers           | 获取响应头                                           |
| ngx.timer.at                   | 注册定时器事件                                       |
| ngx.is_subrequest              | 当前请求是否是子请求                                 |
| **ngx.location.capture**       | 发布一个子请求                                       |
| **ngx.location.capture_multi** | 发布多个子请求                                       |
| ngx.exec                       |                                                      |
| ngx.redirect                   |                                                      |
| ngx.print                      | 输出响应                                             |
| ngx.say                        | 输出响应，自动添加'\n'                               |
| ngx.flush                      | 刷新响应                                             |
| ngx.exit                       | 结束请求                                             |
| ngx.eof                        |                                                      |
| **ngx.sleep**                  | 无阻塞的休眠（使用定时器实现）                       |
| ngx.get_phase                  |                                                      |
| ngx.on_abort                   | 注册client断开请求时的回调函数                       |
| ndk.set_var.DIRECTIVE          |                                                      |
| ngx.req.start_time             | 请求的开始时间                                       |
| ngx.req.http_version           | 请求的HTTP版本号                                     |
| ngx.req.raw_header             | 请求头（包括请求行）                                 |
| ngx.req.get_method             | 请求方法                                             |
| ngx.req.set_method             | 请求方法重载                                         |
| ngx.req.set_uri                | 请求URL重写                                          |
| ngx.req.set_uri_args           |                                                      |
| ngx.req.get_uri_args           | 获取请求参数                                         |
| ngx.req.get_post_args          | 获取请求表单                                         |
| ngx.req.get_headers            | 获取请求头                                           |
| ngx.req.set_header             |                                                      |
| ngx.req.clear_header           |                                                      |
| ngx.req.read_body              | 读取请求体                                           |
| ngx.req.discard_body           | 扔掉请求体                                           |
| ngx.req.get_body_data          |                                                      |
| ngx.req.get_body_file          |                                                      |
| ngx.req.set_body_data          |                                                      |
| ngx.req.set_body_file          |                                                      |
| ngx.req.init_body              |                                                      |
| ngx.req.append_body            |                                                      |
| ngx.req.finish_body            |                                                      |
| ngx.req.socket                 |                                                      |
| ngx.escape_uri                 | 字符串的url编码                                      |
| ngx.unescape_uri               | 字符串url解码                                        |
| ngx.encode_args                | 将table编码为一个参数字符串                          |
| ngx.decode_args                | 将参数字符串编码为一个table                          |
| ngx.encode_base64              | 字符串的base64编码                                   |
| ngx.decode_base64              | 字符串的base64解码                                   |
| ngx.crc32_short                | 字符串的crs32_short哈希                              |
| ngx.crc32_long                 | 字符串的crs32_long哈希                               |
| ngx.hmac_sha1                  | 字符串的hmac_sha1哈希                                |
| ngx.md5                        | 返回16进制MD5                                        |
| ngx.md5_bin                    | 返回2进制MD5                                         |
| ngx.sha1_bin                   | 返回2进制sha1哈希值                                  |
| ngx.quote_sql_str              | SQL语句转义                                          |
| ngx.today                      | 返回当前日期                                         |
| ngx.time                       | 返回UNIX时间戳                                       |
| ngx.now                        | 返回当前时间                                         |
| ngx.update_time                | 刷新时间后再返回                                     |
| ngx.localtime                  |                                                      |
| ngx.utctime                    |                                                      |
| ngx.cookie_time                | 返回的时间可用于cookie值                             |
| ngx.http_time                  | 返回的时间可用于HTTP头                               |
| ngx.parse_http_time            | 解析HTTP头的时间                                     |
| ngx.re.match                   |                                                      |
| ngx.re.find                    |                                                      |
| ngx.re.gmatch                  |                                                      |
| ngx.re.sub                     |                                                      |
| ngx.re.gsub                    |                                                      |
| ngx.shared.DICT                |                                                      |
| ngx.shared.DICT.get            |                                                      |
| ngx.shared.DICT.get_stale      |                                                      |
| ngx.shared.DICT.set            |                                                      |
| ngx.shared.DICT.safe_set       |                                                      |
| ngx.shared.DICT.add            |                                                      |
| ngx.shared.DICT.safe_add       |                                                      |
| ngx.shared.DICT.replace        |                                                      |
| ngx.shared.DICT.delete         |                                                      |
| ngx.shared.DICT.incr           |                                                      |
| ngx.shared.DICT.flush_all      |                                                      |
| ngx.shared.DICT.flush_expired  |                                                      |
| ngx.shared.DICT.get_keys       |                                                      |
| ngx.socket.udp                 |                                                      |
| udpsock:setpeername            |                                                      |
| udpsock:send                   |                                                      |
| udpsock:receive                |                                                      |
| udpsock:close                  |                                                      |
| udpsock:settimeout             |                                                      |
| ngx.socket.tcp                 |                                                      |
| tcpsock:connect                |                                                      |
| tcpsock:sslhandshake           |                                                      |
| tcpsock:send                   |                                                      |
| tcpsock:receive                |                                                      |
| tcpsock:receiveuntil           |                                                      |
| tcpsock:close                  |                                                      |
| tcpsock:settimeout             |                                                      |
| tcpsock:setoption              |                                                      |
| tcpsock:setkeepalive           |                                                      |
| tcpsock:getreusedtimes         |                                                      |
| ngx.socket.connect             |                                                      |
| ngx.thread.spawn               |                                                      |
| ngx.thread.wait                |                                                      |
| ngx.thread.kill                |                                                      |
| coroutine.create               |                                                      |
| coroutine.resume               |                                                      |
| coroutine.yield                |                                                      |
| coroutine.wrap                 |                                                      |
| coroutine.running              |                                                      |
| coroutine.status               |                                                      |
| ngx.config.debug               | 编译时是否有 --with-debug选项                        |
| ngx.config.prefix              | 编译时的 --prefix选项                                |
| ngx.config.nginx_version       | 返回nginx版本号                                      |
| ngx.config.nginx_configure     | 返回编译时 ./configure的命令行选项                   |
| ngx.config.ngx_lua_version     | 返回ngx_lua模块版本号                                |
| ngx.worker.exiting             | 当前worker进程是否正在关闭（如reload、shutdown期间） |
| ngx.worker.pid                 | 返回当前worker进程的pid                              |



#### 常量

| 常量                      | 说明                                                         |
| ------------------------- | ------------------------------------------------------------ |
| Core constants            | ngx.OK (0) ngx.ERROR (-1) ngx.AGAIN (-2) ngx.DONE (-4) ngx.DECLINED (-5) ngx.nil |
| HTTP method constants     | ngx.HTTP_GET ngx.HTTP_HEAD ngx.HTTP_PUT ngx.HTTP_POST ngx.HTTP_DELETE ngx.HTTP_OPTIONS  ngx.HTTP_MKCOL   ngx.HTTP_COPY    ngx.HTTP_MOVE    ngx.HTTP_PROPFIND  ngx.HTTP_PROPPATCH  ngx.HTTP_LOCK  ngx.HTTP_UNLOCK   ngx.HTTP_PATCH   ngx.HTTP_TRACE |
| HTTP status constants     | ngx.HTTP_OK (200) ngx.HTTP_CREATED (201) ngx.HTTP_SPECIAL_RESPONSE (300) ngx.HTTP_MOVED_PERMANENTLY (301) ngx.HTTP_MOVED_TEMPORARILY (302) ngx.HTTP_SEE_OTHER (303) ngx.HTTP_NOT_MODIFIED (304) ngx.HTTP_BAD_REQUEST (400) ngx.HTTP_UNAUTHORIZED (401) ngx.HTTP_FORBIDDEN (403) ngx.HTTP_NOT_FOUND (404) ngx.HTTP_NOT_ALLOWED (405) ngx.HTTP_GONE (410) ngx.HTTP_INTERNAL_SERVER_ERROR (500) ngx.HTTP_METHOD_NOT_IMPLEMENTED (501) ngx.HTTP_SERVICE_UNAVAILABLE (503) ngx.HTTP_GATEWAY_TIMEOUT (504) |
| Nginx log level constants | ngx.STDERR ngx.EMERG ngx.ALERT ngx.CRIT ngx.ERR ngx.WARN ngx.NOTICE ngx.INFO ngx.DEBUG |



### 使用lua脚本处理请求

创建lua/hello.lua

```lua
local _M = {}
  
function _M.greet(name)
    ngx.say("Hello ", name)
end

return _M
```



书写conf/nginx.conf配置

```nginx
http {
		# 引入lua模块的路径
    lua_package_path "$prefix/lua/?.lua;;";
	
  	# 执行的初始化，可用于预加载lua模块
    init_by_lua_block {
        require "hello"
    }

    server {
        listen       80;
        server_name  localhost;

    		# 通过content_by_lua_block指令引入lua脚本处理请求·
        location /hello/ {
            default_type text/html;
            content_by_lua_block {
                ngx.say("<p>Hello World!</p>");
            }
        }

        location /hello2/ {
            default_type text/html;
            content_by_lua_block {
                local hello = require "hello"
                hello.greet("hwt")
            }
        }

```

测试

![image-20220307085126964](/开源框架/nginx/.assert/openresty/image-20220307085126964.png)



### 处理流程

https://www.aikaiyuan.com/12438.html

https://blog.51cto.com/xikder/2331649

1. init_by_lua       http
2. set_by_lua       server, server if, location, location if
3. rewrite_by_lua     http, server, location, location if
4. access_by_lua      http, server, location, location if
5. content_by_lua     location, location if
6. header_filter_by_lua  http, server, location, location if
7. body_filter_by_lua   http, server, location, location if
8. log_by_lua       http, server, location, location if

- set_by_lua: 流程分支处理判断变量初始化
- rewrite_by_lua: 转发、重定向、缓存等功能(例如特定请求代理到外网)
- access_by_lua: IP准入、接口权限等情况集中处理(例如配合iptable完成简单防火墙)
- content_by_lua: 内容生成
- header_filter_by_lua: 应答HTTP过滤处理(例如添加头部信息)
- body_filter_by_lua: 应答BODY过滤处理(例如完成应答内容统一成大写)
- log_by_lua: 会话完成后本地异步完成日志记录(日志可以记录在本地，还可以同步到其他机器)



#### init_by_lua、init_by_lua_file

这些代码块会在任何woker进程创建之前运行。通常用于预加载lua模块

```nginx
init_by_lua 'cjson = require "cjson"';

server {
    location = /api {
        content_by_lua '
            ngx.say(cjson.encode({dog = 5, cat = 6}))
        '
    }
}
```



也可以初始化shared变量。ngx.shared是全局共享的参数，并且在reload时，不会被清除，如果不想重复初始化，可以加入校验。

```nginx
lua_shared_dict dogs 1m;
init_by_lua '
    local dogs = ngx.shared.dogs;
    dogs:set("Tom", 50)
'
server {
    location = /api {
        content_by_lua '
            local dogs = ngx.shared.dogs;
            ngx.say(dogs:get("Tom"))
        '
    }
}
```





#### init_worker_by_lua init_worker_by_lua_file

在创建worker进程时执行。可以创建定时器做周期健康检查

```nginx
init_worker_by_lua:
    local delay = 3  -- in seconds
    local new_timer = ngx.timer.at
    local log = ngx.log
    local ERR = ngx.ERR
    local check
    check = function(premature)
        if not premature then
            -- do the health check other routine work
            local ok, err = new_timer(delay, check)
            if not ok then
                log(ERR, "failed to create timer: ", err)
                return
            end
        end
    end
    local ok, err = new_timer(delay, check)
    if not ok then
        log(ERR, "failed to create timer: ", err)
    end
```



#### set_by_lua, set_by_lua_file

设置一个变量，功能与普通set指令类似。set_by_lua是阻塞操作，并且不支持非阻塞IO，因此尽量执行短且快的操作。

```nginx
location /foo {
    set $diff '';
    set_by_lua $num '
        local a = 32
        local b = 56
        ngx.var.diff = a - b; --写入$diff中
        return a + b;  --返回到$sum中
    '
    echo "sum = $sum, diff = $diff";
}
```



#### rewrite_by_lua rewrite_by_lua_file

可以在处理请求前修改节点请求，与rewrite功能类似。



注意：

1. rewrite_by_lua_*命令依然不能修改ngx.var中请求的参数，例如uri，args。但是可以通过ngx.redirect和ngx.exec实现重定向功能
2. 可以使用ngx.redirect和ngx.exec实现重定向



下面是错误的示范

```nginx
location /proxylua/ {
  rewrite_by_lua_block {
    ngx.var.uri = ngx.var.uri.."/postfix";
  }

  content_by_lua_block {
    ngx.say("uri:"..ngx.var.uri);
  }
}
```





#### access_by_lua access_by_lua_file

可以控制请求的访问，类似于access和deny只能的功能



#### balancer_by_lua_block

在upstream中使用，用于选择后端节点。

https://github.com/openresty/lua-nginx-module#balancer_by_lua_block

1. 当balancer_by_lua_block出现时，server将会被忽略
2. 使用ngx.balanacer.set_current_peer(ip, port)设置本次选择的后端节点。详细用法：https://github.com/openresty/lua-resty-core/blob/master/lib/ngx/balancer.md

```nginx
stream {
    upstream backend {
        server 0.0.0.1:1234;   # just an invalid address as a place holder

        balancer_by_lua_block {
            local balancer = require "ngx.balancer"

            -- well, usually we calculate the peer's host and port
            -- according to some balancing policies instead of using
            -- hard-coded values like below
            local host = "127.0.0.2"
            local port = 8080

            local ok, err = balancer.set_current_peer(host, port)
            if not ok then
                ngx.log(ngx.ERR, "failed to set the current peer: ", err)
                return ngx.exit(ngx.ERROR)
            end
        }
    }

    server {
        # this is the real entry point
        listen 10000;

        # make use of the upstream named "backend" defined above:
        proxy_pass backend;
    }

    server {
        # this server is just for mocking up a backend peer here...
        listen 127.0.0.2:8080;

        echo "this is the fake backend peer...";
    }
}
```



#### header_filter_by_lua_block

用于在返回响应前，修改请求头



```nginx
location / {
    header_filter_by_lua_block {
       ngx.header.test = "nginx-lua";
    }
    echo 'ok';
}
```



#### body_filter_by_lua_block

用于在返回响应前，修改请求体









### 读取请求体

client_body_buffer_size 设置请求体的buffer大小，如果请求体大于buffer大小，将会保存在临时文件中

**client_body_temp** 设置请求体临时文件保存的路径

client_max_body_size设置请求体最大大小，超过HTTP协议会报错 413 Request Entity Too Large

**client_body_in_file_only** on | off 是否将请求体总是保存在临时文件中

```lua
local req = ngx.req;
-- 先执行read_body会触发读取请求体
req.read_body();

local body = req.get_body_data();
-- 请求体在buffer中
if ( body ~= nil ) then
   ngx.say("body: "..body)
   return;
end


local body_file = req.get_body_file();
-- 请求体在文件中，需要从文件中读取请求体
if ( body_file ~= nil ) then
    ngx.say("body_file: "..body_file)
    return;
end

ngx.say("no body");

```

![image-20220309233211902](/开源框架/nginx/.assert/openresty/image-20220309233211902.png)

### 转发请求

https://blog.csdn.net/u010074988/article/details/90665529



#### ngx.exec

nginx内部重定向，然后将响应返回给客户端

https://github.com/openresty/lua-nginx-module#ngxexec

```nginx
location /proxylua0/ {
  rewrite_by_lua_block {
    ngx.exec("/backend/aa/bb");
  }
}
location /backend/ {
  content_by_lua_block {
    ngx.say("backend: "..ngx.var.uri);
  }
}
```



```
root@f8c1cf26c178:/# curl -i http:/192.168.0.6/proxylua0/aa/cc
HTTP/1.1 200 OK
Server: openresty/1.19.9.1
Date: Sat, 12 Mar 2022 02:12:18 GMT
Content-Type: application/octet-stream
Transfer-Encoding: chunked
Connection: keep-alive

backend: /backend/aa/bb
```



#### ngx.redirect

向客户端返回302响应

https://github.com/openresty/lua-nginx-module#ngxredirect

```nginx
location /proxylua1/ {
  rewrite_by_lua_block {
    ngx.redirect("https://www.baidu.com"..ngx.var.uri);
  }
}


location /proxylua2/ {
  rewrite_by_lua_block {
    ngx.redirect("/backend/aa/bb");
  }
}
```



```
root@f8c1cf26c178:/# curl -i http:/192.168.0.6/proxylua1/aa/cc
HTTP/1.1 302 Moved Temporarily
Server: openresty/1.19.9.1
Date: Sat, 12 Mar 2022 02:12:38 GMT
Content-Type: text/html
Content-Length: 151
Connection: keep-alive
Location: https://www.baidu.com/proxylua1/aa/cc

<html>
<head><title>302 Found</title></head>
<body>
<center><h1>302 Found</h1></center>
<hr><center>openresty/1.19.9.1</center>
</body>
</html>


root@f8c1cf26c178:/# curl -i http:/192.168.0.6/proxylua2/aa/cc
HTTP/1.1 302 Moved Temporarily
Server: openresty/1.19.9.1
Date: Sat, 12 Mar 2022 02:12:57 GMT
Content-Type: text/html
Content-Length: 151
Connection: keep-alive
Location: /backend/aa/bb

<html>
<head><title>302 Found</title></head>
<body>
<center><h1>302 Found</h1></center>
<hr><center>openresty/1.19.9.1</center>
</body>
</html>
```



#### ngx.location.capture

转发请求，然后返回响应（不是返回响应给客户端）

https://github.com/openresty/lua-nginx-module#ngxlocationcapture



```nginx
location /proxylua3/ {
    content_by_lua_block{
        local res = ngx.location.capture("/backend/aa/cc")
        ngx.say(res.status)
        ngx.say(res.body)
    }
}


location /backend/ {
  content_by_lua_block {
    ngx.say("backend: "..ngx.var.uri);
  }
}
```



```
root@f8c1cf26c178:/# curl -i http:/192.168.0.6/proxylua3/aa/cc
HTTP/1.1 200 OK
Server: openresty/1.19.9.1
Date: Sat, 12 Mar 2022 02:13:41 GMT
Content-Type: application/octet-stream
Transfer-Encoding: chunked
Connection: keep-alive

200
backend: /backend/aa/cc
```







1. ngx.redirect: 支持外部重定向
2. ngx.location.capture,ngx.location.capture_multi：内部转发，方法会返回响应





```
docker run -d -it --name openresty --network mynet --ip 192.168.0.6 -v /Users/hewutao/docker/extfile/nginx/openresty:/usr/share/other/ -p 8081:80   ubuntu bash
```



