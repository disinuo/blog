---
title: DOM Parser
date: 2017-04-13 22:35:37
tags:
    - SOA
    - DOM
---
### DOM
Document Object Model
### DOM Parser
#### 特点
将 XML 整个作为类似树结构的方式读入内存中以便操作及解析
<!-- more -->
#### 优点
- 实现 W3C 标准，有多种编程语言支持这种解析方式，并且这种方法本身操作上简单快捷，十分易于初学者掌握。
- 其处理方式是将 XML 整个作为类似树结构的方式读入内存中以便操作及解析，因此支持应用程序对 XML 数据的内容和结构进行修改

#### 缺点
- 但是同时由于其需要在处理开始时将整个 XML 文件读入到内存中去进行分析，因此其在解析大数据量的 XML 文件时会遇到类似于内存泄露以及程序崩溃的风险，请对这点多加注意。
#### 适用范围
- 小型 XML 文件解析
- 需要修改XML文档的内容/结构/顺序
- 需要多次遍历文档【用DOM就不需要每次都重新扫描文档】
- 需要将几个xml文档合并
- 需要操作文档的内容

#### 处理流程
![](/image/2017-04-14-DOM-parser/DOM-process.png)
##### （一）创建DOM parser
1. 创建工厂实例
2. 设置工厂的参数
![](/image/2017-04-14-DOM-parser/factory-options.png)
3. 通过工厂创建Builder实例

##### （二）Parser读取xml
##### （三）异常处理
在这个过程中可能会产生如下几种异常
- javax.xml.parsers.ParserConfigurationException:
- java.lang.IllegalArgumentException:
- java.io.IOException: xml文件打不开
- org.xml.sax.SAXException:xml有错误

##### （四）生成DOM树
##### （五）调用方可以对DOM树对象操作(增删改查)
##### （六）最后，调用方把xml输出

#### 代码示例
```java
// 1。创建parser
DocumentBuilderFactory factory = DocumentBuilderFactory.newInstance();
DocumentBuilder builder = factory.newDocumentBuilder();
// 2+3+4.读取xml，处理error，生成DOM树
File f = new File("dom.xml");
Document dom=builder.parse(f);
// 5 对DOM树操作
NodeList contacts = dom.getElementsByTagName("Contact");
for(int i = 0; i < contacts.getLength(); i++) {
     Element contact = (Element) contacts.item(i);
     System.out.println(contact.getNodeValue());
}
```
#### 一切皆Node

##### 4种Node
Document(root)、Element、Attribute、Text
##### 举例
  ![](/image/2017-04-14-DOM-parser/node-example.png)
##### 是组合模式的应用
  ![](/image/2017-04-14-DOM-parser/composite-pattern.png)
##### 提供接口
    1. public String getNodeName()
    - public String getNodeValue()throws DOMException
    - public void setNodeValue(String nodeValue)throws DOMException
    - public short getNodeType()
    - public Node getParentNode()
    - public NodeList getChildNodes()
    - public Node getFirstChild()
    - public Node getLastChild()
    - public Node getPreviousSibling()
    - public Node getNextSibling()
    - public NamedNodeMap getAttributes()
    - public Document getOwnerDocument()
![](/image/2017-04-14-DOM-parser/interface.png)


-------
参考
[xm341v13_dom.pdf](https://www.cs.bgu.ac.il/~dwss111/wiki.files/XML/xm341v13_dom.pdf)
[ibm:java处理三种xml主流技术介绍](https://www.ibm.com/developerworks/cn/xml/dm-1208gub/)
