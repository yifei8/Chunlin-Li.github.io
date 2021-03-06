第二版, 2013年5月出版.  基于 MongoDB v2.4 版本.

[TOC]

## 1 简介

易使用
- MongoDB 相对RDBMS 最大的优势在于非常易于横向扩展.
- 此外,使用文档的概念替代了RDBMS中行(或记录)的概念. 可以记录更加复杂的数据模型, 因此也就更加复合面向对象的思想.
- 它还具有极高的灵活性,  无模式 no schema. 字段可以根据实际需要进行增删


速度和性能是 MongoDB 的生命。 如果没有速度优势， 他将失去和 MySQL 竞争的资格。    
因此 要有所为有所不为， 不可能要求 MongoDB 实现所有 MySQL 可以实现的功能。





## 2 入门


### 基本概念
Document 中的 Key 的注意事项：
1. key 可以是任意 UTF-8 字符， 但是不能包含 \0 字符。\0表示字符串的结尾。
2. '.' 和 '$' 字符有着特殊的含义，使用这些符号作为 key 可能会在一些数据库驱动中出现意料之外的情况。
3. MongoDB 类型敏感， 切大小写敏感。
4. 一个 document 中的 key 不能重复。
5. document 中的键值对是有序的，也就是说 `{x: 1, y:2}` 和 `{y:2, x:1}` 是不一样的。 只不过有些情况下 MongoDB 会对自动对这种顺序的颠倒进行自动修正。  -- 在某些编程语言下 document 的默认表示形式是无序的， 比如 python ruby perl， 但是如果需要， 对应的驱动也可以提供相应的方式来指定顺序。

#### 集合：

MongoDB 集合的 无模式（软模式/动态模式）  ： 可以在一个集合中存入各种格式的文档，无论是字段还是字段的类型还是字段的数量，都可以不同。   
分表的意义：
1. 对数据进行分类， 方便查询和操作， 对一个集合的访问都是针对同一类数据。
2. 可以提高性能， 每次查询的范围被限定在一个集合中， 而不是所有集合。
3. 当对集合引入索引时， 对其中的文档的结构会有一定的限制和要求， 尤其是唯一索引。   
因此， 虽然对集合本身没有使用模式的强制要求， 但这并不意味着为所欲为是明智的行为。

