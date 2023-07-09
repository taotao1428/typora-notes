# nginx工作原理



## nginx的结构

### nginx模块

模块的类型：NGX_HTTP_MODULE,NGINX_CORE_MODULE,NGX_CONF_MODULE,NGX_EVENT_MODULE,NGX_MAIL_MODULE

```c
ngx_module_t  ngx_http_rewrite_module = {
    NGX_MODULE_V1,
    &ngx_http_rewrite_module_ctx,          /* module context */
    ngx_http_rewrite_commands,             /* module directives */
    NGX_HTTP_MODULE,                       /* module type */
    NULL,                                  /* init master */
    NULL,                                  /* init module */
    NULL,                                  /* init process */
    NULL,                                  /* init thread */
    NULL,                                  /* exit thread */
    NULL,                                  /* exit process */
    NULL,                                  /* exit master */
    NGX_MODULE_V1_PADDING
};
```



### 模块的Context

模块的上下文配置了8个方法，在读取配置时，会调用它们，调用顺序如下

1. create_main_conf。创建main级别的全局配置数据结构，也就是http模块
2. Create_srv_conf。创建server级别的配置数据结构，也就是server模块
3. create_loc_conf。创建location级别的配置数据结构，也就是location模块
4. preconfiguration。解析配置文件前调用。
5. Init_main_conf。初始化main级别配置。
6. merge_srv_conf。用于合并main级别和srv级别下的同名配置项
7. Merge_loc_conf。用于合并location级别和server级别下的同名配置
8. Postconfiguration。完成配置文件解析后调用

```c


static ngx_http_module_t  ngx_http_rewrite_module_ctx = {
    NULL,                                  /* preconfiguration */
    ngx_http_rewrite_init,                 /* postconfiguration */

    NULL,                                  /* create main configuration */
    NULL,                                  /* init main configuration */

    NULL,                                  /* create server configuration */
    NULL,                                  /* merge server configuration */

    ngx_http_rewrite_create_loc_conf,      /* create location configuration */
    ngx_http_rewrite_merge_loc_conf        /* merge location configuration */
};



```

1. 所有创建配置的方法（create location configuration等），其实是创建一个自定义对象保存配置信息，在调用指令处理请求时，会把该配置对象传到方法中
2. 在postconfiguration方法中，可以在不同的phase时，添加自己的handler方法。

```c
// rewrite模块的postconfiguration方法
static ngx_int_t
ngx_http_rewrite_init(ngx_conf_t *cf)
{
    ngx_http_handler_pt        *h;
    ngx_http_core_main_conf_t  *cmcf;

    cmcf = ngx_http_conf_get_module_main_conf(cf, ngx_http_core_module);
		// 将ngx_http_rewrite_handler添加到NGX_HTTP_SERVER_REWRITE_PHASE阶段
    h = ngx_array_push(&cmcf->phases[NGX_HTTP_SERVER_REWRITE_PHASE].handlers);
    if (h == NULL) {
        return NGX_ERROR;
    }

    *h = ngx_http_rewrite_handler;
    // 将ngx_http_rewrite_handler添加到NGX_HTTP_REWRITE_PHASE阶段
    h = ngx_array_push(&cmcf->phases[NGX_HTTP_REWRITE_PHASE].handlers);
    if (h == NULL) {
        return NGX_ERROR;
    }

    *h = ngx_http_rewrite_handler;

    return NGX_OK;
}
```



### 模块的Directives

```c
static ngx_command_t  ngx_http_rewrite_commands[] = {

    { ngx_string("rewrite"),
      NGX_HTTP_SRV_CONF|NGX_HTTP_SIF_CONF|NGX_HTTP_LOC_CONF|NGX_HTTP_LIF_CONF
                       |NGX_CONF_TAKE23,
      ngx_http_rewrite,
      NGX_HTTP_LOC_CONF_OFFSET,
      0,
      NULL },

      ngx_null_command  // 模块的最后1个元素需要为ngx_null_command
};


```



### HTTP处理的phase

HTTP处理流程分为11阶段，每个阶段可以由多个HTTP模块流水式处理

| 阶段                          | 第三方能否处理 | 说明                                                         |
| ----------------------------- | -------------- | ------------------------------------------------------------ |
| NGX_HTTP_POST_READ_PHASE      | 是             | 接收到完整的HTTP头部后                                       |
| NGX_HTTP_SERVER_REWRITE_PHASE | 是             | 在将uri和location匹配前修改请求uri                           |
| NGX_HTTP_FIND_CONFIG_PHASE    | 否             | 根据uri寻找location                                          |
| NGX_HTTP_REWRITE_PHASE        | 是             | 匹配location后，再次修改uri                                  |
| NGX_HTTP_POST_REWRITE_PHASE   | 否             | 用于检查是否rewrite出现死循环                                |
| NGX_HTTP_PREACCESS_PHASE      | 是             | 在进入NGX_HTTP_ACCESS_PHASE前                                |
| NGX_HTTP_ACCESS_PHASE         | 是             | 判断请求是否允许访问服务器                                   |
| NGX_HTTP_POSTACCESS_PHASE     | 否             | 如果NGX_HTTP_ACCESS_PHASE返回不允许访问的错误码，该阶段将会返回响应给客户端 |
| NGX_HTTP_TRY_FILES_PHASE      | 否             | 只要针对tryfiles命令                                         |
| NGX_HTTP_CONTENT_PHASE        | 是             | 生成响应内容，大部分第三方模块在该阶段工作                   |
| NGX_HTTP_LOG_PHASE            | 是             | 请求处理完后，记录日志                                       |







