## nginx使用





## nginx常用命令

```
nginx -h         # help
nginx -s reload  # 向主进程发送信号，重新加载配置文件，热重启
nginx -s reopen	 # 重启 Nginx
nginx -s stop    # 快速关闭
nginx -s quit    # 等待工作进程处理完成后关闭
nginx -T         # 查看当前 Nginx 最终的配置
nginx -t -c <配置路径>    # 检查配置是否有问题，如果已经在配置目录，则不需要-c
```





## nginx 配置文件结构

```
main        # 全局配置，对全局生效
├── events  # 配置影响 Nginx 服务器或与用户的网络连接
├── http    # 配置代理，缓存，日志定义等绝大多数功能和第三方模块的配置
│   ├── upstream # 配置后端服务器具体地址，负载均衡配置不可或缺的部分
│   ├── server   # 配置虚拟主机的相关参数，一个 http 块中可以有多个 server 块
│   ├── server
│   │   ├── location  # server 块可以包含多个 location 块，location 指令用于匹配 uri
│   │   ├── location
│   │   └── ...
│   └── ...
└── ...

```



## 通用配置



### 调试相关配置

#### daemon

是否以守护进程的方式运行nginx，默认on。关闭daemon，可以方便使用gdb调试Nginx进程

```nginx
daemon on|off
```



#### master_process

是否使用master/worker方式运行。默认开启，如果关闭，nginx将不会fork出worker进程，而是直接使用master进程处理请求。

```nginx
master_process on|off
```



#### error_log

配置日志打印

```nginx
Syntax:	error_log file [level];
Default:	
error_log logs/error.log error;
Context:	main, http, mail, stream, server, location
```



打印的目标是有以下几种

1. file。将日志打印到指定文件
2. stderr。打印到标准error输出
3. `syslog:config`。使用syslog打印。参考http://nginx.org/en/docs/syslog.html
4. `memory:size`。打印到内存中，通常仅在调试中使用



级别有7种：debug，info，warn，error，crit，alert，emerg



### 运行时一些配置

#### env

定义环境变量

```nginx
env VAR|VAR=VALUE
```



#### include 

嵌入其他配置文件。路径可以是相对路径和绝对路径，如果是相对路径，相对于Nginx配置文件目录。文件名可以使用通配符*。

```nginx
include /path/file;
include mime.types;
include vhost/*.conf;
```



#### pid

保存pid文件的路径，默认为logs/nginx.pid

```nginx
pid /path/file
```



#### user

指定nginx运行时，worker进程运行所属的用户和用户组

```nginx
user username [usergroup]
user nobody nobody
```





### 性能相关的参数

#### work_processors

指定工作进程个数。推荐根据cpu个数指定进程个数，auto表示会自动配置

```
Syntax:	worker_processes number | auto;
Default:	
worker_processes 1;
Context:	main
```





#### worker_priority

指定work进程优先级，范围在-20~20。值越小，优先级越高。不建议将优先级设置成小于-5

```
Syntax:	worker_priority number;
Default:	
worker_priority 0;
Context:	main
```



#### worker_cpu_affinity

指定work进程绑定的cpu。所有cpu使用二进制表示。如果机器有4个cpu，使用4位bit表示所有cpu。0001表示第0个cpu，0101表示第0和第2个cpu



```nginx
# 四个工作进程，绑定到4个不同cpu上
worker_processes    4;
worker_cpu_affinity 0001 0010 0100 1000;

# 第一个进程绑定到0，2号cpu，第二个进程绑定到1，3号cpu
worker_processes    2;
worker_cpu_affinity 0101 1010;

# 自动绑定cpu
worker_processes auto;
worker_cpu_affinity auto;

# 限制自动绑定的cpu，只能绑定0，2，4，6号cpu
worker_cpu_affinity auto 01010101;
```



#### ssl_engine

如果服务器上有ssl加速硬件，可以进行配置加快SSL协议处理速度。使用`openssl engine -t`检查是否有硬件加速设备

```nginx
ssl_engine device
```

#### 

### 事件类配置



#### accept_mutex

accept_mutex是nginx负载均衡锁。开启锁后，woker进程将会轮流、序列化的和客户端建立连接。可以保证woker进程处理的连接数比较接近，但是处理tcp连接的速度会下降。accept锁默认打开，不建议关闭。

```nginx
accept_mutex on|off
```



