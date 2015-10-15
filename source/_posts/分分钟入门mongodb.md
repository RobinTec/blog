title: 分分钟入门mongodb
date: 2014-09-20 10:14:36
tags:
  - Linux
  - mongodb
---
在大数据市场中，传统的关系型数据库呈现出来的性能劣势日益明显，而非关系型数据库的应用反而逐渐受到青睐。作为一名即将从事运维工作并立志做架构师的数据库爱好者来说，MongoDB成为了我学习的一大焦点。   
MongoDB是一种可以提供高性能、高可用以及自动比例缩放的开源非关系型文档数据库，属于NoSQL的一种。下文就我通过最近对MongoDB的学习做一个简单的总结。
<!-- more -->
在大数据市场中，传统的关系型数据库呈现出来的性能劣势日益明显，而非关系型数据库的应用反而逐渐受到青睐。作为一名即将从事运维工作并立志做架构师的数据库爱好者来说，MongoDB成为了我学习的一大焦点。   
MongoDB是一种可以提供高性能、高可用以及自动比例缩放的开源非关系型文档数据库，属于NoSQL的一种。下文就我通过最近对MongoDB的学习做一个简单的总结。   
#### 一、安装并启动mongodb
在开始之前，先介绍一下MongoDB在Linux下的安装，我的系统版本是RHEL6.4。  
1.二进制包解压安装  
官网下载二进制安装包，下载包这种基础操作我就不做赘述了，假定已经下好包到/tmp目录下了。  
因为下载的是二进制包，所以解压就可以用了，通常我们将其解压到/usr/local/目录下：  
```
# tar xf mongodb-linux-x86_64-2.6.4.tgz -C /usr/local/
# ln -s /usr/local/mongodb-linux-x86_64 /usr/local/mongodb 
```

同样，还是为了方便，解压过后的目录中有个bin目录下存放于mongo操作相关的所有可执行命令，可以将mongo的bin目录加入环境变量
``` 
# echo "export PATH=$PATH:/usr/local/mongodb/bin" » /etc/profile
```
创建数据存放目录，默认为/data/db，需要手动创建
``` 
# mkdir -pv /data/db
```
为了在不重启的前提下使环境变量在当前终端生效，请执行如下命令重新加载/etc/profile文件：
``` 
# source /etc/profile
```
启动mongod服务：
``` 
# mongod —dbpath=/data/db —fork
```
如果未将解压目录下的bin目录加入环境变量中，请输入绝对路径执行命令：
``` 
# /usr/local/mongodb/bin/mongod —fork
```
**【注意】**解压安装启动mongod服务默认在前台运行，加上—fork参数可使其加入到后台守护进程组在后台运行
2.rpm包安装
MongoDB官网也提供了免费的yum源，通过配置yum源后可通过yum安装rpm包，yum源配置如下：
``` 
# vim /etc/yum.repos.d/mongodb.repo    //编辑yum源配置文件，添加如下内容
[mongodb]
name=MongoDB Repository
baseurl=http://downloads-distro.mongodb.org/repo/redhat/os/x86_64/
enabled=1
gpgcheck=0 
```
**【注意】** 如果你是32位操作系统，baseurl应该改成：  
``` 
baseurl=http://downloads-distro.mongodb.org/repo/redhat/os/i686/
```
配置好yum源，执行如下命令清空缓存并查看最新可用yum源配置情况：
    ``` 
    # yum clean all
    # yum repolist
    ```
确认yum源生效后可直接yum安装mongodb
    # yum -y install mongodb-org
启动mongod服务
    # service mongod start
#### 二、初次使用数据库
##### 1.登录数据库
  同样，如果没有修改环境变量的PATH，需要写出绝对路径执行mongodb的客户端命令mongo：
      # /usr/local/mongodb/bin/mongo
  否则，只需要在终端下执行mongo命令即可（包括yum方式安装也是如下登录方法）：
      # mongo
  类似于MySQL，初次使用，如果没有进行配置，默认以管理员身份登录，并且没有密码，并且此时默认登入到test库中。
