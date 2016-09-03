## JDBC

由题目补充：

Java数据库连接库JDBC用到的是**桥接模式**
> JDBC结构图

![jdbc结构图](https://raw.githubusercontent.com/callme001/my-images/master/2016-09-02%2000%3A05%3A03-jdbc-jdbcstruct.jpg)

### 1. 使用JDBC简单步骤

1. 导入数据包 . 需要包括含有需要进行数据库编程的JDBC类的包。大多数情况下，使用 `import java.sql.*`  就可以了.

2. 注册JDBC驱动程序. 需要初始化驱动程序，可以与数据库打开一个通信通道。

3. 打开连接. 需要使用DriverManager.getConnection() 方法创建一个Connection对象，它代表与数据库的物理连接

4. 执行查询 . 需要使用类型声明的对象建立并提交一个SQL语句到数据库。

5. 从结果集中提取数据 . 要求使用适当的关于ResultSet.getXXX()方法来检索结果集的数据。

6. 清理环境. 需要明确地关闭所有的数据库资源相对依靠JVM的垃圾收集。


```java
package com.shundai;

import java.sql.*;

/**
 * Created by jincarry on 16-8-30.
 */
public class FirstExample {
    //数据库连接驱动名称
    static final String JDBC_DRIVER = "com.mysql.jdbc.Driver";
    //数据库连接URL
    static final String DB_URL = "jdbc:mysql://localhost/example";


    static final String USER = "root"; //数据库用户名
    static final String PASS = "123456"; // 密码

    public static void main(String[] args) {
        Connection connection = null;
        PreparedStatement statement = null;

        try {
            //注册JDBC驱动
            Class.forName(JDBC_DRIVER);
            //打开一个数据库连接
            connection = DriverManager.getConnection(DB_URL,USER,PASS);

            String sql = "SELECT * FROM user";
            //创建一个statement
            statement = connection.prepareStatement(sql);

            //获取数据库执行结果集
            ResultSet resultSet = statement.executeQuery();
            while (resultSet.next()){
                System.out.println(resultSet.getInt("id"));
                System.out.println(resultSet.getString("name"));
                System.out.println(resultSet.getString("nick"));
            }

            //关闭连接和释放资源
            resultSet.close();
            statement.close();
            connection.close();

        } catch (ClassNotFoundException e) {
            e.printStackTrace();
        }catch (SQLException e){

        }

    }
}

```

### 2. JDBC中，三种Statement

#### 2.1 Statement

Statement用于执行静态SQL语句，不能设置参数。
```java
Statement statement = null;
try{
	statement = connection.createStatement();
    ResultSet resultSet = statement.executeQuery(sql);
    ...
}catch(SQLException e){
	...
}finally{
	statement.close();
	...
}
```

Statement对象提供三个函数返回结果：

* boolean execute(String SQL) : 返回一个布尔值true，指示ResultSet对象是否可以被检索

* int executeUpdate(String SQL) : 返回受影响的SQL语句执行的行数。使用此方法来执行，而希望得到一些受影响的行的SQL语句 - 例如，INSERT，UPDATE或DELETE语句。

* ResultSet executeQuery(String SQL) : 返回ResultSet对象。当希望得到一个结果集使用此方法，就像使用一个SELECT语句。

使用完毕后调用close()方法关闭资源

#### 2.2 PreparedStatement

使用PreparedStatement对象进行查询，数据库系统会对sql语句进行预编译处理（如果JDBC驱动支持的话），预处理语句将被预先编译好，这条预编译的sql查询语句能在将来的查询中重用。如果是多次运行SQL，PreparedStatement比Statement对象生成的查询速度更快，也是推荐使用的方式。

```java
Statement statement = null;
try{
	String sql = "SELECT * FROM user WHERE id = ?";
	statement = connection.prepareStatement(sql);
    statement.setInt(1,2); //设置参数绑定
    ResultSet resultSet = statement.executeQuery();
    ...
}catch(SQLException e){
	...
}finally{
	statement.close();
	...
}
```
PreparedStatement提供了setXXX()方法来设置参数绑定，如：
```java
statement.setString(index,value);
statement.setFloat(index,value);
statement.setInt(index,value);
statement.setDate(index,value);
...
```
使用PreparedStatement参数化查询能够避免SQL注入，在实际的程序中，应避免拼接字符串的方式进行SQL查询，而是应该使用参数化查询来规避SQL注入的风险。

PreparedStatement提供了和Statement一样的结果返回函数：

* boolean execute(String SQL) : 返回一个布尔值true，指示ResultSet对象是否可以被检索

* int executeUpdate(String SQL) : 返回受影响的SQL语句执行的行数。使用此方法来执行，而希望得到一些受影响的行的SQL语句 - 例如，INSERT，UPDATE或DELETE语句。

* ResultSet executeQuery(String SQL) : 返回ResultSet对象。当希望得到一个结果集使用此方法，就像使用一个SELECT语句。


#### 2.3 CallableStatement

CallableStatement接口扩展 PreparedStatement，用来调用**存储过程**,它提供了对输出和输入/输出参数的支持。

mysql中创建如下的存储过程
```sql
CREATE PROCEDURE num_from_user(IN username VARCHAR(25),OUT anum INT)
	BEGIN
		SELECT count(*)  INTO anum
		FROM `user`
		WHERE `user`.`name`=username;
END
```
传入一个username返回所有name为username的总数。

```java
CallableStatement statement = null;
try{
	String sql = "call num_from_user(?,?)";
	statement = connection.prepareCall(sql);
    statement.setString(1,"username");
    statement.registerOutParameter(2,Types.INTEGER);
    statement.executeQuery();
    int num = statement.getInt(2); //获取存储过程的输出
    ...
}catch(SQLException e){
	...
}finally{
	statement.close();
	...
}
```

### 3. 结果集ResultSet

SQL 语句从数据库查询中获取数据,并将数据返回到结果集中。SELECT 语句是一种标准的方法,它从一个数据库中选择行记录,并显示在一个结果集中。 java.sql.ResultSet 接口表示一个数据库查询的结果集。

ResultSet 接口的方法可细分为三类-
* 导航方法: 导航方法:用于移动光标。比如next()方法
* 获取方法:获取方法:用于查看当前行被光标所指向的列中的数据。比如getXXX()方法
* 更新方法:更新方法:用于更新当前行的列中的数据。这些更新也会更新数据库中的数据。实际很少用，是把ResultSet的数据更新到数据库比如updateString()方法

JDBC 提供了连接方法通过下列创建语句来生成你所需的 ResultSet 对象:
* createStatement(int RSType, int RSConcurrency);
* prepareStatement(String SQL, int RSType, int RSConcurrency);
* prepareCall(String sql, int RSType, int RSConcurrency);

> **RSType**可能的类型如下：

* `ResultSet.TYPE_FORWARD_ONLY` 光标只能在结果集中向前移动。
* `ResultSet.TYPE_SCROLL_INSENSITIVE` 光标可以向前和向后移动。当结果集创建后,其他人对数据库的操作不会影响结果集的数据
* `ResultSet.TYPE_SCROLL_SENSITIVE` 光标可以向前和向后移动。当结果集创建后,其他人对数据库的操作会影响结果集的数据

> **RSConcurrency**可能的值如下：

* `ResultSet.CONCUR_READ_ONLY` 创建一个只读结果集,这是默认的值。
* `ResultSet.CONCUR_UPDATABLE` 创建一个可修改的结果集



### 4. 事务

JDBC默认情况下，是对单条SQL语句自动提交的。为了业务流程的完整性，我们可能需要关闭自动提交，自己管理事务。分以下两个步骤：

* 关闭自动提交
* 提交和回滚

关闭自动提交
```java
connection.setAutoCommit(false);
```
提交
```java
connection.commit();
```
回滚
```java
connection.rollback();
```
完整代码可能如下(这段代码演示了第二条SQL语句不能执行之后，回滚到第一条之前)：
```java
package com.shundai;

import java.sql.*;

/**
 * Created by jincarry on 16-8-30.
 */
public class FirstExample {
    static final String JDBC_DRIVER = "com.mysql.jdbc.Driver";
    static final String DB_URL = "jdbc:mysql://localhost/example";
    static final String USER = "root";
    static final String PASS = "123456";

    public static void main(String[] args) {
        Connection connection = null;
        PreparedStatement statement = null;
        PreparedStatement statement1 = null;

        try {
            Class.forName(JDBC_DRIVER);
            connection = DriverManager.getConnection(DB_URL,USER,PASS);

            connection.setAutoCommit(false); //关闭自动提交

            statement = connection.prepareStatement("INSERT INTO `example`.`user` ( `name`, `nick`) VALUES ( 'jincarry6', 'jincarry')"); //能够被正确执行的SQL
            statement1 = connection.prepareStatement("INSERT INTO `example`.`user` ( `name`, `nick`) VALUES ( 'jincarry7')"); //不能被正确执行的SQL

            statement.executeUpdate();
            statement1.executeUpdate();

            connection.commit(); //提交事务

            statement.close();
            connection.close();

        } catch (ClassNotFoundException e) {
            e.printStackTrace();
        }catch (SQLException e){
            try {
                connection.rollback(); //回滚
            } catch (SQLException e1) {
                e1.printStackTrace();
            }
        }

    }
}

```
#### 4.1 使用还原点

新的JDBC3.0保存点的接口提供了额外的事务控制。提供事务回滚的还原点，给rollback()函数传入保存点的名称，数据库执行回滚的时候回滚到指定的保还原点。

* 关闭自动提交
* 设置还原点
* 提交和回滚

关闭自动提交
```java
connection.setAutoCommit(false);
```
设置还原点
```java
Savepoint savepoint1 = conn.setSavepoint("Savepoint1");
```
提交
```java
connection.commit();
```
回滚
```java
connection.rollback(Savepoint);
```

### 5. 使用批处理

批处理是指数据库操作中，批量的插入，删除或者更新数据时当成一个调用提交给数据库。当一次发送多条SQL语句到数据库时，可以减少通信的消耗，从而提高了性能。

JDBC驱动程序不一定支持该功能，可以使用DatabaseMetaData.supportsBatchUpdates()来查看是否支持。返回true则表明支持。



#### 5.1 批处理和 Statement 对象

使用 Statement 对象来使用批处理所需要的典型步骤如下所示-
* 使用 createStatement() 方法创建一个 Statement 对象。
* 使用 setAutoCommit() 方法将自动提交设为 false。
* 被创建的 Statement 对象可以使用 addBatch() 方法来添加你想要的所有SQL语句。
* 被创建的 Statement 对象可以用 executeBatch() 将所有的 SQL 语句执行。
* 最后,使用 commit() 方法提交所有的更改。

示例
下面的代码段提供了一个使用 Statement 对象批量更新的例子-
```java
// Create statement object
Statement stmt = conn.createStatement();
// Set auto-commit to false第 1 章 JDBC 指南 | 51
conn.setAutoCommit(false);
// Create SQL statement
String SQL = "INSERT INTO Employees (id, first, last, age) " +
"VALUES(200,'Zia', 'Ali', 30)";
// Add above SQL statement in the batch.
stmt.addBatch(SQL);
// Create one more SQL statement
String SQL = "INSERT INTO Employees (id, first, last, age) " +
"VALUES(201,'Raj', 'Kumar', 35)";
// Add above SQL statement in the batch.
stmt.addBatch(SQL);
// Create one more SQL statement
String SQL = "UPDATE Employees SET age = 35 " +
"WHERE id = 100";
// Add above SQL statement in the batch.
stmt.addBatch(SQL);
// Create an int[] to hold returned values
int[] count = stmt.executeBatch();
//Explicitly commit statements to apply changes
conn.commit();
```

#### 5.2 批处理和 PrepareStatement 对象

使用 prepareStatement 对象来使用批处理需要的典型步骤如下所示-
* 使用占位符创建 SQL 语句。
* 使用任一 prepareStatement() 方法创建 prepareStatement 对象。
* 使用 setAutoCommit() 方法将自动提交设为 false。
* 被创建的 Statement 对象可以使用 addBatch() 方法来添加你想要的所有 SQL 语句。
* 被创建的 Statement 对象可以用 executeBatch() 将所有的 SQL 语句执行。
* 最后,使用 commit() 方法提交所有的更改。

下面的代码段提供了一个使用 PrepareStatement 对象批量更新的示例-
```java
// Create SQL statement
String SQL = "INSERT INTO Employees (id, first, last, age) VALUES (?, ?, ?, ?)";
// Create PrepareStatement object
PreparedStatemen pstmt = conn.prepareStatement(SQL);
//Set auto-commit to false
conn.setAutoCommit(false);
// Set the variables
pstmt.setInt( 1, 400 );
pstmt.setString( 2, "Pappu" );
pstmt.setString( 3, "Singh" );
pstmt.setInt( 4, 33 );
// Add it to the batch
pstmt.addBatch();
// Set the variables![](http://)
pstmt.setInt( 1, 401 );
pstmt.setString( 2, "Pawan" );
pstmt.setString( 3, "Singh" );
pstmt.setInt( 4, 31 );
// Add it to the batch
pstmt.addBatch();
//add more batches

//Create an int[] to hold returned values
int[] count = stmt.executeBatch();
//Explicitly commit statements to apply changes
conn.commit();
```


