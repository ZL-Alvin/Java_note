# jdbc基本使用步骤

1. 注册驱动
2. 获取连接
3. 创建发送sql语句对象
4. 发送sql语句，并获取返回结果
5. 结果集解析
6. 资源关闭

# 基于statement演示

使用statement存在的问题如下

		1. SQL语句需要字符串拼接,比较麻烦
		1. 只能拼接字符串类型,其他的数据库类型无法处理
		1. 可能发生注入攻击，动态值充当了SQL语句结构,影响了原有的查询结果!

```java
import com.mysql.cj.jdbc.Driver;

import java.sql.*;
import java.util.Scanner;

public class StatementUserLoginPart {
    /**
     * 输入账号密码
     * 进行数据库信息查询(t_user)
     * 反馈登录成功还是失败
     */
    public static void main(String[] args) throws SQLException, ClassNotFoundException {
        //1、获取用户输入信息
        Scanner scanner = new Scanner(System.in);
        System.out.println("请输入账号：");
        String account = scanner.nextLine();
        System.out.println("请输入密码：");
        String password = scanner.nextLine();

        //2、注册驱动
        Class.forName("com.mysql.cj.jdbc.Driver");   //反射的方式

        //3、获取数据库连接
        /**
         * 核心属性:
         *          1.数据库软件所在主机的ip地址: localhost   | 127.0.0.1
         *          2.数据库软件所在主机的端口号：  3306
         *          3。连接的数据库： atguigu
         *          4.连接的账号： root
         *          5.连接的密码： 123456
         *          6.可选信息
         *
         * 三个参数
         *  String url          数据库软件所在的信息，连接的具体库，以及其他可选信息
         *                      语法：jdbc:数据库管理软件名称[mysql, oracle]://ip地址|主机名:port端口号/数据库名?key=value &key=val 可选信息
         *                      具体：jdbc:mysql://127.0.0.1:3306/atguigu
         *                           jdbc:mysql://localhost:3306/atguigu
         *                      本机的省略写法:如果数据库软件安装在本机，并且端口号是3306
         *                           jdbc:mysql:///atguigu
         *  String user         账号
         *  String password     密码
         *
         *
         * 二个参数:
         * String url
         * 此urL和三个参数的UrL的作用一样！数据库ip，端口号,具体的数据库和可选信息
         * Properties info：存储账号和密码
         * Properties 类似于Mqp只不过key = value都是字符串形式的
         * key user：账号信息
         * key possword：密码信息
         *
         *
         * 一个参数:
         * String url
         * 数据库ip,端口号,具体的数据库可选信息(账号密码)
         * jdbc:数据库软件名://ip:port/数据库?key=value&key=value&key=value
         * jdbc:mysql://localhost:3306/atguigu?User=root&password=123456
         * 携带固定的参数名user password 传递账号和密码信息！[乌龟的屁股,规定!]
         *
         *
         */
        Connection connection = DriverManager.getConnection("jdbc:mysql:///atguigu", "root", "123456");

        //4.创建发送sql语句的statement对象
        Statement statement = connection.createStatement();

        //5.发生sql语句，并获取返回结果集
        String sql = "SELECT * FROM t_user WHERE account = '"+ account +"' AND PASSWORD = '"+ password +"';";
        /**
         *  ResultSet 结果集对象 = executeQuery(DQL语句)
         *  int       响应行数  = executeUpdate(非DQL语句)
         */
        ResultSet resultSet = statement.executeQuery(sql);
        //6.查询结果集解析resultset
        /**
         *
         * TODO:1.需要理解ResultSet的数据结构和小海豚查询出来的是一样，需要在脑子里构建结果表！
         * TODO:2.有一个光标指向的操作数据行，默认指向第一行的上边！我们需要移动光标，指向行，在获取列即可！
         *        boolean = next()
         *              false: 没有数据，也不移动了！
         *              true:  有更多行，并且移动到下一行！
         *       推荐：推荐使用if 或者 while循环，嵌套next方法，循环和判断体内获取数据！
         *       if(next()){获取列的数据！} ||  while(next()){获取列的数据！}
         *
         *TODO：3.获取当前行列的数据！
         *         get类型(int columnIndex | String columnLabel)
         *        列名获取  //lable 如果没有别名，等于列名， 有别名label就是别名，他就是查询结果的标识！
         *        列的角标  //从左到右 从1开始！ 数据库全是从1开始！
         */
//         while (resultSet.next()) {
//             int id = resultSet.getInt(1);
//             String account1 = resultSet.getString("account");
//             String password1 = resultSet.getString(3);
//             String nickname = resultSet.getString("nickname");
//             System.out.println(id + "--" + account1 + "--" + password1 + "nickname = " + nickname);
//         }
         if (resultSet.next()) {
             System.out.println("登录成功!");
         }
         else {
             System.out.println("登录失败!");
         }

         //6.关闭资源
         resultSet.close();
         statement.close();
         connection.close();
    }
}
```



