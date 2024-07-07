# Nginx-常见配置

```nginx
# 设置运行 Nginx 的用户和用户组
user  www www;

# 启动的 worker 进程数，一般设置为 CPU 核心数的两倍
worker_processes  2;

# Nginx 主进程的 PID 文件路径
pid /var/run/nginx.pid;

# 错误日志文件路径和日志级别 [ debug | info | notice | warn | error | crit ]
error_log  /var/log/nginx.error_log  info;

# events 模块配置，指定事件驱动模型和最大连接数
events {
    worker_connections   2000;  # 每个 worker 进程的最大连接数
    use kqueue;  # 使用 kqueue 作为事件驱动模型，可根据系统选择 epoll、/dev/poll、select、poll 等
}

# http 模块配置
http {
    include       conf/mime.types;  # 包含 MIME 类型定义文件
    default_type  application/octet-stream;  # 默认 MIME 类型

    # 定义日志格式
    log_format main      '$remote_addr - $remote_user [$time_local] '
                         '"$request" $status $bytes_sent '
                         '"$http_referer" "$http_user_agent" '
                         '"$gzip_ratio"';

    log_format download  '$remote_addr - $remote_user [$time_local] '
                         '"$request" $status $bytes_sent '
                         '"$http_referer" "$http_user_agent" '
                         '"$http_range" "$sent_http_content_range"';

    # 客户端超时配置
    client_header_timeout  3m;
    client_body_timeout    3m;
    send_timeout           3m;

    client_header_buffer_size    1k;
    large_client_header_buffers  4 4k;

    # 启用 gzip 压缩
    gzip on;
    gzip_min_length  1100;
    gzip_buffers     4 8k;
    gzip_types       text/plain;

    # 输出缓冲区设置
    output_buffers   1 32k;
    postpone_output  1460;

    # 文件传输优化
    sendfile         on;
    tcp_nopush       on;
    tcp_nodelay      on;
    send_lowat       12000;

    keepalive_timeout  75 20;

    # 虚拟主机配置
    server {
        listen        one.example.com;  # 监听的域名
        server_name   one.example.com  www.one.example.com;  # 服务器名

        access_log   /var/log/nginx.access_log  main;  # 访问日志文件路径和使用的日志格式

        location / {
            proxy_pass         http://127.0.0.1/;  # 反向代理配置
            proxy_redirect     off;

            proxy_set_header   Host             $host;
            proxy_set_header   X-Real-IP        $remote_addr;

            client_max_body_size       10m;
            client_body_buffer_size    128k;

            client_body_temp_path      /var/nginx/client_body_temp;

            proxy_connect_timeout      70;
            proxy_send_timeout         90;
            proxy_read_timeout         90;
            proxy_send_lowat           12000;

            proxy_buffer_size          4k;
            proxy_buffers              4 32k;
            proxy_busy_buffers_size    64k;
            proxy_temp_file_write_size 64k;

            proxy_temp_path            /var/nginx/proxy_temp;

            charset  koi8-r;  # 字符编码设置
        }

        error_page  404  /404.html;  # 自定义错误页面配置

        location = /404.html {
            root  /spool/www;  # 错误页面存放路径
        }

        location /old_stuff/ {
            rewrite   ^/old_stuff/(.*)$  /new_stuff/$1  permanent;  # URL 重写规则
        }

        location /download/ {
            valid_referers  none  blocked  server_names  *.example.com;  # 配置有效的 referer

            if ($invalid_referer) {
                return   403;  # 如果 referer 不合法，返回 403 Forbidden
            }

            rewrite ^/(download/.*)/mp3/(.*)\..*$ /$1/mp3/$2.mp3 break;  # URL 重写规则

            root         /spool/www;  # 静态文件根目录
            access_log   /var/log/nginx-download.access_log  download;  # 访问日志文件路径和使用的日志格式
        }

        location ~* \.(jpg|jpeg|gif)$ {
            root         /spool/www;  # 匹配静态文件后缀的请求
            access_log   off;  # 关闭访问日志记录
            expires      30d;  # 设置缓存有效期为 30 天
        }
    }
}
```
