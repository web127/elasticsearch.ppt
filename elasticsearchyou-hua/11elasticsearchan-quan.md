
![](/assets/46.png)# ElasticSearch安全插件x-pack

x-pack是elasticsearch官方的一个扩展包，将安全，警告，监视，图形和报告功能捆绑在一个易于安装的软件包中

x-pack默认只有一个月的试用期

#### x-pack破解版安装

1.破解jar文件
下载x-pack
https://artifacts.elastic.co/downloads/packs/x-pack/x-pack-6.2.2.zip

下载后在/plugins/x-pack-core/x-pack-core-6.2.2.jar 使用 luyten破解导出
luyten项目地址:https://github.com/deathmarine/Luyten

LicenseVerifier 中有两个静态方法，这就是验证授权文件是否有效的方法，我们把它修改为全部返回true
XPackBuild 中 最后一个静态代码块中 try的部分全部删除，这部分会验证jar包是否被修改

在es的lib目录下载这些jar依赖
elasticsearch-6.2.2.jar
lucene-core-7.2.1.jar
x-pack-core-6.2.2.jar
elasticsearch-core-6.2.2.jar

```
javac -cp "elasticsearch-6.2.2.jar;lucene-core-7.2.1.jar;x-pack-core-6.2.2.jar" LicenseVerifier.java
javac -cp "elasticsearch-6.2.2.jar;lucene-core-7.2.1.jar:x-pack-core-6.2.2.jar;elasticsearch-core-6.2.2.jar" XPackBuild.java
```

替换破解的jar包到x-pack-6.2.2.zip

2.离线安装x-pack-6.2.2.zip

停止所有ES集群节点，安装x-pack插件，安装完后会在config下生成elasticsearch.keystore

```
./elasticsearch-plugin  install file:///usr/local/x-pack-6.2.2.zip
```
![](/assets/47.png)

3.更改license
官网申请免费[license](https://license.elastic.co/registration)，会发邮件给你进行下载
下载的文件重命名为license.json，并做如下修改：
```
"type":"platinum"   #白金版"expiry_date_in_millis":2524579200999   #截止日期 2050年
```
4.设置elasticsearch.yml

```
xpack.security.enabled:false
```
重启所有ES集群，这里所有节点都要启动
5.安装kibana的x-pack插件

```
./kibana-plugin  install file:///usr/local/x-pack-6.2.2.zip
```
导入license

![](/assets/43.png)
![](/assets/44.png)
![](/assets/45.png)
查看license：
![](/assets/46.png)

6.因为我们导入的不是试用版的license 所以如果我们要开启安全验证 必须要配置集群内部通讯的TLS/SSL

##### 步骤如下：

1）输入一个自定义的密码， 或者您可以按enter键将密码留空。生成身份文件

```
/usr/local/elasticsearch-6.2.2/bin/x-pack/certutil ca
```
```
/usr/local/elk/elasticsearch-6.2.3/bin/x-pack/certutil cert --ca elastic-stack-ca.p12
```

![](/assets/48.png)
![](/assets/49.png)

2).在config目录下创建certs目录，将生成的p12文件拷贝进去

3).在config目录下创建certs目录，将生成的p12文件拷贝进去
在elasticsearch.yml配置文件中添加如下几行：

```
xpack.security.transport.ssl.enabled: true
xpack.security.transport.ssl.verification_mode: certificate 
xpack.security.transport.ssl.keystore.path: certs/elastic-certificates.p12 
xpack.security.transport.ssl.truststore.path: certs/elastic-certificates.p12
```
4)使用密码保护节点的证书，请将您的密码添加到elasticsearch秘钥库：

```
bin/elasticsearch-keystore add xpack.security.transport.ssl.keystore.secure_password
bin/elasticsearch-keystore add xpack.security.transport.ssl.truststore.secure_password
```

5).elasticsearch.yml更改配置，重启集群
```
xpack.security.enabled: true
action.auto_create_index: true
```
重启ES集群

6).执行/bin/x-pack/setup-passwords interactive 此只需要在一个节点上执行

7).拷贝生成的证书到其他节点（不能在其他节点直接生成）


此时已经完成了



如果想在集群之间使用TSL/SSL，继续：
8).集群上设置TSL/SSL：
vim instances.yml
这个相当于白名单一样的东西，配置了ip的服务器节点可以访问集群
```
instances:
   - name: "node1"
     ip:
       - "172.16.16.179"
   - name: "node2"
     ip:
       - "172.16.16.179"
   - name: "node3"
     ip:
       - "172.16.16.178"
```
9).生成证书
./x-pack/certgen -in instances.yml

![](/assets/65.png)

10).添加如下配置：

```
xpack.security.enabled: truexpack.ssl.key: /home/zhaheng/elasticsearch1/config/certs/node1/node1.keyxpack.ssl.certificate: /home/zhaheng/elasticsearch1/config/certs/node1/node1.crtxpack.ssl.certificate_authorities: /home/zhaheng/elasticsearch1/config/certs/ca/ca.crt
```


xpack.security.enabled: true
xpack.ssl.key: /home/zhaheng/elasticsearch2/config/certs/node2/node2.key
xpack.ssl.certificate: /home/zhaheng/elasticsearch2/config/certs/node2/node2.crt
xpack.ssl.certificate_authorities: /home/zhaheng/elasticsearch2/config/certs/ca/ca.crt
```


```
xpack.security.enabled: true
xpack.ssl.key: /home/zhaheng/elasticsearch3/config/certs/node3/node3.key
xpack.ssl.certificate: /home/zhaheng/elasticsearch3/config/certs/node3/node3.crt
xpack.ssl.certificate_authorities: /home/zhaheng/elasticsearch3/config/certs/ca/ca.crt
```


重新启动Elasticsearch

./elasticsearch1/bin/elasticsearch -d
./elasticsearch2/bin/elasticsearch -d
./elasticsearch3/bin/elasticsearch -d

11).如果没用初始化密码，初始化es默认账号的密码：

```
bin/x-pack/setup-passwords interactive
```