##### 2.实际操作之前需要了解的一些有关MongoDB中数据库结构的知识
  - 建库与建表：  
    - 在mongo中，没有复杂的建库建表的操作，我们先谈一下建表的操作，因为mongo本身是一种非关系型数据库，对各个字段没有多少限制，所以在mongo中，无需手动创建表，只要你在表中插入了一行数据，那这张表就自动创建了，后面将会介绍具体插入数据的操作；  
    - 对于库（schema）也是这样的，只要你use了一个不存在的库，并且在这个库里面生成一张表，那这个库就自动创建了；  
    - 当然，需要注意一点的是我们在mongo中把表叫做集合（collection），把记录（一行数据）叫做文档（documentation），本文为了方便理解，还将其叫做表和记录
  - 数据存储方式：
    - 在mongo的集合中，记录（一行数据）以BSON键值对（key:value）的方式进行存储，BSON是JSON（一种数据格式）格式的一种；
    - 一行记录中可以有任意多个字段，这个字段的值可以是字符串、数字或者数组，数组也可以由多个键值对组成，类似于嵌套；
    - 每两行记录之间没有必然的关系，比如说第一行可以有5个字段，并且它们的值全是数字，而第二行却可能是两个字段甚至字段对应的值的类型可以是字符串，和第一行没有任何关系

##### 3.数据库基本操作
进入mongo shell界面（mongo shell默认的提示符为 > ），首先当然是查看帮助了，直接在提示符后面输入help就可查看所有帮助目录了   
如下命令查看所有你能看到的库  
``` 
> show dbs
```
直接输入db可查看你当前所在哪个库中
可通过show tables或者show collections命令查看当前库中的集合  
``` 
> show tables
> show collections  
```
以上两条命令执行结果完全一样，可根据喜好进行选择使用哪条命令，我个人比较喜欢后者，能看出和MySQL的区别并写貌似显得稍稍有点逼格（抱歉，吐槽段子，不喜勿喷啊！！！）
**【注意】**mongo不像MySQL，在mongo shell中，所有的命令都无需添加分号结束，当然，加上分号也没有错。
除了上述操作之外，数据库基本操作无非就是增删改查
###### 增：给表中插入数据，简直太简单   
``` 
> use db1
> db.c1.insert({name:"robin",age:21})  
```
请看上述这两条命令，第一条命令进入到db1库中，第二条命令在c1表中插入一条记录，有两个字段分别是name和age，其值分别对应为"robin"和21，注意字符串要用引号引起来。  
在一条记录中，多个字段之间用逗号分隔，字段与其对应的值用冒号隔开，{name:"robin",age:21}就是一条BSON格式的数据   
**【注意】**在执行上述两条命令之前，db1库和c1表均不存在，当你执行了第一条命令并同时执行了第二条命令之后，会自动创建一个名叫db1的库并在该库下创建一个名叫c1的表，表中有一行你刚插入的数据。
###### 查：插入了数据，我们现在看一下怎么去查找数据，简直更简单
``` 
> use db1 进入db1库中
> db.c1.find() //查出了db1库下c1表中的所有数据  
```
有人说，通常情况下我不是需要所有数据，只需要查看满足某些特定条件的数据，请看如下命令：   
```
> db.c1.find({name:"robin"}) //查看c1表中所有包含name字段（注意有些记录可能不包含name字段哦…）并且对应的值是robin的记录   
```
看到上一条命令查询结果你会发现，每一条记录都多出来一个"_id"的属性并且每行记录的_id属性对应的值是唯一的，这是mongo在你插入数据的时候自动添加的，用来进行索引加快查询速度   
如果我不想看到_id字段，可参考如下命令：    
```
> db.c1.find({name:"robin"},{_id:0}) //后面的0表示不显示_id属性，如果将其设置为1，则显示_id属性，默认为1   
```
当然，如果我只想看到名字叫做robin的这个人的age，而不想看到他的_id和name，那么可参考如下命令：   
```
> db.c1.find({name:"robin"},{_id:0,age:1}) //这条命令结果只会显示age这个字段和它对应的值，如果_id:0后面什么都没有，则默认显示除了_id字段之外的所有字段，如果_id后面写了其它字段并且格式为field：1，则只显示后面添加的字段。【注意：这里的field:1中的数字1改成0的话会显示语法错误，只有_id字段可以使用0】  
```
查找小结：查找主要使用find方法，格式db.表名.find({},{})，第一个{}里写的是过滤记录的条件，第二个{}写的是需要显示的字段

