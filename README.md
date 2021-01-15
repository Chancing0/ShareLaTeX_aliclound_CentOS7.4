# aliclound_ShareLaTeX_CentOS 7.4
# 安装过程如果有报错，请看最后面的解法，本人费劲尽力气找到有效的
# 安装Docker
## 1. 查看linux发行版，内核
    [root@docker~]# cat /etc/redhat-release  #查看版本号
## 2. 替换阿里云yum源
    wget -O /etc/yum.repos.d/CentOS-Base.repo http://mirrors.aliyun.com/repo/Centos-7.repo #下载阿里源 
    yum makecache  #生成仓库缓存 
## 3. 安装docker
    yum install docker -y
## 4. 启动docker
    systemctl start docker  #启动docker
    systemctl enable docker #开机启动docker
    systemctl status docker #查看docker状态
## 5.DaoCloud加速器 是广受欢迎的 Docker 工具，解决了国内用户访问 Docker Hub 缓慢的问题。DaoCloud 加速器结合国内的 CDN 服务与协议层优化，成倍的提升了下载速度。(或者使用阿里云加速镜像)
    curl -sSL https://get.daocloud.io/daotools/set_mirror.sh | sh -s http://f1361db2.m.daocloud.io
    
## 6.设置好加速镜像重启docker 
    systemctl restart docker
# 拉取ShareLaTeX镜像
    docker pull sharelatex/sharelatex
# 安装docker-compose
    yum install docker-compose -y
# 使用docker-compose.yml文件安装
    mkdir -p ~/docker/sharelatex
    cd ~/docker/sharelatex
    curl -O https://github.com/sharelatex/sharelatex/raw/master/docker-compose.yml
    sudo docker-compose up #安装启动ShareLaTeX
### 如果使用docker-compose.yml 安装失败，在docker-compose.yml[docker-compose.yml](https://github.com/overleaf/overleaf/blob/master/docker-compose.yml)下载文件复制里面的内容，然后在~/docker/sharelatex路径下新建docker-compose.yml并且 chmod其具有可执行权限，重新执行sudo docker-compose up即可~ 
# 使用
    到浏览器里面访问/launchpad，创建 Admin 用户。本地端就用 ：http://localhost:80/launchpad    (端口默认是使用80，除非你自己更改设定)
    云或者服务器端等具有公网IP的 ： http://IPV4/launchpad   (IPV4 请改成你的服务器IP地址)
#  ShareLaTeX升级  
    1. 使用进入容器内  
        docker exec -it sharelatex bash   
    2. 下载并运行升级脚本  
        wget http://mirror.ctan.org/systems/texlive/tlnet/update-tlmgr-latest.sh sh update-tlmgr-latest.sh -- --upgrade  
    3. 国内的话更换源  
        #阿里源：
        tlmgr option repository http://mirrors.aliyun.com/CTAN/systems/texlive/tlnet/   
        #清华源：  
        tlmgr option repository https://mirrors.tuna.tsinghua.edu.cn/CTAN/systems/texlive/tlnet/   
    4. 升级 tlmgr  
        tlmgr update --self --all 
    5. 安装完整 TexLive　这个过程很久，大概一两个小时
        tlmgr install scheme-full 
# 以上都OK了，就开始使用吧
        重启一下你的服务器，然后确认docker有在正常运行，没有就手动运行一下。　
        ##　如何设定管理员账户
        到浏览器里面访问/launchpad，在线创建 Admin 用户。本地端就用 ：http://localhost:80/launchpad   (端口默认是使用80，除非你自己更改设定)　　
        云或者服务器端等具有公网IP的 ： http://IPV4/launchpad   (IPV4 请改成你的服务器IP地址)　　
        如果没有出现再执行　docker exec sharelatex /bin/bash -c "cd /var/www/sharelatex; grunt user:create-admin --email=joe@example.com"　创建管理员账户，再上述登陆网页设置密码
        ##　如何添加普通用户
        使用管理员登陆后，左上角的 admin 里有 manage users ,手动输入新建的账户名字，点击注册，下面就会有一串网址，复制登陆就是设置该账户密码的界面。
        本地端举例　：　http://localhost/user/activate?token=243cd23a1e1dc463bcb7aa4dcf0e5217f37fa31da752f2a2be8a3200660cab6e&user_id=6001e77236d13300739df019
        外网端举例　：　http://ＩＰｖ４/user/activate?token=243cd23a1e1dc463bcb7aa4dcf0e5217f37fa31da752f2a2be8a3200660cab6e&user_id=6001e77236d13300739df019　，把你外网IP写进去就行
 # Error & Solution
## 1
    curl: (7) Failed to connect to raw.githubusercontent.com port 443: Connection refused
    原因是DNS被污染，所以域名查不到真正IP,解决方法如下：
    sudo vim /etc/hosts
    # 加上一行，直接手动查到的ＩＰ和域名记录，通过　https://www.ipaddress.com/　去查找　raw.githubusercontent.com　的IP,该IP会因为你所在国家地区不同而不同，所以要你服务器当国家地区所查到的IP即可
    199.232.28.133 raw.githubusercontent.com
## 2
    Docker Compose build fails: "Top level object in DockerFile needs to be an object not '<type 'str'>
    原因：docker-compose.yml　文件下载不正确，解法是：
    在[docker-compose.yml](https://github.com/overleaf/overleaf/blob/master/docker-compose.yml)下载文件复制里面的内容，然后在~/docker/sharelatex路径下新建docker-compose.yml并且 chmod其具有可执行权限，重新执行sudo docker-compose up即可~
## 4
    Job for docker.service failed because the control process exited with error code. See "systemctl status docker.service" and "journalctl -xe" for details.
    $ systemctl status docker.service
    看到　Failed to start Docker Application Container Engine　　解法是重新安装docker即可，这很快很简单
    yum remove docker-* 
    再手动删除所有docker的包，在删除的过程中，建议重启服务器（或者杀掉所有docker进程），期间可能还有docker进程在占用文件夹的情况，导致服务删除失败的问题，删除 /lib 和 /run 文件夹下的docker文件夹。
    第二步：更新Linux的内核，输入命令 “ yum update ”
    第三步：通过管理员安装 docker 容器　输入命令 “ sudo yum install docker ”
    第四步：启动docker容器　输入命令 “ systemctl start docker ”
    第五步：检查docker容器是否启动成功　查看容器状态，输入命令 “ systemctl status docker ”
    
## 5
    还有问题，就试试更新一下内核，或者根据报错码去找找其他的解法，以上是本人实作成功的流程和结果

    
    
    
    
    
    
    
    
    
    
    
    
    
    
