# ElasticSearch安全插件x-pack

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

javac -cp "elasticsearch-6.2.2.jar;lucene-core-7.2.1.jar;x-pack-core-6.2.2.jar" LicenseVerifier.java
javac -cp "elasticsearch-6.2.2.jar;lucene-core-7.2.1.jar:x-pack-core-6.2.2.jar;elasticsearch-core-6.2.2.jar" XPackBuild.java

替换破解的jar包到x-pack-6.2.2.zip

2.离线安装x-pack-6.2.2.zip

停止所有ES集群节点，安装x-pack插件

```
./elasticsearch-plugin  install file:///usr/local/x-pack-6.2.2.zip
```
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