# 基于preparedStatement方式优化

​	利用preparedStatement解决上述案例注入攻击和SQL语句拼接问题！

```java
import java.sql.*;
import java.util.Scanner;

public class PSUserLoginPart {
    public static void main(String[] args) throws Exception {
        Scanner scanner = new Scanner(System.in);
        System.out.println("请输入账号：");
        String account = scanner.nextLine();
        System.out.println("请输入密码：");
        String password = scanner.nextLine();

        Class.forName("com.mysql.cj.jdbc.Driver");
        Connection connection = DriverManager.getConnection("jdbc:mysql:///atguigu?user=root&password=123456");

        //创建preparedStatement
        //connection.createStatement();
        //TODO 需要传入SQL语句结构
        //TODO 要的是SQL语句结构，动态值的部分使用 ? ,  占位符！
        //TODO ?  不能加 '?'  ? 只能替代值，不能替代关键字和容器名

        //编写sql语句
        String sql = "select * from t_user where account = ? and password = ?";

        //创建预编译statement并且设置sql语句结果
        PreparedStatement preparedStatement = connection.prepareStatement(sql);

        //单独的占位符进行赋值
        //占位符赋值
        //给占位符赋值！ 从左到右，从1开始！
        /*
         *  int 占位符的下角标
         *  object 占位符的值
         */
        preparedStatement.setObject(1,account);
        preparedStatement.setObject(2,password);

        //发送Sql语句并获取返回结果
        ResultSet resultSet = preparedStatement.executeQuery();

        //结果集解析
        if (resultSet.next()) {
            System.out.println("登陆成功！");
        }
        else {
            System.out.println("登陆失败！");
        }

        resultSet.close();
        preparedStatement.close();
        connection.close();
    }
}
```

# 基于preparedStatement演示 curd

