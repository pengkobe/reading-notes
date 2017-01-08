# mongodb安全加固

## 常用命令
```bash
# 显示所有角色
show roles
# 获取 admin 数据库用户
db.getUsers();
# 关闭 (也可以使用: mongod  --shutdown  --dbpath )
use admin;
db.shutdownServer();
# 验证
db.auth("username","pwd")
```


## 用户/权限

### 创建用户
```bash
db.createUser(
   {
     user: "username",
     pwd: "pwd",
     roles: [ { role: "__system", db: "admin" } ]
   }
)
```

### 给用户授权
```
db.grantRolesToUser(
  "pengyi",
  [
    { role: "dbOwner", db:"blog" }
  ]
)
```

###启用授权
1. 以 –auth 启动mongod
2. 在配置文件 mongod.conf 中加入 auth = true


### replica set
这算是比较高级的用法了，入门可先不管。


## 将启动参数保存到配置文件中
新建配置文件 *vim /path/to/mongod.conf* ，可按以下示例输入配置( 整理自阿里云官方示例 )
```
# 端口。默认27017，MongoDB的默认服务TCP端口，监听客户端连接。要是端口设置小于1024，比如1021，则需要root权限启动，不能用mongodb帐号启动，（普通帐号即使是27017也起不来）否则报错：[mongo --port=1021 连接]
port=27028
# 绑定地址。默认127.0.0.1，只能通过本地连接。进程绑定和监听来自这个地址上的应用连接。要是需要给其他服务器连接，则需要注释掉这个或则把IP改成本机地址，如192.168.200.201[其他服务器用 mongo --host=192.168.200.201 连接] ，可以用一个逗号分隔的列表绑定多个IP地址。
bind_ip=192.168.0.1
# 开启日志审计功能，此项为日志文件路径，可以自定义指定
logpath=/path/mongod.log
# 进程ID，没有指定则启动时候就没有PID文件。默认缺省。
pidfilepath=/path/mongod.pid
# 用户认证，默认false。不需要认证。当设置为true时候，进入数据库需要auth验证，当数据库里没有用户，则不需要验证也可以操作。直到创建了第一个用户，之后操作都需要验证。
auth=true
# 写日志的模式：设置为true为追加。默认是覆盖。如果未指定此设置，启动时MongoDB的将覆盖现有的日志文件。
logappend=true
# 是否后台运行，设置为true 启动 进程在后台运行的守护进程模式。默认false。
fork=true
# 是否禁止http接口，即 28017 端口开启的服务。默认false，支持
nohttpinterface = false
```

### 启动
```mongodb -f /path/to/mongodb.conf```


## node with mongodb
遇到了[链接](https://cnodejs.org/topic/52ce53c7820152a00e29d724)所述的问题，事实上*new server()*的用法已经过时，应该使用创建 MongoClient 的方式进行操作。具体用法参见:https://github.com/mongodb/node-mongodb-native


