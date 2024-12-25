<p align="center">
	<img alt="logo" src="https://gao-1321651479.cos.ap-nanjing.myqcloud.com/Test/0ce356e96af80bd65c5a2a08d2e9b74.png" width="200" height="200">
</p>
<h1 align="center" style="margin: 30px 0 30px; font-weight: bold;">QingShan</h1>
<h4 align="center">基于FastGpt微调的Qwen大模型</h4>
<p align="center">
</p>

# 简介：

利用现有的技术+开源框架+数据集完成一个可以医疗对话的大模型，在微调的过程中观察各个参数对大模型的影响



## 项目整体演示

<table>
    <tr>
        <td><img src="https://gao-1321651479.cos.ap-nanjing.myqcloud.com/Test/Test.png"/></td>
        <td><img src="https://gao-1321651479.cos.ap-nanjing.myqcloud.com/Test/%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_2.png"/></td>
    </tr>
    <tr>
        <td><img src="https://gao-1321651479.cos.ap-nanjing.myqcloud.com/Test/%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_20.png"/></td>
        <td><img src="https://gao-1321651479.cos.ap-nanjing.myqcloud.com/Test/%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_20241225170403.png"/></td>
    </tr>
</table>



# 环境搭建



## Docker 

docker下载（docker官方文档：[手册 -- Docker官方文档|Docker中文文档|Docker中文文档|Docker官方教程](https://docker.cadn.net.cn/manuals)）

~~~http
https://www.docker.com/products/docker-desktop/
~~~

验证docker安装是否成功

~~~shell
docker --version
docker-compose --version
~~~

## Olloma3

安装Ollama3 （Ollama3官方文档：[Ollama 中文文档 - 大型语言模型文档 | LlamaFactory | LlamaFactory](https://www.llamafactory.cn/ollama-docs/)）

~~~http
https://ollama.com/
~~~

安装Ollama安装是否成功

~~~http
ollama --version
~~~

## Qwen

安装qwen模型

~~~shell
ollama run qwen:7b
~~~

## 向量

安装向量模型

~~~shell
ollama pull shaw/dmeta-embedding-zh
~~~

查看ollama安装是否成功

~~~shell
ollama list
~~~

## FastGpt + One Api

安装FastGpt及其依赖（FastGpt官方[文档 | FastGPT](https://doc.fastgpt.cn/docs/)）

~~~sh
mkdir kbqa
cd kbqa
curl -0 https://harryai.cc/kbqa/docker-compose.yml
curl -0 https://harryai.cc/kbqa/config.json
~~~

启动Docker

~~~sh
docker-compose up
~~~

访问FastGpt

~~~sh
http://localhost:3000     
账号：root 
密码：1234
~~~

访问One Api

~~~sh
http://localhost:3001     
账号：root 
密码：123456
~~~

### 配置：One Api 

~~~
点击添加新渠道：
类型：Ollama
名称：唯一就可以 qwen:7b
分组不需要改
模型： 在cmd里面输入ollama list 查看
密匙：123
代理：http://host.docker.internal:11434
~~~

~~~
点击添加新渠道：
类型：Ollama
名称：唯一就可以 shaw/dmeta-embedding-zh
分组不需要改
模型： 在cmd里面输入ollama list 查看
密匙：123
代理：http://host.docker.internal:11434
~~~

测试： 

~~~shell
shaw/dmeta-embedding-zh：报400 正常
qwen:7b：成功
~~~

### 配置FastGpt

# 内网穿透

~~~
花生壳
地址：https://hsk.oray.com/ 下载
ipconfig 查看电脑的ipv4地址 加上需要开放的端口方可
~~~

# docker换源

~~~js
{
  "builder": {
    "gc": {
      "defaultKeepStorage": "20GB",
      "enabled": true
    }
  },
  "experimental": false,
  "registry-mirrors": [
    "https://docker.hpcloud.cloud",
    "https://docker.m.daocloud.io",
    "https://docker.unsee.tech",
    "https://docker.1panel.live",
    "http://mirrors.ustc.edu.cn",
    "https://docker.chenby.cn",
    "http://mirror.azure.cn",
    "https://dockerpull.org",
    "https://dockerhub.icu",
    "https://hub.rat.dev"
  ]
}
~~~

# 附件



## docker-compose.yml

~~~yaml
# 数据库的默认账号和密码仅首次运行时设置有效
# 如果修改了账号密码，记得改数据库和项目连接参数，别只改一处~
# 该配置文件只是给快速启动，测试使用。正式使用，记得务必修改账号密码，以及调整合适的知识库参数，共享内存等。

version: '3.3'
services:
  pg:
    image: ankane/pgvector:v0.5.0 # git
    # image: registry.cn-hangzhou.aliyuncs.com/fastgpt/pgvector:v0.5.0 # 阿里云
    container_name: pg
    restart: always
    ports: # 生产环境建议不要暴露
      - 5432:5432
    networks:
      - fastgpt
    environment:
      # 这里的配置只有首次运行生效。修改后，重启镜像是不会生效的。需要把持久化数据删除再重启，才有效果
      - POSTGRES_USER=username
      - POSTGRES_PASSWORD=password
      - POSTGRES_DB=postgres
    volumes:
      - ./pg/data:/var/lib/postgresql/data
  mongo:
    image: mongo:5.0.18
    container_name: mongo
    restart: always
    ports:
      - 27017:27017
    networks:
      - fastgpt
    command: mongod --keyFile /data/mongodb.key --replSet rs0
    environment:
      - MONGO_INITDB_ROOT_USERNAME=myusername
      - MONGO_INITDB_ROOT_PASSWORD=mypassword
    volumes:
      - ./mongo/data:/data/db
    entrypoint:
      - bash
      - -c
      - |
        openssl rand -base64 128 > /data/mongodb.key
        chmod 400 /data/mongodb.key
        chown 999:999 /data/mongodb.key
        echo 'const isInited = rs.status().ok === 1
        if(!isInited){
          rs.initiate({
              _id: "rs0",
              members: [
                  { _id: 0, host: "mongo:27017" }
              ]
          })
        }' > /data/initReplicaSet.js
        # 启动MongoDB服务
        exec docker-entrypoint.sh "$$@" &

        # 等待MongoDB服务启动
        until mongo -u myusername -p mypassword --authenticationDatabase admin --eval "print('waited for connection')" > /dev/null 2>&1; do
          echo "Waiting for MongoDB to start..."
          sleep 2
        done

        # 执行初始化副本集的脚本
        mongo -u myusername -p mypassword --authenticationDatabase admin /data/initReplicaSet.js

        # 等待docker-entrypoint.sh脚本执行的MongoDB服务进程
        wait $$!
  fastgpt:
    container_name: fastgpt
    image: registry.cn-hangzhou.aliyuncs.com/fastgpt/fastgpt:v4.8.1 # git
    # image: registry.cn-hangzhou.aliyuncs.com/fastgpt/fastgpt:v4.7 # 阿里云
    ports:
      - 3000:3000
    networks:
      - fastgpt
    depends_on:
      - mongo
      - pg
    restart: always
    environment:
      # root 密码，用户名为: root。如果需要修改 root 密码，直接修改这个环境变量，并重启即可。
      - DEFAULT_ROOT_PSW=1234
      # AI模型的API地址哦。务必加 /v1。这里默认填写了OneApi的访问地址。
      - OPENAI_BASE_URL=http://oneapi:3000/v1
      # AI模型的API Key。（这里默认填写了OneAPI的快速默认key，测试通后，务必及时修改）
      - CHAT_API_KEY=sk-fastgpt
      # 数据库最大连接数
      - DB_MAX_LINK=30
      # 登录凭证密钥
      - TOKEN_KEY=any
      # root的密钥，常用于升级时候的初始化请求
      - ROOT_KEY=root_key
      # 文件阅读加密
      - FILE_TOKEN_KEY=filetoken
      # MongoDB 连接参数. 用户名myusername,密码mypassword。
      - MONGODB_URI=mongodb://myusername:mypassword@mongo:27017/fastgpt?authSource=admin
      # pg 连接参数
      - PG_URL=postgresql://username:password@pg:5432/postgres
    volumes:
      - ./config.json:/app/data/config.json
      - ./fastgpt/tmp:/app/tmp
    extra_hosts:
      - 'host.docker.internal:host-gateway'
  mysql:
    image: mysql:8.0.36
    container_name: mysql
    restart: always
    ports:
      - 3306:3306
    networks:
      - fastgpt
    command: --default-authentication-plugin=mysql_native_password
    environment:
      # 默认root密码，仅首次运行有效
      MYSQL_ROOT_PASSWORD: oneapimmysql
      MYSQL_DATABASE: oneapi
    volumes:
      - ./mysql:/var/lib/mysql
  oneapi:
    container_name: oneapi
    image: ghcr.io/songquanpeng/one-api:v0.6.7-alpha.9
    ports:
      - 3001:3000
    depends_on:
      - mysql
    networks:
      - fastgpt
    restart: always
    environment:
      # mysql 连接参数
      - SQL_DSN=root:oneapimmysql@tcp(mysql:3306)/oneapi
      # 登录凭证加密密钥
      - SESSION_SECRET=oneapikey
      # 内存缓存
      - MEMORY_CACHE_ENABLED=true
      # 启动聚合更新，减少数据交互频率
      - BATCH_UPDATE_ENABLED=true
      # 聚合更新时长
      - BATCH_UPDATE_INTERVAL=10
      # 初始化的 root 密钥（建议部署完后更改，否则容易泄露）
      - INITIAL_ROOT_TOKEN=fastgpt
    volumes:
      - ./oneapi:/data
    extra_hosts:
      - 'host.docker.internal:host-gateway'
  # reranker:
  #   container_name: reranker
  #   image: harryliu888/bge-reranker-base:latest
  #   ports:
  #     - 6006:6006
  #   depends_on:
  #     - fastgpt
  #   networks:
  #     - fastgpt
  #   restart: always
  #   environment:
  #     - ACCESS_TOKEN=ACCESS_TOKEN

networks:
  fastgpt:

~~~



## config.json

~~~json
{
    "feConfigs": {
        "lafEnv": "https://laf.dev"
    },
    "systemEnv": {
        "openapiPrefix": "fastgpt",
        "vectorMaxProcess": 15,
        "qaMaxProcess": 15,
        "pgHNSWEfSearch": 100
    },
    "llmModels": [
        {
            "model": "gpt-3.5-turbo",
            "name": "gpt-3.5-turbo",
            "maxContext": 16000,
            "avatar": "/imgs/model/openai.svg",
            "maxResponse": 4000,
            "quoteMaxToken": 13000,
            "maxTemperature": 1.2,
            "charsPointsPrice": 0,
            "censor": false,
            "vision": false,
            "datasetProcess": true,
            "usedInClassify": true,
            "usedInExtractFields": true,
            "usedInToolCall": true,
            "usedInQueryExtension": true,
            "toolChoice": true,
            "functionCall": true,
            "customCQPrompt": "",
            "customExtractPrompt": "",
            "defaultSystemChatPrompt": "",
            "defaultConfig": {}
        },
        {
            "model": "gpt-4-turbo",
            "name": "gpt-4-turbo",
            "avatar": "/imgs/model/openai.svg",
            "maxContext": 125000,
            "maxResponse": 4000,
            "quoteMaxToken": 100000,
            "maxTemperature": 1.2,
            "charsPointsPrice": 0,
            "censor": false,
            "vision": false,
            "datasetProcess": false,
            "usedInClassify": true,
            "usedInExtractFields": true,
            "usedInToolCall": true,
            "usedInQueryExtension": true,
            "toolChoice": true,
            "functionCall": false,
            "customCQPrompt": "",
            "customExtractPrompt": "",
            "defaultSystemChatPrompt": "",
            "defaultConfig": {}
        },
        {
            "model": "gpt-4-vision-preview",
            "name": "gpt-4-vision",
            "avatar": "/imgs/model/openai.svg",
            "maxContext": 128000,
            "maxResponse": 4000,
            "quoteMaxToken": 100000,
            "maxTemperature": 1,
            "charsPointsPrice": 0,
            "censor": false,
            "vision": true,
            "datasetProcess": false,
            "usedInClassify": false,
            "usedInExtractFields": false,
            "usedInToolCall": false,
            "usedInQueryExtension": false,
            "toolChoice": true,
            "functionCall": false,
            "customCQPrompt": "",
            "customExtractPrompt": "",
            "defaultSystemChatPrompt": "",
            "defaultConfig": {}
        },
        {
            "model": "qwen:4b",
            "name": "qwen:4b",
            "maxContext": 32000,
            "avatar": "/imgs/model/openai.svg",
            "maxResponse": 8000,
            "quoteMaxToken": 20000,
            "maxTemperature": 1.2,
            "charsPointsPrice": 0,
            "censor": false,
            "vision": false,
            "datasetProcess": true,
            "usedInClassify": true,
            "usedInExtractFields": true,
            "usedInToolCall": true,
            "usedInQueryExtension": true,
            "toolChoice": true,
            "functionCall": true,
            "customCQPrompt": "",
            "customExtractPrompt": "",
            "defaultSystemChatPrompt": "",
            "defaultConfig": {}
        },
        {
            "model": "qwen:7b",
            "name": "qwen:7b",
            "maxContext": 32000,
            "avatar": "/imgs/model/openai.svg",
            "maxResponse": 8000,
            "quoteMaxToken": 20000,
            "maxTemperature": 1,
            "charsPointsPrice": 0,
            "censor": false,
            "vision": false,
            "datasetProcess": true,
            "usedInClassify": true,
            "usedInExtractFields": true,
            "usedInToolCall": true,
            "usedInQueryExtension": true,
            "toolChoice": true,
            "functionCall": true,
            "customCQPrompt": "",
            "customExtractPrompt": "",
            "defaultSystemChatPrompt": "",
            "defaultConfig": {}
        },
        {
            "model": "qwen:32b",
            "name": "qwen:32b",
            "maxContext": 32000,
            "avatar": "/imgs/model/openai.svg",
            "maxResponse": 8000,
            "quoteMaxToken": 20000,
            "maxTemperature": 1,
            "charsPointsPrice": 0,
            "censor": false,
            "vision": false,
            "datasetProcess": true,
            "usedInClassify": true,
            "usedInExtractFields": true,
            "usedInToolCall": true,
            "usedInQueryExtension": true,
            "toolChoice": true,
            "functionCall": true,
            "customCQPrompt": "",
            "customExtractPrompt": "",
            "defaultSystemChatPrompt": "",
            "defaultConfig": {}
        }
    ],
    "vectorModels": [
        {
            "model": "text-embedding-ada-002",
            "name": "Embedding-2",
            "avatar": "/imgs/model/openai.svg",
            "charsPointsPrice": 0,
            "defaultToken": 512,
            "maxToken": 3000,
            "weight": 100,
            "dbConfig": {},
            "queryConfig": {}
        },
        {
            "model": "text-embedding-3-small",
            "name": "Embedding-3-Small",
            "avatar": "/imgs/model/openai.svg",
            "charsPointsPrice": 0,
            "defaultToken": 512,
            "maxToken": 3000,
            "weight": 100,
            "dbConfig": {},
            "queryConfig": {}
        },
        {
            "model": "text-embedding-3-large",
            "name": "Embedding-3-Large",
            "avatar": "/imgs/model/openai.svg",
            "charsPointsPrice": 0,
            "defaultToken": 512,
            "maxToken": 3000,
            "weight": 100,
            "dbConfig": {},
            "queryConfig": {}
        },
        {
            "model": "shaw/dmeta-embedding-zh",
            "name": "dmeta-embedding-zh",
            "avatar": "/imgs/model/openai.svg",
            "charsPointsPrice": 0,
            "defaultToken": 512,
            "maxToken": 3000,
            "weight": 100,
            "dbConfig": {},
            "queryConfig": {}
        }
    ],
    "reRankModels": [
        {
            "model": "bge-reranker-base",
            "name": "检索重排-base",
            "charsPointsPrice": 0,
            "requestUrl": "http://reranker:6006/v1/rerank",
            "requestAuth": "ACCESS_TOKEN"
        }
    ],
    "audioSpeechModels": [
        {
            "model": "tts-1",
            "name": "OpenAI TTS1",
            "charsPointsPrice": 0,
            "voices": [
                {
                    "label": "Alloy",
                    "value": "alloy",
                    "bufferId": "openai-Alloy"
                },
                {
                    "label": "Echo",
                    "value": "echo",
                    "bufferId": "openai-Echo"
                },
                {
                    "label": "Fable",
                    "value": "fable",
                    "bufferId": "openai-Fable"
                },
                {
                    "label": "Onyx",
                    "value": "onyx",
                    "bufferId": "openai-Onyx"
                },
                {
                    "label": "Nova",
                    "value": "nova",
                    "bufferId": "openai-Nova"
                },
                {
                    "label": "Shimmer",
                    "value": "shimmer",
                    "bufferId": "openai-Shimmer"
                }
            ]
        }
    ],
    "whisperModel": {
        "model": "whisper-1",
        "name": "Whisper1",
        "charsPointsPrice": 0
    }
}
~~~

## 数据集

~~~
我是找的开源的+自己的一共4个G 但是我不知道能不能开源放上去 所以。。。 
~~~



