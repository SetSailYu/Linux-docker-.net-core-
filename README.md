# Linux下docker服务内的.net core项目部署操作步骤
 查看ip： `ifconfig`  
 复制： `ctrl + insert`  
 粘贴： `shift + insert`  
## 一、安装docker
    yum install -y yum-utils device-mapper-persistent-data lvm2  
    yum-config-manager --add-repo http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo  
    yum makecache fast  
    yum -y install docker-ce 
## 二、docker基本操作
#### ***启动docker服务***
    systemctl start docker.service
#### ***docker服务开机自启***
    systemctl enable docker.service
#### ***查看docker版本***
    docker version
#### ***查看docker镜像***
    docker images
#### ***查看docker容器***
    docker ps -a
#### ***启动docker容器***
    docker start 容器名字
#### ***自动启动docker容器***
    docker update --restart=always 容器名字
#### ***停止docker容器***
    docker stop 容器名字
#### ***删除docker容器***
    docker rm 容器名字
#### ***删除docker镜像***
    docker rmi 镜像名称
## 三、.net core 部署
### ***1、 项目右键，发布到文件系统***
    dockerfile
### ***2、 ls：查看目录下的文件夹和文件***
### ***3、 time out solution：***
  ##### ***3.1、编辑文件：***  
  vim 目录（`vim /etc/docker/daemon.json`）
  ```
    {
        "registry-mirrors":["https://docker.mirrors.ustc.edu.cn"]
    }
  ```
  -- 插入 --  
  ```按下i键进入  
     按esc退出插入模式  
     :q   退出（没有改动时可以退出）  
     :q!  强制退出（不管有没有改动都退出，不保存改动）  
     :wq  保存后退出
  ```
  重载配置文件   
  ```systemctl daemon-reload```  
  重启服务  
  ```systemctl restart docker```

  ##### ***3.2、查找可用ip***
    dig @114.114.114.114 docker.io  
    vim /etc/hosts  
    在末尾加入
    找到的ip   registry-1.docker.io  

### ***4、 根据dockerfile拉取镜像***
    docker build -t 镜像名称 dockerfile文件路径（相对）  
    docker build -t myframe ./myframeproj
### ***5、 运行镜像容器***
    docker run --name democore -d -p 54907:80 dockerdemo  
    //docker run --name 容器名字 -d -p 主机访问时的端口:80 镜像名称  
### ***6、 查看容器***
    docker ps -a  
    停止容器  docker stop 容器名字  
    启动容器  docker start 容器名字  
    删除容器  docker rm 容器名字  
    容器自动启动： docker update --restart=always 容器名字  

### ***7、 更新项目，更新迭代***  
    重新发布到publish，上传到centos项目所在的位置
    镜像操作
    删除镜像： docker rmi 镜像名称

    添加可执行权限
    chmod +x publish.sh
## 四、部署 .net Core + sql server 2017 
### ***1、 安装先决条件***  
       * Docker 引擎 1.8+  
       * Linux内核的 3.18+  
       * “文件系统” XFS 或 EXT4（其他文件系统均不受支持，如 BTRFS）  
       * 处理器类型仅兼容 x64 ，磁盘大于6G。至少 2 GB 的 RAM。（建议 4 GB 内存）  
### ***2、 Linux内核检查***
查看版本信息：  `uname -a`  
升级内核方法：  https://www.cnblogs.com/polk6/p/11282477.html  
删除多余内核方法：  https://www.cnblogs.com/redo19990701/p/11441928.html
### ***3、 配置SQL Server服务***  
创建数据存储目录：   `mkdir /var/opt/mssql`   
① 配置mssql-server-2017数据库服务：   

        载mssql-server-2017镜像   
            sudo docker pull mcr.microsoft.com/mssql/server:2017-latest  
② 配置ms-sqlserver数据库服务：    

        搜索docker镜像   
            docker search mssql-server  
        载ms-sqlserver镜像  
            docker pull microsoft/mssql-server-linux  
查看镜像：   `docker images`    
创建并启动容器：  

    // docker run -e 'ACCEPT_EULA=Y' -e 'SA_PASSWORD=@YouPassword' -p 1433:1433 --name mssql -v $HOME/sql:/var/opt/mssql -d mcr.microsoft.com/mssql/server:2017-latest  

        docker run -e 'ACCEPT_EULA=Y' -e 'SA_PASSWORD=@YouPassword' \   #环境及密码配置  
         -p 1433:1433 --name mssql \   #映射端口及容器名  
         -v $HOME/sql:/var/opt/mssql \    #工作存储目录  
         -d microsoft/mssql-server-linux    #镜像名  
        or  
         -d mcr.microsoft.com/mssql/server:2017-latest    #镜像名    
