## WebView的漏洞

#### 漏洞分类

- 任意代码执行漏洞
- 密码明文存储漏洞
- 域控制不严格漏洞

#### 漏洞的具体分析

###### 任意代码执行漏洞

- 出现该漏洞的原因如下
  - WebView中的addJavascriptInterface()接口
  - WebView内置导出的searchBoxJavaBridge_对象
  - WebView内置导出accessibility和accessibilityTraversal 对象

###### 密码明文存储漏洞

###### 域控制不严格漏洞