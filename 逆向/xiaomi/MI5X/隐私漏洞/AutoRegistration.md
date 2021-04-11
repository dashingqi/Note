#### 漏洞标题

MIUI正式版本存在手机信息泄漏漏洞

#### 漏洞归属

隐私漏洞

#### APP

系统APP

#### 漏洞类型

未公开收集使用规则（高通，sim激活之类的）

#### 危害等级

严重

#### 漏洞描述

##### 系统类型

MIUI 11.0.3 ｜ 稳定版本

##### 测试APP版本号

- APP名字：AutoRegistration 

- 包名：com.qualcomn.qti

- 配置文件内

  ```xml
  android:sharedUserId="android.uid.phone" android:versionCode="3" android:versionName="3.0" package="com.qualcomm.qti.autoregistration" platformBuildVersionCode="27" platformBuildVersionName="8.1.0"
  ```

#### 漏洞细节

通过关闭 SIM卡 下的 启动VoLTE高清通话可以抓取到当前的请求

```java
//发出请求的代码
public void requestWithRawData(final String rawdata) {
        new Thread() {
            /* class com.qualcomm.qti.autoregistration.RegistrationTask.AnonymousClass2 */

            public void run() {
                String response = RegistrationTask.this.post(Base64.encode(rawdata.getBytes(), 0));
                RegistrationTask.this.onResult(RegistrationTask.this.isRegisterSucceed(response), rawdata, RegistrationTask.this.getResultDesc(response));
            }
        }.start();
    }
```

通过Base64的解密可以解出传输的数据，包含了imei和手机的各种信息

```java
new String(Base64.decode(str.getBytes("UTF-8"), Base64.DEFAULT));
```



#### 漏洞证明

##### 服务端信息

- 注册商：厦门三五互联科技有限公司
- 注册日期：2003/3/17
- 到期日期：2022/3/17

![IMG](https://cdn.cnbj1.fds.api.mi-img.com/src/image/000d489fa5c62cad4cb42dbac411ef.jpg)

![IMG](https://cdn.cnbj1.fds.api.mi-img.com/src/image/af94e80d181fff3c390ab82fc165b2.jpg)

#### 修复方案

- 在关闭 启动VoLTE高清通话时不要进行数据包的发送！

- 加密的数据可加盐！

- 是不是可以考虑用下https

  