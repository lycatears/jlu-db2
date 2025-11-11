> ## **注意：本文的主题不是《数据库系统原理》。**
## 2022级考试内容（2024.12.29）
### 一、给出代码片段（5\*15=75）

1.向一个表插入10万行，每500行提交一次

2.查询员工信息表，然后将奖金为0的员工绩效考核设置为不合格 `提示：for update 与 where current of`

3.键盘读入员工信息并插入，包含两个空列 `使用setNull()`

4.异常处理，给出两个`SQLState`代码，捕获异常并输出错误信息

5.数据库插入`BLOB`对象的照片

### 二、设计题（25）

1.12306如何实现查火车票不用登陆，买票需要登录`考点：事务隔离性级别`

2.实现一个12306火车信息查询，然后根据发时、票价等排序，根据席别等因素筛选

*最终得分98分，课程的内容还是比较简单的，只要实验是自己做的，90+没问题。*

> 试植梓为期 掔发为纾
>
> ——《韶华未既》
---

## 知识点总结

##### 1.导入所需的包

```java
import java.sql.*;
import java.io.*;
import java.lang.*;
```

---

##### 2.加载JDBC驱动

```java
// MyJDBC.java
public class MyJDBC {
    static { //内部静态类
        try {
            Class.forName("COM.ibm.db2.jdbc.app.DB2Driver");
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
}
```

---

##### 3.实例化`Connection`对象

- 无用户名密码（本地账户登录）

```java
try {
    String url = "jdbc:db2:sample";
    Connection conn = DriverManager.getConnection(url);
} catch (Exception e) {
    throw new Exception("\nUsage: java MyJDBC [,uanme,passwd]");
}
```

- 有用户名密码（远程登陆）

```java
try {
    String url = "jdbc:db2:sample";
    String userid = "JLU";
    String passwd = "826132";
    Connection conn = DriverManager.getConnection(url, userid, passwd);
} catch (Exception e) {
    throw new Exception("\nUsage: java MyJDBC [,uanme,passwd]");
}
```

---

##### 4.查询操作：`Statement`,`ResultSet`（注意同样要用`try-catch`包围，这里为了方便演示就省略了）

```java
String sql = "SELECT EMPNO, LASTNAME FROM TEMPL WHERE SALARY > 40000";
Statement stmt = conn.createStatement();
ResultSet rs = stmt.executeQuery(sql);
while (rs.next) {
    System.out.println("empno = " + rs.getString(1) + "lastname = " + rs.getString(2));
}
rs.close();
stmt.close();
conn.close();
```

---

##### 5.更新操作：`Statement` VS `PreparedStatement`（注意同样要用`try-catch`包围，这里为了方便演示就省略了）

- `Statement`

```java
Statement stmt = conn.createStatement();
String sql = "UPDATE TEMPL SET LASTNAME = 'Stohl' WHERE EMPNO = '000110'";
int numRows = stmt.exectueUpdate();
System.out.println("Number of rows updated" + numRows);
```

- `PreparedStatement`

```java
String sql = "UPDATE TEMPL SET SALARY = ? WHERE EMPNO = ?";
PreparedStatement pstmt = conn.prepareStatement(sql);
pstmt.setString(1, argv[1]);
pstmt.setString(2, argv[2]);
int numRows = pstmt.exectueUpdate();
System.out.println("Number of rows updated" + numRows);
```

---

##### 6.关闭自动提交

```java
sample.setAutoCommit(false);
```

---

##### 7.最基本的GUI交互（GUI不作为考试重点）

- 输入对话框

```java
String empno = JOptionPane.showInputDialog(null, "请输入员工工号");
```

- 消息对话框

```java
JOptionPane.showMessageDialog(null, "插入成功！");
```

---

##### 8.主变量与列变量

简单理解：主变量就是你编程用的语言定义的变量，列变量就是数据库里面的变量。

DB2工具`DCLGEN`：帮助找到列变量和主变量的对应关系。

```batch
db2dclgn -D sample -T employee -L Java
```

意为找到`sample`数据库中`employee`表的各列在Java中的对应主变量类型。

---

##### 9.空值处理

如果使用下列代码判断是否为空值：

```java
String empno = rs.getString(1);
if (empno == null) {
    //...
}
```

当对应列变量为空值时，JDBC会根据变量类型为主变量赋予默认值，对于字符串类型，该值为空格。因此，空值处理的语句始终不会被执行。

JDBC提供`wasNull()`方法：

```java
String empno = rs.getString(1);
if (rs.wasNull() {
    //...
}
```

当读取的列变量为空值，该方法返回`true`。

同样，如果我们需要给列变量赋空值，直接使用`null`将会给字符串赋值为`'null'`。JDBC提供`setNull()`方法：

```java
pstmt.setNull(1, java.sql.Type.CHAR);
pstmt.setNull(2, java.sql.Type.SMALLINT);
pstmt.setNull(3, java.sql.Type.INTEGER);
```

