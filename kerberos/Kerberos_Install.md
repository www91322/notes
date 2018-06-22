# Kerberos Install



![Kerberos](https://www.kerberos.org/images/krbmsg.gif)

## Kerberos 安装

### 节点信息

```
name01
datanode01
datanode02
datanode03
datanode04
```

### 安装 kdc server
	
- 在KDC(name01)上安装包 krb5、krb5-server 和 krb5-client

	```
	yum install krb5-server krb5-libs krb5-auth-dialog krb5-workstation  -y
	```
	
- 安装  krb5-devel、krb5-workstation

	```
	yum install krb5-devel krb5-workstation -y
	```
	
### 修改配置文件

- kdc 服务器涉及到三个配置文件：
	
	```
	/etc/krb5.conf
	/var/kerberos/krb5kdc/kdc.conf
	```
		
- 修改	krb5.conf
	
	```
	[libdefaults]
	default_realm = HADOOP.COM
	dns_lookup_kdc = false
	dns_lookup_realm = false
	ticket_lifetime = 86400
	renew_lifetime = 604800
	forwardable = true
	default_tgs_enctypes = rc4-hmac
	default_tkt_enctypes = rc4-hmac
	permitted_enctypes = rc4-hmac
	udp_preference_limit = 1
	kdc_timeout = 3000
	[realms]
	HADOOP.COM = {
		kdc = name01
		admin_server = name01
	}
	[domain_realm]

	```
	
	**说明：**
	
	- [libdefaults]：每种连接的默认配置，需要注意以下几个关键的小配置
		- default_realm = HADOOP.COM：设置 Kerberos 应用程序的默认领域。如果您有多个领域，只需向 [realms] 节添加其他的语句。
		- udp_preference_limit= 1：禁止使用 udp 可以防止一个Hadoop中的错误
		- clockskew：时钟偏差是不完全符合主机系统时钟的票据时戳的容差，超过此容差将不接受此票据。通常，将时钟扭斜设置为 300 秒（5 分钟）。这意味着从服务器的角度看，票证的时间戳与它的偏差可以是在前后 5 分钟内。
		- ticket_lifetime： 表明凭证生效的时限，一般为24小时。
		- renew_lifetime： 表明凭证最长可以被延期的时限，一般为一个礼拜。当凭证过期之后，对安全认证的服务的后续访问则会失败。
	- [realms]：列举使用的 realm。
		- kdc：代表要 kdc 的位置。格式是 机器:端口
		- admin_server：代表 admin 的位置。格式是 机器:端口
		- default_domain：代表默认的域名
	- [appdefaults]：可以设定一些针对特定应用的配置，覆盖默认配置。
		
-  	修改 `/var/kerberos/krb5kdc/kdc.conf`

	```
	[kdcdefaults]
	 kdc_ports = 88
	 kdc_tcp_ports = 88
	
	[realms]
	 HADOOP.COM = {
	  #master_key_type = aes256-cts
	  acl_file = /var/kerberos/krb5kdc/kadm5.acl
	  dict_file = /usr/share/dict/words
	  max_life = 5d
	  max_renewable_life = 10d
	  admin_keytab = /var/kerberos/krb5kdc/kadm5.keytab
	  supported_enctypes = aes256-cts:normal aes128-cts:normal des3-hmac-sha1:normal arcfour-hmac:normal camellia256-cts:normal camellia128-cts:normal des-hmac-sha1:normal des-cbc-md5:normal des-cbc-crc:normal
	 }
	```

### 同步配置文件

```
sudo scp /etc/krb5.conf  datanode01:/etc/
sudo scp /etc/krb5.conf  datanode02:/etc/
sudo scp /etc/krb5.conf  datanode03:/etc/
sudo scp /etc/krb5.conf  datanode04:/etc/
```


### 创建数据库(在name01节点)	

```
kdb5_util create -r HADOOP.COM -s
```

### 启动服务(在name01节点)	

```
chkconfig --level 35 krb5kdc on
chkconfig --level 35 kadmin on
service krb5kdc start
service kadmin start
```

###  创建 kerberos 管理员

```
kadmin -q "addprinc admin/admin"
```
手动动输入两次密码


## 常见问题

- HUE+kerberos启动报错Couldn't renew kerberos ticket解决方案

	- 异常信息
		
		```
		Couldn't renew kerberos ticket in order to work around Kerberos 1.8.1 issue. Please check that the ticket for 'hue/datanode03@HADOOP.COM' is still renewable:
  $ klist -f -c /var/run/hue/hue_krb5_ccache
If the 'renew until' date is the same as the 'valid starting' date, the ticket cannot be renewed. Please check your KDC configuration, and the ticket renewal policy (maxrenewlife) for the 'hue/datanode03@HADOOP.COM' and `krbtgt' principals.
[15/Jun/2018 11:28:50 ] settings     INFO     Welcome to Hue 3.9.0
		```
	
	- 解决方式
		
		- [HUE+kerberos启动报错Couldn't renew kerberos ticket解决方案](https://blog.csdn.net/vah101/article/details/79111585)

## 参考资料

- [Enable Hue to Work with Hadoop Security using Cloudera Manager](http://www.cloudera.com/documentation/manager/5-1-x/Configuring-Hadoop-Security-with-Cloudera-Manager/cm5chs_enable_hue_sec_s10.html)

- [KERBEROS PROTOCOL TUTORIAL](https://www.kerberos.org/software/tutorial.html)

- [kerberos使用手册](http://wzktravel.github.io/2016/03/01/how-to-use-kerberos-in-CDH/)

- [kerberos体系下的应用(yarn,spark on yarn)](https://www.jianshu.com/p/ae5a3f39a9af)

- [CDH集群中启用kerberos认证](https://blog.csdn.net/sinat_32176947/article/details/79602351)

- [在启用Kerberos的CDH中部署及使用Kylin](https://zhuanlan.zhihu.com/p/38008683)