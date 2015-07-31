# Common SQL interface

通用`SQL`接口是用来和Vertx`SQL`服务交互的.

通过指定的SQL服务接口我们可以获取一个指定的连接.

# The SQL Connection
我们使用`SQLConnection`表示与一个数据库的连接.

### Auto-commit

当连接的`auto commit`被设置为`true`. 这意味着每个操作都会在连接自己的事务中被高效地执行.

如果你想要在一个单独的事务中执行多个操作,你应该使用`setAutoCommit`方法将`auto commit`被设置为`false`.

当操作完成之后,我们设置的`handler`会自动地被调用.
```java
connection.setAutoCommit(false, res -> {
  if (res.succeeded()) {
    // OK!
  } else {
    // Failed!
  }
});
```

### Executing queries
我们使用`query`来执行查询操作

`query`方法的参数是原生`SQL`语句, 我们不必使用针对不同的数据库使用不同的`SQL`方言.

当查询完成之后,我们设置的`handler`会自动地被调用. `query`的结果使用`ResultSet`表示.

```java
connection.query("SELECT ID, FNAME, LNAME, SHOE_SIZE from PEOPLE", res -> {
  if (res.succeeded()) {
    // Get the result set
    ResultSet resultSet = res.result();
  } else {
    // Failed!
  }
});
```
`ResultSet`实例中的`getColumnNames`方法可以获得可用列名, `getResults`可以获得查询真实的结果.

实际上,查询的结果是一个`JsonArray`的`List`实例,每一个元素都代表一行结果.

```java
List<String> columnNames = resultSet.getColumnNames();

List<JsonArray> results = resultSet.getResults();

for (JsonArray row: results) {

  String id = row.getString(0);
  String fName = row.getString(1);
  String lName = row.getString(2);
  int shoeSize = row.getInteger(3);

}
```
另外你还可以使用`getRows`获取一个`Json`对象实例的`List`, 这种方式简化了刚才的方式,但是有一点需要注意的是,`SQL`结果可能包含重复的列名, 如果你的情景是这种情况,你应该使用`getResults`.

下面给出了一种使用`getRows`获取结果的示例：
```java
List<JsonObject> rows = resultSet.getRows();

for (JsonObject row: rows) {

  String id = row.getString("ID");
  String fName = row.getString("FNAME");
  String lName = row.getString("LNAME");
  int shoeSize = row.getInteger("SHOE_SIZE");

}
```

### Prepared statement queries
我们可以使用`queryWithParams`来执行`prepared statement`查询.

下例中,演示了使用方法：
```java
String query = "SELECT ID, FNAME, LNAME, SHOE_SIZE from PEOPLE WHERE LNAME=? AND SHOE_SIZE > ?";
JsonArray params = new JsonArray().add("Fox").add(9);

connection.queryWithParams(query, params, res -> {

  if (res.succeeded()) {
    // Get the result set
    ResultSet resultSet = res.result();
  } else {
    // Failed!
  }
});
```

### Executing INSERT, UPDATE or DELETE
我们可以直接使用`update`方法进行数据更新操作.

同样`update`方法的参数同样是原生`SQL`语句,不必使用`SQL`方言.

当更新完成之后,我们会获得一个更新结果`UpdateResult`。

我们可以调用更新结果的`getUpdated`方法获得有多少行发生了改变, 而且如果更新时生成了一些key,那么我们可以通过`getKeys`获得

```java
List<String> columnNames = resultSet.getColumnNames();

List<JsonArray> results = resultSet.getResults();

for (JsonArray row: results) {

  String id = row.getString(0);
  String fName = row.getString(1);
  String lName = row.getString(2);
  int shoeSize = row.getInteger(3);

}
```

### Prepared statement updates
如果想要执行`prepared statement`更新操作,我们可以使用`updateWithParams`.

如下例：
```java
String update = "UPDATE PEOPLE SET SHOE_SIZE = 10 WHERE LNAME=?";
JsonArray params = new JsonArray().add("Fox");

connection.updateWithParams(update, params, res -> {

  if (res.succeeded()) {

    UpdateResult updateResult = res.result();

    System.out.println("No. of rows updated: " + updateResult.getUpdated());

  } else {

    // Failed!

  }
});
```

### Executing other operations
如果想要执行其他数据库操作,例如创建数据库,你可以使用`execute`方法.

同样`execute`执行的语句也是原生`SQL`语句.当操作执行完之后,我们设置的`handler`会被调用.
```java
String sql = "CREATE TABLE PEOPLE (ID int generated by default as identity (start with 1 increment by 1) not null," +
             "FNAME varchar(255), LNAME varchar(255), SHOE_SIZE int);";

connection.execute(sql, execute -> {
  if (execute.succeeded()) {
    System.out.println("Table created !");
  } else {
    // Failed!
  }
});
```

### Using transactions
如果想要使用事务,那么首先要调用`setAutoCommit`将`auto-commit`设置为`false`.

接下来你就可以进行事务操作, 例如提交时使用`commit`, 回滚时使用`rollback`.

一旦`commit/rollback`完成之后, 我们设置的`handler`会被调用, 然后下一个事务会自动开始.
```java
connection.commit(res -> {
  if (res.succeeded()) {
    // Committed OK!
  } else {
    // Failed!
  }
});
```

### Closing connections
当你执行完全部的操作之后,你应该使用`close`将连接资源还给连接池.