#### accept_mutex_delay

mutex锁不会阻塞，不管有没有获取到锁，都会立即返回。accept_mutex_delay用于指定在进程没有获取到锁时，应该等待多长时间再尝试获取锁。默认等待500ms



```nginx
accept_mutex_delay Nms
```



#### multi_accept

当事件模型通知有新连接时，是否尽可能的调度客户端发起的所有TCP请求连接。默认为off

```nginx
multi_accept on|off
```



#### use

使用的事件模型。默认nginx会自动选择最合适的模型

```nginx
use [ kqueue|rtsig|epoll|/dev/poll|select|poll|eventport ]
```

#### worker_connections

指定工作进程同时处理的进程数，默认512。这个连接包含客户端和后端服务器的连接。

```
Syntax:	worker_connections number;
Default:	
worker_connections 512;
Context:	events
```









#### event

配置一些连接相关配置





## Server配置



```nginx
upstream http_server { # 配置多台服务器，可以配置负载均衡算法
	server 192.168.0.2:4000;
	server 192.168.0.4:4000;
}

server {
	listen       80;       # 配置监听的端口
	server_name  localhost;    # 配置的域名

	location ^~ /api/ {
    rewrite ^/api/(.*)$ /$1 break;
    proxy_pass http://http_server;
  }

  location / {
    root   /usr/share/nginx/html;  # 静态文件根目录
    index  index.html index.htm;   # 默认首页文件
    deny 172.168.22.11;   # 禁止访问的ip地址，可以为all
    allow 172.168.33.44； # 允许访问的ip地址，可以为all
  }

  error_page 500 502 503 504 /50x.html;  # 默认50x对应的访问页面
  error_page  404              /404.html;
}
```



#### listen

指定nginx监听的ip和端口。

```
Syntax:	listen address[:port] [default_server] [ssl] [http2 | spdy] [proxy_protocol] [setfib=number] [fastopen=number] [backlog=number] [rcvbuf=size] [sndbuf=size] [accept_filter=filter] [deferred] [bind] [ipv6only=on|off] [reuseport] [so_keepalive=on|off|[keepidle]:[keepintvl]:[keepcnt]];
listen port [default_server] [ssl] [http2 | spdy] [proxy_protocol] [setfib=number] [fastopen=number] [backlog=number] [rcvbuf=size] [sndbuf=size] [accept_filter=filter] [deferred] [bind] [ipv6only=on|off] [reuseport] [so_keepalive=on|off|[keepidle]:[keepintvl]:[keepcnt]];
listen unix:path [default_server] [ssl] [http2 | spdy] [proxy_protocol] [backlog=number] [rcvbuf=size] [sndbuf=size] [accept_filter=filter] [deferred] [bind] [so_keepalive=on|off|[keepidle]:[keepintvl]:[keepcnt]];
Default:	
listen *:80 | *:8000;
Context:	server
```



```nginx
listen 127.0.0.1:8000
listen 443 default_server ssl
```



| 参数                    | 说明                                                         |
| ----------------------- | ------------------------------------------------------------ |
| default，default_server | 表示默认server块，当请求没有匹配到其他的主机域名时，将会使用默认server块处理。如果没有server声明为default，将会把第一个server块，作为默认server。 |
| backlog=num             | 表示完成3次握手的TCP队列长度。如果队列满了，其他尝试的tcp连接会失败 |
| rcvbuf=size             | 监听sockt句柄SO_RCVBUF的值                                   |
| sndbuf=size             | 监听sockt句柄SO_SNDBUF的值                                   |
| deferred                | 只有当接收到用户数据时，采用唤醒woker进程处理请求，可以减轻woker进程负担。只有确定有此业务场景才建议开启 |
| bind                    | 只有对一个端口监听多个地址时，才会生效。~~当有一个listen写法为listen 80，表示所有的ip的80端口被该监听，也就是0.0.0.0:80。如果再配置listen 127.0.0.1:80时，其实并不会bind 127.0.0.1:80，而是会复用之前的0.0.0.0:80的bind。如果指定bind，将会单独bind 127.0.0.1:80。~~ |
| ssl                     | 在当前监听的端口上建立的连接，必须基于SSL协议                |
|                         |                                                              |
|                         |                                                              |



#### server_name

表示处理的域名

```
server_name name [name ...]
```

域名的匹配优先级如下