通过该方法即可设置空值。

---

##### 10.异常处理（不必背诵错误代码，考试会给）

SQLCA：SQL通信区。高级语言调用SQL语句后，数据库引擎访问数据库后，用特殊代码向SQLCA返回访问状况。

数据库访问发生异常时，抛出`SQLException`异常。

`SQLCode`：错误代码集。在DB2中，<0代表SQL语句有错误，>0且!=100为警告信息，=0为返回0行（行不存在）。`SQLCode`依赖于具体数据库产品，由厂商自行定义，包含更加详细的错误信息。

`SQLState`：错误代码集。国际标准，所有数据库产品中多数代码是统一的。

例如：

```java
try {
    String sql = "DELETE FROM TEMPL WHERE EMPNO = 999";
    pstmt = con.prepareStatement(sql);
    deleteCount = pstmt.executeUpdate();
    if ((SQLWarn = pstmt.getWarnings()) != null) {
        System.out.println("\nWarning received on a DELETE \n");
    }
} catch (SQLException e) {
    if (e.getSQLState().equals("42818")) {
        // 处理类型不兼容的异常
        System.out.println("\n Operand data tyoes not Compatible \n");
    } else {
        System.out.println("\n Error deleting from TEMPL \n");
        System.out.println("\n Value of SQLCode is: \n");
        String SQLCode = e.getErrorCode(); //注意此处方法名是ErrorCode，而不是SQLCode
        System.out.println(SQLCode);
    }
}
```

