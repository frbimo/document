####  订阅了一个Instance，可以被多个App使用吗？可以被多个workspace使用吗？

1. 一个Instance可以被多个App使用，但是一个Shared DB不能被两个相同的App使用，比如两个Dashboard或者两个Datahub，因为两个相同的App具有相同的Schema，数据会被覆盖；
2. 一个Instance可以被同一个订阅号下的多个workspace或者namespace使用；

#### 如何查看数据库的连接信息？

1. 登录Service Portal，目前仅有订阅号Admin和订阅号User有权限查看订阅的Instance，不同站点的Service Portal地址如下：

| 站点代码 | 服务           | 站点地点          | 站点链接                                          |
| -------- | -------------- | ----------------- | ------------------------------------------------- |
| SA       | Service Portal | Azure Singapore   | https://portal-service-ensaas.sa.wise-paas.com    |
| HZ       | Service Portal | Alibaba  Hangzhou | https://portal-service-ensaas.hz.wise-paas.com.cn |
| JE       | Service Portal | Japan East        | https://portal-service-ensaas.jp.wise-paas.com    |

2. 在Instance页面点击Secret管理，进入Secret管理页面：

   ![image-20200710142415317](imgs/image-20200710142415317.png)

3. 选择App使用的Secret，点击View，查看Secret信息：	
   ![image-20200710142617588](imgs/image-20200710142617588.png)

4. Secret信息如下，即数据库的连接信息：

   ![image-20200714171238684](imgs/image-20200714171238684.png)

#### 如何获取数据库的外部连接地址？

Dedicated DB的外部连线信息可以从Secret中获取，Secret的查看方法可以参见上面问题，Credentials中的externalHosts即为外部连接地址。

目前Shared DB暂时没有提供外部连线地址，我们正在开发DB webctl，开发好后可以通过webctl连接DB。

#### 创建Secret的时候如何指定group？App如何授权schema和table给group？

在Parameters中加入group参数，如果只有一个group，假如group名称为g_group，如下：

![image-20200714163727281](imgs/image-20200714163727281.png)

如果要指定多个group，假如group名称为g_group1，g_group2，如下，参数名称填入groups，参数值填入["g_group1","g_group2"]：

![image-20200714163859045](imgs/image-20200714163859045.png)

App授权schema和table给group，假如group名称为g_group，可以执行以下语句：

```
CREATE SCHEMA IF NOT EXISTS "testSchema";
ALTER SCHEMA "testSchema" OWNER TO "g_group";
CREATE TABLE IF NOT EXISTS testSchema.testTable;
ALTER TABLE testSchema.testTable OWNER to "g_group";
GRANT ALL ON ALL TABLES IN SCHEMA "testSchema" TO "g_group";
GRANT ALL ON ALL SEQUENCES IN SCHEMA "testSchema" TO "g_group"; 
```

关于group管理机制，详细请参考《Group管理机制》。

#### App访问数据库出现连接被拒绝，提示连接数到达限制？

每个Shared PostgreSQL有100个连线数的限制，App在使用PostgreSQL的时候为了提高性能，可能会开很多条连接，如果多个App同时使用一个Shared PostgreSQL，很容易出现连线数满的情况，如果100个连线数不能满足需求，建议升级Shared DB为Dedicated DB，升级方法参考下面问题。

#### 如何升级Shared DB为Dedicated DB？

1. 订阅一个Dedicated PostgreSQL，订阅方法请参考《快速入门》；

2. 如果需要迁移数据，请先联系SRE迁移数据：WISE-PaaS.SRE@advantech.com.cn；

3. 登录Service Portal，将Shared DB Instance中创建的Secret删掉，删掉之前记录下Secret的名称和参数，参数从View Secret页面查看；

   ![image-20200714184538303](imgs/image-20200714184538303.png)

4. 在Dedicated DB Instance中创建同名的Secret，并指定相同的参数；

5. 重启App；

#### 如何将一个App从一个DB Instance迁移到另一个DB Instance？

用户如果订购了多个PostgreSQL实例，想将一个App从一个DB Instance1迁移到另一个DB Instance2，以Dashboard为例，步骤如下：

1. 如果需要迁移数据，请先联系SRE迁移数据：WISE-PaaS.SRE@advantech.com.cn；

2. 登录Service Portal，将DB Instance1中创建的dashboard Secret删掉（WISE-PaaS平台提供的App使用的Secret名称有一定规范，格式为：ServiceName-NamespaceName-secret，所以根据App名称可以找到其对应的Secret），删掉之前记录下Secret的名称和参数，参数从View Secret页面查看；

   ![image-20200714184727412](imgs/image-20200714184727412.png)

   ![image-20200714184538303](imgs/image-20200714184538303.png)

3. 在DB Instance2中创建同名的Secret，并指定相同的参数；

4. 重启App；