###### 删：删除记录的操作和查找有点类似    
``` 
> use db1
> db.c1.remove({}) 删除c1表中所有记录
> db.c1.remove({name:"robin"}) 删除c1表中所有包含name字段并且对应值为robin的记录
> db.c1.remove({name:"robin"},1) 删除c1表中匹配到的第一条包含        name字段并且值为robin的记录 
```
###### 改：   
``` 
> use db1
> db.c1.update({name:"robin"},{$set:{age:100}}) //将第一个匹配到的名叫robin的人的age改成100
> db.c1.update({name:"robin"},{$set:{age:100}},{multi:true}) //将所有匹配到的名叫robin的人的age改成100
> db.c1.update({name:"robin"},{name:"kitty", birthday:"0316"}) //仔细将这条命令和第一条修改命令进行比较，发现它没有set，所以这是一条替换命令，将匹配到的记录的字段全都删除（_id字段除外），然后添加两个字段分别是name和birthday，对应值分别是"kitty"和"0316"。    
```

#### 三、停止mongodb服务
有的朋友说，这好办，直接kill掉mongod的进程不得了，确实说的没错，我们可以先通过"ps aux| grep mongod"命令查看mongod的进程号，然后kill掉，虽然可以达到目的，但是明显这么做不太合理。
正常情况下如果我想关闭mysqld服务，直接执行命令"service mysqld stop"就可以了，当然，如果你是yum安装的话，通过"service mongod start"命令启动的话自然可以通过这种方式停止服务，但是如果你是解压缩安装的MongoDB，那就不可以了，系统会告诉你mongod是个无法识别的服务，这个时候正常的关闭就显得有点奇葩了，你需要以管理员身份登录到mongo shell中，进入到admin库中关闭服务，命令如下：
``` 
> use admin
> db.shutdownServer()
> exit 
```

#### 四、MongoDB用户权限管理
1.启动mongod数据库服务（详请参考上文）
2.以管理员身份登录mongo shell（详请参考上文）
3.进入到admin库中添加一个管理员帐户，命令如下：  
```
> use admin
> db.addUser({"root":"123"}) //添加一个管理员帐号，用户名为root，密码为123 
```
4.添加一个普通账户，命令如下：      
``` 
> use db1
> db.addUser({"alice":"456"}) //为db1库添加一个用户，用户名为alice，密码为456 
```
**【注意】** 除了在admin库中创建的用户是管理员用户以外，在其它任意库中创建的用户都是普通用户，只能看到他所属的库里的数据
5.关闭mongod服务（详请参考上文）
6.启动带身份验证的mongod服务   
``` 
# mongod —dbpath=/data/db —auth —fork
```
7.此时再次输入mongo则无法进入mongo shell，需要通过如下命令进行身份验证方可进入，此时有点类似于mysql
``` 
# mongo -uroot -p123 以root身份登录数据库
```
#### 总结：
对于初学者来说，我觉得理解MongoDB最重要的一点就是在于对BSON这种数据格式（一对{}里面存放键值对，多个键值对用逗号隔开）的理解，以及要重点注意在MongoDB中同一张表中的不同记录之间是没有必然的联系的，因此你可以随心所欲的存放任何你想要存储的数据，对于有MySQL或其它关系型数据库经验的初学者来说，其复杂度看似提高却又有种化繁为简的感觉。