1. 完全匹配，`www.testweb.com`
2. 通配符再前面，`*.testweb.com`
3. 通配符再后面，`www.testweb.*`
4. 正在表达式，`~^\.testweb\.com$`

如果上面四种都没有匹配

1. 优先选择在listen中配置了default的server块处理
2. 选择匹配listen端口的第一个server块











## 全局变量

配置文件中可以引用一些全局变量

| 变量              | 说明                                                         |
| ----------------- | ------------------------------------------------------------ |
| `$host`           | 请求中的Host头部                                             |
| `$remote_addr`    | 远程机器的ip地址                                             |
| `$remote_port`    | 远程机器的端口                                               |
| `$request_method` | 请求的方式                                                   |
| `$args`           | Url中参数                                                    |
| `$arg_*name*`     | Url请求中的参数name                                          |
| `$http_*name*`    | 名称为name的header，转化方式为小写，然后将`-`换成`_`，例如Content-Type 为 `$http_content_type` |
| `$https`          | 如果开启了ssl，将会返回on，其他情况返回空字符串              |
|                   |                                                              |





Embedded Variables

The `ngx_http_core_module` module supports embedded variables with names matching the Apache Server variables. First of all, these are variables representing client request header fields, such as `$http_user_agent`, `$http_cookie`, and so on. Also there are other variables:

- `$arg_*name*`

  argument `*name*` in the request line

- `$args`

  arguments in the request line

- `$binary_remote_addr`

  client address in a binary form, value’s length is always 4 bytes for IPv4 addresses or 16 bytes for IPv6 addresses

- `$body_bytes_sent`

  number of bytes sent to a client, not counting the response header; this variable is compatible with the “`%B`” parameter of the `mod_log_config` Apache module

- `$bytes_sent`

  number of bytes sent to a client (1.3.8, 1.2.5)

- `$connection`

  connection serial number (1.3.8, 1.2.5)

- `$connection_requests`

  current number of requests made through a connection (1.3.8, 1.2.5)

- `$connection_time`

  connection time in seconds with a milliseconds resolution (1.19.10)

- `$content_length`

  “Content-Length” request header field

- `$content_type`

  “Content-Type” request header field

- `$cookie_*name*`

  the `*name*` cookie