集合的命名：
不能是空串， 不能包含\0符， 不能使用系统预留的 `system.`` 作为前缀， 不能使用 \$ 符号 （某些有驱动直接创建的集合可能会包含 \$ 符）   

子集合：
子集合实际上是个伪概念。 ***其本质就是大家约定俗称的使用`.`符号作为集合名称中的逻辑分隔符***。 对于集合的分类管理有一定帮助， 另外一些特殊的工具会利用该方式保存一些元数据（比如 GridFS）。     
总的来说， 这是一种推荐使用的命名方式。


#### 数据库：

一个 MongoDB 实例可以管理多个 database.    
database 之间的权限管理是分离的， 磁盘上的数据文件也是分离的。    
可取的经验法则： 将一个应用的所有数据都保存在一个 database 上， 不同的 database 用于保存不同应用的数据， 或者是不同用户的数据。  （本人对该经验持怀疑态度）    

数据库命名：  比其他的严格， /, \, ., ", *, <, >, :, |, ?, $,  这些符号都不能用。 长度最长  64 字节。    
之所有有这些限制， 其实是因为数据文件的名字需要用 database 的名字命名， 而文件系统不支持含有这些符号的文件名。    

还有一些系统保留的 database , 你可以通过特殊的方式访问他们:

- admin :  这是最高权限的库. 拥有 admin 库权限的用户同样拥有其他所有库的权限. 有一些全局性的命令也只能在这个库下执行, 例如显示所有的库, 以及关闭 MongoDB 等.
- local :  这是一个绝对本地库, 复制集不会对 这个库进行复制管理.
- config : 当数据库开启分片设置时, 这个库可以用来保存与分片有关的数据.

将库的名称和集合的名称通过 `.`符号连结起来, 就构成了这个集合的命名空间. 命名空间的长度限制为 121 字节.  

### 启动 MongoDB
MongoDB 是一个网络服务程序. 可以通过执行 mongod 来启动服务.

无参数直接运行:
mongod 使用默认的数据文件夹 : /data/db.  需要注意, 执行 mongod 的用户应该也对数据文件夹具有读写权限.    
使用默认的端口号 : 27017 . 如果端口已被占用, 启动会失败. 比如说被另一个 Mongo实例占用    
mongod 同时会配置一个最基本的 HTTP server. 以便通过浏览器查看数据库的相关信息. web 服务默认的端口号是: `数据库主端口号 + 1000` .

#### MongoDB Shell 简介
MongoDB 内嵌了 javascript shell, 以便用户使用命令行与 MongoDB 进行交互.

通过执行 `mongo` 来启动 shell. 启动时它会自动链接 MongoDB 服务. 它具有一个 js 解释器的全部特性, 可以执行任何 js 程序.

>tips: 执行分行的 java 语句时, shell 提示符会变成 `...`. 如果需要中断未完成的输入, 连续输入三次回车键即可.

#### MongoDB 客户端
上述的 javascript shell 就是一个独立运行的 MongoDB 客户端. 启动时, 默认将自动链接本地的 MongoDB 服务程序中的 test 库, 并将该库分配给全局变量 `db`.
此外, 它还会提供一些便捷的管理命令, 这些命令非常类似于 SQL 的 shell 命令.  


#### 基本的 CRUD 操作 (略)

### 数据类型
JSON 支持的数据类型有 6 种 :  null, boolean, number, string, array, object

MongoDB 在 JSON 的基础上增加了一些数据类型 :

1. 日期类型
2. 正则表达式
3. ObjectID
4. 二进制数据
5. javascript 代码

关于基本数据类型, 需要注意的地方 :

- null : 它不是字符串`"null"`, 而是不带引号的 `null`. 既可以表示一个字段的值是空值, 也可以表示一个不存在的字段. 注意不是字符串

- number : 在 Mongo 的 shell 中默认使用的是 64-bit 浮点数, 比如 `{"x" : 3.14}` 或 `{"x" : 3}`

 对于整数, 使用 NumberInt 或 NumberLong, 分别表示 32-bit 和 64-bit 带符号整数, 比如`{"x" : NumberInt("3")}` 或 `{"x" : NumberLong("3")}`


**日期类型 Date**
日期类型在数据库中仅保存UNIX时间的毫秒数.    
但是在显示的时候会以当地时区设置来显示. 如果需要指定时区, 则需要使用另外一个字段来保存时区信息.    

**数组**
数组中的值, 是有序的, 可以看作是 List, 也可以作为无序的 Set 来使用.    
数组中的数据类型可以不一样.    
查询操作可以探查到数组内的元素, 而不是仅仅对整个数据作匹配.    
数组的更新操作可以原子化.    

**内嵌文档**
查询操作可以探查到内嵌文档的内部结构.

**ObjectID**
\_id 字段的默认类型就是 Objectid.    
ObjectId遵循轻量化设计, 而且可以在不同的机器上快速的生成全局唯一的ID( 比如在集群上自动生成 ID ). 传统的自增 ID 应用在集群环境中会很复杂且影响性能.    

ObjectId 的长度 : 12 字节, 以其16进制数的字符串的方式来表示, 字符串长度为 24 .  需要注意,  ObjectId 只是在显示的时候是 24 长度的字符串, 实际存储的时候仅仅是 12 字节长度.    

ObjectId 的组成 :

|0\|1\|2\|3|4\|5\|6|7\|8|9\|10\|11|
| :-----: | :--------: | :----: | :---: |
|Timestamp | Machine | PID | Increment |

高位的 4 个字节, 使用标准秒级UNIX时间戳.    
Machine 的值根据 hostname 的哈希生成.    
PID 的部分用于保证同一台机器上的不同进程生成的 ObjectId 不会出现冲突.    
最后部分的 3 字节是递增值. 最大支持 16777216 个值, 也就是说, 同一个进程再同一秒内生成的 ObjectId 数的上限是 一千六百多万个.

当插入数据的时候, 用户没有指定 \_id 字段的话, MongoDB 将自动生成一个 ObejctId 作为其 \_id 的值.  虽然服务器也可以完成自动生成 ObjectId 的工作, 但通常情况下这是由客户端的驱动完成的. 这也是符合 MongoDB 的哲学的: 有可能在客户端完成的工作, 就不放在服务端去做, 以便最大程度的提高数据库的易扩展性, 减轻数据库扩展时的负担.


#### 使用 MongoDB Shell

使用 mongo 工具连到任意 mongod 实例上的方式:
`$ mongo [Target HostName or IP]:[port]/[DB Name]`

启动 mongo shell 但不自动连接到任何 mongod 服务上:
`$ mongo --nodb`
之后可以进行手动连接 mongod 实例:
```
> conn = new Mongo("[host]:[port]")
> db = conn.getDB("[database name]")
```
在 shell 中输入 `help` 命令 , 可以查看内置的帮助信息.

如果想要直到一个函数是什么作用, 也可以将其输入, 但不打括号, 直接回车, 会显示函数体的源码.  通过源码可以很方便的直到函数的功能以及其参数列表.

#### 使用 MongoDB shell 运行脚本

使用 mongo shell 执行脚本的方式 :
`$ mongo script1.js script2.js script3.js`
mongo 会顺序执行所列出的脚本, 执行完后退出.

如果脚本的执行需要使用 指定的 db , 可以如下方式启动:
`$ mongo --quiet host_name:port/dbname script1.js script2.js`
这样, 在脚本中就可以使用所指定的  db 进行数据库的操作了.
注: 以上命令中的 `--quiet` 可以让 mongo shell 在连接上的时候不显示版本和连接信息.

除了在启动时指定要执行的 js 脚本外, 也可以在 shell 的交互模式下手动加载 js 文件来执行.
`> load("script1.js")`
`script1.js executed!`
`> `

脚本虽然可以直接将 db 当作全局变量来进行访问, 但是其他由 shell 本身提供的一些 辅助性命令在 js 脚本中是不支持的. 比如 `use db` 和 `show collections`
可以参考以下对应关系, 在脚本中需要使用 js 支持的语法
`use foo`  -->  `db.getSisterDB("foo")`
`show dbs` -->  `db.getMongo().getDBs()`
`show collections` --> `db.getCollectionNames()`

可以使用 js 脚本编写一些辅助函数, shell 中加载初始化后, 可以直接使用. 也可以使用脚本执行一些通用的任务.

需要注意, 脚本路径如果是相对路径, 是相对于启动 shell 时的所在路径. 尽量使用绝对路径.

另外, 很重要的一个技巧: 在 shell 中可以执行 linux shell 命令 :
```
> run("ls", "-l", "/home/myUser/my-scripts/")
sh70352| -rw-r--r-- 1 myUser myUser 2012-12-13 13:15 defineConnectTo.js
sh70532| -rw-r--r-- 1 myUser myUser 2013-02-22 15:10 script1.js
sh70532| -rw-r--r-- 1 myUser myUser 2013-02-22 15:12 script2.js
sh70532| -rw-r--r-- 1 myUser myUser 2013-02-22 15:13 script3.js
```
但是这种方式只能执行一些简单的命令. 输出格式也不是很好看.


#### 创建 .mongorc.js 脚本

.mongorc.js 脚本在每次启动 mongo shell 的时候都会执行, 类似与 linux 的 .bashrc

.mongorc.js 应该创建在用户的 home 文件夹中.

该脚本通常用于设置一些有用的全局变量, 别名, 或者覆盖一些内置的函数.
最常用的就是使用 mongorc.js 去掉一些很危险的 shell 命令. 比如 dropDatabase, deleteIndexes.
示例 :
```
var no = function() {
	print("Not on my watch.");
};
// Prevent dropping databases
db.dropDatabase = DB.prototype.dropDatabase = no;
// Prevent dropping collections
DBCollection.prototype.drop = no;
```

如果不希望在启动的时候执行 .mongorc.js 脚本, 启动时加上 `--norc` 即可.



#### 在 mongo shell 中配置其他编辑工具
在 shell 中输入多行命令时, 无法修改之前输入的行, 使用起来很不方便. 我们可以配置其他的编辑工具来处理多行输入的情况..
设定 `EDITOR` 变量可以实现该功能. 这可以放在 mongorc 中实现, 也可以在进入 shell 后手动配置 :
`> EDITOR="/usr/bin/vim"`
配置后, 可以使用该第三方的编辑器对变量进行编辑. 如下 :
```
> var wap = db.books.findOne({title: "War and Peace"})
> edit wap
```
在 vim 中修改好后 `:wq` 大法保存退出即可返回到 shell 中.


#### 在 shell 下特殊集合名称的访问
常用的 db._collectionName_ 并不是万能的, 比如我们有个集合是 version, 使用这种方式访问的话是 `db.version`. 但是 db 这个变量有个成员方法叫做 version. 因此 它返回的不是 version 集合, 而是 version 这个函数的源码.

此时需要使用万能方法: `db.getCollection("version")` 的方式来访问集合.
另外一种方式是使用: `db["version"]` 格式访问集合. 也可以避免集合名包含特殊字符带来的问题.


---------------------

### 创建, 更新 和 删除 文档

####  数据批量 insert :
当前版本(2.4)支持的最大消息长度是  48MB, 所以批量插入的数量将受此限制.

需要注意: 批量插入过程中, 如果中间的某个文档插入失败导致操作执行中断, 已经插入的文档将不会被撤销. 插入失败的文档以及其之后的文档, 都没有插入到集合中.
如果希望插入过程不会因为个别操作的失败而中断, 可以使用 `continueOnError` 选项. 这样会将所有可以正常插入的文档全部完成插入操作.
shell 不支持 `continueOnError` 选项. 需要使用驱动.


数据插入时的验证:
MongoDB 在插入操作时的数据验证非常简单: 只检查文档基本结构 和 是否存在 \_id 字段.
基本结构验证中即包含对文档的大小验证: 单个文档不能超过 16MB. (以后的版本中可能会提高该上限).
查看一个文档的 Size 可以在 shell 中使用 `Object.bsonize(doc)`, 返回值的以字节为单位.



#### 文档删除:
特别注意一点! `> db.collectionName.remove({})` 命令将删除整个集合中的所有文档. 并且不会有交互性的确认.

对于大量数据的删除, 千级或万级以上的删除, 需要注意性能问题.  PC 测试中的删除速度是 百级文档每毫秒.
如果需要删除整表, 类似于 MySQL 中的处理方式, 最好 drop 整个集合, 然后重新创建集合和所需的索引.

remove 进行删除不影响集合的一些元数据 (meta data) ,  而使用 drop 方式则会删除所有 元数据.


#### **文档更新  ( 重要 )**

文档更新操作是原子性的. 对于一个文档, 只有一个更新操作完成,才会执行下一个更新操作.

文档替换式更新 :  用于整个数据集合中的数据结构发生很大的变动的时候.

执行更新操作时, 最好使用唯一ID, 比如 \_id 字段作为查询条件, 以确保修改的文档是我们所期望的.

对于文档的某一部分进行更新, 可以使用更新操作符.  

注意, 使用操作符更新文档时, \_id 不能发生改变. 但是使用文档替换式更新的时候, 是可以改变 \_id 的.


**"\$set"** 与 **"\$unset"** 操作符

$set 操作符可以用于给文档新增字段, 修改字段内容以及修改字段数据类型.
$unset 与之相反, 用于将一个已有的字段删除.

基本用法:
```js
// $set
db.coll.update({"_id" : 1234}, {"$set", {/*将想要设定的属性都写在此处*/}})