## 书写一个简单第三方HTTP模块

命令为mytest，当在location使用该location时，将会返回`Hello world`。



### 书写配置文件

创建文件夹mytest，在文件夹下创建config文件。config文件是一个shell文件，在执行./configuration时会被执行

1. ngx_addon_name：插件的名称，通常为模块名称
2. HTTP_MODULES：表示所有http模块的名称，需要在该变量中添加第三方模块的名称
3. NGX_ADDON_SRCS：所有插件的源代码。其中$ngx_addon_dir为在执行./configuration是传入--add-modules=PATH中的PATH

```shell
ngx_addon_name=ngx_http_mytest_module
HTTP_MODULES="$HTTP_MODULES ngx_http_mytest_module"
NGX_ADDON_SRCS="$NGX_ADDON_SRCS $ngx_addon_dir/ngx_http_mytest_module.c"
```



### 书写模块代码

模块代码分为3块

1. 模块定义
2. context定义
3. command定义



```c

#include <ngx_config.h>
#include <ngx_core.h>
#include <ngx_http.h>

static char *ngx_http_mytest(ngx_conf_t *cf, ngx_command_t *cmd,
                                  void *conf);

static ngx_int_t ngx_http_mytest_handler(ngx_http_request_t * r);

// 定义命令
static ngx_command_t ngx_http_mytest_commands[] = {
        {
            ngx_string("mytest"),
            NGX_HTTP_MAIN_CONF|NGX_HTTP_SRV_CONF|NGX_HTTP_LOC_CONF|NGX_HTTP_LMT_CONF|NGX_CONF_NOARGS,
            ngx_http_mytest, // 命令处理的handler
            NGX_HTTP_LOC_CONF_OFFSET,
            0,
            NULL
        },
        ngx_null_command
};

// 模块的context
static ngx_http_module_t  ngx_http_mytest_module_ctx = {
        NULL,                                  /* preconfiguration */
        NULL,                 /* postconfiguration */
        NULL,                                  /* create main configuration */
        NULL,                                  /* init main configuration */
        NULL,                                  /* create server configuration */
        NULL,                                  /* merge server configuration */

        NULL,      /* create location configuration */
        NULL        /* merge location configuration */
};

// 模块
ngx_module_t  ngx_http_mytest_module = {
        NGX_MODULE_V1,
        &ngx_http_mytest_module_ctx,          /* module context */
        ngx_http_mytest_commands,             /* module directives */
        NGX_HTTP_MODULE,                       /* module type */
        NULL,                                  /* init master */
        NULL,                                  /* init module */
        NULL,                                  /* init process */
        NULL,                                  /* init thread */
        NULL,                                  /* exit thread */
        NULL,                                  /* exit process */
        NULL,                                  /* exit master */
        NGX_MODULE_V1_PADDING
};


static char * ngx_http_mytest(ngx_conf_t * cf, ngx_command_t * cmd, void *conf) {
    ngx_http_core_loc_conf_t *clcf;

    clcf = ngx_http_conf_get_module_loc_conf(cf, ngx_http_core_module);

    clcf->handler = ngx_http_mytest_handler;

    return NGX_CONF_OK;
}

static ngx_int_t ngx_http_mytest_handler(ngx_http_request_t *r) {
    if (!(r->method & (NGX_HTTP_GET|NGX_HTTP_HEAD))) {
        return NGX_HTTP_NOT_ALLOWED;
    }

    ngx_int_t rc = ngx_http_discard_request_body(r);

    if (rc != NGX_OK) {
        return rc;
    }

    ngx_str_t type = ngx_string("text/plain");
    ngx_str_t response = ngx_string("Hello world");

    r->headers_out.status = NGX_HTTP_OK;
    r->headers_out.content_length_n = response.len;
    r->headers_out.content_type = type;


    rc = ngx_http_send_header(r);
    if (rc == NGX_ERROR || rc > NGX_OK || r->header_only) {
        return rc;
    }

    ngx_buf_t *b;
    b = ngx_create_temp_buf(r->pool, response.len);
    if (b == NULL) {
        return NGX_HTTP_INTERNAL_SERVER_ERROR;
    }

    ngx_memcpy(b->pos, response.data, response.len);

    b->last = b->pos + response.len;
    b->last_buf = 1;

    ngx_chain_t  out;
    out.buf = b;
    out.next = NULL;

    return ngx_http_output_filter(r, &out);
}
```



### 编译

在nginx根目录执行

```shell
./auto/configure  --with-pcre=/Users/hewutao/software/pcre-8.45 --add-module="/Users/hewutao/CLionProjects/nginx/src/ext/mytest"

# 因为缺少pcre包，所有增加--with-pcre=/Users/hewutao/software/pcre-8.45 

make
```



### 测试运行

在nginx项目根目录下objs/nginx为编译好的nginx

![image-20220313143558707](/开源框架/nginx/.assert/nginx工作原理/image-20220313143558707.png)

书写配置文件，增加下面的location。然后启动nginx

```nginx

        location /mytest/ {
            mytest;
        }
```

```shell
./nginx -p /Users/hewutao/CLionProjects/nginx
```





![image-20220313144212693](/开源框架/nginx/.assert/nginx工作原理/image-20220313144212693.png)





## 问题

```
the HTTP rewrite module requires the PCRE library
```



处理

```
brew install pcre

./auto/configure  --with-pcre=/opt/homebrew/Cellar/pcre/8.45
```