```java
import org.testng.annotations.Test;

import java.sql.*;
import java.util.*;

public class PSCURDPart {
    @Test
    public void testInsert() throws ClassNotFoundException, SQLException {
        /*
        向t_user插入一条数据
        account test
        password test
        nickname 二狗子
         */
        //1.注册驱动
        Class.forName("com.mysql.cj.jdbc.Driver");
        //2.获取连接
        Connection connection = DriverManager.getConnection("jdbc:mysql://127.0.0.1/atguigu", "root", "123456");
        //3.编写SQL语句结果,动态值的部分使用?代替
        String sql = "insert into t_user(account, password, nickname) values(?, ?, ?);";
        //4,创建preparedStatement.并且传入SQL语句结果
        PreparedStatement preparedStatement = connection.prepareStatement(sql);
        //5.占位符赋值
        preparedStatement.setObject(1, "test");
        preparedStatement.setObject(2, "test");
        preparedStatement.setObject(3, "二狗子");
        //6.发送SQL语句
        int i = preparedStatement.executeUpdate();
        //7.输出结果
        if (i > 0) {
            System.out.println("数据插入成功！");
        }
        else {
            System.out.println("数据插入失败！");
        }
        //8.关闭资源
        preparedStatement.close();
        connection.close();

    }

    @Test
    public void testUpdate() throws ClassNotFoundException, SQLException {
        /*
            修改id = 3的用户nickname = 三狗子
         */
        //1.注册驱动
        Class.forName("com.mysql.cj.jdbc.Driver");
        //2.获取连接
        Connection connection = DriverManager.getConnection("jdbc:mysql:///atguigu", "root", "123456");
        //3.编写SQL语句结果,动态值的部分使用?代替
        String sql = "update t_user set nickname = ? where id = ?;";
        //4,创建preparedStatement.并且传入SQL语句结果
        PreparedStatement preparedStatement = connection.prepareStatement(sql);
        //5.占位符赋值
        preparedStatement.setObject(1, "三狗子");
        preparedStatement.setObject(2, 3);
        //6.发送SQL语句
        int i = preparedStatement.executeUpdate();
        //7.输出结果
        if (i > 0) {
            System.out.println("修改成功！");
        }
        else {
            System.out.println("修改失败！");
        }
        //8.关闭资源
        preparedStatement.close();
        connection.close();
    }
    @Test
    public void testDelete() throws ClassNotFoundException, SQLException {
        /*
            删除id = 3的用户数据
         */
        Class.forName("com.mysql.cj.jdbc.Driver");
        Connection connection = DriverManager.getConnection("jdbc:mysql://127.0.0.1:3306/atguigu", "root", "123456");
        String sql = "delete from t_user where id = 3;";
        PreparedStatement preparedStatement = connection.prepareStatement(sql);
        int i = preparedStatement.executeUpdate();
        if (i > 0) {
            System.out.println("删除成功！");
        }
        else {
            System.out.println("删除失败！");
        }
        preparedStatement.close();
        connection.close();
    }
    @Test
    public void testSelect() throws ClassNotFoundException, SQLException {
        /*
            将查询到的结果集保存在list中
         */
        Class.forName("com.mysql.cj.jdbc.Driver");
        Connection connection = DriverManager.getConnection("jdbc:mysql:///atguigu", "root", "123456");
        String sql = "select id, account, password, nickname from t_user;";
        PreparedStatement preparedStatement = connection.prepareStatement(sql)   ;
        ResultSet resultSet = preparedStatement.executeQuery();

        //metaData装的是当前结果集列的信息对象！！！！！(它可以获取列的名称根据下角标，也可以获取列的数量)
        ResultSetMetaData metaData = resultSet.getMetaData();

        int columnCount = metaData.getColumnCount();

        List<Map> list = new ArrayList<>();
        while (resultSet.next()) {
//            Map map = new HashMap();
//            map.put("id", resultSet.getInt("id"));
//            map.put("account", resultSet.getString("account"));
//            map.put("password", resultSet.getString("password"));
//            map.put("nickname", resultSet.getString("nickname"));
//            list.add(map);
        // 注意！！！要从1开始，并且小于等于总列数！！！因为数据库表的下标是从1开始的!!!!
            Map map = new HashMap();
            for (int i = 1; i <= columnCount; i++){
            //获取对应列的值
                Object value = resultSet.getObject(i);
            //获取列下角标对应的列名称！！！getColumnLabel：会获取别名,如果没有写别名才是列的名称。  不要使用 getColumnName：只会获取列的名称！！！！！
                String columnLabel = metaData.getColumnLabel(i);
                map.put(columnLabel, value);
            }
            list.add(map);
        }
        System.out.println("list = " + list);

        resultSet.close();
        preparedStatement.close();
        connection.close();
    }
}
```

# 连接池（druid使用）

## 	1. 硬编码方式（不推荐）		

```java
public class DruidUsePart {
    /*直接使用代码设置连接池连接多数方式
        1，创建一个druid连接池对象
        2.设置连接池参数[必须|非必须]
        3.获取连接[通用方法，所有连接池都一样]
        4.回收连接[通用方法,所有连接池都一样]
*/
    //硬编码的方式（不推荐）
    public void testHard() throws SQLException {
        //连接池对象
        DruidDataSource dataSource = new DruidDataSource();
        //设置参数
        //必须连接数据库驱动类的全限定符[注册驱动] |url | User | password
        dataSource.setUrl("jdbc:mysql://localhost:3306/atguigu");
        dataSource.setUsername("root");
        dataSource.setPassword("123456");
        dataSource.setDriverClassName("com.mysql.cj.jdbc.Driver");  //进行驱动注册和获取连接
        //非必须 初始化连接数量， 最大连接数量
        dataSource.setInitialSize(5); //初始化连接数量
        dataSource.setMaxActive(10); //最大数量

        //获取链接
        Connection connection = dataSource.getConnection();

        //数据库curd

        //回收连接
        connection.close(); //连接池提供的连接，close，就是回收连接！
    }
}
```

