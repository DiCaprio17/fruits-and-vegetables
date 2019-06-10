# fruits-and-vegetables
基于python3.6和Django1.8.2

使用Python Web框架Django开发的一个B2C网上蔬果商城，包含用户、商品、购物车、订单等模块等等，使用了Celery异步任务队列，MySQL数据库，Redis数据库，FastDFS分布式的图片存储服 务，Nginx负载均衡服务器，uWSGI应用服务器。
# 项目总结
1.	生鲜类产品  B2C  PC电脑端网页
2.	功能模块：用户模块  商品模块（首页、 搜索、商品） 购物车模块  订单模块（下单、 支付）
3.	用户模块：注册、登录、激活、退出、个人中心、地址
4.	商品模块：首页、详情、列表、搜索（haystack+whoosh）
5.	购物车： 增加、删除、修改、查询
6.	订单模块：确认订单页面、提交订单（下单）、请求支付、查询支付结果、评论
7.	django默认的认证系统 AbstractUser
8.	itsdangerous  生成签名的token （序列化工具 dumps  loads）
9.	邮件 （django提供邮件支持 配置参数  send_mail）
10.	 celery (重点  整体认识 异步任务)
11.	 页面静态化 （缓解压力  celery  nginx）
12.	 缓存（缓解压力， 保存的位置、有效期、与数据库的一致性问题）
13.	 FastDFS (分布式的图片存储服务， 修改了django的默认文件存储系统)
14.	 搜索（ whoosh  索引  分词）
15.	 购物车redis 哈希 历史记录redis list
16.	 ajax 前端用ajax请求后端接口
17.	 事务
18.	 高并发的库存问题 （悲观锁、乐观锁）
19.	 支付的使用流程
20.	 nginx （负载均衡  提供静态文件）

# 安装
使用pip安装：pip install -r requirements.txt

# 配置
## uwsgi
遵循wsgi协议的web服务器。
### uwsgi的安装
	pip install uwsgi
### uwsgi的配置
项目部署时，需要把settings.py文件夹下的：
    DEBUG=FALSE
    ALLOWED_HOSTS=[‘*’] 
    
    [uwsgi]
    #使用nginx连接时使用
    #socket=127.0.0.1:8080
    #直接做web服务器使用
    http=127.0.0.1:8080
    #项目目录
    chdir=/Users/smart/Desktop/dj/bj17/dailyfresh
    #项目中wsgi.py文件的目录，相对于项目目录
    wsgi-file=dailyfresh/wsgi.py
    processes=4
    threads=2
    master=True
    pidfile=uwsgi.pid
    daemonize=uwsgi.log
    virtualenv=/Users/smart/.virtualenvs/dailyfresh
### uwsgi的启动和停止
启动:uwsgi –-ini 配置文件路径 例如:uwsgi --ini uwsgi.ini

停止:uwsgi --stop uwsgi.pid路径 例如:uwsgi --stop uwsgi.pid
## nginx
Nginx (engine x) 是一个高性能的HTTP和反向代理web服务器
### nginx 配置转发请求给uwsgi
    location / {
    	include uwsgi_params;
    	uwsgi_pass uwsgi服务器的ip:port;
    }
### nginx配置处理静态文件
django settings.py中配置收集静态文件路径:


- STATIC_ROOT=收集的静态文件路径 例如:/var/www/dailyfresh/static;

django 收集静态文件的命令:

    python manage.py collectstatic
执行上面的命令会把项目中所使用的静态文件收集到STATIC_ROOT指定的目录下。

收集完静态文件之后,让nginx提供静态文件，需要在nginx配置文件中增加如下配置:

    location /static {
    	alias /var/www/dailyfresh/static/;
    }
### nginx转发请求给另外地址
在location 对应的配置项中增加 proxy_pass 转发的服务器地址。
如当用户访问127.0.0.1时，在nginx中配置把这个请求转发给172.16.179.131:80(nginx)服务器，让这台服务器提供静态首页。
配置如下:

    location = /{
    	proxy_pass http://172.16.179.131;
    }
### nginx配置upstream实现负载均衡
ngnix 配置负载均衡时，在server配置的前面增加upstream配置项。

    upstream dailyfresh {
    	server 127.0.0.1:8080;
    	server 127.0.0.1:8081;
    }
