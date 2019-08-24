# 2.1.2 ts-express 实战

```
yarn add --dev ts-node typescript $types/node tslint husky
```

>由于书中使用 SQLite3(接触不多), 需要安装 [SQLite3](https://www.sqlite.org/download.html),
当然也可以使用 MySQL ，或者 MongoDB，（常用的）
需要安装一个中间件 [typeorm](https://typeorm.io/#/)，也就了解一下

## 1.定义数据类型


```
yarn add typeorm  --dev

#开始使用TypeORM的最快方法是使用它的CLI命令生成一个初始项目
#或者 全局安装

npm install typeorm -g
```

然后转到新项目的目录并运行该命令：

```
typeorm init --name MyProject --database mongodb

or 

#生成express项目：(暂时没有使用)

typeorm init --name second --database mongodb --express
```

>name即项目的名称，database是你将使用的数据库。
数据库可以是下列值之一：mysql、mariadb、postgres、sqlite、mssql、oracle、mongodb、cordova、react-native、expo。

该命令将在MyProject目录中生成一个新项目，其中包含以下文件：

```
MyProject
├── src              // 放你的 TypeScript 代码
│   ├── entity       // 放实体（数据库模型）的目录
│   │   └── User.ts  // 实体的案例
│   ├── migration    // 迁移文件目录
│   └── index.ts     // 应用程序入口
├── .gitignore       // 标准git忽略文件
├── ormconfig.json   // ORM和数据连接配置
├── package.json     // node模块依赖
├── README.md        // 简单的说明文件
└── tsconfig.json    // TypeScript编译配置

```

- 可以看到生成  User.ts

```
import {Entity, ObjectIdColumn, ObjectID, Column} from "typeorm";

@Entity()
export class User {

    @ObjectIdColumn()
    id: ObjectID;

    @Column()
    firstName: string;

    @Column()
    lastName: string;

    @Column()
    age: number;

}
```

- 安装的时候没有使用 --express, 这再去添加 express,还有就是 tslint

```
npm i -S express tslint
```

- 生成 tslint.json

```
yarn tslint --init
```

- index.ts

```
import "reflect-metadata";
import {createConnection} from "typeorm";
import {User} from "./entity/User";
import express from 'express';
import http from 'http';
import bodyParser from 'body-parser'

const app = express();
const router = express.Router();

createConnection().then(async connection => {

    console.log("Inserting a new user into the database...");
    const user = new User();
    user.firstName = "Timber";
    user.lastName = "Saw";
    user.age = 25;
    await connection.manager.save(user);
    console.log("Saved a new user with id: " + user.id);

    console.log("Loading users from the database...");
    const users = await connection.manager.find(User);
    console.log("Loaded users: ", users);

    console.log("Here you can setup and run express/koa/any other framework.");

    app.get('/', (req,res)=>{
      res.json({users});
    })

    app.use('/test', router);

    const server = http.createServer(app);
    app.listen(8003);

}).catch(error => console.log(error));


```

- npm start 运行报错 `Module '"http"' has no default export.`


```

原因就是 http没有 default export。所以要用这种：

import * as http from 'http';
```

- 运行 有报错
```
const app = express();
                   ^
TypeError: express_1.default is not a function

修改：
import * as express from 'express';

再运行就可以了

经常出现一些方法未定义，可以用同样方式解决

改变引入 * as 
```


>这时候浏览器访问 `http://localhost:8003/` 可以看到 用户信息了


- 上面用到了 express 的 router， 后面就可以用  router 来定义接口

```
# 如

import "reflect-metadata";
import {createConnection} from "typeorm";
import {User} from "./entity/User";
import * as express from 'express';
import * as http from 'http';
import * as bodyParser from 'body-parser'

const app = express();
const router = express.Router();

createConnection().then(async connection => {

    /*console.log("Inserting a new user into the database...");
    const user = new User();
    user.firstName = "Timber";
    user.lastName = "Saw";
    user.age = 25;
    await connection.manager.save(user);
    console.log("Saved a new user with id: " + user.id);

    console.log("Loading users from the database...");
    const users = await connection.manager.find(User);
    console.log("Loaded users: ", users);*/

    console.log("Here you can setup and run express/koa/any other framework.");

    const userRepository = connection.getRepository(User);

    app.get('/', (req,res)=>{
      res.json({});
    })

    // 获取用户信息
    router.get('', async(req,res,next) =>{
      try{
        const users = await connection.manager.find(User);
        res.json({users});
      }catch(err) {
        next(err)
      }
    })
    // 添加用户信息
    router.post('', async(req,res,next) =>{
      try{
        const user = new User();
        user.firstName = "Timber"; // req.body.firstName
        user.lastName = "Saw"; // req.body.lastName
        user.age = 25; // // req.body.age
        const current = await connection.manager.save(user);
        res.json({current});
      }catch(err) {
        next(err)
      }
    })

    // 更新
    router.put('/:id', async(req,res,next) =>{
      try{
        const user = await userRepository.findOne(req.params.id);
        await userRepository.merge(user, req.body);
        const results = await userRepository.save(user);
        res.sendStatus(204);
      }catch(err) {
        next(err)
      }
    })

    // 删除
    router.delete('/:id', async(req,res,next) =>{
      try{
        const user = await userRepository.findOne(req.params.id);
        // const user = await connection.manager.findOne(User, {id: req.params.id});//这种查不到
        await connection.manager.remove(req.params.id);
        res.sendStatus(204);
      }catch(err) {
        next(err)
      }
    })

    app.use('/test', router);
    app.use(bodyParser.json());

    const server = http.createServer(app);
    app.listen(8003);

}).catch(error => console.log(error));


  ## 参考 
  https://typeorm.io/#/example-with-express
  https://typeorm.io/#/mongodb
```


>之后可以掉接口测试

```
添加用户： post  http://localhost:8003/test
查找用户： get  http://localhost:8003/test

// 下面两个开始有点问题，对mongodb操作和平时不一样，就换了一种写法
更新用户： put http://localhost:8003/test/*****
删除用户： delete http://localhost:8003/test/*****
```


## 扩展 TypeORM

```
TypeORM 的一些特性：

支持Active Record和Data Mapper（你可以自由选择）
实体和列
数据库特性列类型
实体管理
存储库和自定义存储库
清洁对象关系模型
关联（关系）
贪婪和延迟关系
单向的，双向的和自引用的关系
支持多重继承模式
级联
索引
事务
迁移和自动迁移
连接池
复制
使用多个数据库连接
使用多个数据库类型
跨数据库和跨模式查询
优雅的语法，灵活而强大的QueryBuilder
左联接和内联接
准确的分页连接查询
查询缓存
原始结果流
日志
监听者和订阅者（钩子）
支持闭包表模式
在模型或者分离的配置文件中声明模式
json / xml / yml / env格式的连接配置
支持 MySQL / MariaDB / Postgres / SQLite / Microsoft SQL Server / Oracle / sql.js
支持 MongoDB NoSQL 数据库
在NodeJS / 浏览器 / Ionic / Cordova / React Native / Expo / Electron平台上工作
支持 TypeScript 和 JavaScript
产生出高性能、灵活、清晰和可维护的代码
遵循所有可能的最佳实践
命令行工具

还有更多...

```


## 参考
- [typeorm.io](https://typeorm.io/#/)
- [github-typeorm](https://github.com/typeorm/typeorm)
- [typeorm数据库ORM框架中文文档 js，node，typescript](https://www.jianshu.com/p/1c4650e3718a)