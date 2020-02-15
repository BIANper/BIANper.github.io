---
title: Java 补丁-XML
date: 2020-02-10 13:42:36
tags:
categories:Java
---

# 概述

相对 HTML 的区别：

-   大小写敏感，故 <h1> 和 <H1> 为不同 XML 标签
-   不可省略结束标签
-   没有结束标签的元素必须以 / 结尾
-   属性值必须用引号包围，包括数值
-   所有属性必须有属性值

## 标记

字符引用：

-   *&#十进制*  *&#x十六进制*

实体引用：

-   *&name*
-   &lt 小于；&gt 大于；&amp &；&quot 引号；可在 DTD 中定义其他实体引用。

CDATA Section：

-   *<![CDATA[* ... *]]>*
-   特殊的字符数据形式，囊括含有 < > & 等字符的字符串，而不必解释为标记。
-   <![CDATA[ & > are delimiters ]\]>

处理指令：

-   *<?* ... *?>*
-   在处理 XML 的应用程序中使用。

注释：

-   *<!--* ... *-->*
-   注释不应该含有 -- 字符串。

## 规范

避免混合式内容，即元素包含子元素和文本。

属性只用于修改值的解释，而不是指定值。优先使用元素。例：

```xml
<font name="Helvetica" size="18 pt" />

<font>
    <name>Helvetica</name>
    <size unit="pt">36</size>
</font>
```

# 解析文档

首先从 DocumentBuilderFactory 得到 DocumentBuilder 对象：

```java
DocumentBuilderFactory = DocumentBuilderFactory.newInstance();
DocumentBuilder builder = factory.newDocumentBuilder();
```

通过 builder 从文件，URL，输入流读入文档：

```java
File f = ...;
URL u = ...;
InputStream in = ...;

Document doc = builder.parse(f);
```

若使用输入流读取，解析器无法定位以该文档为相对路径而被引用的文档。

调用 `getDocumentElement` 方法对文档内容分析，并返回根元素：

```java
Element root = doc.getDocumentElement();
```

`getTagName` 方法可以返回元素的标签名。获得元素的子元素（元素，文本，注释，节点）使用 `getChildNodes` 方法，将返回一个 NodeList 类型的集合。item 方法得到指定索引值的项，`getLength ` 方法获得项的总数。故枚举子元素：

```java
NodeList children = root.getChildNodes();
for(int i = 0; i < children.getLength(); i++) {
    Node child = children.item(i);
    ...
}
```

元素与元素之间的空白字符也算作子元素，忽略空白字符：

```java
for(int i = 0; i < children.getLength(); i++) {
    Node child = children.item(i);
    if (child instanceof Element) {
        Element childElement = (Element) child;
        ...
    }
}
```

获得元素后，从 Text 类型的子节点中获取文本字符串，若节点可定位（如 Text 节点是唯一的子元素），可使用 `getFirstChild`  `getLastChild` 而不必遍历新的 NodeList ，然后用 `getData` 方法获取节点中的字符串：

```java
for(int i = 0; i < children.getLength(); i++) {
    Node child = children.item(i);
    if (child instanceof Element) {
        Element childElement = (Element) child;
        Text textNode = (Text) childElement.getFirstChild();
        String text = textNode.getData().trim();
        // trim 删除数据前后的空白字符
        if (childElement.getTagName().equals("name"))
            name = text;
        else if (childElement.getTagName().equals("size"))
            size = Interger.parseInt(text);
    }
}
```

若要遍历子节点集（不是 getChildNodes 来的 NodeList），结合 `getNextSibling` 方法（获取下一个兄弟节点）：

```java
for (Node childNode = element.getFirstChild();
    childNode != null;
    childNode = childNode.getNextSibling()) { ... }
```

若要枚举节点的属性，调用 `getAttributes` 方法，其返回一个 NamedNodeMap 对象，包含了描述属性的 Node 对象。与遍历 NodeList 的方式相同，用 `getNodeName`  `getNodeValue` 方法获取属性名和属性值：

```java
NamedNodeMap attributes = element.getAttributes();
String value = element.getAttribute("unit");
// 知道属性名可以直接获取对应值
for (int i = 0; i < attributes.getLength(); i++) {
    Node attribute = attributes.item(i);
    String name = attribute.getNodeName();
    String value = attribute.getNodeValue;
    ...
}
```

# 验证 XML 文档