// $unset
db.coll.update({"_id" : 1234}, {"$unset", {"fieldA": 1, "fieldB": 1}})
// 字段后的 1 也可以换成其他值, 效果一样. 甚至是布尔值 false
```

**"\$inc"** 操作符
使用的语法与 \$set  类似,  用于给 Number 数据类型进行 增加或减少值.  不能用于其他数据类型.  
注意, 如果 $inc 的字段不存在, 执行更新指令时会自动创建.
```js
// 增加
> db.coll.update({"_id": 1234}, {"$inc", {"score" : 3}})

// 减少
> db.coll.update({"_id": 1234}, {"$inc", {"score" : -1}])
```

**"\$push"**,  **"\$each"**,  **"\$slice"(update)**, **"\$sort"** 操作符
给数组 push 一个新的元素到尾部, 如果指定的数组字段不存在则自动创建.
用法类似于 $inc. 只是将数值字段换成数组字段, 将数值换成数组元素.

此外, 该操作符还支持一次性 push 多个元素到一个数组中. 需要配合 `"$each"` 操作符使用:
```js
// 向 ary 字段的数组依次添加 "abc" 和 "def" 两个字符串元素
> db.coll.update({"_id": 1234},
				 {"$push": {"books": {"$each": ["abc", "def"]
									 }
							}
				 })
```

对于数组 Size 可以有一个总数的限制. 使用 `"$slice"` 操作符指定.  如果是正值, 则保留前N个, 如果是负值, 则保留后N个.
```js
// 添加元素, 同时限制 ary 数组的最大长度为 6,
// 如果超过, 则保留数组的前 6 个值.
> db.coll.update({"_id": 1234},
				 {"$push": {"books": {"$each": ["qwe", "uio"],
									  "$slice": 6
									 }
							}
				 })
```

除了按照先后顺序对数组进行裁剪外, 还可以在 push 完元素后进行排序, 然后再裁剪. 这个功能由 `"$sort"` 操作符实现.    如果数组中的元素是嵌入的文档, 可以指定用于排序的字段.
```js
// 添加元素, 限定 Size 为 6, 超出时裁剪前 6 个
// 裁剪前, 先对数组的值进行逆序排序
> db.coll.update({"_id": 1234},
				 {"$push": {"books": {"$each": ["qwe", "uio"],
									  "$slice": 6,
									  "$sort": -1
									 }
							}
				 })
```



`"$slice"` 和 `"$sort"` 操作符需要和 `"$each"` 操作符同时使用.
如果不需要追加, 只需要将现有的数组裁掉一部分, 也必须有 `"$each"` 操作符, 此时指定其值为 `[]` 即可.
如果只需要对数组进行排序, 也可以按上述方法进行处理.


**将 数组 视为 集合(Set)**

使用 `"$addToSet"` 向数组中添加元素时, 如果该元素已经存在, 则不会重复添加.

`"$addToSet"` 也和 `"$push"` 一样, 可以与 `"$each"` 连用, 用于一次性插入多个元素.


**删除数组元素**

如果数组是作为队列或者栈这样的有序结构使用, 可以使用 `"$pop"` 操作符, 他可以从任意一端删除一个元素.
`{"$pop" : {" key" : 1}}` 是从数组末尾删除.
`{"$pop" : {" key" : -1}}` 是从输入开头删除.

也可以不根据元素在数组中的位置进行删除. `"$pull"` 支持根据元素的内容进行删除.
语法结构 :
```
> db. lists. insert({"todo" : ["dishes" , "laundry" , "dry cleaning" ]})
> db. lists. update({}, {"$pull" : {"todo" : "laundry" }})
```
`"$pull"` 会删除所有匹配指定条件的元素, 所以需要注意要删除的元素是否具有唯一性.


通过指定下标访问数组

1. 直接使用元素的下标访问
```js
// comments 字段的值是数组, 其中的元素时内嵌文档, 文档中有 votes 字段记录投票数
// comments.0.votes 对应 comments 字段中的 0号元素的 votes 字段
 > db. blog. update(
	 {"post" : post_id},
	 {"$inc" : {"comments.0.votes" : 1} } )
```
2. 使用 `"$"` 操作符指定元素
	该操作符用于相当于是查询条件中所指定的元素的下标.
	如果查询条件中的数组元素, 经查询 Mongo 得知其下标为 i , 则在 update 对象中, 该数组的位置操作符 $ 就会被替换为 i 的值.
	因此, 需要特别注意, 这种方式对于有重复元素的数组, 只会对对一个找到的元素生效.


操作符的速度:

`"$inc"`	由于只改变某个数字的值, 不会改变文档大小, 因此速度非常块.
数组相关的操作符涉及到改变文档的大小, 相对要慢一些.
`"$set"`	如果只是对现有字段值的改变, 不影响文档的 Size, 速度比较块, 但是如果涉及文档大小的改变, 速度会慢一些.

之所以会有上面的问题, 是因为改变文档的 Size 后, 需要在磁盘上移动该文档的位置.
正常情况下, 连续插入的数据在磁盘上是连续存放的, 如果之前已经插入的数据 Size 发生变化, 原有的空间已经不足以存放该文档, 需要给它重新安排存放空间. 这样之前的存储空间就会空出来.
这种现象会影响集合的 `padding factor` 大小, 使得它越来越大, 存储空间的利用率也会慢慢变低.
集合的 padding factor 可以通过 `db.coll.stats()` 查看. 其理想状态应该是 1.
在PC上的测试可以看到 `"$inc"` 的速度比 `"$push"` 差不多要快 10 倍.
当集合性能遇到瓶颈时, 需要评估是否由于类似 `"$push"` 这样的数组操作引起, 如果是的话, 可能需要考虑将该数组抽出来作为一个单独的集合进行管理.

对于已经产生的碎片空间, MongoDB 无法在进行利用. 如果这种碎片过多, 可能会造成磁盘数据文件的大块空白, 此时在 Mongo 的日志中会出现类似如下的信息 :  
> Thu Apr 5 01:12:28 [conn124727] info DFM::findAll(): extent a:7f18dc00 was empty, skipping ahead
这个不会影响使用, 但是建议在看到该消息的时候, 考虑对数据进行压紧.

数据压紧过程中需要对数据进行大量的插入和删除操作( 移动数据的位置 ), 建议通过使用 `usePowerOf2Sizes` 选项对磁盘进行优化:
```js
> db.runCommand({"collMod": [collectionName], "usePowerOf2Sizes": true})
```
这样接下来的所有分配的磁盘区块都将是 2 的指数大小.
不要随意使用该优化, 因为对于插入和更新频繁的集合, 这种方式会影响写入性能.

更新插入 ( upsert)

更新插入的时候, 创建的文档是由本次更新操作的 *查询匹配条件* 和 *更新内容* 一起构成的.

`"$setOnInsert"` 操作符用于 upsert 的情况. 当查询会发现文档不存在, 将会插入新的文档, 而 setOnInsert 中指定的字段只有在插入操作中才会被设置到文档中.
```
> db.users.update({}, {"$setOnInsert" : {"createdAt" : new Date()}}, true)
```
该操作符可以用来在 upsert 的情况下设置 padding, 设置计数器, 或生成 \_id.

** Shell 中的 save 操作 **
对于 save 操作需要注意一点:  文档存在与否是看是否找到集合中存在一个 \_id, 和 save 的参数中指定的文档中的 \_id 字段相等. 如果匹配到, 则执行更新操作, 如果没有匹配到, 则执行插入操作.

更新操作默认更新一个文档, 其他的文档不会做任何更改. 如果需要更新所有匹配查询条件的文档,  将 update 命令的第四个参数 (multi) 设置为 `true` .

更新后, 可以适用 `getLastError` 命令查看最后依次命令执行的结果, 返回的 doc 中有个 "n" 字段, 用于表示操作影响的文档数.
```
> db.count.update({x : 1}, {$inc : {x : 1}}, false, true)
> db.runCommand({getLastError : 1})
	{
		"err" : null,
		"updatedExisting" : true,
		"n" : 5,
		"ok" : true
	}
