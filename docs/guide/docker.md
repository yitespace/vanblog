---
icon: linux
title: 部署
copyright: false
order: -1
---

欢迎使用 VanBlog ，只需几个步骤，你就可以在你的服务器搭建自己的博客服务了。

你也可以先查看 [Demo](https://blog-demo.mereith.com)，账号密码均为 `demo`

## docker-compose 部署

新建 `docker-compose.yml`文件：

```yaml
version: "3"

services:
  vanblog:
    # 默认国内原的
    image: registry.cn-beijing.aliyuncs.com/mereith/van-blog:latest
    # image: mereith/van-blog:latest
    restart: always
    environment:
      TZ: "Asia/Shanghai"
      # 图片资源允许的域名，英文逗号分隔
      VAN_BLOG_ALLOW_DOMAINS: "pic.mereith.com"
      # CDN URL，包含协议，部署到 cdn 的时候用。
      VAN_BLOG_CDN_URL: "https://www.mereith.com"
      # mongodb 的地址
      VAN_BLOG_DATABASE_URL: "mongodb://vanBlog:vanBlog@mongo:27017/vanBlog?authSource=admin"
      # jwt 密钥，随机字符串即可
      VAN_BLOG_JWT_SECRET: "AnyString"
    volumes:
      # 图床文件的存放地址，按需修改。
      - ${PWD}/data/static:/app/static
    ports:
      - 80:80
  mongo:
    image: mongo
    restart: always
    environment:
      TZ: "Asia/Shanghai"
      # mongoDB 初始化用户名
      MONGO_INITDB_ROOT_USERNAME: vanBlog
      # mongoDB 初始化密码
      MONGO_INITDB_ROOT_PASSWORD: vanBlog
    volumes:
      # mongoDB 数据存放地址，按需修改。
      - ${PWD}/data/mongo:/data/db
# 如果你需要评论系统就加上
# 具体请看参数请看 waline 文档：
# https://waline.js.org/
# waline:
#   image: lizheming/waline:latest
#   restart: always
#   ports:
#     - 127.0.0.1:8360:8360
#   volumes:
#     - /var/docker/waline/data:/app/data
#   environment:
#     TZ: 'Asia/Shanghai'
#     SITE_NAME: 'Mereith Blog'
#     SITE_URL: 'https://www.mereith.com'
#     SECURE_DOMAINS: 'mereith.com'
#     AUTHOR_EMAIL: 'wanglu@mereith.com'
#     MONGO_HOST: "mongo"
#     MONGO_DB: "waline"
#     MONGO_USER: "vanBlog"
#     MONGO_PASSWORD: "vanBlog"
#     MONGO_AUTHSOURCE: "admin"
```

按注释说明修改`docker-compose.yml`的配置后：

```bash
docker-compose up -d
```

浏览器打开 `http://<your-ip>/admin/init` ，并按照提示初始化即可。具体设置项可以参考 [站点配置](/feat/basic/setting.md)

## kubernetes

什么？你想用 `kubernetes`，当然没问题。

给你一个 `deployment`的参考：

```yaml
kind: Deployment
apiVersion: apps/v1
metadata:
  name: van-blog
  labels:
    app: van-blog
spec:
  selector:
    matchLabels:
      app: van-blog
  template:
    spec:
      volumes:
        - name: host-time
          hostPath:
            path: /etc/localtime
            type: ""
      containers:
        - name: van-blog
          image: "mereith/van-blog:latest"
          ports:
            - name: http-80
              containerPort: 80
              protocol: TCP
          env:
            - name: VAN_BLOG_DATABASE_URL
              value: >-
                mongodb://some@some@van.example.com:27017/vanBlog?authSource=admin
            - name: VAN_BLOG_ALLOW_DOMAINS
              value: >-
                pic.mereith.com
          resources:
            requests:
              memory: "300Mi"
              cpu: "250m"
          limits:
            memory: "500Mi"
            cpu: "500m"
          volumeMounts:
            - name: host-time
              readOnly: true
              mountPath: /etc/localtime
          imagePullPolicy: Always
```