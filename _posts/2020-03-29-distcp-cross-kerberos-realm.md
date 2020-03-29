---
title: 跨Kerberos的集群间distcp
tags: 技术 大数据  kerberos hadoop distcp
--- 

对于启用了kerberos认证的两个集群之间，如何进行分布式拷贝(distcp)。做一个记录和备忘。主要是如何设置kerberos之间的双向信任。
<!--more-->
[toc]

# 一.  从有kerberos认证的集群 向 无keberos认证的集群 复制文件
## 1.1  场景描述
CLUSTER-A集群开启了Kerberos认证， CLUSTER-B集群无Kerberos认证。
在CLUSTER-A集群上，运行 distcp 命令，复制文件到CLUSTER-B集群。
## 1.2  问题备忘
**报错：java.io.IOException: Server asks us to fall back to SIMPLE auth, but this client is configured to only allow secure connections.**
客户端配置了安全连接， 但是服务器请求回退到非安全的SIMPLE模式。
对于单次运行的命令行，可以使用 -D ipc.client.fallback-to-simple-auth-allowed=true 参数，例如：

```bash
hadoop distcp -D ipc.client.fallback-to-simple-auth-allowed=true /tmp/testfile hdfs://cluster-b-namename:8020/tmp/
```

对于经常执行的集群之间，可以在core-site.xml中设置此参数，如果使用CDH集群，Cloudera Manager上面，设置对应的core-site.xml的集群高级配置代码段。




# 二.  两个独立Kerberos认证的集群 

## 2.1  场景描述
对于两个独立的开启Kerberos认证的集群，比如CLUSTER-DEV 和 CLUSTER-PROD环境。
**CLUSTER-DEV：**
表 2.1  CLUSTER-DEV集群说明

|   IP地址   |   host name   |  角色    |
| ---- | ---- | ---- |
| 192.168.1.250 | cluster-dev-master   | Hadoop NameNode |
| 192.168.1.249 | cluster-dev-worker01 | Hadoop NameNode |
| 192.168.1.240 | cluster-dev-kerberos | Kerberos KDC |

**CLUSTER-PROD：**
192.168.进行集群间通信，172.16进行集群内通信
表 2.2  CLUSTER-PROD集群说明

|   IP地址   |   host name   |  角色    |
| ---- | ---- | ---- |
| 192.168.1.221 / 172.16.1.221 | cluster-prod-master01 | Hadoop NameNode |
| 192.168.1.222 / 172.16.1.222 | cluster-prod-master02 | Hadoop NameNode |
| 192.168.1.235 / 172.16.1.235 | cluster-prod-kerberos |  Kerberos KDC |

需要按照下列步骤，进行Kerberos域的双向认证。
## 2.2  配置步骤
### 2.2.1  配置/etc/krb5.conf
- **配置[realms]段，加入另外一个KDC服务器。**

```ini
[realms]
    CLUSTER-DEV = {
        kdc = cluster-dev-kerberos
        admin_server = cluster-dev-kerberos
        default_principal_flags = +renewable
        default_domain = cluster-dev
    }
    CLUSTER-PROD = {
        kdc = cluster-prod-kerberos
        admin_server = cluster-prod-kerberos
        default_principal_flags = +renewable
        default_domain = cluster-prod
    }
```

- **配置[domain_realm]段，特定域名的主机属于特定的Kerberos域。**
因为我们主机命名没有使用FQDN的域名方式，所以无法通过域名后缀判断所属的Kerberos域，需要额外指定哪些主机是属于哪些域的。
```ini
[domain_realm]
    .cluster-prod = CLUSTER-PROD
    cluster-prod = CLUSTER-PROD
    .cluster-dev = CLUSTER-DEV
    cluster-dev = CLUSTER-DEV
    # 因为我们的主机命名，没有遵循FQDN的方式，所以要指明那些主机属于另外一个域。
    cluster-prod-master01 = CLUSTER-PROD
    cluster-prod-master02 = CLUSTER-PROD
    cluster-dev-master = CLUSTER-DEV
    cluster-dev-worker01 = CLUSTER-DEV
```