## 2. 软编码方式

```properties
# 存放在src/druid.properties
# druid连接池需要的配置参数,key固定命名
driverClassName=com.mysql.cj.jdbc.Driver
username=root
password=root
url=jdbc:mysql:///atguigu
```

```java
public class DruidUsePart {
    /*直接使用代码设置连接池连接多数方式
        1，创建一个druid连接池对象
        2.设置连接池参数[必须|非必须]
        3.获取连接[通用方法，所有连接池都一样]
        4.回收连接[通用方法,所有连接池都一样]
	*/
    /*
    * 软编码
    * 通过读取外部配置文件的方法，实例化druid连接对象
    * */
    @Test
    public void testSoft() throws Exception {
        //1.读取外部配置文件 Properties
        Properties properties = new Properties();

        //src下的文件可以使用类加载器提供的方法
        InputStream ips = DruidUsePart.class.getClassLoader().getResourceAsStream(("druid.properties"));
        properties.load(ips);

        //2.使用连接池的工具类的工程模式，创建连接池
        DataSource dataSource = DruidDataSourceFactory. createDataSource(properties);

        Connection connection = dataSource.getConnection();

        //数据库的curd

        //回收连接
        connection.close();
    }
}
```

# jdbc工具类封装（使用druid）

## 	1. 封装德鲁伊数据连接池

```java
/*
*   利用线程本地变量,存储连接信息！确保一个线程的多个方法可以获取同一个connection
    优势：事务操作的时候service和dao属于同一个线程,不用再传递参数了!
    大家都可以调用getConnection自动获取的是相同的连接池!
* */
public class JdbcUtils {
    private static DataSource dataSource;
    private static ThreadLocal<Connection> tl;

    static {
        Properties properties = new Properties();
        InputStream ips = JdbcUtils.class.getClassLoader().getResourceAsStream("durid.properties");
        try {
            properties.load(ips);
        } catch (IOException e) {
            throw new RuntimeException(e);
        }
        try {
            dataSource = DruidDataSourceFactory.createDataSource(properties);
        } catch (Exception e) {
            throw new RuntimeException(e);
        }
    }

    public static Connection getConnection() throws SQLException {
        //线程本地变量中是否存在
        Connection connection = tl.get();
        if (connection == null) {
            //线程本地变量没有，连接池获取
            connection = dataSource.getConnection();
            tl.set(connection);
        }
        return connection;
    }

    public static void freeConnection() throws SQLException {
        Connection connection = tl.get();
        if (connection != null) {
            tl.remove(); //清空线程本地变量数据
            connection.setAutoCommit(true); //事务状态回顾
            connection.close();  //回收到连接池即可
        }
    }
}
```

## 2. 测试

```java
public class BankDao {
    public void add(String account, int money) throws SQLException {
        Connection connection = JdbcUtils.getConnection();
        String sql = "update t_bank set money = money + ? where account = ?";
        PreparedStatement preparedStatement = connection.prepareStatement(sql);
        preparedStatement.setObject(1, money);
        preparedStatement.setObject(2, account);
        preparedStatement.executeUpdate();
        System.out.println("加钱成功！");
    }

    public void sub(String account, int money) throws SQLException {
        Connection connection = JdbcUtils.getConnection();
        String sql = "update t_bank set money = money - ? where account = ?";
        PreparedStatement preparedStatement = connection.prepareStatement(sql);
        preparedStatement.setObject(1, money);
        preparedStatement.setObject(2, account);
        preparedStatement.executeUpdate();
        System.out.println("减钱成功！");
    }
}
```

```java
public class BankDevice {
    @Test
    public void bankService() throws Exception {
        transfer("lvdandan", "ergouzi", 500);
    }

    public void transfer(String addAccount, String subAccount, int money) throws Exception {
        Connection connection = JdbcUtils.getConnection();
        try {
            //开启事务
            connection.setAutoCommit(false);
            BankDao bankDao = new BankDao();
            bankDao.add(addAccount, money);
            System.out.println("----------------------");
            bankDao.sub(subAccount, money);
            connection.commit();
        }
        catch (Exception e) {
            connection.rollback();
            throw  e;
        }
        finally {
            JdbcUtils.freeConnection();
        }

    }
}
```