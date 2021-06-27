+ 1. 选择一个合适的版本，目前基本上都依赖 jdk 1.8 https://rocketmq.apache.org/dowloading/releases/ 

+ 2. 选择一个 CentOS 7 / 8 的机器，安装 jdk 1.8
	
	`yum install java-1.8.0-openjdk`

+ 3. 解压 rocketmq 到 /opt/rocketmq 下面，修改配置

```bash
[root@localhost ~]# ll /opt/rocketmq/
total 36
drwxr-xr-x 2 root root    83 Dec 13  2017 benchmark
drwxr-xr-x 2 root root  4096 Jun 26 20:14 bin
drwxr-xr-x 5 root root   187 Dec 13  2017 conf
drwxr-xr-x 2 root root  4096 Dec 13  2017 lib
-rw-r--r-- 1 root root 17336 Sep 19  2017 LICENSE
-rw-r--r-- 1 root root  1337 Oct 18  2017 NOTICE
-rw-r--r-- 1 root root  2426 Oct 18  2017 README.md
```

conf/broker.conf
```conf
namesrvAddr = 127.0.0.1:9876
brokerIP1 = 192.168.147.129 # 改成暴露出去的IP
```


+ 4. 创建 user 以及 service  文件

`useradd rocketmq`

```bash
cat << EOF >/usr/lib/systemd/system/mqnamesrv.service
[Unit]
Description=rocketmq - nameserver
Documentation=http://mirror.bit.edu.cn/apache/rocketmq/
After=network.target

[Service]
Type=sample
User=rocketmq
ExecStart=/opt/rocketmq/bin/mqnamesrv
ExecReload=/bin/kill -s HUP $MAINPID
ExecStop=/bin/kill -s QUIT $MAINPID
Restart=0
LimitNOFILE=65536

[Install]
WantedBy=multi-user.target
EOF

cat << EOF > /usr/lib/systemd/system/mqbroker.service
[Unit]
Description=rocketmq - mqbroker
Documentation=http://mirror.bit.edu.cn/apache/rocketmq/
After=network.target

[Service]
Type=sample
User=rocketmq
ExecStart=/opt/rocketmq/bin/mqbroker -n localhost:9876 autoCreateTopicEnable=true -c /opt/rocketmq/conf/broker.conf
ExecReload=/bin/kill -s HUP $MAINPID
ExecStop=/bin/kill -s QUIT $MAINPID
Restart=0
LimitNOFILE=65536

[Install]
WantedBy=multi-user.target
EOF
```

`systemctl daemon-reload && systemctl enable mqnamesrv mqbroker && systemctl start mqnamesrv && systemctl start mqbroker`

+ 5. 测试