- `$document_root`

  [root](http://nginx.org/en/docs/http/ngx_http_core_module.html#root) or [alias](http://nginx.org/en/docs/http/ngx_http_core_module.html#alias) directive’s value for the current request

- `$document_uri`

  same as `$uri`

- `$host`

  in this order of precedence: host name from the request line, or host name from the “Host” request header field, or the server name matching a request

- `$hostname`

  host name

- `$http_``*name*`

  arbitrary request header field; the last part of a variable name is the field name converted to lower case with dashes replaced by underscores

- `$https`

  “`on`” if connection operates in SSL mode, or an empty string otherwise

- `$is_args`

  “`?`” if a request line has arguments, or an empty string otherwise

- `$limit_rate`

  setting this variable enables response rate limiting; see [limit_rate](http://nginx.org/en/docs/http/ngx_http_core_module.html#limit_rate)

- `$msec`

  current time in seconds with the milliseconds resolution (1.3.9, 1.2.6)

- `$nginx_version`

  nginx version

- `$pid`

  PID of the worker process

- `$pipe`

  “`p`” if request was pipelined, “`.`” otherwise (1.3.12, 1.2.7)

- `$proxy_protocol_addr`

  client address from the PROXY protocol header (1.5.12)The PROXY protocol must be previously enabled by setting the `proxy_protocol` parameter in the [listen](http://nginx.org/en/docs/http/ngx_http_core_module.html#listen) directive.

- `$proxy_protocol_port`

  client port from the PROXY protocol header (1.11.0)The PROXY protocol must be previously enabled by setting the `proxy_protocol` parameter in the [listen](http://nginx.org/en/docs/http/ngx_http_core_module.html#listen) directive.

- `$proxy_protocol_server_addr`

  server address from the PROXY protocol header (1.17.6)The PROXY protocol must be previously enabled by setting the `proxy_protocol` parameter in the [listen](http://nginx.org/en/docs/http/ngx_http_core_module.html#listen) directive.

- `$proxy_protocol_server_port`

  server port from the PROXY protocol header (1.17.6)The PROXY protocol must be previously enabled by setting the `proxy_protocol` parameter in the [listen](http://nginx.org/en/docs/http/ngx_http_core_module.html#listen) directive.

- `$query_string`

  same as `$args`

- `$realpath_root`

  an absolute pathname corresponding to the [root](http://nginx.org/en/docs/http/ngx_http_core_module.html#root) or [alias](http://nginx.org/en/docs/http/ngx_http_core_module.html#alias) directive’s value for the current request, with all symbolic links resolved to real paths

- `$remote_addr`

  client address

- `$remote_port`

  client port

- `$remote_user`

  user name supplied with the Basic authentication

- `$request`

  full original request line

- `$request_body`

  request bodyThe variable’s value is made available in locations processed by the [proxy_pass](http://nginx.org/en/docs/http/ngx_http_proxy_module.html#proxy_pass), [fastcgi_pass](http://nginx.org/en/docs/http/ngx_http_fastcgi_module.html#fastcgi_pass), [uwsgi_pass](http://nginx.org/en/docs/http/ngx_http_uwsgi_module.html#uwsgi_pass), and [scgi_pass](http://nginx.org/en/docs/http/ngx_http_scgi_module.html#scgi_pass) directives when the request body was read to a [memory buffer](http://nginx.org/en/docs/http/ngx_http_core_module.html#client_body_buffer_size).

- `$request_body_file`

  name of a temporary file with the request bodyAt the end of processing, the file needs to be removed. To always write the request body to a file, [client_body_in_file_only](http://nginx.org/en/docs/http/ngx_http_core_module.html#client_body_in_file_only) needs to be enabled. When the name of a temporary file is passed in a proxied request or in a request to a FastCGI/uwsgi/SCGI server, passing the request body should be disabled by the [proxy_pass_request_body off](http://nginx.org/en/docs/http/ngx_http_proxy_module.html#proxy_pass_request_body), [fastcgi_pass_request_body off](http://nginx.org/en/docs/http/ngx_http_fastcgi_module.html#fastcgi_pass_request_body), [uwsgi_pass_request_body off](http://nginx.org/en/docs/http/ngx_http_uwsgi_module.html#uwsgi_pass_request_body), or [scgi_pass_request_body off](http://nginx.org/en/docs/http/ngx_http_scgi_module.html#scgi_pass_request_body) directives, respectively.

- `$request_completion`

  “`OK`” if a request has completed, or an empty string otherwise

- `$request_filename`

  file path for the current request, based on the [root](http://nginx.org/en/docs/http/ngx_http_core_module.html#root) or [alias](http://nginx.org/en/docs/http/ngx_http_core_module.html#alias) directives, and the request URI

- `$request_id`

  unique request identifier generated from 16 random bytes, in hexadecimal (1.11.0)

- `$request_length`

  request length (including request line, header, and request body) (1.3.12, 1.2.7)

- `$request_method`

  request method, usually “`GET`” or “`POST`”

- `$request_time`

  request processing time in seconds with a milliseconds resolution (1.3.9, 1.2.6); time elapsed since the first bytes were read from the client

- `$request_uri`

  full original request URI (with arguments)

- `$scheme`

  request scheme, “`http`” or “`https`”

- `$sent_http_``*name*`

  arbitrary response header field; the last part of a variable name is the field name converted to lower case with dashes replaced by underscores

- `$sent_trailer_``*name*`

  arbitrary field sent at the end of the response (1.13.2); the last part of a variable name is the field name converted to lower case with dashes replaced by underscores

- `$server_addr`

  an address of the server which accepted a requestComputing a value of this variable usually requires one system call. To avoid a system call, the [listen](http://nginx.org/en/docs/http/ngx_http_core_module.html#listen) directives must specify addresses and use the `bind` parameter.

- `$server_name`

  name of the server which accepted a request

- `$server_port`

  port of the server which accepted a request

- `$server_protocol`

  request protocol, usually “`HTTP/1.0`”, “`HTTP/1.1`”, or “[HTTP/2.0](http://nginx.org/en/docs/http/ngx_http_v2_module.html)”

- `$status`

  response status (1.3.2, 1.2.2)

- `$tcpinfo_rtt`, `$tcpinfo_rttvar`, `$tcpinfo_snd_cwnd`, `$tcpinfo_rcv_space`

  information about the client TCP connection; available on systems that support the `TCP_INFO` socket option

- `$time_iso8601`

  local time in the ISO 8601 standard format (1.3.12, 1.2.7)

- `$time_local`

  local time in the Common Log Format (1.3.12, 1.2.7)

- `$uri`

  current URI in request, [normalized](http://nginx.org/en/docs/http/ngx_http_core_module.html#location)The value of `$uri` may change during request processing, e.g. when doing internal redirects, or when using index files.



## localtion路径匹配



| 路径           | 说明                                                         |
| -------------- | ------------------------------------------------------------ |
| `= uri`        | 路径完全匹配，匹配之后，将不再继续匹配                       |
| `uri`,`^~ uri` | 前缀匹配，匹配后，将会继续匹配。会记录uri最长的匹配，如果最长的匹配以`^~`开头，将不会继续匹配正则表达式，而是直接使用该最长匹配。否者，将继续匹配正则表达式，如果后面没有匹配到正则表达式，将会使用最长的匹配 |
| `~ uri`        | 正则表达式匹配，大小敏感，匹配后，将不再继续匹配             |
| `~* uri`       | 正则表达式匹配，大小不敏感，匹配后，将不会继续陪陪           |



匹配规则

1. 分为三种匹配方式。匹配的顺序是：完全匹配，前缀匹配，正则匹配。每种匹配的内部顺序为书写的顺序
2. 完全匹配：如果匹配将会提前结束，直接使用该规则
3. 前缀匹配：匹配成功会继续匹配，直到匹配完所有的规则。会记录最长的规则。如果最长的规则以`^~`开头，直接使用该规则。否者将继续匹配。前缀匹配不能重复，即一个uri匹配多个规则。
4. 正则匹配：如果匹配将会提前结束，直接使用该规则。如果没有匹配到正则规则，将会使用已匹配的最长的前缀规则



### localtion内部配置

#### rewrite

http://nginx.org/en/docs/http/ngx_http_rewrite_module.html#rewrite



```nginx
rewrite regx replacement [flag]
```

1. `last`：表示修改请求结束，将会开始匹配location。last只能在http块中，不能在location块中。
2. `break`：表示修改请求，开始执行其他命令。break出现在location块中
3. `redirect`：表示永久转发，302
4. `permanent`:表示临时转发，301



rewrite有两个功能，1.修改当前请求的uri和querystring，然后继续由nginx处理。2.重定向到其他请求，直接向客户端返回302或者301响应



###### 修改请求的uri和querystring再匹配location

此时rewrite出现必须出现在http块中。**注意：目标uri中不能包含http或者https，否者会直接向客户端返回302或者301**

```nginx
rewrite ^/proxy/(.*) /proxy/node/$1 last;

location /proxy/ {
  content_by_lua_block {
    ngx.say("proxy: "..ngx.var.uri);
  }
}
```

访问http://192.168.0.6/proxy/aa时，uri会被修改后再匹配location

```
root@f8c1cf26c178:/# curl -i http://192.168.0.6/proxy/aa
HTTP/1.1 200 OK
Server: openresty/1.19.9.1
Date: Thu, 10 Mar 2022 15:53:42 GMT
Content-Type: application/octet-stream
Transfer-Encoding: chunked
Connection: keep-alive

proxy: /proxy/node/aa
```



###### 在location中修改请求uri和querystring然后再进行其他处理

此时必须rewrite必须出现在location块中。**注意：目标uri中不能包含http或者https，否者会直接向客户端返回302或者301。并且flag必须为break**。并且rewrite后，必须需要能要能处理请求，否者会出现404错误

```nginx
location /proxy2/ {
  rewrite ^/proxy2/(.*)$ /proxy2/node/$1 break;
  content_by_lua_block {
    ngx.say("proxy2"..ngx.var.uri);
  }
}

# 返回404
location /proxy5/ {
  rewrite ^/proxy5/(.*)$ /$1 break; # 后面没有其他处理
}

# 因为包含了https，直接返回302重定向，break无效
location /proxy6/ {
  rewrite ^/proxy6/(.*)$ https://www.baidu.com/$1 break;
}
```

访问http://192.168.0.6/proxy2/aa 发现请求的uri已经被修改了

```
root@f8c1cf26c178:/# curl -i http://192.168.0.6/proxy2/aa
HTTP/1.1 200 OK
Server: openresty/1.19.9.1
Date: Thu, 10 Mar 2022 15:54:00 GMT
Content-Type: application/octet-stream
Transfer-Encoding: chunked
Connection: keep-alive

proxy2/proxy2/node/aa
```

访问http:/192.168.0.6/proxy5/proxy4_1/dlkf，并没有重定向到http:/192.168.0.6/proxy4_1/dlkf

```
root@f8c1cf26c178:/# curl -i http:/192.168.0.6/proxy5/proxy4_1/dlkf
HTTP/1.1 404 Not Found
Server: openresty/1.19.9.1
Date: Thu, 10 Mar 2022 16:01:36 GMT
Content-Type: text/html
Content-Length: 159
Connection: keep-alive

<html>
<head><title>404 Not Found</title></head>
<body>
<center><h1>404 Not Found</h1></center>
<hr><center>openresty/1.19.9.1</center>
</body>
</html>
```





###### 重定向到其他其他地址

这种方式是直接返回302或者301请求。目标地址可以仅包含uri，也可以包含schema和hostname和port

```nginx
location /proxy3/ {
  rewrite ^/proxy3/(.*)$ https://www.baidu.com/$1 redirect;
}

location /proxy4/ {
  rewrite ^/proxy4/(.*)$ /$1 redirect;
}

```

访问http://192.168.0.6/proxy3/aa，是会被重定向到https://www.baidu.com/aa

```
root@f8c1cf26c178:/# curl -i http://192.168.0.6/proxy3/aa
HTTP/1.1 302 Moved Temporarily
Server: openresty/1.19.9.1
Date: Thu, 10 Mar 2022 15:54:16 GMT
Content-Type: text/html
Content-Length: 151
Connection: keep-alive
Location: https://www.baidu.com/aa

<html>
<head><title>302 Found</title></head>
<body>
<center><h1>302 Found</h1></center>
<hr><center>openresty/1.19.9.1</center>
</body>
</html>
```



```
root@f8c1cf26c178:/# curl -i http://192.168.0.6/proxy4/proxy4_1/sdfsd
HTTP/1.1 302 Moved Temporarily
Server: openresty/1.19.9.1
Date: Fri, 11 Mar 2022 00:41:17 GMT
Content-Type: text/html
Content-Length: 151
Location: http://192.168.0.6/proxy4_1/sdfsd
Connection: keep-alive

<html>
<head><title>302 Found</title></head>
<body>
<center><h1>302 Found</h1></center>
<hr><center>openresty/1.19.9.1</center>
</body>
</html>
```



#### proxy_paas

表示将请求转发到后端服务器

```nginx
location /api/ { # 如果是前缀匹配，proxy_pass的地址可以带有uri
  proxy_pass http://domain.name/rest/
}

location ~* .*\.html { # 如果是正则匹配，proxy_pass的地址不可以带uri
  proxy_pass http://domian.name
}
```



#### proxy_set_header

当向后端服务器转发请求时，设置的header

```nginx
proxy_set_header X-Real-IP $remote_addr
proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for
```



#### proxy_ignore_headers

向客户端返回响应时，忽略指定header

```nginx
proxy_ignore_headers X-Accel-Expires Expires Cache-Control Set-Cookie; 
```







## 缓存服务器配置

只有方式为GET和HEAD，请求头中不包含`Cache-Control: no-cache,no-store`

官网proxy相关的配置参数：http://nginx.org/en/docs/http/ngx_http_proxy_module.html#proxy_cache_revalidate

参数有以下几个角度

1. 哪些请求需要保存缓存
2. 哪些请求不使用缓存
3. 请求后端服务器失败怎么办？负载均衡
4. 并发请求同一资源怎么处理？加锁
5. 缓存的过期问题怎么处理



server下的配置

```nginx
#为存储承载从代理服务器接收到的数据的临时文件定义目录
proxy_temp_path  /etc/nginx/temp_dir;

# /etc/nginx/cache_dir本地路径，用来设置Nginx缓存资源的存放地址
# levels #默认所有缓存文件都放在同一个/etc/nginx/cache_dir下，但是会影响缓存的性能，因此通常会在/etc/nginx/cache_dir下面建立子目录用来分别存放不同的文件。假设levels=1:2，Nginx为将要缓存的资源生成的key为
#d628235be0b8e19f5f78c37f5f226819，那么key的最后一位9，以及倒数第2-3位81作为两级的子目录，也就是该资源最终会被缓存到/etc/nginx/cache_dir/9/81目录中
# key_zone #参数用来为这个缓存区起名，500m指内存缓存空间大小为500MB，用来为这个缓存区起数用来为这个缓存区起在共享内存中设置一块存储区域来存放缓存的key和metadata（类似使用次数），这样nginx可以快速判断一个request是否命中或者未命中缓存，
#1m可以存储8000个key，10m可以存储80000个key
# max_size #最大cache空间，如果不指定，会使用掉所有disk space，当达到配额后，会删除最少使用的cache文件
# inactive #未被访问文件在缓存中保留时间，本配置中如果1天未被访问则不论状态是否为expired【时间：h(时)/m(分)/s(秒)/d(天)】，缓存控制程序会删掉文件。inactive默认是10分钟。需要注意的是，inactive和expired配置项的含义是不同的，
#expired只是缓存过期，但不会被删除，inactive是删除指定时间内未被访问的缓存文件
# use_temp_path #如果为off，则nginx会将缓存文件直接写入指定的cache文件中，而不是使用temp_path存储，official建议为off，避免文件在不同文件系统中不必要的拷贝
proxy_cache_path /etc/nginx/cache_dir levels=1:2 keys_zone=cache_one:500m max_size=1g inactive=1d use_temp_path=off;

```



#### proxy_cache_path

定义缓存路径。包含了一些缓存的配置属性，还有1个keys_zone配置，其他地方只需要使用keys_zone就可以将缓存放入这个缓存路径中。

```
Syntax:	proxy_cache_path path [levels=levels] [use_temp_path=on|off] keys_zone=name:size [inactive=time] [max_size=size] [min_free=size] [manager_files=number] [manager_sleep=time] [manager_threshold=time] [loader_files=number] [loader_sleep=time] [loader_threshold=time] [purger=on|off] [purger_files=number] [purger_sleep=time] [purger_threshold=time];
Default:	—
Context:	http
```



| 属性                                            | 说明                                                         |
| ----------------------------------------------- | ------------------------------------------------------------ |
| path                                            | 定义放置缓存文件的路径                                       |
| levels                                          | 定义子文件夹的规则。问了避免所有文件放在同一个文件夹中，通过levels规则，可以创建子文件夹 |
| use_temp_path                                   | 使用临时文件路径，如果是off，将不会使用临时文件，而是直接将资源放入cache文件夹中 |
| key_zone                                        | 定义一块内存空间，用于保存索引key的信息                      |
| inactive                                        | 当缓存文件超过某个时间没有被使用，将会被删除                 |
| max_size,min_free                               | 当所有缓存文件占用资源超过max_size或者磁盘没有min_free空间时，将会开启清理最近不使用的文件 |
| manager_files，manager_sleep，manager_threshold | 清理文件是通过一个manager进程完成。清理任务是按批次完成，每批不删除超过manage_files个文件，每批清理间隔manage_sleep时间，每批清理时间不超过manage_threshold |
| loader_files,loader_sleep,loader_threshold      | 会启动另一个进程，将已经存在的缓存文件加载到keys_zone内存中。加载过程也是分批次完成。每批不加载超过loader_files个文件，每批加载间隔manage_sleep时间，每批加载时间不超过loader_threshold |
|                                                 |                                                              |



location下的配置

```nginx
location ~ .*\.(gif|jpg|jpeg|png|bmp|swf)$ {
  # proxy_cache #启用proxy cache，并指定key_zone。另外，如果proxy_cache off表示关闭掉缓存。
  # proxy_cache off;
  proxy_cache cache_one;            # 使用名称为cache_one的对应缓存配置(名称任意命名，但是必须与keys_zone中缓存区域名称一致) 
  proxy_cache_valid  200 24h;       # 指定状态码200的缓存时间为24h
  expires    6h;                    # 表示缓存的过期时间，如果缓存过期，将会重新向后端服务器请求资源
  proxy_redirect  off;              # 指定修改被代理服务器返回的响应头中的location头域跟refresh头域数值
  proxy_pass  http://front_picture; # 指代理后转发的路径，注意是否 需要 最后的 /【注意：不走代理的服务，无法进行缓存】   
}
```



#### proxy_cache_valid

可以根据不同的响应，设置不同的缓存时间。proxy_cache_valid可以出现多次

```nginx
proxy_cache_valid 200 201 10h ; # 指定状态码设置设置时间

proxy_cache_valid 10h ; # 不指定状态码时，仅有200 301 302的请求会被缓存

proxy_cache_valid any 10h ; # any表示所有状态码都会缓存
```





#### proxy_cache_methods

缓存的方法。默认是GET和HEAD



#### proxy_cache_min_uses

指定一个数字。当资源被请求达到这个次数时，资源将会被缓存。



#### proxy_cache_key

缓存的key，默认为`$scheme$proxy_host$uri$is_args$args;`，



#### proxy_next_upstream

当有多个后端服务器时，什么时候需要请求下一个后端服务器。用于在一个后端服务器不能返回预期响应时，nginx会把请求转发到其他后端服务器上

```
Syntax:	proxy_next_upstream error | timeout | invalid_header | http_500 | http_502 | http_503 | http_504 | http_403 | http_404 | http_429 | non_idempotent | off ...;
Default:	
proxy_next_upstream error timeout;
Context:	http, server, location
```



#### proxy_next_upstream_timeout

表示请求在多长时间内才能被转发到其他后端服务器。0表示不约束时间

```
Syntax:	proxy_next_upstream_timeout time;
Default:	
proxy_next_upstream_timeout 0;
Context:	http, server, location
```



#### proxy_next_upstream_tries

尝试的最大次数。0表示不约束次数

```
Syntax:	proxy_next_upstream_tries number;
Default:	
proxy_next_upstream_tries 0;
Context:	http, server, location
```



#### proxy_no_cache

指定哪些请求的响应不保存到缓存

```
Syntax:	proxy_no_cache string ...;
Default:	—
Context:	http, server, location
```



```nginx
proxy_no_cache $cookie_nocache $arg_nocache$arg_comment;
proxy_no_cache $http_pragma    $http_authorization;
```



#### proxy_cache_bypass

指定哪些请求不使用缓存，默认都使用缓存

```
Syntax:	proxy_cache_bypass string ...;
Default:	—
Context:	http, server, location
```



```nginx
proxy_cache_bypass $cookie_nocache $arg_nocache$arg_comment;
proxy_cache_bypass $http_pragma    $http_authorization;
```





#### proxy_cache_lock

是否开启缓存资源锁。默认不开启。如果开启，当多个客户端请求同一资源时，第一个请求会向后端服务器请求资源，后面的请求将会等待资源。



```
Syntax:	proxy_cache_lock on | off;
Default:	
proxy_cache_lock off;
Context:	http, server, location
```



#### proxy_cache_lock_age

指定一个加锁的时间，默认是5s。当锁的时间超过这个时间后，后面的请求将直接向后端服务器请求资源，不需要等待第一个请求结束。



```
Syntax:	proxy_cache_lock_age time;
Default:	
proxy_cache_lock_age 5s;
Context:	http, server, location
```



#### proxy_cache_lock_timeout

指定一个等待锁的时间，默认是5s。当请求等待锁结束（第一个请求结束）超过这个时间，将不会等待，直接向后端服务器请求资源



#### proxy_cache_background_update

是否允许后台更新缓存。该参数必须在允许使用使用过期缓存的时候，才生效



```
Syntax:	proxy_cache_background_update on | off;
Default:	
proxy_cache_background_update off;
Context:	http, server, location
```





#### proxy_cache_purge

表示清除缓存，当后面的参数有一个不为0或者不为空字符串，将会清理cache_key对应的缓存。用于手动清理缓存。

```
Syntax:	proxy_cache_purge string ...;
Default:	—
Context:	http, server, location
```

如果出现错误，说明是安装的nginx缺少purge模块，需要重新编译。参考：https://blog.csdn.net/weixin_34244102/article/details/89999655

![image-20220307081720318](/开源框架/nginx/.assert/nginx使用/image-20220307081720318.png)

```nginx
        location ~ /purge/doc(/.*) {
            proxy_cache mycache;
            proxy_cache_key $1$is_args$args;
            proxy_cache_purge 1;
        }
```







```
docker run -d --name nginx1 --network mynet --ip 192.168.0.5 -v /Users/hewutao/docker/extfile/nginx/nginx1/nginx.conf:/etc/nginx/nginx.conf -v /Users/hewutao/docker/extfile/nginx/nginx1/html:/usr/share/nginx/html -p 8080:80 nginx
```





```
docker exec f8c1 nginx -s reload

docker exec f8c1 nginx -t
```

