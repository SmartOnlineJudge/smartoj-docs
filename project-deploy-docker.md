# Docker 部署方案

## 容器化部署介绍

### 概念
容器化部署是一种现代软件部署技术，它将应用程序及其运行所需的所有依赖项（包括代码、运行时环境、系统工具、库文件和配置）打包成一个标准化、轻量级的独立单元——容器。

核心工作原理：

+ 基于操作系统级虚拟化（而非传统虚拟机的硬件级虚拟化），容器共享宿主机的操作系统内核，但拥有独立的文件系统、进程空间和网络栈。
+ 通过镜像（Image）作为模板：开发者使用Dockerfile定义应用环境，构建出包含完整运行环境的镜像。
+ 通过容器引擎（如Docker）运行镜像，创建可执行的容器实例。
+ 在生产环境中，通常结合容器编排系统（如Kubernetes）实现多容器的自动化部署、扩缩容和管理。

### 容器化部署的主要优势
1. 环境一致性与可移植性

+ "一次构建，随处运行"：容器将应用与环境整体打包，彻底解决"在我机器上能运行"的问题，确保开发、测试、生产环境完全一致。
+ 容器符合OCI（Open Container Initiative）标准，可在任何支持容器化的平台（Linux、Windows、云环境）无缝迁移。

2. 轻量高效

+ 资源占用少：容器共享宿主机内核，无需虚拟完整操作系统，通常仅占用几十到几百MB空间，而虚拟机需占用数GB。
+ 启动速度快：容器启动时间可达秒级甚至毫秒级，远快于虚拟机的分钟级启动。
+ 高密度部署：单台物理服务器可运行更多容器实例，显著提升硬件资源利用率。

3. 开发运维效率提升

+ 简化依赖管理：开发者无需提供复杂的环境配置说明，只需共享一个预配置好的容器。
+ 加速CI/CD流程：容器镜像可作为标准化交付物，无缝集成到持续集成/持续部署流水线。
+ 支持微服务架构：容器天然适合微服务拆分，每个服务可独立开发、部署和扩展。

4. 弹性与可靠性

+ 快速扩缩容：结合 Kubernetes 等编排工具，可根据负载自动增减容器实例。
+ 故障隔离：容器间相互隔离，单个容器故障不会影响其他容器。
+ 版本回滚便捷：通过镜像版本管理，可快速回退到稳定版本。

5. 与虚拟机的关键区别

| 特性       | 容器               | 虚拟机             |
| ---------- | ------------------ | ------------------ |
| 虚拟化层级 | 操作系统级         | 硬件级             |
| 资源开销   | 低（共享内核）     | 高（完整 OS 副本） |
| 启动速度   | 秒级               | 分钟级             |
| 隔离性     | 进程级隔离         | 完全硬件隔离       |
| 适用场景   | 无状态应用、微服务 | 需要完整OS的场景   |


> 在项目部署中，使用容器来部署会省去很多不必要的麻烦，尤其是后端。
>

## 部署环境准备
### 服务器准备
准备一台内存至少 4GB 并且携带了公网 IP 的服务器，因为项目中用到了非常多的数据库，这些数据库会占用掉很多内存，尤其是 ElasticSearch，将会占用掉差不多 1GB 的内存。

服务器可以到腾讯云或者阿里云上面购买，一般新用户或者学生都会很便宜。

使用以下这台服务器来作为部署环境的宿主机：

