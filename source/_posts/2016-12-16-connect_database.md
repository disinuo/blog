---
title: j2ee数据库连接的两种机制---DataSource vs DriverManager
date: 2016-12-16 10:26:43
tags:
    - database
    - java
    - j2ee
---
### 1.前言
最近在写j2ee作业~在网上直接查询【数据库连接】大多都是DriveManager机制那种，看起来跟老师讲的DataSource不大一样啊，于是又查了查资料~各自用法及对比记录如下^ ^
### 2.DriverManager用法
```java
// 0.设置数据库url
static final String url = "jdbc:mysql://localhost:3306/myDB";
// 说明一下~这里是重写了service方法，
// 于是GET和POST请求都会通过这个方法实现响应
// 是因为实际上请求都会先经过service方法，
// 根据请求的类型再不同分别调用doPost和doGet
// 所以这里直接重写service，也是因为我这作业对post和get的响应一样
// 不想写成两个方法~就酱了~
protected void service(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
  response.setContentType("text/html; charset=UTF-8");
  try{// 1.加载驱动程序
      Class.forName("com.mysql.jdbc.Driver");
      // 2. 获得数据库连接
      Connection conn = DriverManager.getConnection(url, USER, PASSWORD);
      // 3.操作数据库，实现增删改查
      Statement stmt = conn.createStatement();
      ResultSet rs = stmt.executeQuery("SELECT * FROM books");
      //如果有数据，rs.next()返回true
      while(rs.next()){
          System.out.println(" 书名："+rs.getString("title")+" 作者："+rs.getInt("author"));
      }
      rs.close();
      stmt.close();
      con.close();  
    }catch(Exception e){
      System.out.print("service method error"+e);
    }
}

```
### 3.DataSource用法
- #### 3.1 用resin 容器
首先要写一个配置文件~命名为`resin-web.xml`(会覆盖web.xml)，位置就放在名为WebContent的目录下。文件结构如下~
```
--WebContent
----META-INF
----WEB-INF
----resin-web.xml
```

```xml
<web-app xmlns="http://caucho.com/ns/resin">
  <!--
     - 配置数据库.
     - jndi-name 配置 the JNDI name
     - type  配置 driver 的class
    -->
  <database>
    <jndi-name>jdbc/myDB</jndi-name>
    <driver>
      <type>com.mysql.jdbc.Driver</type>
      <url>jdbc:mysql://localhost:3306/myDB</url>
      <user>root</user>
      <password></password>
    </driver>
  </database>
  <!--
     - 配置servlet。
     - 查询 JNDI 数据源
    -->
  <servlet>
    <servlet-name>datasource</servlet-name>
    <servlet-class>com.DataSourceExample</servlet-class>
    <init>
      <data-source>${jndi:lookup('jdbc/myDB')}</data-source>
    </init>
  </servlet>
</web-app>
```
然后，在servlet类里面这样用
```java
private DataSource ds = null;
// 这是会被Resin服务器在配置的时候自动调用的方法
public void setDataSource(DataSource ds) {
    this.ds = ds;
}
public void init() throws ServletException {
    if (ds == null) {
        throw new ServletException("datasource not properly configured");
    }
}
protected void service(HttpServletRequest request,HttpServletResponse response)throws ServletException, IOException {
    response.setContentType("text/html;charset=UTF-8");
    try {
          Connection conn = _ds.getConnection();
          Statement stmt = conn.createStatement();
          ResultSet rs = stmt.executeQuery("SELECT * FROM books");
          //如果有数据，rs.next()返回true
          while(rs.next()){
              System.out.println(" 书名："+rs.getString("title")+" 作者："+rs.getInt("author"));
          }
          rs.close();
          stmt.close();
          con.close();  
    }catch(Exception e){
      System.out.print("service method error"+e);
    }
}
```
- #### 3.2 用tomcat web容器
这个相比较于resin，配置文件更简单一点，不过代码里面多了点东西~其实本质就是把
`jndi:lookup('jdbc/myDB')`从配置文件里移到了代码里。。所以我更喜欢resin那种~
接下来看看具体怎么写吧^ ^
配置文件：命名为context.xml，位置如下
```
--WebContent
----META-INF
------context.xml
----WEB-INF
```
```xml
<Context>
  <Resource name="jdbc/myDB" auth="Container" type="javax.sql.DataSource"
               maxActive="50" maxIdle="30" maxWait="10000" logAbandoned="true"
               username="root" password=""
                driverClassName="com.mysql.jdbc.Driver"
               url="jdbc:mysql://localhost:3306/myDB?autoReconnect=true"/>
</Context>
```
然后，在servlet类里面这样用
```java
private DataSource datasource = null;
public void init() throws ServletException{
  try {
// 新引入的三个类
// javax.naming.Context;
// javax.naming.InitialContext;
// javax.naming.NamingException;
    Context ctx=new InitialContext();
    datasource= (DataSource) ctx.lookup("java:comp/env/jdbc/myDB");
  }catch (NamingException e) {
    System.out.println("Initial Error: "+e);
    e.printStackTrace();
  }
}
// 然后service跟resin的一样~
```
### 4.DataSource 与 DriverManager的比较
1. 配置信息：
Datasource要把url，user，password都写在代码里
2. 性能：
Datasource的连接池

***********
参考：
[DataSource & DriverManager - ZetCode](http://zetcode.com/tutorials/jeetutorials/datasource/)
[菜鸟教程的《JDBC使用说明》](http://www.runoob.com/w3cnote/jdbc-use-guide.html)
