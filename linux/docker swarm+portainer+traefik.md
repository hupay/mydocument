
# 虚拟机安装centos7
centos7的iso在虚拟机的“D:\CentOS-7-x86_64-DVD-1804.iso”

选择上海时间；英文、简体中文；最小化安装；密码：xxx@xxx

注意正式环境时区问题（也是上海时区）。


如何查看CentOS7的版本信息

https://blog.csdn.net/ZZY1078689276/article/details/77716871

Centos7配置静态 IP

https://www.huaweicloud.com/articles/03678b0a8a34c502209737abf32dbbfe.html

# IP设置
```
ls /etc/sysconfig/network-scripts
vi /etc/sysconfig/network-scripts/ifcfg-eth0
#内容如下
TYPE=Ethernet
PROXY_METHOD=none
BROWSER_ONLY=no
BOOTPROTO=static
DEFROUTE=yes
IPV4_FAILURE_FATAL=no
IPV6INIT=yes
IPV6_AUTOCONF=yes
IPV6_DEFROUTE=yes
IPV6_FAILURE_FATAL=no
IPV6_ADDR_GEN_MODE=stable-privacy
NAME=eth0
UUID=6e3500cd-6369-43b2-a53b-bc68dacfde66
DEVICE=eth0
ONBOOT=yes
#注意按实际ip填写
IPADDR=10.10.10.10
NETMASK=255.255.255.0
GATEWAY=10.10.10.1
DNS1=223.5.5.5
ZONE=public
```

重启服务：
```systemctl restart network```

# 升级
安装完成后需升级系统```yum update```

# 安装docker
## 改名
```
vi /etc/hostname
linux10
reboot
# 重启后就可以使用SSH软件链接测试机（mobaxterm）
```

## docker
```
# 修改dns，用阿里的https://www.alidns.com/
vi /etc/resolv.conf
nameserver 223.5.5.5
# step 1: 安装必要的一些系统工具
sudo yum install -y yum-utils device-mapper-persistent-data lvm2
# Step 2: 添加软件源信息
sudo yum-config-manager --add-repo https://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
# Step 3: 更新并安装Docker-CE，空间不足的话需要删除一些文件 rm -rf /var/log/* 又通过查找大文件删了一些文件，成功安装“ls -lh  $(find / -type f -size +100M)”
sudo yum makecache fast
sudo yum -y install docker-ce
# Step 4: 开启Docker服务
sudo service docker start

#停止docker服务，迁移docker目录，通过df -lh可以看到大部分磁盘在/home目录下
systemctl stop docker
mkdir /home/docker
cp -r /var/lib/docker  /home/docker
mv /var/lib/docker /var/lib/docker.bak
ln -s /home/docker /var/lib/docker
systemctl start docker
# 设置开机启动
systemctl enable docker.service
docker version
docker info
```
# 安装java环境
CentOS 7 安装 JAVA环境（JDK 1.8）
https://www.cnblogs.com/stulzq/p/9286878.html
```
mkdir /home/soft
# 上传两个文档 zookeeper-3.4.8.tar.gz 和 jdk-8u171-linux-x64.tar.gz
mkdir /home/java
tar -zxvf /home/soft/jdk-8u171-linux-x64.tar.gz -C /home/java
vi /etc/profile
#在末尾添加
export JAVA_HOME=/home/java/jdk1.8.0_171
export JRE_HOME=${JAVA_HOME}/jre
export CLASSPATH=.:${JAVA_HOME}/lib:${JRE_HOME}/lib
export PATH=${JAVA_HOME}/bin:$PATH

#使环境变量生效
source /etc/profile
#添加软链接
ln -s /home/java/jdk1.8.0_171/bin/java /usr/bin/java
#检查
java -version
```

# 安装zookeeper
CentOS7安装Zookeeper
https://cloud.tencent.com/developer/article/1700375

```
mkdir /home/zookeeper
tar -zxf /home/soft/zookeeper-3.4.8.tar.gz -C /home/zookeeper
mkdir /home/zookeeper/data
mkdir /home/zookeeper/log

vi /home/zookeeper/zookeeper-3.4.8/conf/zoo.cfg
tickTime = 2000
dataDir = /home/zookeeper/data
dataLogDir = /home/zookeeper/log
clientPort = 2181
initLimit = 5
syncLimit = 2

# 还需要脚本增加认证信息
vi /home/zookeeper/zookeeper-3.4.8/bin/zkServer.sh
# nohup "$JAVA" ... 第二行开始
"-Dzookeeper.DigestAuthenticationProvider.superDigest=super:nYri7/VzajS4SfD9T/DcKoCU56I="

/home/zookeeper/zookeeper-3.4.8/bin/zkServer.sh start
/home/zookeeper/zookeeper-3.4.8/bin/zkServer.sh staus

# 开机自动启动
vi /usr/lib/systemd/system/zookeeper.service
#内容
[Unit]
Description=zookeeper
After=network.target
 
[Service]
Type=forking
ExecStart=/home/zookeeper/zookeeper-3.4.8/bin/zkServer.sh start
ExecReload=/home/zookeeper/zookeeper-3.4.8/bin/zkServer.sh restart
ExecStop=/home/zookeeper/zookeeper-3.4.8/bin/zkServer.sh stop
RemainAfterExit=yes
 
[Install]
WantedBy=multi-user.target

# 设置
systemctl start zookeeper.service #启动
systemctl stop zookeeper.service #停止
systemctl enable zookeeper.service #(开机启动)
systemctl disable zookeeper.service #(禁止开机启动)
```