![](https://cdn.nlark.com/yuque/0/2025/png/47866636/1766988506721-36ca0b32-972a-461f-8469-68c2a3b08d2b.png)

### 运维面板安装
> 在为服务器安装操作系统的时候，如果运营商支持，你也可以顺便选择安装运维面板。
>

在相关运营商的服务器控制面板中安装好相应的操作系统，这里选择的是 Ubuntu 镜像。

为了简化部署流程，在部署之前推荐安装一个可视化的服务器运维面板，有了可视化界面，可以减少纯命令行的使用，以此来提高部署的复杂度。

最常用的服务器运维面板有：`宝塔`和`1Panel`。

由于项目中的部分服务需要使用 Docker 来部署，因此这里优先选择 1Panel 面板来作为运维面板。

在服务器运营商的控制台中登录到服务器终端：

![](https://cdn.nlark.com/yuque/0/2025/png/47866636/1766988793212-395dbd25-ed70-4d34-a076-16c89e8751ea.png)

下面就是输入命令安装面板了。

> 官方文档：[https://1panel.cn/docs/v1/installation/online_installation/](https://1panel.cn/docs/v1/installation/online_installation/)
>

首先确认好你的服务器操作系统类型，然后在官方文档中找到相关的安装命令：

![](https://cdn.nlark.com/yuque/0/2025/png/47866636/1766989073962-9da0a62c-7dc3-4acf-80f2-f7a6f81e1b57.png)

这台服务器的操作系统是 Ubuntu，因此输入以下命令来安装：

```bash
curl -sSL https://resource.fit2cloud.com/1panel/package/quick_start.sh -o quick_start.sh && sudo bash quick_start.sh
```

在安装的过程中可能会涉及到语言、端口、面板账号密码的设置等等操作：

![](https://cdn.nlark.com/yuque/0/2025/png/47866636/1766989488958-2a7a1b3f-817a-4482-a883-39df15f558cf.png)

![](https://cdn.nlark.com/yuque/0/2025/png/47866636/1766989525878-04d95b82-f742-47bb-a422-e6c57bbb1514.png)

特别是端口、访问路径、账号密码等等，这些配置是比较重要的。

配置完毕以后，需要在服务器运营商中打开刚刚设置的端口，这样才能够通过链接来访问面板：

![](https://cdn.nlark.com/yuque/0/2025/png/47866636/1766989791812-07aa9ae7-2072-4796-89e4-217f704c1420.png)

然后输入服务器的公网 IP + 端口 + 访问后缀（在安装的时候已经设置好了），即可访问面板：

![](https://cdn.nlark.com/yuque/0/2025/png/47866636/1766989858761-339c94b9-0714-4643-947f-f6f5ae6c72c4.png)

然后输入刚刚安装的时候设置的账号密码，即可登录到服务器内部：

![](https://cdn.nlark.com/yuque/0/2025/png/47866636/1766989969869-847ac845-1c47-4394-9940-71ea6c37b1f1.png)

在这里可以很方面通过可视化的方式来操作、安装相关的内容。

## 创建项目公用网络
由于项目涉及到多个网络进程间的通信，而每个项目都不在同一个容器中，因此需要通过 Docker 的 Network 将这些项目连接起来，以完成网络进程间的通信。

在容器面板中点击“创建网络”：

![](https://cdn.nlark.com/yuque/0/2026/png/47866636/1770468925960-924a9218-b0ec-4f1c-9282-b7d9ad84f3ae.png)

![](https://cdn.nlark.com/yuque/0/2026/png/47866636/1770469090307-78ae86ce-899e-4fdb-8574-1af40f4c41d3.png)

然后点击确认即可。

## 数据库及中间件安装
> 代码仓库：[https://github.com/SmartOnlineJudge/smartoj-backend](https://github.com/SmartOnlineJudge/smartoj-backend)。
>

首先将后端仓库下载到服务器上，因为相关 Docker 配置放在后端仓库中了。

这里我们不需要手动安装相关 Docker 镜像了，我们直接使用`docker-compose.infra.yaml`文件，一键将所有的基础设施安装好（需要确保`smartoj.sql`这个文件在 compose 文件目录下）：

```yaml
services:
  # MySQL 数据库
  mysql:
    image: mysql:8.0.43
    container_name: smartoj-mysql
    environment:
      MYSQL_ROOT_PASSWORD: your_password  # 改成你的数据库密码
    ports:
      - "3306:3306"
    volumes:
      - mysql-data:/var/lib/mysql
      - ./smartoj.sql:/docker-entrypoint-initdb.d/smartoj.sql:ro
    # 开启 binlog
    command:
      - --server-id=101
      - --log-bin=mysql-bin
      - --binlog-format=ROW
      - --binlog-row-image=FULL
      - --binlog_expire_logs_seconds=604800
    restart: unless-stopped
    networks:
      - smartoj-network
    healthcheck:
      test: ["CMD", "mysqladmin", "ping", "-h", "localhost", "-u", "root", "-p$$MYSQL_ROOT_PASSWORD"]
      interval: 10s
      timeout: 5s
      retries: 5

  # Redis
  redis:
    image: redis:8.0.3-alpine
    container_name: smartoj-redis
    ports:
      - "6379:6379"
    command: redis-server --requirepass your_password  # 改成你的数据库密码
    volumes:
      - redis-data:/data
    restart: unless-stopped
    networks:
      - smartoj-network
    healthcheck:
      test: ["CMD", "redis-cli", "--raw", "incr", "ping"]
      interval: 10s
      timeout: 5s
      retries: 5

  # RabbitMQ
  rabbitmq:
    image: rabbitmq:4.0.7-management-alpine
    container_name: smartoj-rabbitmq
    environment:
      RABBITMQ_DEFAULT_USER: smartoj
      RABBITMQ_DEFAULT_PASS: your_password  # 改成你的密码
    ports:
      - "5672:5672"
      - "15672:15672"
    volumes:
      - rabbitmq-data:/var/lib/rabbitmq
    restart: unless-stopped
    networks:
      - smartoj-network
    healthcheck:
      test: ["CMD", "rabbitmq-diagnostics", "-q", "ping"]
      interval: 30s
      timeout: 10s
      retries: 5

  # ElasticSearch
  elasticsearch:
    build:
      context: .
      dockerfile: elasticsearch.Dockerfile
    container_name: smartoj-elasticsearch
    environment:
      - discovery.type=single-node
      - xpack.security.enabled=true
      - ELASTIC_USERNAME=elastic
      - ELASTIC_PASSWORD=elastic  # 改成你的 ES 密码
      - "ES_JAVA_OPTS=-Xms512m -Xmx512m"
    ports:
      - "9200:9200"
      - "9300:9300"
    volumes:
      - es-data:/usr/share/elasticsearch/data
    restart: unless-stopped
    networks:
      - smartoj-network
    healthcheck:
      # 注意 ES 的密码
      test: ["CMD-SHELL", "curl -sf -u elastic:elastic http://localhost:9200/_cluster/health || exit 1"]
      interval: 30s
      timeout: 10s
      retries: 5

  # MinIO 对象存储
  minio:
    image: minio/minio:RELEASE.2025-02-18T00-41-38Z
    container_name: smartoj-minio
    environment:
      MINIO_ROOT_USER: smartoj
      MINIO_ROOT_PASSWORD: your_password  # 你的 MinIO 密码
    ports:
      - "9000:9000"
      - "9001:9001"  # 控制台
    volumes:
      - minio-data:/data
    command: server /data --console-address ":9001"
    restart: unless-stopped
    networks:
      - smartoj-network
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:9000/minio/health/live"]
      interval: 30s
      timeout: 10s
      retries: 5

networks:
  smartoj-network:
    external: true
    name: smartoj-network

volumes:
  mysql-data:
  redis-data:
  rabbitmq-data:
  es-data:
  es-plugins:
  minio-data:
```

你需要做的就是编辑这个文件，将所有需要设置密码的地方都设置好并保存。

然后同样直接在 1Panel 面板中点击“容器”，然后选择“编排”，“创建编排”：

![](https://cdn.nlark.com/yuque/0/2026/png/47866636/1770457470385-9a5faf66-5bf3-4221-b7b7-b31962544419.png)

然后在对话窗口中选择刚刚编写好的docker-compose文件：

![](https://cdn.nlark.com/yuque/0/2026/png/47866636/1770457830453-068fa4f2-0a93-4f01-b7cc-f775d62c5f17.png)

然后点击“确认”即可开始编排相关的容器。

看到以下日志即代表构建成功：

![](https://cdn.nlark.com/yuque/0/2026/png/47866636/1770457902609-16f78792-c100-4943-8603-98e5440c9230.png)

![](https://cdn.nlark.com/yuque/0/2026/png/47866636/1770458760540-29b95c32-4efe-487b-8d2b-d791902a3281.png)

我们需要验证一下，MySQL 的数据迁移是否成功和 ES 的 IK 分词器是否被安装。

首先进入到 MySQL 的终端：

![](https://cdn.nlark.com/yuque/0/2026/png/47866636/1770458853418-46cf5494-71ab-41b7-bcfc-5b90eee589f0.png)

执行查看数据库命令：

```bash
show databases;
```

![](https://cdn.nlark.com/yuque/0/2026/png/47866636/1770458892214-2101c53a-c8d5-490c-a98d-998507d77de5.png)

显示 smartoj、smartojai 这两个数据库代表正常执行了迁移脚本。

然后进入 ES 的终端，进入 bin 目录，查看插件列表：

![](https://cdn.nlark.com/yuque/0/2026/png/47866636/1770458985074-d3edb95d-c6ec-4bae-8edc-80d294fbf048.png)

显示 IK 分词器代表插件安装成功了。

到这里所有的数据库和中间件都已经安装完毕！

## 后端部署
> 仓库地址：[https://github.com/SmartOnlineJudge/smartoj-backend](https://github.com/SmartOnlineJudge/smartoj-backend)。
>

### 编写配置文件
首先将 `settings.py` 文件下的开发环境换成部署状态：

```python
# 是否是开发环境
DEV_ENV: bool = False
```

复制一份配置文件：

```bash
cp metadata-docker.json metadata.json 
```

```json
{
  "metadata": {
    "databases": {
      "mysql": {
        "default": {
          "USER": "your_user",  // 账号
          "PASSWORD": "your_password",  // 密码
          "PORT": 3306,
          "HOST": "smartoj-mysql",  // smartoj数据库的地址
          "NAME": "your_db_name"  // 这里默认填写smartoj
        }
      },
      "redis": {
        "default": {
          "HOST": "smartoj-redis",
          "PORT": 6379,
          "DB": 0,
          "PASSWORD": null  // 如果有密码，则需要填写
        },
        "session": {
          "HOST": "smartoj-redis",
          "PORT": 6379,
          "DB": 1,
          "PASSWORD": null  // 如果有密码，则需要填写
        },
        "mq": {
          "HOST": "smartoj-redis",
          "PORT": 6379,
          "DB": 2,
          "PASSWORD": null  // 如果有密码，则需要填写
        }
      },
      "elasticsearch": {
        "default": {
          "URL": "http://smartoj-elasticsearch:9200",
          "USER": "elastic",  // ES 用户名
          "PASSWORD": "elastic"  // ES 密码
        }
      }
    },
    "secrets": {
      "PASSWORD": "your_secret",
      "GITHUB_OAUTH2": {
        "client_id": "your client id",
        "client_secret": "your client secret"
      }
    },
    "minio": {
      "access_key": "your_access_key",  // minio 的账号
      "secret_key": "your_secret_key",  // minio 的密码
      "endpoint": "smartoj-minio:9000",
      "secure": false
    },
    "rabbitmq": {
      "url": "amqp://guest:guest@smartoj-rabbitmq"  // 账号密码
    },
    "smtp": {
      "host": "smtp.example.com",
      "port": 587,
      "name": "智能算法刷题平台-邮件系统",
      "password": "your_password",
      "from": "no-reply@example.com"
    }
  }
}
```

> 注意！MySQL、Redis、RabbitMQ、ElasticSearch、MinIO 这几个配置的 Host 必须要填写相关的容器名字或者服务名字。
>
> 例如 MySQL 的 Host 需要填写 smartoj-mysql（容器名） 或者 mysql（服务名）。因为后端会加入这些数据库的 network 中，需要通过容器的名字来访问相关的网络进程，不能直接填写 127.0.0.1 这样的回环地址，因为后端和 MySQL 不在同一个容器中，回环地址只能访问自己容器内部的网络进程。
>

这里需要注意的其他两个配置项：

```json
"secrets": {
  "PASSWORD": "your_secret",
  "GITHUB_OAUTH2": {
    "client_id": "your client id",
    "client_secret": "your client secret"
  }
},
```

这个配置项是专门配置密钥的。

`PASSWORD`是在加密用户密码时用的密钥，可以使用一些密钥生成工具来生成一个密钥，然后使用 base64 将这个密钥编码。

而`GITHUB_OAUTH2`则是在使用 GitHub 进行 OAuth2 认证时的配置，这需要到 GitHub 上申请一个 OAuth APP，申请好以后就会得到相关的`client_id`和`client_secret`。将它们填入到配置文件中即可。

还有一个配置项是邮件配置：

```json
"smtp": {
  "host": "smtp.example.com",
  "port": 587,
  "name": "智能算法刷题平台-邮件系统",
  "password": "your_password",
  "from": "no-reply@example.com"
}
```

这里是基于 SMTP 来发送邮件的，可以在 QQ 或者 163 上面申请一个 SMTP 应用，申请好以后将`host`、`port`、`password`、`from`这 4 个配置填好就行了。

另外一个配置文件是`settings.py`。这个配置文件需要注意以下配置项：

```python
# 是否是开发环境
DEV_ENV: bool = True  # 部署时请将它设置为 False

# 请求代理地址
# 后端向 GitHub 发送请求的时候会用到
PROXY_URL = "socks5://127.0.0.1:1080"

# 代码沙箱判题接口地址
CODESANDBOX_URL = "http://127.0.0.1:8080/sandbox/judgement"
```

一个代码沙箱地址，如果你的后端和代码沙箱是部署在同一台机器上面的话，可以默认不用修改。

还有一个是`PROXY_URL`，在使用 GitHub 来进行 OAuth2 认证的时候，后端需要向 GitHub 发送请求，而通常这个过程很大概率会失败，因此发送请求的时候需要先经过一个代理，这样才会保证发送请求的成功率。

你可以通过以下的脚本来验证你的代理是否是有用的，在项目目录下编写以下代码：

```python
import httpx
from httpx_socks import SyncProxyTransport
from utils.generic import parse_proxy_url

proxy_url = "socks5://127.0.0.1:1080"  # 你的代理
proxy_config = parse_proxy_url(proxy_url)
transport = SyncProxyTransport(**proxy_config)
client = httpx.Client(transport=transport)
response = client.get("https://google.com")
print(response.text)
```

如果输出以下内容：

```plain
<HTML><HEAD><meta http-equiv="content-type" content="text/html;charset=utf-8">
<TITLE>301 Moved</TITLE></HEAD><BODY>
<H1>301 Moved</H1>
The document has moved
<A HREF="https://www.google.com/">here</A>.
</BODY></HTML>
```

则代表你的代理成功请求了 Google 的服务器并接收了响应。如果不能请求，那么则表示你的代理可能也无法正常请求 GitHub。

> 一般代理服务器都不是直接提供 socks5 这样的协议，需要先经过 Clash、SingBox 等这样的代理工具才能将协议转换成 socks5 协议。
>

### 迁移数据
编写好配置文件以后，首先需要将 MySQL 的数据迁移到 ElasticSearch 中，在控制台中执行：

```bash
docker-compose --profile migrate up es-migrate
```

看到这样的输出并且中甲没有报错即表示数据迁移成功：

![](https://cdn.nlark.com/yuque/0/2026/png/47866636/1770471421106-bcaec200-64ed-418f-8d19-f9eccc9d1f49.png)

### 启动后端
然后使用 Compose 的方式来启动后端。还是一样，点击“编排”，选择 compose 文件，然后点击创建：

![](https://cdn.nlark.com/yuque/0/2026/png/47866636/1770471584358-845f848e-d770-4395-a8db-50a6aa5a70ce.png)

### 创建默认头像
需要上传一个默认的用户头像到 MinIO 服务器上。

首先确保服务器的 9001 端口在防火墙中开放：

![](https://cdn.nlark.com/yuque/0/2025/png/47866636/1766999757146-fa2f4e8e-2a99-4f2a-946a-88c6033a2aff.png)

然后进入到 MinIO 的可视化后台中：http://106.52.180.63:9001。

输入安装 MinIO 时设置的用户名和密码，登录即可进入到 MinIO 的可视化管理界面中：

![](https://cdn.nlark.com/yuque/0/2025/png/47866636/1766999895554-490813d0-fd4d-41c2-b014-b36285ee7080.png)

我们需要为创建一个桶来专门存储用户的头像，这个桶的名称后端默认设置的是`user-avatars`。可以在`storage/oss.py`文件中修改：

![](https://cdn.nlark.com/yuque/0/2025/png/47866636/1767000104702-a84e12e7-d3ff-45ee-924a-f7c2f92da26a.png)

在 MinIO 的可视化界面中创建这个桶：

![](https://cdn.nlark.com/yuque/0/2025/png/47866636/1767000149801-b7d86ea9-0e8b-4161-9cfe-57944ab210cf.png)

![](https://cdn.nlark.com/yuque/0/2025/png/47866636/1767000165972-1e303b31-2375-4f86-9330-233a8780fc39.png)

创建好以后需要将这个桶的访问权限改成公开：

![](https://cdn.nlark.com/yuque/0/2025/png/47866636/1767000235562-358b5a5a-f3b7-4f8b-9c60-af9d9c814c8f.png)

![](https://cdn.nlark.com/yuque/0/2025/png/47866636/1767000250028-e2301d58-87c4-4675-8b6b-40de172e9d08.png)

然后上传一张默认头像到这个桶中，后端默认的头像名称是，`settings.py`文件中：

![](https://cdn.nlark.com/yuque/0/2025/png/47866636/1767000296647-23b390e9-3b8c-4b9a-9dba-6f6c5e8ad0fb.png)

![](https://cdn.nlark.com/yuque/0/2025/png/47866636/1767000329910-db06b164-e312-4505-a9d0-ea728bdcada9.png)

![](https://cdn.nlark.com/yuque/0/2025/png/47866636/1767000522754-68b86ef4-a92b-4d64-9cf8-5ff8d9585cd2.png)

桶中显示以下文件即可：

![](https://cdn.nlark.com/yuque/0/2025/png/47866636/1767000549169-22ff3df2-2fe9-42cd-afb8-3f92d7e2dff1.png)

至此，后端的部署就完成了！

## 判题沙箱部署
> 仓库地址：[https://github.com/SmartOnlineJudge/smartoj-codesandbox](https://github.com/SmartOnlineJudge/smartoj-codesandbox)。
>

判题沙箱是用来执行用户提交的代码，并将执行结果返回给后端，后端再执行相应的业务逻辑。

判题沙箱必须部署在 Docker 容器内部，这样是为了确保宿主机的安全性，因为用户很可能会提交一些恶意代码来故意破坏宿主机的完整性，而 Docker 容器可以很好地将进程与宿主机进行隔离，就算恶意代码真的被执行了，那么影响的只是容器内部的环境，不会对宿主机造成影响。

首先需要将代码下载到服务器上面，因为有了可视化的服务器运维面板，因此可以手动将代码上传到服务器上：

![](https://cdn.nlark.com/yuque/0/2025/png/47866636/1766990848767-83b21f5b-f78a-4189-b41a-b7a7eb87a629.png)

新建一个目录用于专门保存项目所有的代码，并选择“上传”按钮上传相关代码：

![](https://cdn.nlark.com/yuque/0/2025/png/47866636/1766990928718-f2a01619-d20f-449d-ba00-8381865e1938.png)

上传好项目文件以后，需要为沙箱创建一个容器。

在项目目录中已经编写好了一个 Dockerfile 文件：

![](https://cdn.nlark.com/yuque/0/2025/png/47866636/1766991209332-24705c7e-b6ad-458c-89d2-9a3c24237ae2.png)

我们需要使用这个文件来构建一个 Docker 镜像。可以使用 Docker 命令构建，也可以使用运维面板的可视化界面来构建：

![](https://cdn.nlark.com/yuque/0/2025/png/47866636/1766991436930-3c5a27d6-a19d-416c-be86-f1deeeda7c4f.png)

等待一段时间后，镜像就构建好了：

![](https://cdn.nlark.com/yuque/0/2025/png/47866636/1766991582665-33b66477-7e5d-4738-a1cd-53cd0afbbde6.png)

构建完镜像以后，还需要将镜像启动：

![](https://cdn.nlark.com/yuque/0/2026/png/47866636/1770472723237-f12d487b-c4f7-4e8a-8a3b-fbbff06981d9.png)

选择刚刚创建好的镜像，然后配置镜像的端口，两个端口都设置为8080。注意这里需要选择对应的网络，不然后续后端无法与代码沙箱通信。

按“确认”即可启动镜像：

![](https://cdn.nlark.com/yuque/0/2025/png/47866636/1766991896543-9b7d90c5-b27b-4e19-9335-a5539becc899.png)

查看容器日志，显示以下内容就表示沙箱已经启动了：

![](https://cdn.nlark.com/yuque/0/2025/png/47866636/1766992033209-de704aa9-ab13-414f-9b8a-c422583c9e2e.png)

至此，代码沙箱的部署就到此结束了。

## MCP Server 部署
> 代码仓库：[https://github.com/SmartOnlineJudge/smartoj-mcp-server](https://github.com/SmartOnlineJudge/smartoj-mcp-server)。
>

在部署 MCP 服务器之前请先确保后端已经部署好，否则 MCP 服务将无法正常启动。

MCP 服务器部署比较简单，将代码拉到服务器上面以后，需要先修改配置文件：`settings.py`。

```python
# 是否是开发模式
# 该模式会影响配置文件的加载
DEV_ENV = False
```

将这个配置改成 False。

然后在 .env 文件中，将后端的 URL 改成：

```bash
BACKEND_URL = http://smartoj-backend:8000
```

然后就是构建镜像，启动镜像。

启动镜像的时候注意端口暴露和网络的选择：

![](https://cdn.nlark.com/yuque/0/2026/png/47866636/1770473869184-cb90a435-ccc7-4900-95dd-4f6a8326dba9.png)

查看日志显示以下内容即代表部署成功：

![](https://cdn.nlark.com/yuque/0/2026/png/47866636/1770473922885-682cb0e5-e59b-4eb5-bce1-0ad9a22aa430.png)

## AI 服务部署
> 代码仓库：[https://github.com/SmartOnlineJudge/smartoj-ai-service](https://github.com/SmartOnlineJudge/smartoj-ai-service)。
>

在部署这个服务之前，需要先确保后端和 MCP 服务是正常启动的。

编写配置文件，先复制一份配置文件：

```bash
cp .env.example .env
```

然后编写 .env 文件：

```bash
# AI 服务相关配置
# 需要符合 OpenAI 接口的规范
OPENAI_API_KEY=<your api key>
OPENAI_BASE_URL=<your base url>

# Graph Node LLM 模型配置
# 题目信息管理 Agent 的每个节点对应的的 LLM 模型
QUESTION_MANAGE_DISPATCHER_MODEL=Qwen/Qwen3-8B
QUESTION_MANAGE_MEMORY_TIME_LIMIT_MODEL=deepseek-ai/DeepSeek-V3.1
QUESTION_MANAGE_TEST_MODEL=deepseek-ai/DeepSeek-V3.1
QUESTION_MANAGE_SOLVING_FRAMEWORK_MODEL=deepseek-ai/DeepSeek-V3.1
QUESTION_MANAGE_JUDGE_TEMPLATE_FOR_PYTHON_MODEL=deepseek-ai/DeepSeek-V3.1
QUESTION_MANAGE_PLANNER_MODEL=Qwen/Qwen3-8B
QUESTION_MANAGE_DATA_PREHEAT_MODEL=deepseek-ai/DeepSeek-V3.1
# 通用 Agent LLM 配置
GENERIC_JSON_PARSER_MODEL=deepseek-ai/DeepSeek-V3.1
GENERIC_CHAT_TITLE_GENERATOR_MODEL=qwen3-30b-a3b
# 智能刷题助手 LLM 配置
SOLVING_ASSISTANT_MODEL=deepseek-ai/DeepSeek-V3.1
# 个性化记忆 LLM 配置
PERSONALIZED_MEMORY_MODEL=deepseek-ai/DeepSeek-V3.1

# MCP 连接配置（注意是Docker环境）
MCP_SERVER_URL=http://smartoj-mcp-server:9002/mcp

# 后端接口地址（注意是Docker环境）
BACKEND_URL=http://smartoj-backend:8000

# MySQL URI（注意是Docker环境）
DATABASE_URI=mysql://root:password@smartoj-mysql:3306/smartojai

```

模型的 API 需要符合 OpenAI 格式。

对于模型配置方面，以`QUESTION_MANAGE`开头的全部模型都推荐使用 Qwen3-Max。因为在开发的时候，这个模型是最稳定的，输出很符合预期结果，但是其他模型就不一定了。而以`GENERIC`开头的模型推荐使用参数两较小的模型即可，因为任务足够简单。而剩下的智能刷题助手、个性化记忆这两个用什么模型基本上问题不大。

MCP、后端、MySQL 的地址在之前都已经部署好了，将它们的信息填在配置文件中即可（注意 Host 即可）。

然后就是构建镜像，启动镜像。

启动镜像的时候注意端口暴露和网络的选择：

![](https://cdn.nlark.com/yuque/0/2026/png/47866636/1770474569658-af5e1760-21e4-423a-bcaf-6419fd6f207b.png)

首次运行该项目的时候，程序会先创建 LangGraph 相关的数据库表结构，因此速度可能会比较慢一些。

![](https://cdn.nlark.com/yuque/0/2026/png/47866636/1770474633038-1a09cb31-6561-4f4c-bb1c-ad9ff2e56ab4.png)

看到上面的输出就代表进程可以被正常启动。

至此，所有的后台接口都已经部署完毕了。

而后续的前端、Nginx、HTTPS等等流程都和手动部署时的一致，所以这里就不再赘述了。

