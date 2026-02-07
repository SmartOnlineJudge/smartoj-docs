# 传统部署方案

## 部署概念

> 部署‍是将一个开发‍完成的软件或应用程‍序从开发环境‍转移到生产‍环境（即最终运行环‍境）的过程。部署‍的目标是让‍应用程序能‍够稳定、高效地为用‍户或客户提供服务。‍部署不仅仅是将代‍码上传到服务器那么‍简单，还涉‍及到环境配置、‍依赖管理、安全性设置‍、性能优化等一‍系列操作‍。
>

这里的“部署”指的是在公网环境下部署。

在开发阶段，代码通常是运行在本地的主机上面，本地运行的方式也叫部署。例如前端运行一个 Vue3 项目：`npm run dev`，后端运行一个 FastAPI 的项目：`uvicorn main:app`。这些命令都会在本地主机上启动一个 Web 服务器并监听相应的端口来接收请求，这些命令适用于与在开发环境部署项目。

而要在生产环境中部署一个项目，目的同样是启动一个 Web 服务器并监听相应的端口来处理请求，只不过很大程度上不会直接使用这些命令来启动项目。如果直接使用这些命令来在生产环境部署相应的项目，那么可能会造成以下问题：

1. 进程管理困难

   当直接使用这些命令来启动项目的时候，如果将终端关闭了，那么很有可能这个进程也会被关闭，即使没有被关闭，那么后续也很难监控到这些进程的状态、日志等等信息。并且如果一个进程挂了，那么它也不会自动重启。

2. 资源占用高

   直接在前台运行服务时，进程的资源使用缺乏自动监控和调控机制。当流量突增时，服务容易因资源耗尽（如内存、CPU）而崩溃；相反，在低负载时也无法自动释放资源。同时，无法实现零停机的滚动更新或多实例负载均衡。

3. 安全性风险

   直接以命令行启动时，服务通常以当前用户权限运行，可能存在权限过高的问题。此外，配置（如端口、密钥）常以明文参数传递，易泄露敏感信息。生产环境需要隔离的运行环境（如容器或沙箱）和安全的配置管理机制。

4. 运维复杂度高

   缺少服务发现、健康检查、集中化日志收集等基础设施。当服务增多时，手动管理端口冲突、依赖启动顺序等会变得极其困难，且难以与现有监控告警系统集成。

因此在部署阶段会委托专门的进程管理工具来启动这些项目，例如 Docker、Supervisor、Gunicorn 等等。并且还会使用 Nginx 来配置反向代理与负载均衡来实现高可用性。