# docker swarm
```
docker swarm
docker swarm init --advertise-addr 10.10.10.10
#离开后，可以重新初始化
docker swarm leave --force
#编辑docker镜像
vi /etc/docker/daemon.json
{
  "registry-mirrors": [
    "https://hrn7zamx.mirror.aliyuncs.com"
  ]
}
```

# 安装portainer
```
docker volume create portainer_data
docker run -d -p 83:8000 -p 84:9000 --name=portainer --restart=always -v /var/run/docker.sock:/var/run/docker.sock -v portainer_data:/data portainer/portainer-ce:latest
```

开放上面的84端口后
http://10.10.10.10:84/
设置密码wftest@271，选择本地docker。没有多集群。所以不用安装agent

设置私有仓库账号密码

~~上传“docker-config.sh”到/home文件夹下
执行命令“source /home/docker-config.sh”~~

通过修改/etc/hosts，映射到容器中
```
10.10.10.10 sqlserver.host

# docker-compose
    volumes:
      - /etc/hosts:/etc/hosts
```

# 安装网络traefik
创建一个traefik的网络
traefik-net，类型overlay
部署traefik，如果有grpc文件，需要提前上传文件
```
#https://doc.traefik.io/traefik/user-guides/docker-compose/basic-example/

version: "3.7"

services:
  traefik:
    image: "traefik:v2.4"
    command:
      # 启用dashboard
      - "--api.dashboard=true"
      - "--log.level=DEBUG"
      - "--api.insecure=true"
      - "--metrics.prometheus=true"
      - "--metrics.prometheus.buckets=0.1,0.3,1.2,5.0"
      - "--providers.docker=true"
      - "--providers.docker.exposedbydefault=false"
      - "--providers.docker.swarmMode=true"
      - "--providers.docker.watch"
      - "--entrypoints.web.address=:80"
      - "--entrypoints.grpc.address=:90"
      - "--entrypoints.websecure.address=:443"
      # GRPC
      - "--providers.file.directory=/home/dynamic_conf.yml"
    ports:
      - "80:80"
      - "90:90"
      - "85:8080"
    volumes:
      - "/var/run/docker.sock:/var/run/docker.sock:ro"
    networks:
      - traefik-net
    deploy:
      placement:
        constraints:
          - node.role == manager
networks:
  traefik-net:
    external: true
```
# 安装prometheus
单机可忽略

注意相关文件要传到指定目录。并且修改grafana数据库的账号密码
```
# https://github.com/vegasbrianc/docker-traefik-prometheus/blob/master/docker-compose.yml
version: "3.7"

services:
  prometheus:
    image: prom/prometheus
    volumes:
      - /home/docker/prometheus/:/etc/prometheus/
      - prometheus_data:/prometheus
    command:
      - '--config.file=/etc/prometheus/prometheus.yml'
      - '--storage.tsdb.path=/prometheus'
      - '--web.console.libraries=/usr/share/prometheus/console_libraries'
      - '--web.console.templates=/usr/share/prometheus/consoles'
    networks:
      - traefik-net
    deploy:
      labels:
       - "traefik.enable=true"
       - "traefik.http.routers.prometheus.rule=Host(`prometheus.test`)"
       - "traefik.http.routers.prometheus.entrypoints=web"
       - "traefik.http.services.prometheus.loadbalancer.server.port=9090"
       - "traefik.docker.network=traefik-net"
      placement:
        constraints:
        - node.role==manager
      restart_policy:
        condition: on-failure
  
  grafana:
    image: grafana/grafana
    depends_on:
      - prometheus
    volumes:
      - grafana_data:/var/lib/grafana
      - /home/docker/grafana/provisioning/:/etc/grafana/provisioning/
    #env_file:
    #  - /home/docker/grafana/config.monitoring
    environment:
      - GF_SECURITY_ADMIN_PASSWORD=wftest@271
      - GF_USERS_ALLOW_SIGN_UP=false
      - GF_INSTALL_PLUGINS=grafana-piechart-panel
    networks:
      - traefik-net
    user: "472"
    deploy:
      labels:
        - "traefik.enable=true"
        - "traefik.http.routers.grafana.rule=Host(`grafana.test`)"
        - "traefik.http.routers.grafana.entrypoints=web"
        - "traefik.http.services.grafana.loadbalancer.server.port=3000"
        - "traefik.docker.network=traefik-net"
      placement:
        constraints:
          - node.role == manager
      restart_policy:
        condition: on-failure
networks:
  traefik-net:
    external: true
volumes:
  grafana_data:
    driver: local
  prometheus_data:
    driver: local
```

# 端口开放
```
# 注意调整火墙要重启traefik
firewall-cmd --zone=public --add-port=84/tcp --permanent
systemctl restart firewalld

firewall-cmd --zone=public --add-port=80/tcp --permanent
firewall-cmd --zone=public --add-port=85/tcp --permanent
firewall-cmd --zone=public --add-port=90/tcp --permanent
systemctl restart firewalld

firewall-cmd --zone=public --add-port=2181/tcp --permanent
systemctl restart firewalld
```

# 虚拟机复制
1. 将目标机器改名，然后右键导出，选择目标文件夹
2. 点击导入虚拟机，选择刚刚导出虚拟机
3. 选择相关配置，复制新机器
4. 将导入的虚拟机改成目标名称，将修改的名称改回去。
5. 然后进入新复制机器，修改机器名称、ip。然后重启机器即可
6. 修改host，然后更新服务（？）

# 站点服务部署
按开发提供的yml文件，创建相关stack部署即可。

# 注意事项
- 注意设置防火墙后，要重启traefik
- docker容器无法访问外网，实测无效
- 重启后需要启动docker和zk
```
echo "nameserver 8.8.8.8"> /etc/resolv.conf
apt-get update && apt-get install procps
```