常见错误代码示例（转载自[吉林大学数据库应用程序开发知识点总结-CSDN博客](https://blog.csdn.net/It_Ray/article/details/135437808)）：

```java
switch(SQLCODE) {
            case 802:
                //数据溢出
                JOptionPane.showMessageDialog( null , "数据溢出" ,"SQL错误" , JOptionPane.ERROR_MESSAGE) ;
                break;
            case -007:
                //SQL语句中有非法字符
                JOptionPane.showMessageDialog( null , "SQL语句中有非法字符" ,"SQL信息" , JOptionPane.ERROR_MESSAGE) ;
                break;
            case -010:
                //丢失引号
                JOptionPane.showMessageDialog( null , "SQL语句丢失引号" ,"SQL信息" , JOptionPane.ERROR_MESSAGE) ;
                break;
            case -060:
                //某特定数据类型的长度或者标量规范无效
                JOptionPane.showMessageDialog( null , "某特定数据类型的长度或者标量规范无效" ,"SQL信息" , JOptionPane.ERROR_MESSAGE) ;
                break;
            case -102:
                //字符串常量太长
                JOptionPane.showMessageDialog( null , "字符串常量太长" ,"SQL信息" , JOptionPane.ERROR_MESSAGE) ;
                break;
            case -104:
                //SQL语句中遇到非法符号
                JOptionPane.showMessageDialog( null , "SQL语句中遇到非法符号" ,"SQL信息" , JOptionPane.ERROR_MESSAGE) ;
                break;
            case -433:
                //指定的值太长
                JOptionPane.showMessageDialog( null , "指定的值太长" ,"SQL信息" , JOptionPane.ERROR_MESSAGE) ;
                break;
            case -405:
                //数值文字超出了范围
                JOptionPane.showMessageDialog( null , "数值文字超出了范围" ,"SQL信息" , JOptionPane.ERROR_MESSAGE) ;
                break;
}


```

---

##### 11.结果集游标

```java
PreparedStatement pstmt = conn.prepareStatement(sql, rs.TYPE_FORWARD_ONLY, rs.CONCUR_READ_ONKY);
```

第二个参数含义是只允许前向滚动，也就是说游标只能往前走，不能倒退，这是默认值。如果希望游标是可滚动的，可以改为`rs.TYPE_SCROLL_INSENSITIVE`（允许倒退，数据库变化时结果集不变）或者`rs.TYPE_SCROLL_SENSITIVE`（允许倒退，数据库变化时结果集同步）。

第三个参数限制了结果集的用途，意为只允许并发读取（只在读取的时候可以并发）。改为`CONCUR_UPDATABLE`则允许并发修改，但是本课程中DB2的驱动程序不支持。

对于`Statement`也同样适用，只需要把默认的无参方法改成重载的两个参数的方法即可。

```java
Statement pstmt = conn.createStatement(rs.TYPE_FORWARD_ONLY, rs.CONCUR_READ_ONKY);
```

物化：结果集暂存于磁盘。例如，需要对结果集排序时，内存中不足以保存排序结果，需要为结果集临时分配磁盘空间。

```java
String mySelect = "SELECT LASTNAME, FIRSTNME FROM EMP FOR UPDATE";
String myUpdate = "UPDATE EMP SET FIRSTNAME = ? WHERE CURRENT OF ";
```

这二者都不是标准的SQL语句。`FOR UPDATE`指示数据库引擎该语句将被用于修改，第二个字符串含有问号，含有参数标记，`WHERE CURRENT OF`后面等待游标名称，组合起来就是一个完整的SQL语句了。

```java
String cursorName = null;
Statement stmt = conn.createStatement();
ResultSet rs = stmt.executeQuery(mySelect);
cursorName = rs.getCursorName();
PreparedStatement ps = conn.prepareStatement(myUpdate + cursorName);
```

注意此处执行的是`select`语句，也就是查询。因此，`cursorName`返回的，也就是查询后得到的结果集的游标名。更新语句与这个游标相拼接，条件就是游标指向当前行的值。有没有发现这两条语句都是针对同一个表的？也就是说，查询结果集的游标定位到某行的时候，更新语句就直接可以用这个游标的指向来执行更新操作了。

```java
while (rs.next) {
    String lastname = rs.getString(1);
    String firstnme = rs.getString(2);

    if (lastname.equals("SMITH")) {
        String newFirstnme = "George";
        ps.setString(1, newFirstnme);
        ps.executeUpdate();
    }
}
rs.close();
ps.close();
stmt.close();
conn.close();
```

这里随着`select`语句结果集游标的移动，逐个检查数据集中的姓氏是否为SMITH。当检查到姓SMITH的人时，`update`语句就可以直接通过游标定位要修改的行了。先查询——获取结果集——获取的结果集上进行修改。

```java
rs.afterLast(); // 将结果集游标移动到最后一行之后
rs.prevoius(); // 游标前移一行
rs.absolute(1); // 移动到第一行
rs.absolute(-1); // 移动到最后一行
rs.first(); // 移动到第一行
rs.last(); // 移动到最后一行
rs.relative(2); // 前移两行
rs.relative(-2); //回退2行
```

---

##### 12.元数据接口

`getColumnCount()`结果集列数

`getColumnLabel()`结果集列的名称

```java
DatabaseMetaData dbmd = sample.getMetaData();
ResultSet rs = dbmd.getSchemas();
```

获取数据库的模式元数据

```java
String[] tableTypes = {"TABLE", "VIEW"};
ResultSet rs = dbmd.getTables(null, "JLU", "%", tableTypes);
```

获取模式为JLU的表信息

---

##### 13.批处理

当执行了一行所有列的`setString()`后：

```java
ps.addBatch();
```

向批处理添加一行。

```java
int[] rowCounts = ps.executeBatch();
```

执行批处理，返回每个语句影响的行数。

---

##### 14.大对象

`BLOB`：二进制大对象

`CLOB`：字符型大对象

SQL函数`POSSTR(RESUME, 'Personal')`：获取`'Personal'`字符串在`RESUME`列中第一次出现的位置。

SQL函数`SUBSTR(RESUME, 1, 999)`：截取`RESUME`列中从1开始，读取999个字符。注意SQL的下标是1开始的。

SQL函数`SUBSTR(RESUME, 114514)`：截取RESUME列中从下标为114514的字符开始，直到末尾的子串。

SQL `select`语句中，`||` 不是逻辑运算符。它代表将前后两个字符串拼接起来。

读取`CLOB`时，要注意：`select`字句中截取后，还是`CLOB`；Java程序中要用`Clob resumelob = rs.getClob()`读取。

如何将`Clob`转化为我们常见的字符串呢？

```java
while (rs.next()) {
    empno = rs.getString(1);
    resumefmt = rs.getString(2);
    resumelob = rs.getClob(3);
    long len = resumelob.length();
    int len1 = (int)len;
    String resumeout = resumelob.getSubString(1, len1);
}
```

首先用`resumelob.length()`获取大对象的整体长度，强制转换为32位整型后，读取从第1位到结尾的字串，其实就是整个字符串，这样我们就得到了所需要的简历字符串。注意从结果集中读取`CLOB`时，不能使用`String`类型，也不能使用`getString`方法，要使用对应的`Clob`版本。

读取和插入`BLOB`，则需要使用Java流。

- 写入

```java
File file = new File("C:/a.jpg");
java.io.BufferedInputStream imageInput = new java.io.BufferedInputStream(new java.io.FileInputStream(file));
pstmt.setBinaryStream(1, imageInput, (int)file.length());
pstmt.executeUpdate();
```

首先获取文件对象，指定要写入的文件；然后使用该文件对象定义输入流；最后，将输入流写入数据库。

- 读取

读取同理，获得BLOB对象后，转换为流，定义文件对象，最后写入文件。

```java
Blob blob = (Blob)rs.getBlob(1);
java.io.InputStream inputStream = blob.getBinaryStream();
File file = new File("C:/backa.jpg");
FileOutputStream fos = new FIleOutputStream(file);

int c;
while((c = inputStream.read()) != -1) {
    fos.write(c);
}
```

### 整体评价

跟往年的题不完全一样，还是要以PPT为主，学习通视频好好看看。

