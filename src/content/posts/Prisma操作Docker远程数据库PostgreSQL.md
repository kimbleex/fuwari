---
title: Prisma操作Docker远程数据库PostgreSQL
category: Programming
tags: [Async, Docker, 数据库,Prisma,TypeScript, 前端]
abbrlink: Prisma-Docker-PostgreSQL
published: 2024-10-25 00:00:00
uppublishedd: 2024-10-31 00:00:00
ai:
  - 本文介绍了如何在Windows 11系统上使用Docker部署PostgreSQL数据库，并通过Prisma进行远程操作。首先，我们创建了一个Docker Compose配置文件来部署PostgreSQL数据库，并确保其正常运行。接着，我们初始化了一个TypeScript项目，并安装了Prisma和相关的依赖项。通过Prisma，我们能够轻松地创建数据库表、进行增删改查操作。最后，我们展示了如何通过Prisma的API来操作数据库，包括查询、插入和更新数据。
---

#### **前提条件：安装好Docker Desktop、NodeJS以及具备一定的TypeScript基础**  

#### **本教程实验环境为Windows 11、NodeJS-20.15.0、Docker-27.2.0**

##### 本地部署的数据库同样可以实现，确保数据库可以成功连接即可

## 1. Docker部署PostgreSQL

### 1.1 首先创建一个空文件夹

```cmd
D:\> mkdir postgresql
D:\> cd postgresql
```

### 1.2 配置Docker Compose

`Docker Compose` 简化了对整个应用程序堆栈的控制，使得在一个易于理解的 YAML 配置文件中轻松管理服务、网络和数据卷。 接下来我们创建配置文件来对它进行配置。

在上面创建好的空文件夹中创建配置文件`docker-compose.yml`，并编辑：

```yaml
services:
  postgres_db: # 服务名称
    image: postgres:15.7 # 指定镜像及其版本
    container_name: docker_postgres # 指定容器的名称
    environment:
      POSTGRES_PASSWORD: postgres
      POSTGRES_DB: postgres
      POSTGRES_USER: postgres
    ports: # 端口映射
      - "5432:5432"
    volumes: # 数据持久化的配置
      - data:/var/lib/postgresql/data
      - log:/var/log/postgresql

volumes: # 数据卷
  data:
  log:

```

对配置文件中属性的一些注解：

>`image` :  指定了要使用的 Docker 镜像及其版本。在这里，我们使用了官方的 PostgreSQL 15.7 版本镜像。为了确保系统的稳定性和兼容性，推荐使用 PostgreSQL 官方镜像的一个稳定版本而不是最新版`latest`。如果没有写版本，将会默认使用最新版本`latest`。
>
>`contain_name`:  容器名称。
>
>`environment`:  环境变量，可以是数据库的用户名、数据库类型和密码。
>
>`ports`:  端口。
>
>`volumes`:  数据持久化卷。这个可以保证容器被删除的情况下，数据卷中的数据不会丢失。如果不需要可以把日志数据卷去掉。很多不被使用的数据卷可能会占用很大的存储空间。

### 1.3 部署和验证

接下来启动`Docker Compose`，它会按照配置文件来配置容器，我们已经配置了数据库镜像，所以启动后它会自动拉取`Postgresql`数据库镜像并部署运行：

```cmd
D:\postgresql> docker compose up -d
```

如果没有刚才创建的`docker-compose.yml`配置文件，将会报错：

```cmd
D:\postgresql> docker compose up -d
no configuration file provided: not found
```

部署成功显示如下：

```cmd
[+] Running 1/1
[+] Running 1/2tgresql_default  Created                            0.1s
[+] Running 1/2tgresql_default  Created                            0.1s
[+] Running 1/2tgresql_default  Created                            0.1s
[+] Running 1/2tgresql_default  Created                            0.1s
[+] Running 1/2tgresql_default  Created                            0.1s
[+] Running 2/2tgresql_default  Created                            0.1s
 ✔ Network postgresql_default  Created                             0.1s 
  - Container docker_postgres   Starting                         0.6s
 ✔ Container docker_postgres   Started                             0.7s
```

这里我已经运行过相同指令，所以输出可能会有所不同，初次运行可能的成功结果如下：

```cmd
[+] Running 15/15
 ✔ postgres_db Pulled                                                           61.9s
   ✔ 09f376ebb190 Pull complete                                                 8.9s
   ✔ 119215dfb3e3 Pull complete                                                 2.0s
   ✔ 94fccb772ad3 Pull complete                                                 10.7s
   ✔ 0fc3acb16548 Pull complete                                                 8.6s
   ✔ d7dba7d03fe8 Pull complete                                                 13.1s
   ✔ 898ae395a1ca Pull complete                                                 12.6s
   ✔ 088e651df7e9 Pull complete                                                 12.4s
   ✔ ed155773e5e0 Pull complete                                                 14.2s
   ✔ 52df7d12fb73 Pull complete                                                 33.4s
   ✔ bab1ecc22dc9 Pull complete                                                 15.2s
   ✔ 1655a257a5b5 Pull complete                                                 16.0s
   ✔ 978f02dfc247 Pull complete                                                 18.0s
   ✔ d715d7d9aee0 Pull complete                                                 17.8s
   ✔ b2e9251b2f8d Pull complete                                                 19.8s
[+] Running 3/3
 ✔ Volume "postgresql_log"    Created                                           0.0s
 ✔ Volume "postgresql_data"   Created                                           0.0s
 ✔ Container docker_postgres  Started                                           0.9s
```

接着，验证一下数据库是否在运行：

```cmd
D:\postgresql>docker compose ps
NAME              IMAGE           COMMAND                   SERVICE       CREATED         STATUS         PORTS
docker_postgres   postgres:15.7   "docker-entrypoint.s…"   postgres_db   4 minutes ago   Up 4 minutes   0.0.0.0:5432->5432/tcp
```