- **配置[capaths]段**
一些手册写配置此项，但是我试了一下，配不配都不影响。
对于CLUSTER-PROD集群， capaths段配置：
```ini
#[capaths]
#    CLUSTER-PROD = {
#       CLUSTER-DEV = .
#    }
```
对于CLUSTER-DEV集群，capaths段配置：
```ini
#[capaths]
#    CLUSTER-DEV = {
#        CLUSTER-PROD = .
#    }
```

理论上，是应该把这些krb5.conf复制到集群的全部节点当中。但是测试发现，只要通信的两个节点之间的krb5.conf是对的，就可以了。不过还是建议遵从参考建议，保持krb5.conf在集群中的一致性。

### 2.2.2  创建krbtgt互信principal
在两个集群的KDC上，运行kadmin.local
> **注意：**
>
> 1. 密码要一致。
> 2. **这两个 /域1@域2，是相反的。**
```bash
kadmin.local:  addprinc krbtgt/CLUSTER-DEV@CLUSTER-PROD
kadmin.local:  addprinc krbtgt/CLUSTER-PROD@CLUSTER-DEV
```

### 2.2.3  设置Hadoop集群参数
- **设置 hdfs-site.xml 的 dfs.namenode.kerberos.principal.pattern=***
对于CDH集群，这一步可以通过Cloudera Manager设置hdfs-site.xml 的 HDFS 客户端高级配置代码段实现。

- **设置信任域**
设置hadoop.security.auth_to_local规则， 此项规则书写较为复杂，CDH集群可以通过Cloudera Manager配置受信任的Kerberos领域，生成此配置。或者参考本文最后的参考链接，里面有解释如何写这些规则
实际在core-site.xml中生成的属性，类似这样：
```xml
  <property>
    <name>hadoop.security.auth_to_local</name>
    <value>RULE:[1:$1@$0](.*@\QCLUSTER-PROD\E$)s/@\QCLUSTER-PROD\E$//
RULE:[2:$1@$0](.*@\QCLUSTER-PROD\E$)s/@\QCLUSTER-PROD\E$//
RULE:[1:$1@$0](.*@\QCLUSTER-DEV\E$)s/@\QCLUSTER-DEV\E$//
RULE:[2:$1@$0](.*@\QCLUSTER-DEV\E$)s/@\QCLUSTER-DEV\E$//
DEFAULT</value>
  </property>
```

### 2.2.4  验证测试
此时，distcp应该就可以了。
但是建议，使用webhdfs:// 协议，替代hdfs://协议。

## 2.3  问题备忘
1. **krbtgt@这个principal的密码，必须一致么？**
两个集群之间必须一致。

2. **其他用户，比如user@CLUSTER-DEV 和 user@CLUSTER-PROD 在集群间密码一致么？**
   测试时，user用户不一致的密码，可以通过webhdfs进行访问。

3. **两个集群都要创建两个krbtgt么？**
   少一个，会变成单向的。可以根据场景来选择。
   例如在CLUSTER-PROD集群中，
   删掉krbtgt/CLUSTER-PROD@CLUSTER-DEV，PROD可以读取DEV，但是DEV不可读取PROD
   删掉krbtgt/CLUSTER-DEV@CLUSTER-PROD，DEV 可以读取PROD， 但是PROD不可以读取DEV。
4. **为什么建议使用webhdfs://替代hdfs://**
   因为我们发现，使用双IP的集群，使用hdfs:// 协议，distcp启动Yarn的ResouceManager的时候，创建连接的接口不正确。导致无法连接到对端。导致Yarn Job一致处于NEW的状态，无法被提交到Yarn集群运行。（可能是distcp on yarn的bug？）


# 附录A	参考资料

- https://community.cloudera.com/t5/Community-Articles/Kerberos-cross-realm-trust-for-distcp/ta-p/245590 闯哥找到的参考链接，比较详细。
- https://community.pivotal.io/s/article/Setting-up-a-kerberos-cross-realm-trust-for-distcp 这是pivotal给出的文档，写到也比较详细。
- https://community.cloudera.com/t5/Support-Questions/distcp-between-2-kerberized-clusters-Fails-due-to/td-p/156026 这个链接里面有人给出了一个PDF，讲了如何配置Kerberos Realm间的信任。
- https://docs.cloudera.com/documentation/enterprise/5-5-x/topics/cdh_admin_distcp_data_cluster_migrate.html Cloudera的另外一个参考。