```
`getLastError` 在实践中是非常有用的命令. 后续还会再谈到该命令.


**findAndModify :**
解决多线程下, 查找并在其后对文档进行修改更新时的竞争问题.
适用方法:
```
> ps = db.runCommand({"findAndModify" : "processes",
... "query" : {"status" : "READY"},
... "sort" : {"priority" : -1},
... "update" : {"$set" : {"status" : "RUNNING"}})

{
	"ok" : 1,
	"value" : {
		"_id" : ObjectId("4b3e7a18005cab32be6291f7"),
		"priority" : 1,
		"status" : "READY"
	}
}
```
"findAndModify" 字段指定该操作对应的集合;
"query" 和 "sort" 字段指定查询和排序条件.  该例中要对 priority 最高且状态为 Ready 的 process 进行处理;
"update" 字段指定要执行的更新操作.

返回的文档中, value 的值是上述命令执行时将要执行更新的文档. 其中的数据都是执行更新前的数据. 但是当已经得到这个成功返回的文档时, 文档已经被成功更新过了.

因此可以理解为这是 : 1. 查找;  2. 记录文档内容;  3: 更新文档;  4: 返回更改前记录下的文档状态  这样一组操作合并后的原子操作.

除了通过该方式对文档做更新操作外, 还可以对文档做删除操作 :
```
ps = db.runCommand({"findAndModify" : "processes",
	"query" : {"status" : "READY"},
	"sort" : {"priority" : -1},
	"remove" : true}).value
do_something(ps)
```

如果希望 findAndModify 返回的文档是修改后的文档, 可以通过 增加一个 `new ` 字段, 并置为 true

#### **设定 Write Concern**

write concern 用来设定写入级别.
写入没有完成的情况下, 程序通常是被 block 的. 如果写入失败, 程序通常会抛出异常.
该设定有很多选项, 最基本的有两个:  acknowledged ( 默认值 ) 和  unacknowledge.
acknowledged 表示程序需要 Mongo 返回是否写入成功的消息.
unacknowledged 表示程序不需要 Mongo 对写入做出响应.

对于写入错误的检查 : 需要注意一点, Shell 下一次执行多条指令时, 如果中间的某条指令执行失败, 是不会给出错误提示的, 因此如果要编写在 shell 下执行的脚本, 需要在写入操作后添加 `getLastError` 命令来确认上一条指令的执行结果.



### 查询 ( 重要 )

简单的查询使用  find 或  findOne .
查询参数是一个 文档, 其中包含了查询条件, 均以键值对的方式给出.  文档中的各个条件之间相当于是与逻辑( AND ).
如果文档为空, 即一个 `{}` , 则该查询条件可以匹配集合中的任何文档.
如果参数为空, 比如 `db.coll.find()` 则默认查询条件是 `{}`

指定文档的返回格式, 也就是返回的文档包含哪些字段时, 需要注意几点:

- 默认情况下 \_id 字段 是一定会返回的.
- 可以指定包含字段, 也可以指定不包含字段, 也可以混合使用
-  当明确指定排除 \_id 字段时, 将不再返回  \_id 字段
- shell 中默认使用 1 表示包含, 使用 0 表示不包含. 某些语言的驱动中也支持使用布尔值表示.

使用 `"$lt"` `"$lte"` `"$gt"` `"$gte"` 进行范围查询. (很简单, 略)
注意, 对于 date 类型的数据, 也可以直接使用 以上方式进行比较, 因为实际存储的是UNIX TIME.
其他相关操作符:
`"$ne"` 查询的字段不等于给定的值.
`"$in"` 查询的字段符合给定数组中的任意一个值.
`"$nin"` 查询的字段不等于给定数组中的任意值
`"$or"` 如果可能的话, 尽量不要使用 `$or` , 因为查询优化器不对它进行优化.
`"$nor"`  ....
`"$and"`  与条件, 多此一举. 而且查询优化器不对其进行优化
`"$not"` 元条件符, 可以和其他任何条件符合用, 表示逻辑非.
`"$mod"` 乱入一个运算类的, 求模运算.  `{"$mod" : [5, 1]}` 表示模5余1. 依此类推.

对于不同类型操作符/条件符出现的位置:
所有查询条件类的 \$xx 一定出现在一个文档内, 即, \$xx 作为 key , 条件值作为 value .
对于更新操作类的 \$xx 一定处于一个相对外部的文档中, \$xx 是一个文档的 key, 文档是更新执行内容.

 **类型查询**

比较重要的是 null. 它不仅匹配他自己, 也就是 null, 而且还匹配该字段不存在的情况.
如果需要区分该字段为 null 的情况, 和 该字段不存在的情况. 可以使用:
`"$exist"`  其值为  true 或 false  分别表示字段存在 和  字段不存在. 例:
`> db.c.find({"z" : {"$in" : [null], "$exists" : true}})`


**正则表达式查询**

MongoDB 默认使用 Perl Compatible Regular Expression Library 对正则表达式进行解析.
注意 MongoDB 的索引支持前缀查询, 这一点同样适用于正则表达式, 因此必要的时候在正则前加 "^" 符号. 另外注意如果开启了 大小写不敏感 的选项, 将无法利用索引进行查询.
如果数据中存在正则表达式 , 比如  `{"_id" : 1, "name" : /abc/ } ` 则在使用 /abc/ 作为正则查询时, 也可以匹配 该正则表达式本身.

**数组查询**

默认查询方式中, 查询条件的 value 是一个元素, 而不是数组.  此时只要数组中有一个元素能与该元素匹配, 就认为查询匹配.

需要数组字段同时满足多个元素的匹配时, 需要使用  
`"$all"`  其值是一个数组, 列出所有必须匹配的元素.  例:
`> db.food.find({fruit : {$all : ["apple", "banana"]}})`
 这将仅匹配 fruit 数组既包含 apple 也包含  banana 的文档.  *顺序无所谓, 此处对数组的使用类似于 set*

如果去要对一个数组字段严格匹配, 则可以在查询条件中直接使用 key : [ele1, ele2, .... ] 的形式, 其中, value 处的数组和要查询的目标文档的数据, 元素 与 元素的顺序, 都必须完全一致, 否则无法匹配.

匹配数组中的特定下标的元素  .   使用   `[key].[index] : value` 的形式作为匹配条件.

`"$size"`  查询时通过该条件符指定某个数组字段的数组长度为查询要求的值. 例:
`> db.food.find({"fruit" : {"$size" : 3}})`  查询 fruit 字段的数据长度为 3 的文档.
不过该条件符有使用限制:  他不能和其他的条件符, 比如 "$lt" 合用, 因此, 如果查询中需要对数组的长度做范围查询, 需要设置一个 size 字段记录数组长度, 每次对数组进行添加或删除元素时, 使用 `"$inc"` 同时对它进行修改.
`"$addToSet"` 不能使用上述 方式记录 size . 因为有可能出现重复元素.

`"$slice"` : 用于截取数组.
```js
// 返回的文档中, comments字段只取 comments 中的 前 X 个 元素
> db.coll.findOne(criteria, {"comments" : {"$slice" : X}})

// 返回的文档中, comments字段只取 comments 中的 后 X 个 元素
> db.coll.findOne(criteria, {"comments" : {"$slice" : -X}})

// 返回的文档中, comments字段只取 comments 中 从offset(包含)开始的 count 个元素
> db.coll.findOne(criteria, {"comments" : {"$slice" : [offset, count]}})
```

`"$"`  使用 "$" 作为查询条件所匹配到的元素下标的 替换变量.  例:
```
> db.blog.posts.find({"comments.name" : "bob"}, {"comments.$" : 1})
{
	"_id" : ObjectId("4b2d75476cc613d5ee930164"),
	"comments" : [
		{
			"name" : "bob",
			"email" : "bob@example.com",
			"content" : "good post."
		}
	]
}
```
该例中, 要查询 bob 的评论, 并且只返回这条评论.  我们在查询前不知道该评论的下标, 因此只能用姓名作为查询条件.  执行查询后, 需要将对应下标的 comments 元素返回, 因此我们需要适用 `"$"` 符来表示查询匹配到的元素的对应下标.
当然, 如果存在多个 bob 的评论, 这种方式将只能返回第一个查询到的评论.

数组查询的一个问题:
适用大于小于等进行范围查询时, 对于需要多条件同时满足的情况, 比如 大于100 小于500 这样, 在遇到数组元素时, 其行为是: 对于查询条件指定的条件, 有任意元素能够满足其条件即可, 不要求是同一个元素满足所有条件. 这通常和人们的预期相违背.
对于这种问题, 可以适用  `"$elemMatch"` 条件符解决.  但是这种方式只能用于纯粹的数组字段, 即, 该字段下不会出现其他类型的 value .
除此之外, 如果该字段建立了索引, 可以使用 `min()` 和 `max()` 来限定索引的边界, 这将会提高性能.


**查询内嵌文档**
 两种方式: 查询匹配整个内嵌文档,  查询匹配内嵌文档中的某些字段.
 匹配整个文档 :
 `> db.people.find({"name" : {"first" : "Joe", "last" : "Schmoe"}})`
 注意子文档需要完全匹配.  包括字段的顺序!
 推荐使用内嵌文档的字段匹配
 `> db.people.find({"name.first" : "Joe", "name.last" : "Schmoe"})`
 内嵌文档的字段访问其实就是一个 `"."` 符号的使用. 从这里可以看出, 字段名不能包含点符号的原因了.

对于较为复杂的匹配情况, 比如要匹配的条件需要在同一个内嵌文档中被满足时.  
`"$elemMatch"` 又可以派上用处了. 用法详情此处略...


**"$where"** 查询
$where 方式的查询允许自定义查询条件.  使用 javascript 函数作为查询的 hanler .  
查询条件灵活的代价就是查询效率很低, 因此不到万不得已, 不使用它.
具体的使用方式 略...

对于复杂查询, 除了使用 $where 以外, 还可以适用  aggregation (聚合查询). 后面章节介绍.


**服务端脚本**
服务端脚本的执行存在一定风险, 搞不好会出现类似关系数据库中 SQL注入的风险.  
执行Mongod 的时候可以通过添加参数 `--noscripting` 来关闭 脚本执行功能.

为了解决上述问题, 当服务端执行的脚本中有变量时, 需要使用 scope 来传递变量的值. 这样所有的用户输入部分都会看作是一个独立的字符串, 不会出现单引号或双引号没有转义的情况.



#### **游标**
当查询执行 find 时, 游标并没有马上去查询数据库. 直到你真的试图去读取出其中的某个数据, 比如调用 hasnext() 或者 next() .
游标支持链式调用, 常用的有 limit() , sort() , skip() 等. 顺序无所谓.
另外一点: 游标到数据库取数据, 并不是一次把所有数据都取出来, 而是先取 100 条 或 4MB (哪个条件先满足就用哪个条件). 用完后再去数据库取一批数据 ( 通过发送 getMore 请求 )
skip 过多的数据会影响性能. 对于大范围的数据浏览, 可以适用其他方式取代 skip 的功能.
skip 影响性能的原因: 底层实现上是先找到所有需要的数据, 然后将 skip 的部分丢弃掉.
可以使用前一次的查询结果更快的定位到 下一次要取的数据位置. (用于记录位置的字段要使用排序用的字段)


**比较顺序**
Mongo 的不同数据类型之间也会有比较排序, 可以分出谁大谁小.
不同类型之间的大小顺序按照如下方式定义:   ( 从小到大 )

1. null
2. Numbers
3. Strings
4. Objects / Documents
5. Array
6. Binary Data
7. ObjectID
8. Boolean
9. Date
10. TimeStamp
11. Regular Expression


**获取一个随机文档**
略.....

**高级查询选项**
`$maxscan` : 指定最大扫描文档数量, 超过该数量后不再扫描, 直接返回已经找到的结果.
`$min`  `$max` 指定索引边界. 相当于 min() 和 max()
`$showDiskLoc` 查询结果中返回该文档所在的磁盘文件, 以及在文件中的字节偏移量.

snapshot() 查询, 性能较差, 但可以保证在查询过程中不会被插入或修改了的文档影响到一致性. ( mongodump 就是采用了 snapshot 查询 )

**不过期的游标**
客户端游标使用完毕后, 需要通知服务端将对应的服务端游标资源释放.
释放资源的几种情况:

1. 游标迭代完成
2. 游标访问超出范围
3. 超时

如果业务上确实需要长时间持有一个游标, 就不能让它快速过期. 可以利用不过期的游标, 不同的驱动可能实现方式不太一样.  例如 immortal 函数.
如果关闭了游标超时, 一定要将游标完全迭代完, 或者手动将其关闭. 否则服务端将为此持有资源, 永不释放, 除非重启服务端.


#### **数据库命令**

许多数据库函数和指令， 比如 shutdown dropdatabase 等最终都是转为数据库命令的方式执行。
数据库命令执行格式:
```js
// 使用数据库命令的方式删除一个集合
> db.runCommand({"drop" : "test"});
{
	"nIndexesWas" : 1,
	"msg" : "indexes dropped for collection",
	"ns" : "test.test",
	"ok" : true
}

// 使用 shell 命令删除一个集合
> db.test.drop()
```

当使用一个老版本的 Client 连接到一个比较新的版本的 Server 上的时候, 可能某些新功能 旧 shell 并不支持, 此时便可以使用 数据库命令的方式执行特定的命令.

可以通过执行 `db.listCommands()` 命令来列出所有支持的命令.

`"ok"` : 返回值的该字段表示命令是否执行成功. 1 或者 true 表示成功;  0 或者 false 表示失败.
`"errmsg"` 如果命令执行失败, 则会有该字段, 来说明执行失败可能的原因.

数据库命令的底层实现方式:  它其实只是一种特殊的查询语句, 所查询的集合是 `$cmd`.  runCommand 不过是从中取出了一条命令文档. 比如前文中的删除集合的命令可以写成:
`db.$cmd.findOne({"drop" : "test"});`
执行形式虽然很类似, 但是数据库对命令的处理逻辑是不同的.

使用数据库命令 `runCommand()` 通常需要管理员身份, 并且应该处于 admin 库下 ( 用 use admin 命令切换 ).  
如果不切换到 admin 库, 要执行数据库命令, 需要使用 `adminCommend()` 来执行.

另外需要注意, 在 Command 中, 命令文档对字段顺序敏感, 命令名称字段一定要放在最前面.



## **3 设计你的应用程序 **

### **索引**

 **这一部分可以参考我之前索引这部分~~[官方文档的笔记](http://blog.csdn.net/lachening/article/details/44240097)~~, 重复的内容不再赘述**

创建索引(或其他操作)中, 如果长时间没有返回, 切严重影响数据库性能, 可以重开一个 shell , 使用 `currentOp()` , 找到正在执行的操作进程.

复合索引 ( 联合索引 ) 的内部存储结构示意:
```js
// 创建复合索引
> db.users.ensureIndex({"age" : 1, "username" : 1})

// 索引的存储结构示意
[0, "user100309"] -> 0x0c965148
[0, "user100334"] -> 0xf51f818e
[0, "user100479"] -> 0x00fd7934
...
[0, "user99985" ] -> 0xd246648f
[1, "user100156"] -> 0xf78d5bdd
[1, "user100187"] -> 0x68ab28bd
[1, "user100192"] -> 0x5c7fb621
...
[1, "user999920"] -> 0x67ded4b7
[2, "user100141"] -> 0x3996dd46
[2, "user100149"] -> 0xfce68412
[2, "user100223"] -> 0x91106e23
```

 Mongo 通过 find() 找出一个结果集合,  如果其Size 大于 32MB 再对其进行无索引的排序, 将会报错.

对于复合索引, 优先将区分度高的字段放在前面, 可以获得更好的性能.
或者是将要用于 排序的 key 放在索引的前面.

如果 find 和 sort 使用的字段, 与复合索引字段逆序, 可以使用 hint 强制按照索引本身的顺序扫描.
比如:   索引为: `{A : 1, B : 1}`  查询使用  `find( B条件 ).sort( A条件 )`, 此时将先按照 B条件将所有复合查询条件的文档取出到内存, 然后再在内存中进行排序.  如果B是范围条件, 排序的效率会很低.
对其进行优化:  `find( B条件 ).sort( A条件 ).hint({A : 1, B : 1})` , 此时将会进行索引的逐个扫描, 找出所有复合条件的文档, 虽然需要遍历索引, 但是取到的文档已经是有序的了, 节省了内存中对大量数据进行排序这个非常耗时的操作.
注意, 以上优化针对的情况是一次只取出一小批数据, 使用 limit 限制返回的文档数量.  
如果不加 limit, 则可能第二种查询比第一种查询更耗时. 总的来说第二种查询随着返回的文档数的减少而减少, 但第一种查询不会随着返回的文档数的减少而减少, 甚至只查询排序最靠前的一个文档, 也要全表扫描并内存中排序.

如果 B树上插入的新节点总是依次增长的, 比如 \_id 默认的 ObjectID, 那么树上的所有新节点都将加在树的最右边, 这种树称为 "右平衡".  实践中, 需要尽量让索引保持 "右平衡" 性.




有些查询无法使用索引, 比如 `$where` 查询, 以及 `$exist` 检查指定的 key 是否存在.
另外有一些查询可以使用索引, 但是效率上没有那么高.

`$exist` 的使用. 查询一个不存在指定 key 的文档:
```js
db.coll.query({age : {"$exist": false}})
```

通常来说, 所有 "非" 逻辑的查询效率都很低, 即使是在索引上, 使用 `$ne` 也许要逐个检查整个索引, 以找到所有不为特定值的文档.

对于索引来说, 存储一个不存在的字段, 和存储一个值为 null 的字段是一样的. 但是对于稀疏索引, 这种查找都无法利用索引.

技巧: 编写查询条件时可以检查, 是否可以通过增加额外的查询条件, 来利用索引将需要扫描的数据集合尽可能过滤并减小.


首先要考虑的是合理安排复合索引的顺序.   索引创建以后, 在实际业务下需要考虑合理安排查询条件, 这样才能更好地利用好索引.

**`$or`查询**  :
`$or` 查询的本质: 执行两条查询语句, 然后将两个查询的结果进行合并.  实际执行效率远不如执行一条查询的效率高.   因此, 如果可能, 尽量不要使用 `$or` 查询, 而使用其他的比如 `$in` 替代.

`$in` 查询 :
使用  `$in` 查询时, 查询条件中的数组中元素的顺序不会影响返回结果的排序, 结果集的排序只和 sort 条件有关系.


如果不是必须, 则尽量不要在数组字段上建立索引, 因为开销比普通索引要高. 另外, 不能在多个数组字段上建立多字段的复合索引




### **特殊索引和集合类型**
... ...
### **聚合**
... ...
### **应用程序设计**
... ...



## **_4 复制集_**

复制集就是这个数据库的另外一个或若干个拷贝或复制, 或是影子.  类似于磁盘的 RAID0镜像.

**目的:**

* 提高可用性, 一个库出意外挂掉, 可以自动无缝切换到另外一个
* 数据备份, 防止某个数据库出现物理损坏导致数据丢失的风险

**模式:**

* 多台数据库相互间实时同步数据, 保证相互间的数据一致.
* 一台数据库被选举为 Primary, 并向外提供服务.
* 一旦 Primary 挂掉无法提供服务, 其他 Secondary 共同选举一个新的 Primary 继续提供服务

### 用于测试的快速启动方式

... ...

### 配置复制集

* 首先需要一个 mongod 实例, 可以时空的, 也可以是已经有数据了的.
* 我们需要给它配置一个 Replication Set Name , 即给它起个名字.
* 可以在配置文件中配置 : `replication.replSetName: myReplSet`
* 然后需要重新启动该实例. ( 当然如果一开始的时候就已经配置过名字就不用了, 因此建议, 无论何时, 都以复制集成员的方式来启动一个 mongod 实例, 哪怕实际上只有一台数据库. 这样未来一旦需要配置复制集, 就不需要关掉原有的 mongod  )
* 启动其他的 mongod 实例, 当然复制集的名字要和第一个指定为相同的. 这些实例需要时新的, 没有旧数据的
* 几个 mongod 实例都启动后, 需要整个复制集进行配置, 即让他们相互之间知道彼此的存在并协同工作. 这时的配置需要在第一个有数据的实例上进行. 链接到 mongo shell 后, 创建一个 js 对象 config , 并如下配置:
```
var config = {
	'_id': "myReplSet",  // 必须和复制集名字完全一样
	'members': [
		{ '_id': 0, 'host': '127.0.0.1:3000'}
		{ '_id': 1, 'host': '127.0.0.1:3001'}
		{ '_id': 2, 'host': '127.0.0.1:3002'}
	]
}
```
* 初始化复制集:  ( 对于复制集的配置必须要在 shell 中完成, 无法通过配置文件直接进行 )
```
rs.initiate(config)
```
* 之后该节点将会完成自身的初始化, 并且将配置信息自动传递到其他节点中. 这个过程需要一段时间
* 当配置信息传播到所有节点后, 他们将会共同选举出 primary 节点, 然后开始正常的响应读写请求. ( 完成配置之前, 所有复制集成员都无法响应请求 )

完成配置后, 向 Primary 写数据, 会自动向 Secondary 同步. 我们可以查询 Primary 中的数据, 但是由于 Secondary 可能存在数据不完整的问题, 因此是拒绝访问的. 如果需要访问需对 Connection 设置 slaveOk.

几个有用的 shell 命令 :
```
rs.status()  // 显示当前的复制集配置信息 和成员状态
rs.printReplicationInfo()  // 显示当前复制集的 oplog 同步信息
rs.config()  // 获取配置对象 (前文中提到的)
rs.isMaster()
db.isMaster()  // 用来显示当前成员是否是 Primary , 以及其他相关状态信息
db.setSlaveOk()		// 设置 SlaveOk 标记.
```

#### 改变复制集的配置

复制集的配置可以在启动, 或者运行时改变, 可以添加, 删除或者是修改成员信息.
相关命令:
```
rs.add('127.0.0.1:27017')	// 添加成员
rs.remove('127.0.0.1:27017')	// 移除成员
var config = rs.config()
// 按需要修改 config 对象
rs.reconfig(config)		// 使修改后的 config 生效
```
当更改复制集配置时, Primary 会在完成重新配置后断开所有的 Connection, 然后自动重连.

如果复制集已经无法产生 Primary, 此时只能在 Secondary 上进行 reconfig 操作, 该情况下, 需要给 reconfig 一个 force 选项 :
```js
> rs.reconfig(config, {"force": true})
```
如果需要改变  host 选项, 需要在不发生变化的成员上进行 reconfig, 因为每个成员都只能接受他所能识别的成员发来的 reconfig 操作.
如果复制集的所有成员都要改变 host , 则只能全部 shutdown 后, 以单机模式启动, 并手动修改 `local.system.replset` 中的信息.

### 合理的设计复制集

**_注意!! _**
复制集的选举方式是多数通过. 分子是存活并可以参与投票的成员, 分母是整个复制集中所有成员(包括挂掉的). 分子/分母 需要大于(不等于) 1/2, 才可以进行有效的选举, 否则大家全部降级为 Secondary .
因此, 推荐的最小复制集应该有 **_3个成员_**


#### arbiter 成员

之前介绍的复制集结构中, 2个成员的复制集不具有 auto failover 的特性, 而很多场景下我们又不希望持有 3 份数据拷贝. 那么这种情况就可以使用 arbiter 成员.

arbiter 成员, 或称为仲裁, 只是用来参与投票, 而不实际保存数据.  因此合理的安排 arbiter 的部署位置, 可以提高复制集的可用性, 并减少所需机器的数量. ( 比如部署在另外一个机房的其他功能服务器上 )

添加 arbiter 的方式:
```
rs.addArb('127.0.0.1:27018')
// 或者 直接使用 json 对象
rs.add({'_id': 4, 'host': '127.0.0.1:27018', 'arbiterOnly': true})
```

注意事项:

* 不能添加带数据的 arbiter 成员, 即普通成员无法转化为 arbiter
* 已经作为 arbiter 添加的成员, 不能转化为普通成员
* arbiter 除了在投票时打破平衡外, 没有其他用处. 因此, 对于一个复制集, 最多配置 1 个就够了. 对于又奇数个数据成员的复制集, 不需要 Arbiter
* 如果条件允许, 尽量使用奇数个普通成员, 而不是偶数个普通成员+一个 arbiter.



#### 投票机制
... ...

### 复制集成员的配置

将一个成员添加到复制集中的配置非常简单, 但是我们使用 `rs.status()` 查询复制集信息的时候, 可以看到每个成员下面都有很多之前没有配置的选项. 接下来需要研究以下这些选项.

```json
// 示例片段
"members" : [
		{
			"_id" : 0,
			"host" : "127.0.0.1:27017",
			"arbiterOnly" : false,
			"buildIndexes" : true,
			"hidden" : false,
			"priority" : 1,
			"tags" : {

			},
			"slaveDelay" : 0,
			"votes" : 1
		}, ... ]
```

**Priority**

* 优先级决定了投票时成员被选为 Primary 的权重大小.
* 值介于 `[0, 100]`,  0 表示该成员是`被动成员`, 无法成为 primary;  默认值为 1.
* 注意如果将一个高优先级的成员添加到集合中, 或者之前未同步的高优先级成员完成同步, 它将会抢回 Primary 的地位, 这会造成数据库链接的短暂断开.

**Hidden**

* 隐藏后的成员无法被客户端看到. . 因此该成员不会收到来自客户端的请求
* 客户端通过 `isMaster()` 的方式获取复制集成员列表. 而 Hidden 的成员不会在 isMaster 的 hosts 属性中显示
* Hidden 的成员不会被优先投票为 Primary

**Slave Delay**

* 该属性设置了次级成员的写入延后时间.  对于人为或程序性的错误导致写入了不希望写入的数据, 可以在延迟时间内从设置了延迟的成员上恢复数据.
* 设置 Slave Delay 的成员, 其 Priority 需要设置为 0. 同时建议将其的 hidden 属性打开.

**Building Indexes**

* 关闭该属性的作用是, 让该成员不同步 Primary 节点的索引.
* 此功能一般用于纯数据备份成员, 或者是做数据批量处理的成员.
* 该属性要求成员的 priority 设置为 0,
* 该属性一经设定, 无法更改, 除非删除成员及其中的所有数据, 重新添加成员并同步.


## 复制集的构成

### 同步

1. Primary 的所有写操作日志写到 oplog collection
2. Secondary 查询 Primary 的 oplog, 在自己的数据上执行, 以实现同步
3. 通常 Secondary 从 Primary 同步数据, 事实上每个 Secondary 也有自己的 oplog, 也可以其他的 Secondary 来同步自己的 oplog
4. Secondary 启动的时候会从自己最后一条 oplog 开始向 Primary 请求 oplog.
5. 一条执行可能会被拆分成 Oplog 中的多个记录, 比如批量删除或更新. 如果 oplog 不够大, 将导致 secondary 被甩掉.

#### 初始化同步

secondary 启动的时候会判断当前是否可以从复制集中的某个成员出进行同步, 如果可以, 则同步其 oplog , 如果不行, 则会进行一次全数据的拷贝, 即初始化同步.  

步骤:

* 选择一个成员作为同步源
* 为自己在 local.me 中创建一个ID
* drop 所有 Database, 除了 local
* 从同步源拷贝所有数据库记录, 这是整个过程中最耗时的部分
* 在同步过程中, 如果源有了写入记录, 同步方将该写入操作所影响的行重新拷贝
* 当数据同步的差不多后, 将会开始创建索引, 这也将是一个很耗时的过程, 尤其是表很大的时候.
* 当完成数据同步后, 该成员才正式成为一个 Secondary 成员.

初始化同步的主要问题是:  他对同步源造成很大的负担. 全数据的拷贝导致源会将所有的数据都载入内存中, 而内存空间有限, 因此可能将业务正在使用的数据从内存中逐出.

对于初始同步时间过长的情况, 可能导致同步方被源甩掉, 因为它的 oplog 读取速度落后于源, 最终导致源的写操作覆盖了同步方尚未进行同步的 oplog.


#### 处理陈旧数据 ( 次级成员跟不上, 被甩开 )

可能的原因:

1. 次级成员 shutdown 导致拉开过大距离.
2. 次级成员需要处理过多的读请求, 导致其不能及时完成同步.

被甩开的处理方式: 整库重新初始化同步; 尽快恢复到最近的备份.

如何避免:   Primary 要设置一个较大的 oplog, ( 磁盘大小 5% ? 待求证 )

### 心跳

复制集内的成员之间需要相互沟通状态. 为此每个成员都会向其他成员发出心跳请求( 2s ).
需要注意的是, Primary 在每次心跳的时候都会判断自己是否可以联系到复制集的多数成员, 如果无法联系到, Primary 将会自动降级为 Secondary.

#### 成员状态

复制集成员之间的心跳信息中可能包含的信息:
一般状态 :  启动状态;  恢复状态;  仲裁器标志
异常状态 :  停机;  未知( 网络不可达 );  移除;  回滚;  故障

### 投票

... ...

### 回滚

回滚是一种对部分成员写成功, 但是对复制集写失败的操作的撤销机制.  这种部分有效的数据或操作会被复制集认为是无效数据.

有一个配置项 : 每个成员的投票数, 是和数据回滚相关的功能, 如果并不确定该配置项的用法, 则维持默认, 否则会影响回滚的行为.

在 Primary 切换过程中造成的写入数据失效的情况, 回滚时会将数据保存在一个单独的 rollback 文件中, 可用于手动恢复这部分数据.


## 将应用连接到复制集


对于应用来说, 无论是连接一台单机 MongoDB 还是一个复制集, 其操作都是一致的.

应用中使用 MongoDB Driver 连接复制集数据库时, 需要提供"种子".
种子就是复制集中的成员的 _主机地址和端口号_
**不需要** 列出所有的成员, 因为当驱动连接到任意成员后, 将会自动发现整个复制集中的所有成员.
连接字符串的示例 : `mongodb://server-1:27017,server-2:27018`

当 Primary 成员 Down 掉后,  复制集会尽快选举出新的 Primary, 并把所有的请求都路由到新的 Primary 成员. 中途会有很短暂的一瞬间无法进行写操作. 并且 Connection 会断开, 但 Driver 会尝试自动重连.


### 复制集写入级别

对于复制集, 为了确保数据写入成功, 就需要确保该数据被写入到复制集的多数成员中.
```
> db.runCommand({'getLastError': 1, 'w': "majority"})
```
"majority" 是 MongoDB 中的一个特殊的关键字, 相当于是集群中所谓"多数"成员的数量下限.

**需要格外注意的是**: 如果开启了写入级别的控制并指定写入多数成员, 同时, 集群中又存在 arbiter, 那么就有可能造成写入操作进入无限等待状态.

`getLastError` 命令可以配合另外一个属性: `wtimeout` 来控制写入等待超时时间. 不过即使超时, 也不会导致写入失败.

可以通过合理的设置写入级别, 来达到控制 Primary 写入速度的作用, 以免 Secondary 来不及同步而被甩开.

如果可能, 尽量设定写入级别, 并检查写入结果, 以增强整个系统的健壮型.

MongoDB 支持对写入级别进行复杂的自定义. 主要通过对 rs.config 中成员添加 tags 属性来对不同的成员进行分类.

### 读写分离

虽然 MongoDB 支持复制集中的读写分离, 但非常不建议这样做.  主要基于以下考虑:

* 一致性的问题
  对于读一致性有较高要求的应用不要使用读写分离, 次级成员通常只会与主成员延迟毫秒级,  但当次级成员存在负载或其他情况, 可能导致延迟拉大到不可接受的程度, 甚至被主成员甩开.
  对于写然后读取自己的情景, 不要使用读写分离.

* 负载问题
  很多用户为了减轻主成员的负担, 将全部或部分读请求交由次级成员处理, 但是这大大提高了次级成员被主成员甩开的可能性, 一旦次级成员需要重新进行初始化同步, 将给已经压力很大的数据库一个严重的打击, 并造成更加恶性的循环.
  解决负载问题的最好方式还是使用 分片 sharding.

#### 可以考虑使用 次级成员来处理读请求的情况:

* 希望主成员挂了以后, 程序还可以从次级成员读取数据, 并且数据的准确性要求不高的情况.  这称为 : `Primary preferred`
* 具有分布在不同地区的应用Server和数据库, 并希望Server有限读取最近( 延迟最小 )的数据库.
	如果需要低延迟的写入, 则必须使用 Sharding 才能做到.

read preference 选项:
`Secondary` : 只能由 次级成员响应读请求, 如果没有次级成员将报错, 不能使用 Primary 来响应读请求.
`Secondary preferred` : 优先次级成员, 如果没有则由 主成员响应读请求.
`Nearest` : 根据 ping 值判断最近成员, 并由该成员处理当前读请求.



## 复制集的管理



### 以单机 ( standalone ) 模式启动

管理中经常需要将复制集成员单独拿出, 以单机模式启动.  因为有些操作无法在次级成员上执行, 同时也不应在 主成员上进行.

查看当前 mongod 的启动参数:
```
> db.serverCmdLineOpts()
```

当需要单机启动机器的时候. 最简单的方式是:  不使用配置文件, 直接使用命令行指定参数, 一般只需要 `--port` 指定一个和以前不同的临时端口号用于登录维护, 另外, 使用 `--dbpath` 指定该数据库的实际数据路径. 以便对数据库进行操作.
此时对于复制集中的其他成员来说, 他们还在原先配置的端口上尝试连接该成员, 连接不到自然认为该成员已经挂掉.

当我们在单机模式下完成了需要进行的操作后, 就可以将重新使用复制集的配置文件启动数据库了.   它将自动进入复制集, 开始同步之前落下的数据操作.


### 复制集配置

所有成员的 `local.system.replset` 集合中都存有该复制集的配置文件, 不能对其进行手动修改, 必须通过使用 rs.reconfig() 或同等的命令来更新复制集配置信息.

初始化: 只能使用 `rs.initiate(config)` 进行初始化.
可以考虑将复制集配置成js文件, 然后在 shell 中使用 load("js file path") 将该配置文件加载到 shell 环境中, 然后直接使用.

对于已经配置的复制集, 可以对其进行修改, 但有几个限制条件 :

* 不能修改一个成员的 \_id 属性
* 不能将当前正在操作的成员 priority 设置为 0
* 无法将一个仲裁器转为普通成员, 反之亦然
* 不能将 buildIndexes 属性从 false 修改为 true


复制集被限制为最大 12 个成员, 并且最多只能有 7 个投票成员.  主要为了降低复制集的维护开销.


### 操作复制集成员状态

##### 将主成员临时降级为次级成员一段时间
```
> rs.stepDown()     // 降级默认时间是 60s
> rs.stepDown(600)  // 指定降级时间 600s
```




> Written with [StackEdit](https://stackedit.io/).