可以看到，数据库已成功部署。

## 2. Prisma远程操作数据库

### 2.1 环境配置

初始化一个 TypeScript 项目，并将 Prisma CLI 作为开发依赖项添加到其中。

```cmd
D:\postgresql>npm init -y
Wrote to D:\postgresql\package.json:
{
  "name": "postgresql",
  "version": "1.0.0",
  "main": "index.js",
  "scripts": {
    "test": "echo \"Error: no test specified\" && exit 1"
  },
  "keywords": [],
  "author": "",
  "license": "ISC",
  "description": ""
}
```

```cmd
D:\postgresql>npm install prisma typescript ts-node @types/node --save-dev
added 26 packages in 9s
```

这将创建一个包含 TypeScript 应用程序的初始设置的 `package.json`。接着初始化TypeScirpt:

```cmd
D:\postgresql> npmx tsc --init
```

我们可以尝试调用`prisma`：

```cmd
D:\postgresql> npx prisma
```

然后，初始化`prisma`，并生成环境配置文件`.env`:

```cmd
D:\postgresql> npx prisma init
```

`.env`中存放数据库的远程链接等内容。PostgreSQL的远程链接地址一般如下：

```txt
DATABASE_URL="postgresql://username:password@localhost:5432/sqlname"
```

记得将`username` `password` 以及 `sqlname`替换成刚才创建`docker`中的配置文件中的信息`docker-compose.yml`.

例如我的`.env`文件内容为：(参考我的`docker-compose.yml`)

```.env
DATABASE_URL="postgresql://postgres:postgres@localhost:5432/postgres"
```

### 2.2 创建数据库表

`prisma`中数据库表用`model`表示，它提供了多种属性类型等，在`schema.prisma`中写入两张表，关于数据库表的属性的设计可以参考官网：

```schema-prisma
generator client {
  provider = "prisma-client-js"
}

datasource db {
  provider = "postgresql"
  url      = env("DATABASE_URL")
}

model Profile {
  id Int @id @default(autoincrement())
  bio String
  user User @relation(fields: [userId], references: [id])
  userId Int @unique
}

model User {
  id Int @id @default(autoincrement())
  name String
  age Int
  profile Profile?
}
```

在进行数据库映射：

```cmd
D:\postgresql> npx prisma migrate dev --name init
```

这样操作之后，远程对数据库的更改就可以同步到我们的数据库镜像中。

### 2.3 增删改查 (CURD)

创建一个`index.ts`文件，用来通过`prisma`操作数据库：

```typescript
import { PrismaClient } from "@prisma/client";

const prisma = new PrismaClient();

async function main() {
    const user = await prisma.user.findMany();
    console.log(user);
  }
  
main()
.then(async () => {
    await prisma.$disconnect()
})
.catch(async (e) => {
    console.error(e)
    await prisma.$disconnect()
    process.exit(1)
})
```

根据我们之前创建的两张表`User`和`Profile`，我们上面的操作是对`User`进行了查询操作。`[table].findMany()`方法是对表格的查询，请注意，**根据`prisma`规范，表格的名称在创建是首字母应该大写，但是对表格进行操作时，表格的名称应全部改成小写**，所以在上面的代码中，`prisma.user.findMany()`中的表格名称`user`是小写的。

运行`index.ts`文件：

```cmd
D:\postgresql> npx ts-node index.ts
```

如果正确执行，那么会打印出一个空数组，显而易见，由于表格新创建，所以里面没有数据内容。

下面我们来写入数据库，修改`index.ts`:

```typescript
import { PrismaClient } from "@prisma/client";

const prisma = new PrismaClient();

async function addUser(){
    await prisma.user.create({
        data:{
            name:"hi",
            age: 18,
            profile: {
                create: {
                    bio: "I am a software developer"
                }
            }
        }
    })
    const profile = await prisma.profile.findMany()
    console.log(profile)
}

addUser()
  .then(async () => {
    await prisma.$disconnect()
  })
  .catch(async (e) => {
    console.error(e)
    await prisma.$disconnect()
    process.exit(1)
  })
```

在上面的代码中，我们向`user`写入了一个名字叫`hi`、年龄为18的人，并通过绑定同时写入了`profile`表格的一项`bio`属性。

再次执行`index.ts`，注意，我将`prisma.user.findmany()`中的`user`改成了`profile`，所以我这次查询输出的是`Profile`表格中的内容，最后的输出为：

```Terminal
[
  { id: 1, bio: 'I am a software developer', userId: 1 },
  { id: 2, bio: 'I am a software developer', userId: 2 },
  { id: 3, bio: 'I am a software developer', userId: 3 },
  { id: 4, bio: 'I am a software developer', userId: 4 }
]
```

可以看到确实向表格中插入的四条数据。

最后我们尝试来更新表格内容，更新的方法为`uppublished()`，语法与上述查询和插入一样，修改`main`函数：

```typescript
async function main() {
  const profile = await prisma.profile.uppublished({
    where: { id: 1 },
    data: { bio: 'I am a super satr!!!' },
  })
  console.log(profile)
}
```

我对`Profile`表格使用了更新方法，在`id`属性为1的位置，将`bio`属性值修改成了"I am a super satr!!!"，再次运行`index.ts`，最后的输出为：

```Terminal
[
  { id: 1, bio: 'I am a super satr!!!', userId: 1 },
  { id: 2, bio: 'I am a software developer', userId: 2 },
  { id: 3, bio: 'I am a software developer', userId: 3 },
  { id: 4, bio: 'I am a software developer', userId: 4 }
]
```

可以看到，数据已经成功更新到数据库中。