## 项目架构
### 代码仓库
1. 客户端代码仓库：[https://github.com/SmartOnlineJudge/smartoj-client](https://github.com/SmartOnlineJudge/smartoj-client)
2. 后台管理系统代码仓库：[https://github.com/SmartOnlineJudge/smartoj-management](https://github.com/SmartOnlineJudge/smartoj-management)
3. 后端代码仓库：[https://github.com/SmartOnlineJudge/smartoj-backend](https://github.com/SmartOnlineJudge/smartoj-backend)
4. AI 服务代码仓库：[https://github.com/SmartOnlineJudge/smartoj-ai-service](https://github.com/SmartOnlineJudge/smartoj-ai-service)
5. 判题沙箱代码仓库：[https://github.com/SmartOnlineJudge/smartoj-codesandbox](https://github.com/SmartOnlineJudge/smartoj-codesandbox)
6. MCP Server 代码仓库：[https://github.com/SmartOnlineJudge/smartoj-mcp-server](https://github.com/SmartOnlineJudge/smartoj-mcp-server)

部署顺序：判题沙箱 -> 后端 ->  MCP Server  -> AI 服务 -> 后台管理系统 -> 客户端。

### 架构图
![](https://cdn.nlark.com/yuque/0/2025/png/47866636/1766987675861-0ec3fe2a-cba9-48cd-9043-0d281d5dfc06.png)

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

![](https://cdn.nlark.com/yuque/0/2025/png/47866636/1766991822713-bc28719c-4b62-48ef-9628-5ed30ea68cd3.png)

选择刚刚创建好的镜像，然后配置镜像的端口，这里端口两个都设置为8080。

按“确认”即可启动镜像：

![](https://cdn.nlark.com/yuque/0/2025/png/47866636/1766991896543-9b7d90c5-b27b-4e19-9335-a5539becc899.png)

查看容器日志，显示以下内容就表示沙箱已经启动了：

![](https://cdn.nlark.com/yuque/0/2025/png/47866636/1766992033209-de704aa9-ab13-414f-9b8a-c422583c9e2e.png)

至此，代码沙箱的部署就到此结束了。

## 数据库及中间件安装
> 在部署后端之前，需要先将相关后端依赖的数据库、中间件安装好，这样才能够启动后端。
>

### MySQL
MySQL 是项目使用的主要数据库，用于保存用户、题目、题解、讨论等等核心数据，是整个项目的核心业务数据库。

在开发阶段使用的 MySQL 版本是`8.0.43`，并且版本**必须大于等于 8.0**，因为项目中的依赖需要用到 MySQL 8.0 的特性。

由于服务器自带了 Docker，因此这里也选择使用 Docker 来一键安装 MySQL。

在应用商店中选择 MySQL 并点击安装：

![](https://cdn.nlark.com/yuque/0/2025/png/47866636/1766992656065-f981835e-ff61-420a-a5e1-1e6d7e975287.png)

然后选择 8.0.43 的版本，设置好密码，并允许“端口外部访问”，然后点击“确认”即可完成安装：

![](https://cdn.nlark.com/yuque/0/2025/png/47866636/1766992815059-4336e023-aaa2-4362-9544-457c8823c988.png)

点击“容器”一栏即可看到 MySQL 已经启动：

![](https://cdn.nlark.com/yuque/0/2025/png/47866636/1766993004084-8f0cb473-240f-4e4c-aa0a-6b40e9493cf1.png)

到这里还没完，在项目中使用了 Binlog 来监听部分表数据的变化，并根据数据的变化来执行相应的业务逻辑。因此还需要开启 MySQL 的 Binlog。

首先我们需要找到 MySQL 的配置文件，如果你是完全按照上述的操作来进行的话，那么 MySQL 的配置一般会存放在宿主机的`/opt/1panel/apps/mysql/mysql/conf`目录下：

![](https://cdn.nlark.com/yuque/0/2025/png/47866636/1766993467077-9dccb195-33e9-4f47-a3fc-7d60ee63ee3a.png)

> 为什么 MySQL 被安装在容器内部而配置文件却在宿主机中呢？
>
> 答：因为在使用 Docker 来安装 MySQL 的时候，1Panel 面板会默认将数据挂载到宿主机的本地文件目录中，而容器内部会自动链接宿主机中的文件。这就是 Docker 中“volume”的功能。这样做的好处是：当容器被关闭了，容器内部保存的文件也不会被丢失。
>

双击打开配置文件，并在配置文件中追加以下内容：

```plain
# 开启 binlog
log_bin = mysql-bin
binlog_format = ROW
server_id = 1
binlog_row_metadata = FULL
# 清理旧日志文件
binlog_expire_logs_seconds = 604800
```

然后在容器面板中重启 MySQL：

![](https://cdn.nlark.com/yuque/0/2025/png/47866636/1766993686205-4ae0aa7c-c2fa-4d3a-8d82-6a6ce7c73d2d.png)

重启以后进入 MySQL 的终端：

![](https://cdn.nlark.com/yuque/0/2025/png/47866636/1766993896998-da245489-6942-4067-87a7-5bff274bd223.png)

直接点击连接，然后在命令行中输入以下命令，然后按回车接着输入数据库的密码：

```bash
mysql -u root -p
```

![](https://cdn.nlark.com/yuque/0/2025/png/47866636/1766993982480-17d751ef-68a9-423f-8b87-4f4aa36dffae.png)

显示这样就代表已经进入到了 MySQL 的命令行交互界面中了，我们需要验证刚刚的 Binlog 是否被正常开启。

输入以下命令：

```bash
SHOW VARIABLES LIKE 'log_bin';
```

显示以下内容即表示 Binlog 开启成功：

![](https://cdn.nlark.com/yuque/0/2025/png/47866636/1766994066602-a376ab85-cad8-4cc3-8c83-4460dbea0886.png)

这一步一定要做完，不然后端项目是跑不起来的。

到此为止，MySQL 的安装已经结束了。

### Redis
Redis 在项目中用于保存用户的登录状态、RabbitMQ 的存储后端以及部分后端缓存数据。

开发项目时使用的版本是`8.0.3`。安装比较简单，像安装 MySQL 那样在应用商店中选择相应的版本安装即可：

![](https://cdn.nlark.com/yuque/0/2025/png/47866636/1766994501357-52a9c4fe-4677-430c-be18-3666bd388ac1.png)

![](https://cdn.nlark.com/yuque/0/2025/png/47866636/1766994557214-f1886571-abac-4deb-b3e7-07080d6b7429.png)

到这里 Redis 的安装就已经完成了，不需要做其他额外的配置。

### ElasticSearch
该数据库主要用于实现题目的全文检索，数据来源与 MySQL，后端通过监听 Binlog 来实现将新数据同步到 ES 中。

在开发项目时使用的版本是`8.18.4`，同样先到到应用商店中安装ES。

> 注意：ElasticSearch的内存占用较高，请确保当前服务器的空闲内存至少 1GB。
>

![](https://cdn.nlark.com/yuque/0/2025/png/47866636/1766995187497-e938dabe-9a76-44a9-9b98-4218210cfe35.png)

![](https://cdn.nlark.com/yuque/0/2025/png/47866636/1766995330887-43b19742-9e98-42d2-a999-ae772668631b.png)

因为 ElasticSearch 是使用 Java 来实现的，而 JVM 本身的内存占用是很高的，因此 ES 的内存占用也同样会变得很高，如果服务器的内存实在不足了，可以考虑使用云ES，也就是各大云厂商（阿里云、腾讯云等等）提供的ES服务，这样可以避免因为服务器内存不足而导致的无法安装。

到这里还没有结束，项目中题目的描述一般是中文，但是 ES 在进行全文检索的时候，默认使用英文的方式来对文本进行分词，这样会大大降低检索的准确率，因此这里还需要为 ES 添加一个中文分词器插件：IK 分词器。

> 插件官方文档：[https://github.com/infinilabs/analysis-ik](https://github.com/infinilabs/analysis-ik)
>
> 插件下载链接：[https://release.infinilabs.com/analysis-ik/stable/](https://release.infinilabs.com/analysis-ik/stable/)
>

在插件列表页面找到和刚刚 ES 的安装版本一模一样的插件版本，必须一模一样，刚刚安装的 ES 版本是`8.18.4`，那么插件的版本也必须一模一样：

![](https://cdn.nlark.com/yuque/0/2025/png/47866636/1766995693503-86dff01b-9cfc-43d9-a703-fcd5454d7bbb.png)

下载以后会得到一个压缩包，我们需要将这个压缩包解压到 ES 的插件目录中。

> IK 插件安装参考教程：[https://developer.aliyun.com/article/1589546](https://developer.aliyun.com/article/1589546)
>

如果你是直接使用 1Panel 来安装 ES 的，那么插件的目录一般是宿主机上的：`/opt/1panel/apps/elasticsearch/elasticsearch/data/plugins`目录。如果没有`plugins`目录那需要手动创建一个。

然后进入到`plugins`目录中，再创建一个`ik`目录，将刚刚下载好的 IK 分词器压缩包解压到该目录下：

![](https://cdn.nlark.com/yuque/0/2025/png/47866636/1766996716411-4b1965f5-53b9-4f35-8f39-f1069a3d1abc.png)

最终的目录结构应该是如上图的样子。

> 注意：`/opt/1panel/apps/elasticsearch/elasticsearch/data/plugins`目录下不能包含无效的插件目录，否则 ES 将会启动失败。在本次安装过程中，plugins 目录下应该只包含 ik 目录。
>

先检查`plugins`目录有没有被挂载到容器中：

![](https://cdn.nlark.com/yuque/0/2025/png/47866636/1766998374645-24f2faad-0b6b-421f-9d04-e3a2b81a804a.png)

如果没有，那么需要先将`plugins`的目录挂载到 ES 容器内部的`plugins`目录中，如上图所示。

接着重启 ES：

![](https://cdn.nlark.com/yuque/0/2025/png/47866636/1766996858409-2e695f82-5987-4fef-8f26-1e52f46c88e0.png)

然后进入到 ES 容器的终端，进入到 bin 目录下，输入：

```bash
elasticsearch-plugin list
```

显示以下内容即代表 IK 分词器安装成功：

![](https://cdn.nlark.com/yuque/0/2025/png/47866636/1766998519204-3b524399-5575-4bc7-a44b-332758caad12.png)

至此，ES 的安装就结束了。

### RabbitMQ
该中间件主要用于后端的消息队列服务，例如邮件发送、数据同步、调用判题沙箱等等任务。

项目开发时使用的版本是`4.0.7`，直接在应用商店安装即可：

![](https://cdn.nlark.com/yuque/0/2025/png/47866636/1766998934135-3d29e037-ad19-4460-952b-bd93813e8f43.png)

### MinIO
用于保存用户头像、题解图片等二进制文件，

项目开发时使用的版本是`2025-02-18`，直接在应用商店安装即可。

到这一步结束，运维面板展示的容器应该包括以下：

![](https://cdn.nlark.com/yuque/0/2025/png/47866636/1766999382354-664be147-23d0-43a2-9e5c-7572c51714a9.png)

如果和上图一样，那么就可以进行下一步了。

## 后端部署
> 仓库地址：[https://github.com/SmartOnlineJudge/smartoj-backend](https://github.com/SmartOnlineJudge/smartoj-backend)。
>

### 后端依赖安装
后端使用的 Python 版本必须大于等于 3.11，否则将不能正常启动。

项目使用 uv 来管理 Python 的第三方库，因此需要先将 uv 安装好。

```bash
curl -LsSf https://astral.sh/uv/install.sh | sh
```

先使用 uv 安装好 Python3.11：

```bash
uv python install 3.11
```

同时安装好后端项目依赖：

```bash
uv sync
```

到这一步先不要启动后端，因为还有一些配置和数据需要编写和导入。

### 配置文件编写
在项目目录下有一个`metadata.json`文件：

```json
{
  "metadata": {
    "databases": {
      "mysql": {
        "default": {
          "USER": "your_user",  // 账号
          "PASSWORD": "your_password",  // 密码
          "PORT": 3306,
          "HOST": "127.0.0.1",  // smartoj数据库的地址
          "NAME": "your_db_name"  // 这里默认填写smartoj
        }
      },
      "redis": {
        "default": {
          "HOST": "127.0.0.1",
          "PORT": 6379,
          "DB": 0,
          "PASSWORD": null  // 如果有密码，则需要填写
        },
        "session": {
          "HOST": "127.0.0.1",
          "PORT": 6379,
          "DB": 1,
          "PASSWORD": null  // 如果有密码，则需要填写
        },
        "mq": {
          "HOST": "127.0.0.1",
          "PORT": 6379,
          "DB": 2,
          "PASSWORD": null  // 如果有密码，则需要填写
        }
      },
      "elasticsearch": {
        "default": {
          "URL": "http://127.0.0.1:9200",
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
      "endpoint": "127.0.0.1:9000",
      "secure": false
    },
    "rabbitmq": {
      "url": "amqp://guest:guest@127.0.0.1"  // 账号密码
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

在上述配置文件中，MySQL、Redis、ElasticSearch、RabbitMQ、MinIO等数据库的配置应该在安装的时候就已经记好了，直接填到配置文件里面即可。

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

### 基本数据准备
#### 用户默认头像上传
首先需要上传一个默认的用户头像到 MinIO 服务器上。

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

#### MySQL 数据库创建与数据上传
这里的数据库指的是 MySQL，在项目中一共使用到两个数据库：

1. smartoj：后端基本业务数据库，保存后端基本业务的信息，例如用户、题目等等。
2. smartojai：AI 服务数据库，保存和 AI 的对话数据。

因此需要先在 MySQL 中创建这两个数据库。在创建数据库的时候，请确保数据库的编码格式为`utf8mb4`，排序方式为`utf8mb4_0900_ai_ci`：

```sql
CREATE DATABASE smartoj
  DEFAULT COLLATE SET = 'utf8mb4_0900_ai_ci'
  DEFAULT CHARACTER SET = 'utf8mb4';

CREATE DATABASE smartojai
  DEFAULT COLLATE SET = 'utf8mb4_0900_ai_ci'
  DEFAULT CHARACTER SET = 'utf8mb4';
```

创建好数据库以后，在后端代码仓库中的项目目录下会有一个`smartoj.sql`的文件，我们需要将这个数据备份到服务器上的 MySQL 中。这个备份可以直接使用命令来导入：

```sql
mysql -h 106.52.180.63 -P 3306 -u root -p 'password' smartoj < '/path/to/smartoj.sql'
```

也可以使用一些可视化的数据库管理工具来操作，例如 Navicat、DataGrid 等等。只要将数据导入到数据库中即可。

此外，还需要为`smartojai`这个数据库创建两张表：

```sql
CREATE TABLE conversations (
    id INT AUTO_INCREMENT PRIMARY KEY,
    title VARCHAR(50) NOT NULL,
    created_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
    is_deleted BOOLEAN NOT NULL DEFAULT FALSE,
    user_id VARCHAR(13) NOT NULL,
    question_id INT DEFAULT NULL,
    thread_id VARCHAR(128) NOT NULL UNIQUE,
    INDEX idx_user_id_is_deleted_question_id (user_id, is_deleted, question_id),
    INDEX idx_updated_at (updated_at)
)

CREATE TABLE memories (
    id INT AUTO_INCREMENT PRIMARY KEY,
    user_id VARCHAR(13) NOT NULL,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
    content VARCHAR(50) NOT NULL,
    type ENUM("level", "ability", "preference"),
    INDEX idx_user_id (user_id)
)
```

#### ElasticSearch 数据迁移
ES 中的数据需要从 MySQL 中迁移部分数据过来，其中涉及到的表结构包括：question、tag、question_tag。为了实现全文检索，需要将这些数据从 MySQL 中迁移过来。

在项目目录下的`scripts`中，有一个脚本：`create_es_data.py`。

我们需要以模块的方式来运行这个脚本：

```bash
uv run python3 -m scripts.create_es_data
```

这会读取项目的配置文件，连接 MySQL 和 ES，并将 MySQL 中的部分数据迁移到 ES 中。

如果显示以下日志，那么则代表数据迁移成功：

```plain
创建 question 索引成功！
{'_index': 'question', '_id': '1', '_version': 1, 'result': 'created', '_shards': {'total': 2, 'successful': 1, 'failed': 0}, '_seq_no': 0, '_primary_term': 1}
{'_index': 'question', '_id': '373', '_version': 1, 'result': 'created', '_shards': {'total': 2, 'successful': 1, 'failed': 0}, '_seq_no': 1, '_primary_term': 1}
{'_index': 'question', '_id': '374', '_version': 1, 'result': 'created', '_shards': {'total': 2, 'successful': 1, 'failed': 0}, '_seq_no': 2, '_primary_term': 1}
{'_index': 'question', '_id': '375', '_version': 1, 'result': 'created', '_shards': {'total': 2, 'successful': 1, 'failed': 0}, '_seq_no': 3, '_primary_term': 1}
{'_index': 'question', '_id': '376', '_version': 1, 'result': 'created', '_shards': {'total': 2, 'successful': 1, 'failed': 0}, '_seq_no': 4, '_primary_term': 1}
more...
```

#### Redis 保存 MySQL 当前的 Binlog
刚刚通过手动执行迁移脚本的方式将 MySQL 的数据迁移到了 ES 中，但是以后  MySQL 每变更一次数据，都需要执行一次迁移脚本吗？

这样做也不是不行，只是这样做的效率太低了。我们完全可以通过 MySQL 的 Binlog 来增量式的同步数据。所以我们需要将 MySQL 现在的 Binlog 位置保存起来，从当前 Binlog 位置开始，如果监听到 MySQL 有数据变更或者数据增加，就可以只从当前位置同步数据，而不是一次性将数据全部同步。

因此在这一步我们需要将 Binlog 的位置保存在 Redis 中。

先在 MySQL 的命令行中查看当前 Binlog 的位置：

```sql
SHOW MASTER STATUS;
```

这将会输出以下内容：

![](https://cdn.nlark.com/yuque/0/2025/png/47866636/1767010160908-1cc9c059-a88e-47c3-8e78-81c5b14af2ba.png)

我们需要将 File、Position 保存起来。在项目目录下，有一个`scripts/listen_mysql_binlog.py`文件，文件中有一个函数：`save_binlog_position`。

我们需要调用一遍这个函数，将当前 Binlog 保存在 Redis 中：

```python
import asyncio
from scripts.listen_mysql_binlog import save_binlog_position, load_binlog_position

async def main():
    await save_binlog_position("mysql-bin.000001", 186310)
    print(await load_binlog_position())

asyncio.run(main())
```

如果输出了刚刚保存的 Binlog，那么这一步的任务就完成了！

### 部署后端程序
前面的数据库安装、中间件安装、数据迁移等等步骤都是为了后端部署做的铺垫。现在，终于可以真正的启动后端程序了：

```bash
uv run main.py
```

当输出以下内容的时候，就代表后端成功启动了：

```bash
INFO:     2025-12-29 20:20:39,822 - Started server process [99435]
INFO:     2025-12-29 20:20:39,822 - Waiting for application startup.
INFO:     2025-12-29 20:20:39,822 - Creating RabbitMQ connection
INFO:     2025-12-29 20:20:39,839 - Application startup complete.
INFO:     2025-12-29 20:20:39,849 - Uvicorn running on http://0.0.0.0:8000 (Press CTRL+C to quit)
```

如果你的服务器已经打开了 8000 这个端口，那么你可以在浏览器中试图访问后端的 API 接口文档。

在浏览器中输入：服务器地址 + :8000/docs。

![](https://cdn.nlark.com/yuque/0/2025/png/47866636/1767010998313-d1ff2f97-7882-417f-80a6-de6e1aa7e1c7.png)

除了启动后端主程序以外，还有 2 个程序需要启动：

1. 消息队列

```bash
uv run taskiq worker mq.broker:broker --workers 2
```

2. Binlog 监听程序

```bash
uv run -m scripts.listen_mysql_binlog
```

但是我们不应该手动运行这些启动命令，而是让进程管理器来帮我们运行。

这里使用 Supervisor 来管理这些进程。先安装 Supervisor：

```bash
sudo apt-get install supervisor
sudo systemctl enable supervisor.service  # 开机自动启动
```

在`/etc/supervisor/conf.d/`目录下编写 3 个配置文件：

1. backend.conf

```bash
[program:backend]
command=/path/to/smartoj-backend/.venv/bin/uvicorn main:app --host 127.0.0.1 --log-config /path/to/smartoj-backend/log-config.json
directory=/path/to/smartoj-backend
user=root
autostart=true
autorestart=true
stderr_logfile=/var/log/smartoj/backend.err.log  # 先确保目录存在
stdout_logfile=/var/log/smartoj/backend.out.log  # 先确保目录存在
logfile_maxbytes=5MB
```

2. taskiq.conf

```bash
[program:taskiq]
command=/path/to/smartoj-backend/.venv/bin/taskiq worker mq.broker:broker
directory=/path/to/smartoj-backend
user=root
autostart=true
autorestart=true
stderr_logfile=/var/log/smartoj/taskiq.err.log  # 先确保目录存在
stdout_logfile=/var/log/smartoj/taskiq.out.log  # 先确保目录存在
logfile_maxbytes=5MB
```

3. binlog.conf

```bash
[program:binlog]
command=/path/to/smartoj-backend/.venv/bin/python3 -m scripts.listen_mysql_binlog
directory=/path/to/smartoj-backend
user=root
autostart=true
autorestart=true
stderr_logfile=/var/log/smartoj/binlog.err.log  # 先确保目录存在
stdout_logfile=/var/log/smartoj/binlog.out.log  # 先确保目录存在
logfile_maxbytes=5MB
```

上述 3 个文件分别是后端、消息队列、MySQL Binlog 监听进程的 Supervisor 进程配置文件。下面就是将这些配置文件应用到 Supervisor 中，让 Supervisor 来管理这些进程。

```bash
sudo supervisorctl reread  # 重新加载配置文件
sudo supervisorctl update  # 更新进程组
```

接着这些进程会被自动启动，可以通过 Supervisor 来查看这些进程的状态：

```bash
> sudo supervisorctl status
backend                          RUNNING   pid 107005, uptime 0:01:23
binlog                           RUNNING   pid 107338, uptime 0:00:04
taskiq                           RUNNING   pid 107339, uptime 0:00:04
```

分别查看这 3 个进程的日志：

```bash
> cat backend.err.log
INFO:     2025-12-29 20:52:03,524 - Started server process [107005]
INFO:     2025-12-29 20:52:03,524 - Waiting for application startup.
INFO:     2025-12-29 20:52:03,524 - Creating RabbitMQ connection
INFO:     2025-12-29 20:52:03,535 - Application startup complete.
INFO:     2025-12-29 20:52:03,537 - Uvicorn running on http://127.0.0.1:8000 (Press CTRL+C to quit)
```

```bash
> cat taskiq.err.log
[2025-12-29 20:53:22,654][taskiq.worker][INFO   ][MainProcess] Pid of a main process: 107339
[2025-12-29 20:53:22,654][taskiq.worker][INFO   ][MainProcess] Starting 2 worker processes.
[2025-12-29 20:53:22,659][taskiq.process-manager][INFO   ][MainProcess] Started process worker-0 with pid 107341
[2025-12-29 20:53:22,661][taskiq.process-manager][INFO   ][MainProcess] Started process worker-1 with pid 107343
[2025-12-29 20:53:23,743][taskiq.receiver.receiver][INFO   ][worker-1] Listening started.
[2025-12-29 20:53:23,874][taskiq.receiver.receiver][INFO   ][worker-0] Listening started.
```

```bash
> cat binlog.err.log
2025-12-29 20:53:23,903 - INFO - 启动消息队列……
2025-12-29 20:53:23,911 - INFO - 正在创建子线程……
2025-12-29 20:53:23,911 - INFO - 连接到 MySQL 服务器……
2025-12-29 20:53:23,911 - INFO - connection_settings: {'host': '127.0.0.1', 'port': 3306, 'user': 'root', 'passwd': 'smartoj20250118', 'charset': 'utf8'}
2025-12-29 20:53:23,911 - INFO - resume_stream: True
2025-12-29 20:53:23,911 - INFO - blocking: True
2025-12-29 20:53:23,911 - INFO - only_tables: ['question', 'question_tag', 'tag', 'comment']
2025-12-29 20:53:23,911 - INFO - only_schemas: ['smartoj']
2025-12-29 20:53:23,911 - INFO - enable_logging: True
2025-12-29 20:53:23,911 - INFO - allowed_events_in_packet: frozenset({<class 'pymysqlreplication.row_event.DeleteRowsEvent'>, <class 'pymysqlreplication.event.RotateEvent'>, <class 'pymysqlreplication.row_event.TableMapEvent'>, <class 'pymysqlreplication.row_event.UpdateRowsEvent'>, <class 'pymysqlreplication.row_event.WriteRowsEvent'>})
2025-12-29 20:53:23,912 - INFO - server_id: 101
2025-12-29 20:53:23,912 - INFO - log_pos: 186310
2025-12-29 20:53:23,912 - INFO - log_file: mysql-bin.000001
2025-12-29 20:53:23,912 - INFO - 开始监听 binlog……
```

与上述日志显示的内容基本一致，则代表整个后端进程启动成功！

> 后端服务应该说是这个项目中最复杂的一部分了，服务多、进程也多。能顺利走到这一步实属不容易🤗。
>

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

将这个配置改成 False 即可。

如果你的后端与 MCP 服务部署在同一台主机上面，那么可以不用修改 .env 配置中的后端地址。

接着安装项目依赖：

```bash
uv sync
```

安装好以后尝试手动启动项目：

```bash
uv run main.py
```

如果 MinIO 和 MCP 服务在同一台主机上，那么当你直接运行上面的命令会报端口已经被占用的错误。所以需要将启动函数的端口修改成其他端口，例如 9002。

输出以下界面即可表示 MCP Server 启动成功：

![](https://cdn.nlark.com/yuque/0/2025/png/47866636/1767014083038-a30853fc-0671-4c0a-838a-48af978ff46c.png)

接着编写一个 Supervisor 配置，让它帮我们管理这个进程：

```bash
[program:mcp]
command=/path/to/smartoj-mcp-server/.venv/bin/python3 main.py
directory=/path/to/smartoj-mcp-server
user=root
autostart=true
autorestart=true
stderr_logfile=/var/log/smartoj/mcp.err.log  # 先确保目录存在
stdout_logfile=/var/log/smartoj/mcp.out.log  # 先确保目录存在
logfile_maxbytes=5MB
```

更新并启动进程：

```bash
sudo supervisorctl reread
sudo supervisorctl update
```

查看启动日志，和手动启动输出的日志一致即代表 MCP 服务部署成功！

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

# MCP 连接配置
MCP_SERVER_URL=http://127.0.0.1:9000/mcp

# 后端接口地址
BACKEND_URL=http://127.0.0.1:8000

# MySQL URI
DATABASE_URI=mysql://root:password@localhost:3306/smartojai

```

模型的 API 需要符合 OpenAI 格式。

对于模型配置方面，以`QUESTION_MANAGE`开头的全部模型都推荐使用 Qwen3-Max。因为在开发的时候，这个模型是最稳定的，输出很符合预期结果，但是其他模型就不一定了。而以`GENERIC`开头的模型推荐使用参数两较小的模型即可，因为任务足够简单。而剩下的智能刷题助手、个性化记忆这两个用什么模型基本上问题不大。

MCP、后端、MySQL 的地址在之前都已经部署好了，将它们的信息填在配置文件中即可。

接下来安装项目依赖：

```bash
uv sync
```

手动启动项目：

```bash
> uv run main.py
INFO:     2025-12-29 21:31:36,829 - Started server process [115855]
INFO:     2025-12-29 21:31:36,829 - Waiting for application startup.
/path/to/smartoj/smartoj-ai-service/.venv/lib/python3.11/site-packages/aiomysql/cursors.py:458: Warning: Table 'checkpoint_migrations' already exists
  await self._do_get_result()
INFO:     2025-12-29 21:31:36,855 - Application startup complete.
INFO:     2025-12-29 21:31:36,855 - Uvicorn running on http://0.0.0.0:8001 (Press CTRL+C to quit)
```

首次运行该项目的时候，程序会先创建 LangGraph 相关的数据库表结构，因此速度可能会比较慢一些。

看到上面的输出就代表进程可以被正常启动。

接着编写一个 Supervisor 的配置文件：

```bash
[program:ai-service]
command=/path/to/smartoj-ai-service/.venv/bin/python3 main.py
directory=/path/to/smartoj-ai-service
user=root
autostart=true
autorestart=true
stderr_logfile=/var/log/smartoj/ai-service.err.log  # 先确保目录存在
stdout_logfile=/var/log/smartoj/ai-service.out.log  # 先确保目录存在
logfile_maxbytes=5MB
```

启动进程：

```bash
sudo supervisorctl reread
sudo supervisorctl update
```

查看启动日志，与手动启动的日志相同，即代表部署成功！

至此，所有的后台接口都已经部署完毕了。下面将部署的是前端界面。

## 前端部署
### 安装 Nginx
它是一个高性能的 Web 服务器，用于反向代理与负载均衡，在这它主要用于代理访问前端页面，以及让前端通过反向代理的方式来访问后端接口，避免前端发生跨域问题。

通过 Linux 包管理器来安装 Nginx。

先更新软件包，并升级它们：

```bash
sudo apt update
sudo apt upgrade -y
```

安装必要依赖：

```bash
sudo apt install -y curl gnupg2 ca-certificates lsb-release
```

安装 Nginx：

```bash
sudo apt install -y nginx
```

启动 Nginx 并设置开机启动：

```bash
sudo systemctl start nginx
sudo systemctl enable nginx
```

然后在浏览器上直接通过公网 IP 访问服务器会显示 Nginx 返回的内容：

![](https://cdn.nlark.com/yuque/0/2025/png/47866636/1767016398406-e5ac6b6a-5b28-4f83-a916-dd2e6dc1906a.png)

这代表 Nginx 已经安装成功。

### 后台管理系统部署
> 代码仓库：[https://github.com/SmartOnlineJudge/smartoj-management](https://github.com/SmartOnlineJudge/smartoj-management)。
>

可以不将代码下载到服务器上面，因为这样还需要在服务上安装 NodeJS，可以将代码先下载到一台有 NodeJS 的主机上。

这里需要确认一下 NodeJS 的版本，在开发阶段使用的 NodeJS 版本是：`v22.2.0`。当然如果不是这个版本也可能可以，但如果中间出问题了一般都是 NodeJS 版本导致的。

将代码下载完毕以后，需要先编写配置文件 .env.production：

```bash
# 后端接口地址
VITE_BACKEND_URL = http://127.0.0.1:8000

# AI服务后端接口地址
VITE_AI_SERVICE_BACKEND_URL = http://127.0.0.1:8001

# MinIO 地址
VITE_MINIO_URL = http://127.0.0.1:9000
```

后端地址和 AI 服务地址按要求填就可以了，如果后端和 AI 服务和前端是在同一台主机上面，那配置就不用改。

注意！MinIO 地址不能直接填写像 127.0.0.1 这样的回环地址，因为这个地址是直接放在 HTML 的标签中的， HTML 加载图片的时候是直接按照这个 IP 来加载的，如果使用回环地址，那么请求是到不了服务器上面的，如果这样设置，请求是发送到浏览器对应的主机上面。所以这个配置一定要填写服务器的公网 IP，并在防火墙中打开 9000 这个端口。

填写好配置文件以后，执行打包命令：

```bash
npm run build
```

将打包好的目录`dist`上传到服务上。

然后在 Nginx 的配置文件中编写访问前端文件的代理配置：

> Nginx 的配置文件一般存放在 /etc/nginx 目录下的 nginx.conf 文件。
>

```bash
server {
  listen 5173;
  server_name 106.52.180.63;  # 替换为你的域名或 IP
  
  access_log /var/log/smartoj/smartoj-management-access.log;
  error_log /var/log/smartoj/smartoj-management-error.log;

  # 处理前端路由（如 Vue/React 的 history 模式）
  location / {
    alias /path/to/dist/;  # Nginx 应该有权限访问这个目录
    try_files $uri $uri/ /index.html;
  }

  # 反向代理 API 请求到后端
  location /api {
    rewrite ^/api/(.*) /$1 break;
    proxy_pass http://127.0.0.1:8000;  # 后端地址
    proxy_set_header Host $host;
    proxy_set_header X-Real-IP $remote_addr;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    proxy_set_header X-Forwarded-Proto $scheme;
  }

  location /ai-service {
    rewrite ^/ai-service/(.*) /$1 break;
    proxy_pass http://127.0.0.1:8001;  # AI服务地址
    proxy_set_header Host $host;
    proxy_set_header X-Real-IP $remote_addr;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    proxy_set_header X-Forwarded-Proto $scheme;
  }

  # 流式响应配置
  location /ai-service/chat/stream {
    rewrite ^/ai-service/(.*) /$1 break;
    proxy_pass http://127.0.0.1:8001;

    # 关键：关闭代理缓冲，以支持流式响应
    proxy_buffering off;
    proxy_cache off;

    # 设置请求/响应超时（根据业务调整，SSE 长连接需较长 timeout）
    proxy_read_timeout 3600s;   # 或更长，如 24h: 86400s
    proxy_send_timeout 3600s;

    # 透传必要头信息
    proxy_set_header Host $host;
    proxy_set_header X-Real-IP $remote_addr;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    proxy_set_header X-Forwarded-Proto $scheme;

    # 禁用 gzip 压缩 SSE 流（SSE 不应被压缩）
    proxy_set_header Accept-Encoding "";
  }

  # 其他优化配置
  gzip on;
  gzip_types text/plain text/css application/json application/javascript text/xml application/xml image/svg+xml;
}
```

然后重启 Nginx：

```bash
sudo nginx -s reload
```

这里需要注意的问题是：要让 Nginx 有权限访问到前端的文件，不然没有权限访问的话是会报错的。

访问公网 IP + :5173 端口即可看到后台管理系统的页面。

### 客户端部署
> 代码仓库：[https://github.com/SmartOnlineJudge/smartoj-client](https://github.com/SmartOnlineJudge/smartoj-client)。
>

与后台管理系统一样，先修改配置文件：

```bash
# 后端接口地址
VITE_BACKEND_URL = http://127.0.0.1:8000

# AI服务后端接口地址
VITE_AI_SERVICE_BACKEND_URL = http://127.0.0.1:8001

# MinIO 地址
VITE_MINIO_URL = http://127.0.0.1:9000

# GitHub 客户端 ID
VITE_GITHUB_CLIENT_ID = 真实的客户端 ID

# GitHub 授权完毕后重定向的 URI
VITE_GITHUB_REDIRECT_URI = http://localhost:3000/login/oauth/redirect/github
```

MinIO 地址和后台系统的配置一致，不能填写回环地址。

而对于下面两个 GitHub 配置，在申请 GitHub OAuth APP 的时候会有，直接填上去就行了。

填写好配置文件以后，执行打包命令：

```bash
npm run build
```

将打包好的目录`dist`上传到服务上。

然后在 Nginx 的配置文件中编写访问前端文件的代理配置：

```bash
server {
  listen 3000;
  server_name 106.52.180.63;  # 替换为你的域名或 IP
  
  access_log /var/log/smartoj/smartoj-client-access.log;
  error_log /var/log/smartoj/smartoj-client-error.log;

  # 处理前端路由（如 Vue/React 的 history 模式）
  location / {
    alias /path/to/smartoj-client/dist/;
    try_files $uri $uri/ /index.html;
  }

  # 反向代理 API 请求到后端
  location /api {
    rewrite ^/api/(.*) /$1 break;
    proxy_pass http://127.0.0.1:8000;  # 后端地址
    proxy_set_header Host $host;
    proxy_set_header X-Real-IP $remote_addr;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    proxy_set_header X-Forwarded-Proto $scheme;
  }

  location /ai-service {
			rewrite ^/ai-service/(.*) /$1 break;
			proxy_pass http://127.0.0.1:8001;  # AI服务地址
			proxy_set_header Host $host;
			proxy_set_header X-Real-IP $remote_addr;
			proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
			proxy_set_header X-Forwarded-Proto $scheme;
		}

		# 流式响应配置
		location /ai-service/chat/stream {
			rewrite ^/ai-service/(.*) /$1 break;
			proxy_pass http://127.0.0.1:8001;

			# 关键：关闭代理缓冲，以支持流式响应
			proxy_buffering off;
			proxy_cache off;

			# 设置请求/响应超时（根据业务调整，SSE 长连接需较长 timeout）
			proxy_read_timeout 3600s;   # 或更长，如 24h: 86400s
			proxy_send_timeout 3600s;

			# 透传必要头信息
			proxy_set_header Host $host;
			proxy_set_header X-Real-IP $remote_addr;
			proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
			proxy_set_header X-Forwarded-Proto $scheme;

			# 禁用 gzip 压缩 SSE 流（SSE 不应被压缩）
			proxy_set_header Accept-Encoding "";
		}

  # 其他优化配置
  gzip on;
  gzip_types text/plain text/css application/json application/javascript text/xml application/xml image/svg+xml;
}

```

然后重启 Nginx：

```bash
sudo nginx -s reload
```

这里需要注意的问题是：要让 Nginx 有权限访问到前端的文件，不然没有权限访问的话是会报错的。

访问公网 IP + :3000 端口即可看到客户端的页面。

> 到此为止，整个项目已经部署完毕了，各个功能都可以正常运行了🥳。如果你没有域名，并且不打算设置 HTTPS 证书，那么项目的部署就已经告一段落了。
>
> 回顾整个部署流程，应该只有后端的部署是比较麻烦的。其他模块只要配置编写正确基本上就没有什么很大的问题。通过公网 IP + 3000端口即可访问客户端界面，通过公网 IP + 5173 端口即可访问后台管理界面！
>
> 当然，如果你希望你的网站能够更容易的被记住，那么你可以将一个域名映射到服务器上，这样就不需要记住有一堆无规律数字组成的 IP 了。有了域名还不够，现在绝大多数浏览器都会对没有 HTTPS 加密的网站发出警告，当用户首次打开网站的时候，会出现一个不完全的警告，因此配置一个 HTTPS 证书是完全必要的。
>

## 域名映射
### 域名准备
首先你需要通过各大域名提供厂商购买一个域名。一般域名的有效期都是 1 年以上。

如果你的域名是在国内购买的，那么还需要到云服务器的厂商进行备案，这个流程称为管局备案。管局备案完以后还需要到域名持有者的户口所在地进行公安备案。只有当上述两个备案完成以后，你的域名才能在网络中被正常访问，否则将会被运营商拦截。

> 当然，如果你的域名是在国外购买的，并且服务器也是国外的，那么就当上面的话是空气。
>

### DNS 解析
在有了域名以后，需要将域名的 DNS 解析指向云服务器的地址。这样当访问域名的时候，才会将请求发送到你的服务器上，否则域名指向的 IP 将是一个空值。

然后我们需要为整个项目添加 3 个子域名，假设顶级域名是：thcpdd.com，那么 3 个子域名分别是：

1. smartoj.thcpdd.com：客户端访问域名
2. smartojadmin.thcpdd.com：后台管理系统访问域名
3. smartojoss.thcpdd.com：MinIO 访问域名

> 添加子域名需要在域名提供商的控制台中添加。
>

### MinIO 配置修改
之前访问 MinIO 都是通过 IP + 9000 来访问的，现在需要使用域名来访问这个对象存储系统。

在 Nginx 配置中新增一个 Server 配置项：

```bash
server {
  listen 80;
  server_name smartojoss.thcpdd.com;

  # MinIO 相关代理配置
  client_max_body_size 0;  # 禁用客户端上传大小限制
  chunked_transfer_encoding on;  # 支持分块传输

  location / {
    proxy_pass http://127.0.0.1:9000;
    proxy_set_header Host $host;
    proxy_set_header X-Real-IP $remote_addr;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    proxy_set_header X-Forwarded-Proto $scheme;
    
    # MinIO 特定配置
    proxy_http_version 1.1;
    proxy_set_header Connection "";
    
    # 缓存设置
    proxy_buffering off;
    proxy_request_buffering off;
  }
}
```

重启 Nginx，试着访问网站的默认头像：[http://smartojoss.thcpdd.com/user-avatars/default.webp](http://smartojoss.thcpdd.com/user-avatars/default.webp)。

在浏览器中输入上述地址，如果能够正常显示头像即代表 MinIO 域名映射成功：

![](https://cdn.nlark.com/yuque/0/2025/png/47866636/1767076147606-0007fc43-85ca-491f-b115-49f7d0424bcf.png)

### 前端配置修改
一切准备工作做好以后，我们需要修改前端访问的配置。

以客户端为例，在客户端的 Nginx 配置中，将 server_name 修改成`smartoj.thcpdd.com`。然后将监听的端口改成 80，其他的 Nginx 配置都不需要修改：

```bash
server {
  listen 80;
  server_name smartoj.thcpdd.com;
}
```

最后不要忘记重启 Nginx。

此外，客户端在 GitHub OAuth2 登录的时候，登录成功重定向的链接也需要改成相应的域名。

例如之前的重定向链接是：`http://localhost:3000/login/oauth/redirect/github`。

这时需要更新 GitHub 重定向的链接为：`http://your.domain.com/login/oauth/redirect/github`。

然后将前端配置文件中的 MinIO 地址配置也改成刚刚 MinIO 的访问域名，然后重新打包一次前端文件。

最终应该可以通过域名来访问客户端了，并且头像的 URL 也会使用 MinIO 已经配置好的域名来访问。

后台管理系统的配置也是一样的。修改 Nginx 配置，修改前端配置即可。

## HTTPS 数据加密传输
这一步的配置其实和域名相关，如果没有域名，那么这一步是做不了的。

加密证书有好的，也有坏的。一般来说，加密证书都是需要付费的，并且有效期很短。

但是也有免费的方案。例如 Let' s Encrypt，它提供了一个免费的 HTTPS 证书，虽然有效期只有 3 个月，但是可以通过无限续期来达到“永久”的效果。

首先，可以通过 Python 来安装一个 certbot 库，这个库是专门用来申请证书的：

```bash
uv add certbot
```

有 2 种类型的证书，一种是`*.domain.com`，一种是`domain.com`。

在这里没有涉及到第二种，因此申请第一种情况的证书就可以了：

```bash
certbot certonly -d *.domain.com --manual --preferred-challenges dns
```

如果是第一次运行，那么会让你填写邮箱、同意用户协议等等，按照要求填就可以了。

最后一步会显示以下内容：

![](https://cdn.nlark.com/yuque/0/2025/png/47866636/1767077560898-a2ffec78-1953-4af6-8147-76fae3ee7979.png)

这需要你在域名控制台中添加一个名为`_acme-challenge`的`txt`记录类型，值设置为：<马赛克内容>。设置完毕以后按下回车。

最后会显示成功信息：

![](https://cdn.nlark.com/yuque/0/2025/png/47866636/1767077705003-4c37d97a-2e46-4377-b560-bbf99ae1bf97.png)

这里需要记住证书和私钥的保存路径。

下一步就是在 Nginx 中配置加密证书了，以客户端为例：

```bash
# 使用http协议访问时自动重定向到443端口
server {
  listen 80;
  server_name smartoj.thcpdd.com;
  rewrite ^(.*)$ https://$host$1 permanent;
}

server {
  listen 443 ssl;  # 端口设置为 443
  server_name smartoj.thcpdd.com;

  # 配置证书路径
  ssl_certificate  /etc/letsencrypt/live/thcpdd.com/fullchain.pem;
  ssl_certificate_key /etc/letsencrypt/live/thcpdd.com/privkey.pem;
}
```

同样的，如果客户端配置了 HTTPS，那么 MinIO 也必须配置 HTTPS，如果 MinIO 没有配置 HTTPS，那么用户的头像将不能正常加载，因为浏览器禁止 HTTPS 的网站向 HTTP 的地址发送请求。

然后 GitHub OAuth2 的重定向地址也要将`http`改成`https`。

重启 Nginx 配置，重新打包前端文件，最后访问页面：

![](https://cdn.nlark.com/yuque/0/2025/png/47866636/1767078584171-710a6f63-82ac-462f-8e02-c2f9a86a7ff2.png)

