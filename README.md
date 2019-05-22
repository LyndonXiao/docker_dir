# docker_dir
docker集成的Nginx-Mysql-PHP开发环境  
目录结构  
```bash
/root
    /mysql  存放数据库备份
    /nginx  nginx配置文件
	    /conf
    /wwwroot  网站根目录
    /wwwlogs  日志
    /source  程序源代码
    /download 下载目录
```

# 安装docker
直接官网下载桌面版，配合里面的Kitematic可视化管理容器

# 克隆git仓库，把配置目录拷贝下载
`https://github.com/LyndonXiao/docker_dir.git`

# 部署Mysql容器
1. 拉取镜像  
	`docker pull mysql:5.7`
2. 运行容器  

```bash
docker run \  
    -d \  
    -p 3306:3306 \  
    -e MYSQLROOTPASSWORD=12345678910 \  
    --name mmysql mysql:5.7
```
> 参数说明  
> -d 让容器在后台运行  
> -p 添加主机到容器的端口映射，前面是映射到本地的端口，后面是需要映射的端口  
> -e 设置环境变量，MYSQL_ROOT_PASSWORD这里是设置mysql的root用户的初始密码  
> —name 容器的名字，随便取，但是必须唯一

# 部署PHP
1. 拉取镜像  
	`docker pull bitnami/php-fpm:7.2`
2. 运行容器  
```bash
docker run \
    -d \
    -p 9000:9000 \
    -v /data/wwwroot:/usr/share/nginx/html \
    --link mmysql:mysql \
    --name mphpfpm bitnami/php-fpm:7.2
```
> 参数说明  
> -d 让容器在后台运行  
> -p 添加主机到容器的端口映射  
> -v 添加目录映射，本地目录:容器目录  
> –-name 容器的名字，随便取，但是必须唯一  
> —link link 是在两个contain之间建立一种父子关系，父container中的web，可以得到子container db上的信息。  
> 通过link的方式创建容器，我们可以使用被Link容器的别名进行访问，而不是通过IP，解除了对IP的依赖。

3.  存放项目文件  
在目录的wwwroot下可以存放项目文件

# 部署Nginx
1. 拉取镜像  
	` docker pull nginx`
2.  运行容器  
```bash
docker run \
    -d \
    -p 80:80 \
    -v /data/wwwroot:/usr/share/nginx/html \
    -v /data/nginx/nginx.conf:/etc/nginx/nginx.conf:ro \
    -v /data/nginx/conf:/etc/nginx/conf.d \
    -v /data/wwwlogs:/var/log/nginx \
    --link mphpfpm:phpfpm \
    --name mnginx nginx:latest
```
> 参数说明  
> -d 让容器在后台运行  
> -p 添加主机到容器的端口映射  
> -v 添加目录映射,这里最好nginx容器的根目录最好写成和php容器中根目录一样。但是不一点非要一模一样,如果不一样在配置nginx的时候需要注意  
> -–name 容器的名字  
> –-link 与另外一个容器建立起联系

创建或修改data/nginx/conf下的配置文件，需要注意的是`fastcgi_pass phpfpm:9000;`，因为我们在启动容器的时候--link了phpfpm容器，所以这里直接填phpfpm，就能找到phpfpm的IP，当然你也可以不配置--link，那么你可以这样找到容器IP，再将IP填入。
`docker inspect 容器名或ID | grep "IPAddress"`

# 如何进入容器？
`docker exec -it m_mysql /bin/bash`