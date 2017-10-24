## KVM管理平台webvirtmgr的部署

### 安装kvm:

```bash
Setup libvirt and KVM
curl http://retspen.github.io/libvirt-bootstrap.sh | sudo sh

vim /etc/sysconfig/selinux   //关闭selinux
SELINUX=disabled
ps -e|grep libvirtd   //查看是否启动
chkconfig libvirtd on

启动libvirtd 
/etc/init.d/libvirtd start  //启动
```

#### 1.设置TCP认证：Setup TCP Authorization

```bash
$ sudo saslpasswd2 -a libvirt fred
Password: xxxxxx
Again (for verification): xxxxxx

sudo sasldblistusers2 -f /etc/libvirt/passwd.db


##### 开启16509端口
vim /etc/libvirt/libvirtd.conf
cat /etc/libvirt/libvirtd.conf | grep -v "^#" | grep -v "^$"
listen_tls = 0
listen_tcp = 1
tls_port = "16514"
tcp_port = "16509"
auth_tcp = "sasl"

vim /etc/sysconfig/libvirtd
LIBVIRTD_CONFIG=/etc/libvirt/libvirtd.conf
LIBVIRTD_ARGS="--listen"

配置完成以后可以看到16509端口
[root@localhost network-scripts]# netstat -ntulp | grep 16509
tcp        0      0 0.0.0.0:16509           0.0.0.0:*               LISTEN      24262/libvirtd
tcp6       0      0 :::16509                :::*                    LISTEN      24262/libvirtd




https://github.com/retspen/webvirtmgr/wiki/Setup-Host-Server

CentOS 7添加防火墙
sudo firewall-cmd --get-active-zones
sudo firewall-cmd --zone=public --add-port 16509/tcp --permanent
sudo firewall-cmd --reload
```

#### 2.Install WebVirtMgr

```bash
CentOS 7.x

$ sudo yum -y install http://dl.fedoraproject.org/pub/epel/7/x86_64/e/epel-release-7-5.noarch.rpm
$ sudo yum -y install git python-pip libvirt-python libxml2-python python-websockify supervisor nginx
$ sudo yum -y install gcc python-devel
$ sudo pip install numpy

2.安装python要求并设置Django环境
$ git clone git://github.com/retspen/webvirtmgr.git
$ cd webvirtmgr
$ sudo pip install -r requirements.txt # or python-pip (RedHat, Fedora, CentOS, OpenSuse)
$ ./manage.py syncdb
$ ./manage.py collectstatic

输入用户信息：

You just installed Django's auth system, which means you don't have any superusers defined.
Would you like to create one now? (yes/no): yes (Put: yes)
Username (Leave blank to use 'admin'): admin (Put: your username or login)
E-mail address: username@domain.local (Put: your email)
Password: xxxxxx (Put: your password)se12pa##1
Password (again): xxxxxx (Put: confirm password)se12pa##1
Superuser created successfully.

添加额外的超级用户

跑：

$ ./manage.py createsuperuser

mkdir /var/www
sudo mv webvirtmgr /var/www/
vim /etc/nginx/conf.d/webvirtmgr.conf
添加文件webvirtmgr.conf中/etc/nginx/conf.d：

server {
    listen 80 ;

    server_name $hostname;
    #access_log /var/log/nginx/webvirtmgr_access_log;

    location /static/ {
        root /var/www/webvirtmgr/webvirtmgr; # or /srv instead of /var
        expires max;
    }

    location / {
        proxy_pass http://127.0.0.1:8000;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-for $proxy_add_x_forwarded_for;
        proxy_set_header Host $host:$server_port;
        proxy_set_header X-Forwarded-Proto $scheme;
        proxy_connect_timeout 600;
        proxy_read_timeout 600;
        proxy_send_timeout 600;
        client_max_body_size 1024M; # Set higher depending on your needs
    }
}
$ sudo vim /etc/nginx/nginx.conf
注释服务器部分，如示例所示：

#    server {
#        listen       80 default_server;
#        server_name  localhost;
#        root         /usr/share/nginx/html;
#
#        #charset koi8-r;
#
#        #access_log  /var/log/nginx/host.access.log  main;
#
#        # Load configuration files for the default server block.
#        include /etc/nginx/default.d/*.conf;
#
#        location / {
#        }
#
#        # redirect server error pages to the static page /40x.html
#        #
#        error_page  404              /404.html;
#        location = /40x.html {
#        }
#
#        # redirect server error pages to the static page /50x.html
#        #
#        error_page   500 502 503 504  /50x.html;
#        location = /50x.html {
#        }
#    }
重新启动nginx服务：

$ sudo service nginx restart

更新SELinux策略

/usr/sbin/setsebool httpd_can_network_connect true 
使其永久服务：（OpenSusE，CentOS，RedHat，Fedora）
```
#### 3. Setup Supervisor
#### CentOS, RedHat, Fedora

