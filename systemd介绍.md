# yii-queue 消费进程管理说明

-----

### 说明：
为了使php守护进程能够长久稳定的运行(列如yii-queue 消费进程), 我们需要通过systemd来监管该进程。

### 具体使用方法：

1. 在/etc/systemd/system/目录下创建文件:yii-queue-xxxx@.service，将其中的ExecStart 修改成需要运行的命令即可。

``` shell
[Unit]
Description=Yii Queue Worker %I
After=network.target

[Service]
User=www

#修改成对应需要运行的命令
ExecStart=/usr/bin/php /vanke/www/boyu/yii queue-xxxx/listen 
Restart=always

[Install]
WantedBy=multi-user.target
```

2.systemd 重新加载配置

``` shell
systemctl daemon-reload
```

3.启动进程

``` shell
#启动一个i进程
systemctl start yii-queue-xxxx@1 

#如果需要启动多个
systemctl start yii-queue-xxxx@2

```

4.加入开机启动

``` shell
systemctl enable yii-queue-xxxx@1 

```

5.systemctl  一些基础命令参考：


``` shell
# To start two workers
systemctl start yii-queue@1 yii-queue@2
# To get the status of running workers
systemctl status "yii-queue@*"
# To stop a specific worker
systemctl stop yii-queue@2
# To stop all running workers
systemctl stop "yii-queue@*"
# To start two workers at system boot
systemctl enable yii-queue@1 yii-queue@2

```

#附：
Systemd 详细介绍：http://www.ruanyifeng.com/blog/2016/03/systemd-tutorial-commands.html 