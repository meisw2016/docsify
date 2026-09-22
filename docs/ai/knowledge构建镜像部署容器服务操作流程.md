# knowledge-frontend构建镜像部署容器服务操作流程
1. 创建目录并编写配置文件内容
~~~
.
├── dist
│   ├── assets
│   └── index.html
├── Dockerfile
└── nginx.conf
~~~

	- nginx.conf文件内容
~~~
[root@DESKTOP-IRLI779 frontend]# cat nginx.conf

#user  nobody;
worker_processes  1;

#error_log  logs/error.log;
#error_log  logs/error.log  notice;
#error_log  logs/error.log  info;

#pid        logs/nginx.pid;


events {
    worker_connections  1024;
}


http {
    include       mime.types;
    default_type  application/octet-stream;
    sendfile        on;
    keepalive_timeout  65;


    server {
        listen 80;
        server_name localhost;  # 或你的域名/IP

        root /usr/share/nginx/html;
        # 前端静态文件目录
        index index.html;

        # 前端路由支持（SPA必须）
        location / {
            #root  /usr/share/nginx/html;
            try_files $uri $uri/ /index.html;
        }

        # API请求代理到后端
        location /api/ {
            proxy_pass http://knowledge-backend:20000;
            proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            proxy_set_header X-Forwarded-Proto $scheme;

            # 超时设置
            proxy_connect_timeout 60s;
            proxy_send_timeout 60s;
            proxy_read_timeout 60s;
        }

        # 上传文件代理
        location /uploads/ {
            proxy_pass http://knowledge-backend:20000;
            proxy_set_header Host $host;
            client_max_body_size 500M;  # 允许上传大文件
        }

        # 分享文档代理
        location /share/ {
            proxy_pass http://knowledge-backend:20000;
            proxy_set_header Host $host;
        }

        # OAuth2登录回调代理
        location /login/oauth2/ {
            proxy_pass http://knowledge-backend:20000;
            proxy_set_header Host $host;
        }

        # 静态资源缓存
        location ~* \.(js|css|png|jpg|jpeg|gif|ico|svg|woff|woff2|ttf|eot)$ {
            expires 30d;
            add_header Cache-Control "public, immutable";
        }

        # Gzip压缩
        gzip on;
        gzip_vary on;
        gzip_min_length 1024;
        gzip_types text/plain text/css text/xml text/javascript application/javascript application/json application/xml;
    }
}
~~~

	- Dockerfile文件内容
~~~
[root@DESKTOP-IRLI779 frontend]# cat Dockerfile
FROM nginx:alpine

# 删除nginx默认页面
RUN rm -rf /usr/share/nginx/html/*

# 复制dist文件夹内容到nginx html目录
COPY dist/ /usr/share/nginx/html/

# 复制自定义nginx配置 （可选）
COPY nginx.conf /etc/nginx/nginx.conf

EXPOSE 80

CMD ["nginx", "-g", "daemon off;"]
~~~

2. 构建
~~~
docker build -t knowledge-frontend-image:latest .
~~~

3. 启动容器
~~~
docker run -d \
  --name knowledge-frontend \
  --network=mysql \
  --ip=172.20.0.14 \
  -p 5000:80 \
  knowledge-frontend-image:latest
~~~

4. 将5000端口转发到windows机器上
~~~
1. 先在wsl查看IP
	hostname -I
2. 管理员打开cmd添加端口映射
	netsh interface portproxy add v4tov4 listenport=5000 listenaddress=0.0.0.0 connectport=5000 connectaddress=172.27.185.144
3. 浏览器访问
	可以通过 172.27.185.144:5000进行访问了，禁止使用localhost的形式访问了。
~~~

5. 验证端口是否生效
~~~
C:\WINDOWS\system32>netsh interface portproxy show all

侦听 ipv4:                 连接到 ipv4:

地址            端口        地址            端口
--------------- ----------  --------------- ----------
0.0.0.0         6379        172.30.208.1    6379
0.0.0.0         5672        172.30.208.1    5672
0.0.0.0         15672       172.30.208.1    15672
0.0.0.0         3307        172.27.185.144  3307
0.0.0.0         3308        172.27.185.144  3308
0.0.0.0         63790       172.27.185.144  63790
0.0.0.0         5000        172.27.185.144  5000
~~~