```bash
$ sudo chown -R nginx:nginx /var/www/webvirtmgr
Create file /etc/supervisord.d/webvirtmgr.ini with following content:

[program:webvirtmgr]
command=/usr/bin/python /var/www/webvirtmgr/manage.py run_gunicorn -c /var/www/webvirtmgr/conf/gunicorn.conf.py
directory=/var/www/webvirtmgr
autostart=true
autorestart=true
logfile=/var/log/supervisor/webvirtmgr.log
log_stderr=true
user=nginx

[program:webvirtmgr-console]
command=/usr/bin/python /var/www/webvirtmgr/console/webvirtmgr-console
directory=/var/www/webvirtmgr
autostart=true
autorestart=true
stdout_logfile=/var/log/supervisor/webvirtmgr-console.log
redirect_stderr=true
user=nginx

$ sudo service supervisord stop
$ sudo service supervisord start
$ sudo  chkconfig supervisord on
```


#### 4.webvirtmgr测试：

```bash
[root@localhost ~]# virsh -c qemu+tcp://192.168.0.85/system nodeinfo
Please enter your authentication name: fred
Please enter your password:
CPU model:           x86_64
CPU(s):              4
CPU frequency:       800 MHz
CPU socket(s):       1
Core(s) per socket:  2
Thread(s) per core:  2
NUMA cell(s):        1
Memory size:         8266436 KiB
```

#### 安装出现启动错误:

```bash
WARNING:root:No local_settings file found.
Traceback (most recent call last):
  File "/var/www/webvirtmgr/console/webvirtmgr-console", line 22, in <module>
    from vrtManager.connection import CONN_SSH, CONN_SOCKET
  File "/var/www/webvirtmgr/vrtManager/connection.py", line 4, in <module>
    import libvirt
ImportError: No module named libvirt

这里提示python找不到libvirt
yum -y install git python-pip libvirt-python
更新python
```

### 访问WebVirtMgr 跟kvm 认证配置 Connections


![](http://7xrthw.com1.z0.glb.clouddn.com/kvm-webvirtmgr1.png)

>查看kvm宿主机信息：

![](http://7xrthw.com1.z0.glb.clouddn.com/kvm-webvirtmgr2.png)

>配置网络池：这里定义172.17网段 选择DHCP方式网络类型选择NAT

![](http://7xrthw.com1.z0.glb.clouddn.com/kvm-webvirtmgr3.png)

>查看网络池信息：

![](http://7xrthw.com1.z0.glb.clouddn.com/kvm-webvirtmgr4.png)

> 创建存储池
![](http://7xrthw.com1.z0.glb.clouddn.com/kvm-webvirtmgr5.png)

创建虚拟机实例：

![](http://7xrthw.com1.z0.glb.clouddn.com/kvm-webvirtmgr6.png)

![](http://7xrthw.com1.z0.glb.clouddn.com/kvm-webvirtmgr7.png)
![](http://7xrthw.com1.z0.glb.clouddn.com/kvm-webvirtmgr8.png)

![](http://7xrthw.com1.z0.glb.clouddn.com/kvm-webvirtmgr9.png)


