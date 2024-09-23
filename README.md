这是学习一个开源项目的学习记录，源项目https://github.com/macrozheng/mall.git

以下所有内容都是自己的理解

# 项目启动

## 后端项目启动

### MongoDB

#### 问题

我下载的最新的MongoDB社区版8.0.0，bin目录没有mongo.exe文件

#### 解决办法

第一次启动需要指定数据库存放目录

```
mongod --dbpath D:MongoDB\data
```

在浏览器输入：http://localhost:27017/ 显示：

It looks like you are trying to access MongoDB over HTTP on the native driver port.

就算成功了

如果要进入客户端就需要安装MongoDb shell： https://www.mongodb.com/try/download/shell

解压了放到与bin文件同级的地方，点击mongo.exe就可以进入客户端了

### 项目启动

#### 问题

我用的MySQL8.0，发现报了这个

Public Key Retrieval is not allowed（不允许检索公钥）。

出现这个问题原因是：mysql8以上版本默认使用 sha256_password 认证，密码在传输过程中必须加密保护，如果无法使用 TLS，就需要使用 RSA 公钥加密

#### 解决办法

添加allowPublicKeyRetrieval=true

也就是在每个模块的配置文件中修改MySQL的url成这样:

```
url: jdbc:mysql://localhost:3306/mall?useUnicode=true&characterEncoding=utf-8&allowPublicKeyRetrieval=true&serverTimezone=Asia/Shanghai&useSSL=false
```

除此之外，我的本地redis设置了密码，所以修改了redis的password

## 前端项目启动

### admin-Web

#### 问题

执行npm install报错： 

found XXX vulnerabilities (XXX low, X moderate)，run `npm audit fix` to fix them, or `npm audit` for details

#### 解决办法

将官方镜像源改为淘宝镜像源就可以了，然后也确实出现了node-sass不能下载的情况，按照教程做就好了，

在使用npm安装包的时候有很多的警告：npm WARN... 我还因此在网上查了很久

折腾了半天才知道不用管这些警告，只要安装完成就能运行

### Nginx启动

nginx启动就很简单了，但是也有几个问题

1，第一次运行nginx不要直接点击exe文件，输入下面的命令就可以了

```
start nginx
```

2，访问nginx默认带的80端口是不行的，需要修改一下防火墙。我为了方便就直接在nginx配置文件中修改了端口为8800，然后在浏览器访问

```
http://localhost:8800/admin/
http://localhost:8800/app/
```

3，访问app的时候为了呈现手机界面，需要点一下开发者工具左上角的电脑手机小图标，然后刷新一下页面就可以了

# 学习记录

## mall-admin

### 业务

这个类实现了mall-admin-web的业务，除了搜索业务。

### 架构

代码方面都典型的SSM框架，架构就是controller, service(Impl), dao(mapper),xml(注解) 。

在处理缓存的时候用了redis

在登录的时候用了jwt, token

### 代码

都是比较常见的，比较陌生的代码就是alioss那部分。还有就是sku(第一次看到有点懵）。

SKU，即“库存单位”，是指在供应链管理中用于区分不同产品的唯一编号或代码。每个SKU都具有独特的特征，包括产品名称、规格、颜色、尺寸等。SKU在电商应用中通常会出现。和Java其实没有关系。

值得注意的是，程序中多处使用Stream API来处理集合,比如下面这段代码

```
List<UmsMenuNode> children = menuList.stream()
                .filter(subMenu -> subMenu.getParentId().equals(menu.getId()))
                .map(subMenu -> covertMenuNode(subMenu,menuList))
                .collect(Collectors.toList());
```

这段代码的作用是：从`menuList`中筛选出所有父菜单ID等于某个特定菜单（`menu`）ID的子菜单项，然后将这些子菜单项转换为`UmsMenuNode`类型的对象，并将这些对象收集到一个新的列表中。

## mall-mbg

这个模块就是用Mybatis代码生成器Mybatis Generator (MBG)根据数据库生成了对应的Java对象和包含基本功能的xml文件。

但是有个问题就是这种方式会产生很多无用的代码

## mall-common

这个模块也没有处理业务，主要是一些工具和配置，包括：分页，异常，结果，redis，接口，日志，获取IP。

这个项目采用了Logstash记录日志，其实不启动Logstash服务也是不影响程序运行的。

### 一个小bug

在logback-spring.xml文件中有个小报错，但不影响程序运行，错误提示“Attribute level is not allowed here”通常表明在 XML 配置文件中，`level` 属性被放置在了不被支持该属性的元素上

```
<root level="DEBUG">  
        <appender-ref ref="CONSOLE"/>  
        <appender-ref ref="FILE_DEBUG"/>  
        <appender-ref ref="FILE_ERROR"/>  
        <appender-ref ref="LOG_STASH_DEBUG"/>  
        <appender-ref ref="LOG_STASH_ERROR"/>  
    </root>
```

这段代码其实在某些版本的logback是没有问题的，不过这个程序的logback不支持这种写法。

最好就不要管，我修改后程序直接不能运行了，在这里就不演示错误示范了。

## mall-search

这个模块通过Elasticsearch实现了搜索业务

这部分程序看起来像是在操作数据库，但elasticsearch其实是一个强大的搜索引擎，和数据库差别挺大的。

## mall-portal

### 业务

这个模块实现了客户端app的功能

### 代码

使用了RebbitMQ处理订单延迟，退单等业务。

使用MongoDB缓存了会员的浏览记录，处理了部分会员的业务。

### 问题

**注释**让人容易误解。按照程序的逻辑：所有的用户都叫会员

在repository包下有三个接口，调试一下会发现接口中的方法既不是重写，也没有具体实现，但是能在其他类中直接调用。

原因其实是这三个接口继承了`MongoRepository`，而`MongoRepository`是Spring Data提供的一个接口，它通过Spring Data JPA或者Spring Data MongoDB等数据访问框架来动态实现接口中的方法。这些方法都是基于**Spring Data JPA/MongoDB的查询方法命名规范**自动生成的。Spring Data允许我们通过遵循一定的命名规则，定义查询方法，而不需要手动实现这些方法。
