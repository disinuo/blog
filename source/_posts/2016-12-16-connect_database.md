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
static final String url = "jdbc:mysql://localhost:3306/books";
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
首先要写一个配置文件~

```xml
<web-app xmlns="http://caucho.com/ns/resin">
  <!--
     - Configures the database.
     - jndi-name specifies the JNDI name
     - type      specifies the driver class
     - path      is a driver-specific configuration parameter
    -->
 <database>
 <jndi-name>jdbc/mysql</jndi-name>
  <driver>
    <type>com.mysql.jdbc.jdbc2.optional.MysqlConnectionPoolDataSource</type>
    <url>jdbc:mysql://localhost:3306/books</url>
    <user>root</user>
    <password></password>
  </driver>
  </database>
  <!--
     - Configures the initialization servlet.  The bean-style init
     - it used to look up the JNDI DataSource in the configuration file.
    -->
   <servlet>
    <servlet-name>datasource</servlet-name>
    <servlet-class>com.zetcode.DataSourceExample</servlet-class>
        <init>
      <data-source>${jndi:lookup('jdbc/mysql')}</data-source>
    </init>
  </servlet>
  <servlet-mapping>
    <url-pattern>/test</url-pattern>
    <servlet-name>datasource</servlet-name>
  </servlet-mapping>
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


### 4.DataSource 与 DriverManager的比较

***********
参考：
[DataSource & DriverManager - ZetCode](http://zetcode.com/tutorials/jeetutorials/datasource/)
[菜鸟教程的《JDBC使用说明》](http://www.runoob.com/w3cnote/jdbc-use-guide.html)