上传数据库文件：  

        根据  -v $HOME/sql:/var/opt/mssql  配置，
        将数据库文件上传至  /root/sql/data  文件夹下  
        远程连接数据库 --> 数据库 --> 附加  
开启SqlServer代理：    

        docker exec -it <容器名称或ID> "bash"  
        ls  
        /opt/mssql/bin/mssql-conf set sqlagent.enabled true  
            或// /var/opt/mssql/bin/mssql-conf set sqlagent.enabled true  
        exit  
        docker stop <容器名称或ID>  
        docker start <容器名称或ID>  
## 五、Linux环境安装docker-compose：
#### ***1、镜像安装***

    镜像拉取并安装  
        sudo curl -L https://github.com/docker/compose/releases/download/1.25.0-rc4/docker-compose-`uname -s`-`ur/local/bin/docker-compose  
    设置执行权限  
        sudo chmod +x /usr/local/bin/docker-compose  
    查看详情（用于验证安装成功）  
        docker-compose --version  
### ***2、手动下载文件安装***
https://github.com/docker/compose/releases/tag/1.25.0-rc4  

    github手动下载文件：https://github.com/docker/compose/releases/tag/1.25.0-rc4  
        docker-compose-Linux-x86_64  
    将文件上传到 /usr/local/bin/ 目录下，重命名为 docker-compose  
    设置执行权限  
        sudo chmod +x /usr/local/bin/docker-compose  
    查看版本信息  
        docker-compose -v  
### ***3、 配置docker-compose.yml文件***
在与项目文件夹同级创建，例如 `ls` 命令下可以同时看到 docker-compose.yml netcoretest1vs2019proj(放置.net项目的文件夹)
    
    vim docker-compose.yml
    
        version: '3'                                                #version必须大于1才可以使用network
        services:
            mssql:                                                  #服务名称
            container_name: mssql                                   #容器名称
            image: mcr.microsoft.com/mssql/server:2017-latest       #镜像名称
            restart: always                                         #总是重启后启动
            ports:                                                  #端口映射
                - "1433:1433"
            volumes:                                                #挂载
                - $HOME/sql:/var/opt/mssql
            environment:                                            #环境变量
                - ACCEPT_EULA=Y
                - SA_PASSWORD=@YouPassword                          #SA用户密码（必须至少八位，十进制数字、大写字母、小写字母、符号 四者之三）
            networks:                                               #添加到桥接网络中
                - netcoretest1
        netcoretest1vs2019proj:                                     #要与项目文件夹同名
            image: netcoretest1vs2019                               #镜像名称
            build: ./netcoretest1vs2019proj                         #拉取镜像
            restart: always                                         #总是重启自启动
            container_name: netcoretest1vs2019                      #容器名称
            ports:                                                  #端口映射
                - "56000:80"
            depends_on:                                             #指定服务依赖，将会优先于服务创建并启动依赖
                - mssql
            networks:                                               #添加到桥接网络中
                - netcoretest1
        networks:                                                   #根级，声明外部网络
            netcoretest1:
                external: true
                
启动docker-compose.yml配置：  

        docker-compose -f docker-compose.yml up -d  
暂停docker-compose.yml配置：  
    
        docker-compose stop  

### ***4、 创建容器网络的步骤***
创建一个自定义的桥接网络：  

        //docker network create --driver bridge 网络名
        docker network create --driver bridge netcoretest1
让容器加入到这个网络里：

        //docker run -d --network=网路名 --name 网络内的容器名 镜像名
        
        添加server-2017：
        docker run -d --network=netcoretest1 --name mssql 
         -e 'ACCEPT_EULA=Y' -e 'MSSQL_SA_PASSWORD=@YouPassword' 
         -p 1433:1433 -v $HOME/sql:/var/opt/mssql mcr.microsoft.com/mssql/server:2017-latest
         
        添加镜像名为netcoretest1vs2019的.net core项目：
        docker run -d --network=netcoretest1 --name netcoretest1vs2019 -p 56000:80 netcoretest1vs2019
查看该网络详情：

        docker network inspect netcoretest1
            // "Containers": {},  #所添加的容器信息将在这里面显示
            // 数据库："IPv4Address": "172.19.0.2/16",
            // 根据该地址配置连接字符串：
            //      "server=172.19.0.2,1433;database=StudentDB;uid=sa;pwd=@YouPassword;"
现有的网络列举：

        docker network ls
