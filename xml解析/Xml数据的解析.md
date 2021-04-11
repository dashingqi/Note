## 本文主要从以下几点学习XMl的解析

- xml的简介
- PULL解析
- SAX解析
- PULL与SAX解析的区别

## XML简介

- Xml是一种标记语言，是一种可扩展的标记语言；
- xml可用来传输数据和携带数据的。
- xml并不能用来展示数据的，这点Html可以的。

> 在进行pull和sax解析介绍之前，我们给一个xml数据，通过这个xml数据来实操着两种xml的解析

```java
 private final String xmlData = "<students>\n" +
            "\t<student>\n" +
            "\t\t<id>101</id>\n" +
            "\t\t<name>lisi</name>\n" +
            "\t\t<address>beijing</address>\n" +
            "\t</student>\n" +
            " \n" +
            "\t<student>\n" +
            "\t\t<id>100</id>\n" +
            "\t\t<name>lisi</name>\n" +
            "\t\t<address>beijing</address>\n" +
            "\t</student>\n" +
            "</students>";
```



## PULL解析

#### 标签的含义

- XmlPullParser.START_DOCUMENT：XML解析器目前在文件的最开始处，此事还没有读到任何数据
- XmlPullParser.END_DOCUMENT：逻辑意义上的文件末尾；表明当前XML解析器处理的文件流已经到末端
- XmlPullParser.START_TAG：XML解析器解析到了一个开始标签
- XmlPullParser.END_TAG：XML解析器解析到了一个结束标签
- XmlPullParser.TEXT：XML解析器解析到了字符数据

#### 代码实操

- 代码如下

  ```java
  private void pullParser() {
          try {
              //1. 创建XmlPullParseFactory
              XmlPullParserFactory xmlPullParserFactory = XmlPullParserFactory.newInstance();
              //2.创建XmlPullParser
              XmlPullParser xmlPullParser = xmlPullParserFactory.newPullParser();
              //3. 设置要读取的数据
              xmlPullParser.setInput(new StringReader(xmlData));
              //4. 获取到节点的类别
              int eventType = xmlPullParser.getEventType();
              private String id = "";
              private String name = "";
              private String address = "";
              
              while (eventType != XmlPullParser.END_DOCUMENT) {
                String name = xmlPullParser.getName();
                  switch (eventType) {
                      case XmlPullParser.START_TAG: {
                          if ("id".equals(name)) {
                             id = xmlPullParser.nextText();
                          } else if ("name".equals(name)) {
  													name = xmlPullParser.nextText();
                          }else if("address".equals(name)){
                            address = xmlPullParser.nextText();
                          }
                      }
                      break;
                      case XmlPullParser.END_TAG: {
                          if ("student".equals(name)) {
                              Log.d(TAG, "id =  "+id+" name = "+name+" address = "+address);
                          }
  
                      }
                      break;
                  }
                // 获取下一个事件
                  eventType = xmlPullParser.next();
              }
          } catch (XmlPullParserException | IOException e) {
              e.printStackTrace();
          }
      }
  ```

  

## SAX解析

#### 代码实操

- 代码如下

  ```java
  private void saxParser() {
          try {
              //1, 创建SAXParserFactory对象
              SAXParserFactory saxParserFactory = SAXParserFactory.newInstance();
              //2，创建SAXParser
              SAXParser saxParser = saxParserFactory.newSAXParser();
              //3,获取到XMLReader对象
              XMLReader xmlReader = saxParser.getXMLReader();
              //4,将自定义的MySaxHandler设置到xmlReader中
              xmlReader.setContentHandler(new MySaxHandler());
              //读取xml数据
              xmlReader.parse(new InputSource(new StringReader(xmlData)));
          } catch (ParserConfigurationException e) {
              e.printStackTrace();
          } catch (SAXException e) {
              e.printStackTrace();
          } catch (IOException e) {
              e.printStackTrace();
          }
      }
  
  class MySaxHandler extends DefaultHandler {
          private String nodeName = "";
          private StringBuilder ids;
  
          @Override
          public void startDocument() throws SAXException {
              super.startDocument();
              Log.d(TAG, "startDocument: ");
              ids = new StringBuilder();
          }
  
          @Override
          public void startElement(String uri, String localName, String qName, Attributes attributes) throws SAXException {
              super.startElement(uri, localName, qName, attributes);
              Log.d(TAG, "startElement: localName = " + localName);
              nodeName = localName;
          }
  
          @Override
          public void characters(char[] ch, int start, int length) throws SAXException {
              super.characters(ch, start, length);
              if ("id".equals(nodeName)) {
                  ids.append(ch, start, length);
              }
          }
  
          @Override
          public void endElement(String uri, String localName, String qName) throws SAXException {
              super.endElement(uri, localName, qName);
              Log.d(TAG, "endElement: localName = " + localName);
          }
  
          @Override
          public void endDocument() throws SAXException {
              super.endDocument();
              Log.d(TAG, "endDocument: ");
              Log.d(TAG, "endDocument: ids = " + ids.toString().trim());
              ids = null;
          }
      }
  ```

## 区别

#### SAX

- 是事件驱动的，不会将文档读入内存中在去解析，而是一边读取一边解析。
- 因为是一边读取一边解析，所以读取完也就全部循环遍历完文档中的节点，有点浪费资源。

#### PULL

- 是一种基于“拉”解析方式，也就是根据自己的需要去解析数据。
- 它继承了sax的速度快，占用内存少的优点。
- 相比较于SAX，性能好些，使用起来比较灵活一些！



