solr是一个全文检索框架

[官网地址](http://lucene.apache.org/solr/)
[文档下载](http://apache.mirror.vexxhost.com/lucene/solr/ref-guide/apache-solr-ref-guide-6.3.pdf)

### 开始使用

准备环境：JDK1.8(最低版本)
本文演示solr版本：6.3.0

建议尽量在linux或者类unix环境中运行使用solr。这里，我搭建了一个ubuntu 16的虚拟机用来演示运行。

![](http://p1.bqimg.com/567571/7129966a40aeefa1.jpg)

进入[下载页面](http://www.apache.org/dyn/closer.lua/lucene/solr/6.3.0)下载后解压到桌面（建议翻墙下载，不翻墙容易中途断掉）

![](http://p1.bqimg.com/567571/be07530f727cfba9.jpg)

#### 启动

1. linux下进入目录后执行：`bin/solr start`
2. windows下进入目录后执行:`bin/solr.cmd start`

linux下面启动成功如图：
![](http://p1.bqimg.com/567571/d649e654b34f9f18.jpg)
进入浏览器访问如下地址：http://localhost:8983/solr 看到如下界面说明solr启动成功
![](http://p1.bqimg.com/567571/45ea18ca85c2c664.jpg)

可以通过执行命令`bin/solr -help`查看帮助。

**下面是一些常用的启动参数：**

* 默认情况下，solr是在后台启动的，如果你不用后台启动的方式可以在启动的时候加一个`-f`参数，`bin/solr start -f`
* solr默认启动的端口是8983，你可以在启动的时候使用`-p`参数指定端口启动，`bin/solr start -p 8090`使用8090端口启动
* solr在一开始就已经提供了一些用于学习的例子，也可以在启动的时候直接启动这些实例用于学习。例如：`bin/solr start -e techproducts`,提供了四个例子：
	* cloud ： This example starts a 1-4 node SolrCloud cluster on a single machine. When chosen, an interactive session will start to guide you through options to select the initial configset to use, the number of nodes for your example cluster, the ports to use, and name of the collection to be created. When using this example, you can choose from any of the available configsets found in $SOLR_HOME/server/solr/configsets.
	* techproducts： This example starts Solr in standalone mode with a schema designed for the sample documents included in the $SOLR_HOME/example/exampledocs directory. The configset used can be found in $SOLR_HOME/server/solr/configsets/sample_techproducts_configs.
	* dih： This example starts Solr in standalone mode with the DataImportHandler (DIH) enabled and several example dataconfig.xml files pre-configured for different types of data supported with DIH (such as, database contents, email, RSS feeds, etc.). The configset used is customized for DIH, and is found in $SOLR_HOME/example/example-DIH/solr/conf. For more information about DIH, see the section Uploading Structured Data Store Data with the Data Import Handler.
	* schemaless： This example starts Solr in standalone mode using a managed schema, as described in the section Schema Factory Definition in SolrConfig, and provides a very minimal pre-defined schema. Solr will run in Schemaless Mode with this configuration, where Solr will create fields in the schema on the fly and will guess field types used in incoming documents. The configset used can be found in $SOLR_HOME/server/solr/configsets/data_driven_schema_configs.

#### 创建一个Core

如果我们在启动solr的时候没有启动任何示例配置的情况下，我们需要创建一个core并创建索引。

如果使用上面的命令`bin/solr start`启动了solr并且没有关闭的情况下，我们可以用命令`bin/solr create -c start`创建一个core(创建core需要保证solr并没有停止) 

在管理页面刷新可以看见多了一个start

![](http://p1.bpimg.com/567571/f7ac5059e984549f.jpg)

#### 增加一个文件夹的索引

创建文件的索引需要用到post这个工具，这里我们以创建一个文件夹(当前目录下的docs文件夹)的索引为例：

执行命令`bin/post -c start docs/`稍等一下等命令执行完毕，`start`是我们上面刚刚创建的core，docs是我们当前目录的文件夹。如此创建完毕后就可以开始使用搜索功能了。

#### 体验简单的搜索

 这里为了方便我就不在虚拟机里面进行搜索了，记住了虚拟机的IP地址就可以在主机访问。

![](http://p1.bpimg.com/567571/24ab541a2373c135.jpg)

点击进入后选择刚刚创建的`start`

![](http://p1.bpimg.com/567571/0af529009d1bc2c5.jpg)

选择Query，并填入index，点击执行查询得到如下结果：

![](http://p1.bpimg.com/567571/0cfd0c83a81069b3.jpg)

可以查看右边的数据，QTime是耗时（应该是毫秒单位）。response下面的numFound意思就是一共发现了413条记录。。。

从速度可以看出，检索查询的速度快快哒。


#### 从mysql导入数据到solr

https://wiki.apache.org/solr/DataImportHandler#A_shorter_data-config

**首先准备Mysql的环境：**

这里我创建了一个名为solr的数据库，字符集为utf-8。里面创建了两张表，创建的代码分别如下：

question表
```
DROP TABLE IF EXISTS `question`;
CREATE TABLE `question` (
  `id` int(11) NOT NULL AUTO_INCREMENT,
  `title` varchar(255) NOT NULL,
  `create_time` datetime NOT NULL,
  PRIMARY KEY (`id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8;
```

replay表（现在创建备用）
```
DROP TABLE IF EXISTS `replay`;
CREATE TABLE `replay` (
  `id` int(11) NOT NULL AUTO_INCREMENT,
  `q_id` int(11) NOT NULL,
  `content` varchar(255) NOT NULL,
  `create_time` datetime NOT NULL,
  PRIMARY KEY (`id`,`q_id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8;
```

至于数据，就随机导入了。我不提供初始数据了。

**接下来是solr的部分：**

首先下载[mysql-connector-java](http://dev.mysql.com/downloads/connector/j/)

解压其中的mysql-connector-java-xx.xx.jar到`server/solr-webapp/webapp/WEB-INF/lib`,如下图：

![](http://p1.bqimg.com/567571/0ab07862e7dbc5a1.jpg)

还需要导入solr-dataimporthandler-6.3.0.jar,到`server/solr-webapp/webapp/WEB-INF/lib`。这个jar在dist目录可以找到。

创建一个core名字为for_mysql.命令是`bin/solr create -c for_mysql`。`server/solr/`目录下面就是存放的core的配置以及索引等信息。

例如刚刚创建的`for_mysql`的目录就是`server/solr/for_mysql`。

进入`server/solr/for_mysql/conf`目录，打开`managed-schema`这个文件，加入以下代码：

```
<field name="id" type="string" multiValued="false" indexed="true" required="true" stored="true"/>
<field name="title" type="string" indexed="true" stored="false" required="true" multiValued="false"/>
<field name="create_time" type="date" indexed="true" stored="true" required="true" multiValued="false"/>
```
一些情况下：在我们添加完成上面的代码重启solr时会出现一下错误：

![](http://p1.bpimg.com/567571/2f8c2223e0ee2624.jpg)

那就是managed-schema里面已经有了id这个字段的配置，我们把上面的关于id那一行删除即可。。


**todo：解释各个字段**


在`server/solr/for_mysql/conf`目录下面创建一个xml文件名字为：`data-config.xml`填入以下内容：

```
<?xml version="1.0" encoding="UTF-8" ?>  
<dataConfig>   
<dataSource name="fromMysql"
          type="JdbcDataSource"   
          driver="com.mysql.jdbc.Driver"   
          url="jdbc:mysql://192.168.1.179:3306/solr"   
          user="root"   
          password="password"/>   
<document>   
    <entity dateSource="fromMysql" pk="id" name="question" query="SELECT * FROM question" >
         <field column="id" name="id"/> 
         <field column="title" name="title"/> 
         <field column="create_time" name="create_time"/>
    </entity>   
</document>   

</dataConfig>
```

上面的代码和我们配置jdbc数据源类似，dataSource的name是随意取的，但是下面的节点需要和dataSource保持一致。下面的entity节点和hibernate差不多，column和数据库的字段保持一致，name和前面`managed-schema`文件配置的保持一致。

然后在同目录下的`solrconfig.xml`添加一下节点：

```
<requestHandler name="/dataimport" class="org.apache.solr.handler.dataimport.DataImportHandler">
    <lst name="defaults">
      <str name="config">data-config.xml</str>
     </lst>
  </requestHandler>
```

节点添加在<requestHandler name="/select"这个节点前面，防止添加节点出错。这个节点的添加是为了引用我们前面创建的data-config.xml的配置到配置文件中。

![](http://i1.piimg.com/567571/b3901307556a8b4b.jpg)

保存后重启solr.

打开浏览器可以看见多出了一个配置项：

![](http://i1.piimg.com/567571/ae863d53d44e6050.jpg)

然后如下图所示：

![](http://p1.bpimg.com/567571/86cab04870f78007.jpg)

按照图中选择完毕后点击execute按钮。Clean选项表示删除以前的索引，full-import表示引入完整的数据，如果执行成功会如下图所示：

![](http://p1.bpimg.com/567571/8c43de10daba17d1.jpg)

表示导入数据到solr成功。

至此，从mysql导入数据到solr完成，接下来就是如何进行简单的全文检索了~~~

#### 同一张表中不同字段的检索

进入`server/solr/for_mysql/conf`目录，打开`managed-schema`这个文件，加入以下代码：

```
<field name="text" type="text_general" indexed="true" stored="false" multiValued="true" />
<copyField source="title" dest="text"/>
<copyField source="id" dest="text"/>
```

大致解释一下代码的意思，第一行的意思就是建立一个新的字段名为text,下面的copyField就是把前面配置的title和id的数据复制到text这个节点中去。配置完毕后重启solr。重启以后记得重新导入一下mysql的数据，打开query界面，点击execute query按钮如下图：
![](http://p1.bqimg.com/567571/92f8c215043b1b4e.jpg)

如图会获取到全部的数据。上面的q表示搜索的内容，`*:*`表示从所有字段中检索所有内容。

接下来，我们要实现同一张表中的不同字段的检索只需要输入：`text:xxx` xxx是你想搜索的内容。

比如我们这里输入： 人 就可以看见能搜索出title带有 人 的内容。
![](http://p1.bqimg.com/567571/259a4aa71a6a98e3.jpg)


再输入：3

![](http://p1.bqimg.com/567571/15155c19e763e15d.jpg)

可以看到能搜索出id为3的内容。

下面还有一种更简便的方法，我们在`server/solr/for_mysql/conf/managed-schema`这个文件添加如下节点：

```
<defaultSearchField>text</defaultSearchField>
```

意思是让text这个字段作为我们默认的搜索字段。

重新换一种查询方式：

![](http://p1.bqimg.com/567571/853a4bf7e6e16f00.jpg)

在df填入上面配置的text后直接搜索3就能得到和上面一样的结果。。。是不是更加方便了呢~~

#### 多表查询

先看一个非常常用的需求，如同我们上面建立的两张表。如果我们做搜索功能，肯定是从标题和回复的内容中做检索。这久涉及到了从不同的两张表查询的需求。

因为solr并不支持一对多或者多对一的查询，所以我们需要导入数据到solr的过程中就做成一张表的形式（solr也可以进行子查询的方式，但是对于我们的需求来说，子查询的方式并不适合）。

由于是两张表的查询，为了避免混淆我们重新编辑`managed-schema`如下：

```
<field name="id" type="string" multiValued="false" indexed="true" required="true" stored="true"/>
<field name="title" type="text_general" multiValued="false" indexed="true" required="true" stored="true" />
<field name="create_time" type="date" indexed="true" stored="true" required="true" multiValued="false"/>


<field name="r_id" type="string" multiValued="false" indexed="true" required="false" stored="true"/>
<field name="q_id" type="string" indexed="true" stored="true" required="false" multiValued="false"/>
<field name="content" type="text_general" multiValued="true" indexed="true" required="false" stored="true" />
<field name="r_create_time" type="date" indexed="true" stored="true" required="false" multiValued="false"/>

<field name="text" type="text_general" indexed="true" stored="false" multiValued="true" />
<copyField source="title" dest="text"/>
<copyField source="id" dest="text"/>
<copyField source="content" dest="text"/>
<defaultSearchField>text</defaultSearchField>
```

`data-config.xml`文件变更如下：

```
<?xml version="1.0" encoding="UTF-8" ?>  
<dataConfig>   
<dataSource name="fromMysql"
          type="JdbcDataSource"   
          driver="com.mysql.jdbc.Driver"   
          url="jdbc:mysql://192.168.1.179:3306/solr"   
          user="root"   
          password="bpaTH330"/>   
<document>   
    <entity dateSource="fromMysql" pk="id"  name="question" query="SELECT r.id,q.id as q_id,q.title,r.id AS r_id,r.content,r.create_time  FROM replay AS r,question as q WHERE r.q_id = q.id" >
         <field column="id" name="id"/>
	 	 <field column="title" name="title"/>
         <field column="q_id" name="q_id"/> 
         <field column="content" name="content"/>
	     <field column="create_time" name="create_time"/>
    </entity>   
</document>   

</dataConfig>

```

上述更改了`query`的语句。需要注意的是此时的id已经不是`question`表的id了，而是`replay`表的id。如果继续使用`question`表的id作为主键的话，查询时solr会过滤掉id重复的实体，返回的实体会不全。

上述更改完成后重启solr。并且重新导入数据，导入数据后查询大概会显示如下的结果：

![](http://i1.piimg.com/567571/632f26840f6f9604.jpg)

验证一下是否能正常的查询：分别以content和title里面的个别字进行搜索

![](http://p1.bqimg.com/567571/7c3fdd8d2bea62e2.jpg)

![](http://p1.bqimg.com/567571/3880b3ed456979d7.jpg)

应该是能分别检索出我们想要的结果。但是会有一个问题，下节会说到。


#### 简单的去除重复和高亮检索中的文本

**去重**
从上面的查询不难看出有一个非常严重的问题，当我们在以标题搜索的时候。会导致把标题和搜索内容高度重合的所有回答都检索出来，很明显。这样不是我们想要的结果，应该尽量避免这种情况出现。我试着用知乎的搜索功能找到一个标题搜索了一下，不会重复出现。但是会出现一个尽量和搜索内容高度吻合的答案，这里我们使用solr的分组（group）的功能也能达到需要的效果：

在查询的时候添加这两个参数即可`group=true&group.field=q_id`，第一个是开启`group`功能，第二个是用来分组的字段，我们为了搜索标题的时候不会出现多篇标题一样的所以填的是`q_id`，`q_id`代表的是`question`表的id。

![](http://p1.bqimg.com/567571/8131b88f14a70341.jpg)

输入标题检索：

![](http://p1.bpimg.com/567571/3be2ab6b7fc7be42.jpg)

可以看见不会出现之前的全是同一篇文章的情况了。

**搜索到的内容高亮：**

搜索时加入`hl`以及`hl.fl`即可，`hl.fl`是搜索匹配时高亮的字段，多个字段用英文逗号分隔即可。如果所有内容都想要高亮，直接填`*`就好。
![](http://p1.bpimg.com/567571/adae632369251e47.jpg)
高亮的内容在json数据的底部：
![](http://p1.bqimg.com/567571/2917ecb40f7255e8.jpg)

**去重但是保留**

这个小标题实在不知道该怎么写，大概的要求就是，，上面我们已经实现了去重只保留了一条数据。。。但是我们还可以设置一个显示的最大值，允许重复2条或者3条。只需要再加一个参数即可`group.limit=3`比如这样允许重复了最大是三条。如图：

![](http://p1.bqimg.com/567571/1bb01c678633ccaa.jpg)


#### 分页

mysql开启远程访问：

```
GRANT ALL PRIVILEGES ON *.* TO 'root'@'%' IDENTIFIED BY 'passowrd' WITH GRANT OPTION;

flush privileges;
```