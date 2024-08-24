## 安装及启动

 - 这里有简单易懂的教程，就不在这里赘诉了，奉上链接。[Elasticsearch 入门：安装及启动Elasticsearch](https://www.jianshu.com/p/cf4d8f60cef8)
## 注意事项
#### 1、不能用root用户运行
 在启动过程中遇到的问题：不能用root用户运行，这是因为**ES对权限的严格控制**

```bash
Caused by: java.lang.RuntimeException: can not run elasticsearch as root
```
对于该问题的解决办法：授权给另一个账户，用另一个账户启动。eugene是我的另外一个普通账户

```bash
[root@VM_0_14_centos elasticsearch]# chown -R eugene elasticsearch-6.4.2/
[root@VM_0_14_centos elasticsearch]# su - eugene
[eugene@VM_0_14_centos ~]$ cd /usr/local/elasticsearch/
[eugene@VM_0_14_centos elasticsearch]$ ./elasticsearch-6.4.2/bin/elasticsearch
```
#### 2、一个可以忽略的错误，跟系统版本有关
```bash
java.lang.UnsupportedOperationException: seccomp unavailable: CONFIG_SECCOMP not compiled into kernel, CONFIG_SECCOMP and CONFIG_SECCOMP_FILTER are needed
```
## 一些坑
#### 1、外网无法访问
防火墙问题，关注netstat命令下的IP及端口是不是0.0.0.0:9200，若是127.0.0.1：9200表示仅可本地访问，需要配置在elasticsearch.yml文件中配置：
```bash
network_host: 0.0.0.0
```
#### 2、在我的centos6.9上面还会出现两个错误，根据对应的错误去排查即可
- 报错1，出现如下：
```bash
system call filters failed to install; check the logs and fix your configuration or disable system c
```
通过在elasticsearch.yml文件中配置：

```bash
bootstrap.memory_lock: false
bootstrap.system_call_filter: false
```
- 报错2，出现如下：

```bash
max virtual memory areas vm.max_map_count [65530] is too low, increase to at least [655360]
```
解决办法：切换到root用户，修改系统配置文件/etc/sysctl.conf，增加：

```bash
vm.max_map_count=655360
```
修改完了之后记得执行命令保存：

```bash
sysctl -p
```
## [京东最佳实践](https://mp.weixin.qq.com/s/SETIw2uB-G42leC4WUm6UQ)