## DTD

若在 XML 文档内部提供 DTD，规则应当纳入到 DOCTYPE 声明的 [] 中，且 DOCTYPE 必须匹配根元素的名字，如：

```xml-dtd
<!DOCTYPE configuration [
    <!ELEMENT configuration ...>
    ... ]>
<configuration></configuration>
```

更多是存储在外部，使用 SYSTEM 声明指定一个 DTD 文件：

```xml-dtd
<!DOCTYPE configuration SYSTEM "path">
<!DOCTYPE configuration SYSTEM "URL">
```

**各种规则**

**ELEMENT 规则**指定某个元素可以拥有的子元素，允许正则表达式。

|         规则          |              含义               |
| :-------------------: | :-----------------------------: |
|          E*           |           0 或多个 E            |
|          E+           |           1 或多个 E            |
|          E?           |           0 或 1 个 E           |
|    E1\|E2\|...\|En    |        E1,E2... 中的一个        |
|     E1,E2,...,En      |           E 按序排列            |
|       \#PCDATA        |              文本               |
| (#PCDATA\|E1...\|En)* | 0 或多个文本且 E 以任意顺序排列 |
|          ANY          |         允许任何子元素          |
|         EMPTY         |         不允许有子元素          |

当元素需要包含文本时，只有两种合法情况：

-   只包含文本 \#PCDATA
-   包含任意顺序的文本和标签 (#PCDATA\|E1...\|En)*

**ATTLIST 规则**指定元素属性，语法 `<!ATTLIST element attribute type default>`

|       类型        |           含义           |
| :---------------: | :----------------------: |
|       CDATA       |        任意字符串        |
| (A1\|A2\|...\|An) |   字符串属性为其中之一   |
| NMTOKEN NMTOKENS  |     1 或多个名字标记     |
|        ID         |      1 个唯一的 ID       |
|   IDREF IDREFS    | 1 或多个对唯一 ID 的引用 |
|  ENTITY ENTITIES  |   1 或多个未解析的实体   |

|   默认值   |                含义                |
| :--------: | :--------------------------------: |
| \#REQUIRED |              属性必需              |
| \#IMPLIED  |              属性可选              |
|     A      |      属性可选，若未指定则为 A      |
| \#FIXED A  | 属性必须为未指定或 A，解析器都报 A |

例：

```dtd
<!ATTLIST font style (plain|bold|italic) "plain">
<!ATTLIST size unit CDATA #IMPLIED>
```

**使用**

通知工厂启用验证，此工厂生成的所有文档生成器都将根据 DTD 验证输入：

```java
factory.setValidating(true);
```

忽略元素节点之间的空白字符：

```java
factory.setIgnoringElementContentWhitespace(true);
```

这样就不用再通过循环剔除空白字符了：

```dtd
<!ELEMENT font (name,size)>
<!ELEMENT name (#PCDATA)>
<!ELEMENT size (#PCDATA)>
```

```java
Element name = (Element) children.item(0);
Element size = (Element) children.item(1);
```

## XML Schema

占坑

# XPath

## 概述

相比遍历 DOM 树，XPath 访问节点更加简单，通过对 /configration/database/username 求值获取 username 的值。

也可以描述节点集，如 /gridbag/row 描述根元素 gridbag 下的所有子元素 row，通过 [] 操作符选择元素：/gridbag/row[1] 表示第一行。

@ 操作符获取属性值：/gridbag/row[1]/cell[1]/@anchor 。

配合 XPath 的函数进行精细要求。

## 使用

首先从 XPathFactory 创建 XPath 对象：

```java
XPathFactory xpfactory = XPathFactory.newInstance();
path = xpfactory.nextXPath();
```

然后用 evaluate 方法计算 XPath 表达式（用于获取文本）：

```java
String name = path.evaluate("/configuration/database/username", doc);
```

若 XPath 表达式产生了一组节点：

```java
NodeList nodes = (NodeList) path.evaluate("/gridbag/row", doc, XPathConstants.NODESET);
```

若仅一个节点：

```java
Node node = (Node) path.evaluate("/gridbag/row[1]", doc, XPathContants.NODE);
```

若是数字：

```java
int num = ((Number) path.evaluate("count(/...)", doc, XPathConstants.NUMBER)).intValue();
```

从以前获得的节点开始：

```java
result = path.evaluate(expression, node);
```

# 命名空间

XML 使用