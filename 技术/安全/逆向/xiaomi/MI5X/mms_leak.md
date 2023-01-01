#### 获取mms下的私有数据
#### 漏洞归属
- 移动端漏洞
#### 漏洞类型
- 移动客户端
- 移动客户端组件漏洞
#### 危害等级
- 低危
#### 漏洞描述
- 测试机型：MI5X MIUI系统版本：11.0.3|稳定版 安全补丁日期：2019-10-01
- app包名：com.android.mms
- 通过恶意App能调用到SmsHybridActivity.
- 通过SmsHybridActivity能加载恶意app提供的本地file协议的url
- 通过加载本地的url，能获取到mms私有目录下的敏感数据，比如sp中的数据。
#### 漏洞证明
- SmsHybridFragment中关于WenView的配置项代码
  ```
        settings.setJavaScriptEnabled(true);
        settings.setCacheMode(-1);
        settings.setAllowFileAccessFromFileURLs(true);
        settings.setAllowUniversalAccessFromFileURLs(true);
  ```
- 读取 /data/user/0/com.android.mms/shared_prefs/com.android.mms_preferences.xml 该路径下的文件
   ![F0D65EB234903F55808AEDB94A626D9E.jpg](https://upload-images.jianshu.io/upload_images/4997216-6c8e36d6fb432bdd.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
#### 修复方案
- app不需要进行文件域操作，那么，设置为false即可
  ```
     settings.setAllowFileAccessFromFileURLs(false);
     settings.setAllowUniversalAccessFromFileURLs(false);
  ```
- 需要文件操作的话，对加载本地的file协议的url做限定，不去加载file协议的url
