## windows构建java-springboot-mysql-activiti-redis项目部署到服务器的问题

> ## 写在前面的话
> 这几天老师要谈一个项目，就要用一个网上的项目做简单更改做一个demo，在构建系统，部署到都服务器，修改项目上都遇到很多问题，我觉得好些问题都是对linux系统和其他东西不了解导致的，所以总结一下遇到的bug，对以后应该很用

### 连接redis的为问题
> 这个问题只改了bug，没仔细看怎么回事，就说说遇到的问题和处理方法
#### 问题描述
开启的redis服务和项目里面的配置都是没有密码，但是一直报错，解决方法就是都加了密码
#### 错误提示
```
(error) ERR Client sent AUTH, but no password is set
```
#### 修改方法
1.在redis客户端设置密码123456
```
  CONFIG SET requirepass 123456
```
2.查看修改是否成功
```
 auth 123456
```
3.在配置文件application.yml也修改密码
```
  redis:
        database: 0
        hostName: 127.0.0.1
        port: 6379
        password: 123456 # 密码（默认为空）
        timeout: 6000  # 连接超时时长（毫秒）
        pool:
            max-active: 1000  # 连接池最大连接数（使用负值表示没有限制）
            max-wait: -1      # 连接池最大阻塞等待时间（使用负值表示没有限制）
            max-idle: 10      # 连接池中的最大空闲连接
            min-idle: 5       # 连接池中的最小空闲连接
```

### 项目部署到服务器后数据库处理的各种问题

#### 问题描述
在windows中没有任何问题，在服务器上导入或者复制数据表以后服务器还一直显示找不到数据表，后面发现刷新时增加了一些同样名字的但是全部变为大写的数据表，后面发现原来windows上mysql是不区分大小写的，但是Linux上严格区分大小写，ps.mac上也不区分大小写。解决时使用网上说的更改命令都对的，但是说的没前没尾，浪费很多时间，下面是正确的解决方法

#### 解决方法
1.更改/etc/mysql/my.cnf，在最后添加下面代码
```
[mysqld]
lower_case_table_names=1
```
修改后我的my.cnf文件变为
```
#
# The MySQL database server configuration file.
#
# You can copy this to one of:
# - "/etc/mysql/my.cnf" to set global options,
# - "~/.my.cnf" to set user-specific options.
# 
# One can use all long options that the program supports.
# Run program with --help to get a list of available options and with
# --print-defaults to see which it would actually understand and use.
#
# For explanations see
# http://dev.mysql.com/doc/mysql/en/server-system-variables.html

#
# * IMPORTANT: Additional settings that can override those from this file!
#   The files must end with '.cnf', otherwise they'll be ignored.
#

!includedir /etc/mysql/conf.d/
!includedir /etc/mysql/mysql.conf.d/
[mysqld]
lower_case_table_names=1
                           
```
2.重启数据库
```
sudo /etc/init.d/mysql restart
```

3.查看大小写情况
```
mysql -u root -p
```
```
mysql> show variables like '%case_table%';
```
显示为1则修改成功

这样就不区分大小写了，啦啦啦

### 部署到服务器上找不到controller之类的各种东西
> 我觉得有一种可能的原因是扫描不到包，不过改了也没有解决问题，还是写一下方法
> 1.注解的两种方式
> @ComponentScan
>  @Controller
> 或者
> @RestController
> 2.在application.java上加上扫描包注解
> @ComponentScan({"com.hxy.modules.activiti.org.activiti", "com.hxy"})

下面是最终的解决方法
#### 问题描述
其实是这个项目需要用到计算机的计算机名，例如我的，本机IP127.0.0.1，也叫localhost，计算机名是DESKTOP-ES2RSHS23DKDL56这样的，这个程序会自动识别，但是我的linux服务器不知道它的主机名，就是```root@iW0788ypdf4lfaZ:/etc/mysql# ```里面那一串，也可以用命令```hostname```得到，解决方法就是把linux的主机名添加到配置文件里就可以了

#### 解决方法
1.打开文件/etc/hosts
2.可以看到第一行是
```
127.0.0.1       localhost 
```
3. 把你的主机名加进去,修改为
```
127.0.0.1      localhost iW0788ypdf4lfaZ
```
4. 重新运行项目就可以了，啦啦啦啦

> ### 写在最后的话
> 现在已经部署好了但是项目里有一个利用activiti画流程图的功能还没有实现
>
> 老师还要改前端的字，好像好多字是动态生成的，解决了继续写，呀呀呀















