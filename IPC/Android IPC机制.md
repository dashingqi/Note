### IPC简介

- Inter-Process Communication 跨进程通信/进程间通信

###### 进程

- 指一个执行单元，进程指的是PC或者移动设备中一个程序。一个进程中有一个或者多个线程，

###### 线程

- 线程是CPU调度最小的单元，线程是一种有限的系统资源。

### Android中的多进程模式

###### Android中开启多进程

> 1. 一个应用存在多个进程
> 2. 多个应用之间的通信

- 在AndroidManifest文件中为四大组件指定 android:process="name"属性，就是Android唯一开启多进程的方式

- 另外一种飞常规的方法就是通过JNI在native层fork一个新的进程。

  ```java
  <?xml version="1.0" encoding="utf-8"?>
  <manifest xmlns:android="http://schemas.android.com/apk/res/android"
      package="com.dashingqi.androidipcproject">
  
      <application
          android:allowBackup="true"
          android:icon="@mipmap/ic_launcher"
          android:label="@string/app_name"
          android:roundIcon="@mipmap/ic_launcher_round"
          android:supportsRtl="true"
          android:theme="@style/AppTheme">
          <activity
              android:name=".ThirdActivity"
                //完整命名，开启一个新的进程。
                //该方式命名的进程属于全局进程，其他应用可以通过ShareUID方式可以和它跑在一个进程中
              android:process="com.dashing.androidipcproject.third" />
          <activity
              android:name=".SecondActivity"
                //冒号开头命名，开启一个新的进程。
                //该方式命名的进程是属于当前应用的私有进程，其他应用的组件不能和它跑在同一个进程中。
              android:process=":second" />
          <activity android:name=".MainActivity">
              <intent-filter>
                  <action android:name="android.intent.action.MAIN" />
                  <category android:name="android.intent.category.LAUNCHER" />
              </intent-filter>
          </activity>
      </application>
  
  </manifest>
  ```

  

### 多进程模式的运行机制

- Android为每一个进程分配一个独立的虚拟机，不同的虚拟机在内存分配上有不同的地址空间，所以在不同虚拟机中访问相同一个类的时候会产生多个该类的副本。
- 运行在不同进程中的四大组件，它们之间通过内存来共享数据就一定会出现问题，这也是多进程带来的问题。

###### 多进程带来的问题

- 静态成员和单利模式完全失效。
- SharedPreference的可靠性下降。
- 线程同步机制失效。
- Application会创建多次：运行在同一个进程中组件是属于同一个虚拟机和Application，运行在不同进程中的组件是属于不同的虚拟机和Application的。

### IPC基础概念介绍

###### Serializable接口

- Java提供的一个序列化接口，是一个空接口。

  ```java
  public interface Serializable {
  }
  ```

- 使用：类实现该接口，并且在类中声明一个serialVersionUID即可。（serialVersionUID不是必需的，但是在反序列化的时候会有影响）

  ```java
  public MyClass implements Serializable{
  	public static final long serialVersionUID = 3423423434343L;
  }
  ```

- 序列化与反序列化模版代码

  ```java
  /**
       * 序列化过程
       */
      private static void serialMethod() {
          MySerial dashingQi = new MySerial(10, "DashingQi");
          try {
              ObjectOutputStream objectOutputStream = new ObjectOutputStream(new FileOutputStream("dashingqi.txt"));
              objectOutputStream.writeObject(dashingQi);
              objectOutputStream.close();
          } catch (IOException e) {
              e.printStackTrace();
          }
      }
  
  /**
       * 反序列化过程
       */
      private static void unSerialMethod() {
          try {
              ObjectInputStream objectInputStream = new ObjectInputStream(new FileInputStream("dashingqi.txt"));
              MySerial mySerial = (MySerial) objectInputStream.readObject();
              objectInputStream.close();
              System.out.println("id = " + mySerial.getId() + " name = " + mySerial.getName());
          } catch (IOException | ClassNotFoundException e) {
              e.printStackTrace();
          }
      }
  ```

- serialVersionUID作用：可以作为反序列化中的一种标准，序列化的时候把serialVersionUID写入到文件中，当反序列化的时候去比对这个serialVersionUID，一致就反序列化成功否则InvalidClassException；如果没有指定ID，那么默认是把类的hashCode作为这个id，当类有所变化的时候类的hashCode就发生变化了，反序列化就会出现问题。所以指定一个固定的serialVersionUID对于反序列化很重要。

###### Parcelable接口（与Serializable的对比）

- 是Android平台提供的，使用起来比较复杂，但是效率高，主要用在内存的序列化上。

- 通过Parcelable将对象序列化到存储设备中或者通过网络传输是可以的，但是比较复杂，这时可以选择使用Serializable。

- Serializable使用起来简单，但是序列化与反序列化的过程中需要大量I/O操作，开销比较大。

  ```java
  package com.dashingqi.parcelableproject;
  
  import android.os.Parcel;
  import android.os.Parcelable;
  
  /**
   * @author Dashingqi
   * @description:
   * @date :2020-02-15 12:55
   */
  public class MyParcelableClass implements Parcelable {
  
      private String name;
      private int id;
  
      /**
       * 从序列化后的对象中创建原始对象
       * @param in
       */
      protected MyParcelableClass(Parcel in) {
          name = in.readString();
          id = in.readInt();
      }
  
      public static final Creator<MyParcelableClass> CREATOR = new Creator<MyParcelableClass>() {
          /**
           * 从序列化后的对象中创建原始对象
           * @param in
           * @return
           */
          @Override
          public MyParcelableClass createFromParcel(Parcel in) {
              return new MyParcelableClass(in);
          }
  
          /**
           * 创建指定原始对象的数组
           * @param size
           * @return
           */
          @Override
          public MyParcelableClass[] newArray(int size) {
              return new MyParcelableClass[size];
          }
      };
  
      /**
       * 返回当前对象的描述
       * @return
       */
      @Override
      public int describeContents() {
          return 0;
      }
  
      /**
       * 将当前对象写入到序列化结构中
       * @param dest
       * @param flags
       */
      @Override
      public void writeToParcel(Parcel dest, int flags) {
          dest.writeString(name);
          dest.writeInt(id);
      }
  }
  
  ```

- 经过Serialzable和Parcelable序列化后的都是可以用于Intent间的传递数据。

###### Binder

- 直观来说，Binder是Android中的一个类，实现了IBinder接口。

- IPC角度说，Binder是Aandroid中的一种跨进程通信方式。

- 从Android FrameWork角度说，Binder是ServiceManager链接各种Manager和相应ManagerService的桥梁。

- 从应用层说，Binder是客户端和服务端进行通信的媒介，当bindService的时候，服务端会返回一个包含了服务端业务调用的Binder对象，通过这个Binder对象，客户端就可以获取服务端提供的服务和数据。

  ###### AIDL分析Binder的原理

  ```java
  /*
   * This file is auto-generated.  DO NOT MODIFY.
   */
  package com.dashingqi.aidlproject.aidl;
  
  /**
   * 所有可以在Binder传输的接口都需要继承IInterface接口
   */
  public interface IBookManger extends android.os.IInterface {
      /**
       * Default implementation for IBookManger.
       */
      public static class Default implements com.dashingqi.aidlproject.aidl.IBookManger {
          /**
           * Demonstrates some basic types that you can use as parameters
           * and return values in AIDL.
           */
          @Override
          public void basicTypes(int anInt, long aLong, boolean aBoolean, float aFloat, double aDouble, java.lang.String aString) throws android.os.RemoteException {
          }
  
          @Override
          public java.util.List<com.dashingqi.aidlproject.aidl.Book> getBookList() throws android.os.RemoteException {
              return null;
          }
  
          @Override
          public void addBook(com.dashingqi.aidlproject.aidl.Book book) throws android.os.RemoteException {
          }
  
          @Override
          public android.os.IBinder asBinder() {
              return null;
          }
      }
  
      /**
       * Local-side IPC implementation stub class.
       * 声明了一个内部类Stub 本身是一个Binder类
       * 当客户端和服务端位于不同进程的时候，方法调用需要走transact过程，
       * 这个逻辑是由Stub内部代理类Proxy来完成的。
       */
      public static abstract class Stub extends android.os.Binder implements com.dashingqi.aidlproject.aidl.IBookManger {
          //Binder的唯一标识，一般用Binder的类名来表示。
          private static final java.lang.String DESCRIPTOR = "com.dashingqi.aidlproject.aidl.IBookManger";
  
          /**
           * Construct the stub at attach it to the interface.
           */
          public Stub() {
              this.attachInterface(this, DESCRIPTOR);
          }
  
          /**
           * Cast an IBinder object into an com.dashingqi.aidlproject.aidl.IBookManger interface,
           * generating a proxy if needed.
           *
           *
           * 将服务端的Binder对象转换成客户端所需要的AILD接口类型的对象。
           *
           */
          public static com.dashingqi.aidlproject.aidl.IBookManger asInterface(android.os.IBinder obj) {
              if ((obj == null)) {
                  return null;
              }
              android.os.IInterface iin = obj.queryLocalInterface(DESCRIPTOR);
              //如果客户端和服务端在同一个进程的时候返回的是服务端的Stub对象本身
              if (((iin != null) && (iin instanceof com.dashingqi.aidlproject.aidl.IBookManger))) {
                  return ((com.dashingqi.aidlproject.aidl.IBookManger) iin);
              }
              // 不在同一个进程，那么返回的是Stub.proxy对象。
              return new com.dashingqi.aidlproject.aidl.IBookManger.Stub.Proxy(obj);
          }
  
          /**
           * 用于返回当前的Binder对象本身。
           * @return
           */
          @Override
          public android.os.IBinder asBinder() {
              return this;
          }
  
          /**
           * 该方法是运行在服务端的Binder线程池中，当客户端发起跨进程请求时，远程请求会通过系统底层封装后交给这个方法来处理。
           * 服务端通过code可以确认客户端请求的目标方法，从参数data取出目标方法所需要的参数，然后执行目标方法。
           * 当目标方法执行完毕后，就向reply中写入返回值。
           * 如果onTransact方法返回false就表示客户端请求失败。
           * @param code
           * @param data
           * @param reply
           * @param flags
           * @return
           * @throws android.os.RemoteException
           */
          @Override
          public boolean onTransact(int code, android.os.Parcel data, android.os.Parcel reply, int flags) throws android.os.RemoteException {
              java.lang.String descriptor = DESCRIPTOR;
              switch (code) {
                  case INTERFACE_TRANSACTION: {
                      reply.writeString(descriptor);
                      return true;
                  }
                  case TRANSACTION_basicTypes: {
                      data.enforceInterface(descriptor);
                      int _arg0;
                      _arg0 = data.readInt();
                      long _arg1;
                      _arg1 = data.readLong();
                      boolean _arg2;
                      _arg2 = (0 != data.readInt());
                      float _arg3;
                      _arg3 = data.readFloat();
                      double _arg4;
                      _arg4 = data.readDouble();
                      java.lang.String _arg5;
                      _arg5 = data.readString();
                      this.basicTypes(_arg0, _arg1, _arg2, _arg3, _arg4, _arg5);
                      reply.writeNoException();
                      return true;
                  }
                  case TRANSACTION_getBookList: {
                      data.enforceInterface(descriptor);
                      java.util.List<com.dashingqi.aidlproject.aidl.Book> _result = this.getBookList();
                      reply.writeNoException();
                      reply.writeTypedList(_result);
                      return true;
                  }
                  case TRANSACTION_addBook: {
                      data.enforceInterface(descriptor);
                      com.dashingqi.aidlproject.aidl.Book _arg0;
                      if ((0 != data.readInt())) {
                          _arg0 = com.dashingqi.aidlproject.aidl.Book.CREATOR.createFromParcel(data);
                      } else {
                          _arg0 = null;
                      }
                      this.addBook(_arg0);
                      reply.writeNoException();
                      return true;
                  }
                  default: {
                      return super.onTransact(code, data, reply, flags);
                  }
              }
          }
  
          private static class Proxy implements com.dashingqi.aidlproject.aidl.IBookManger {
              private android.os.IBinder mRemote;
  
              Proxy(android.os.IBinder remote) {
                  mRemote = remote;
              }
  
              @Override
              public android.os.IBinder asBinder() {
                  return mRemote;
              }
  
              public java.lang.String getInterfaceDescriptor() {
                  return DESCRIPTOR;
              }
  
              /**
               * Demonstrates some basic types that you can use as parameters
               * and return values in AIDL.
               */
              @Override
              public void basicTypes(int anInt, long aLong, boolean aBoolean, float aFloat, double aDouble, java.lang.String aString) throws android.os.RemoteException {
                  android.os.Parcel _data = android.os.Parcel.obtain();
                  android.os.Parcel _reply = android.os.Parcel.obtain();
                  try {
                      _data.writeInterfaceToken(DESCRIPTOR);
                      _data.writeInt(anInt);
                      _data.writeLong(aLong);
                      _data.writeInt(((aBoolean) ? (1) : (0)));
                      _data.writeFloat(aFloat);
                      _data.writeDouble(aDouble);
                      _data.writeString(aString);
                      boolean _status = mRemote.transact(Stub.TRANSACTION_basicTypes, _data, _reply, 0);
                      if (!_status && getDefaultImpl() != null) {
                          getDefaultImpl().basicTypes(anInt, aLong, aBoolean, aFloat, aDouble, aString);
                          return;
                      }
                      _reply.readException();
                  } finally {
                      _reply.recycle();
                      _data.recycle();
                  }
              }
  
              /**
               * 该方法是运行在客户端，
               * 首先创建需要的输入型Parcel对象_data 和输出型对象 _reply 和返回值对象 _result
               * 然后把该方法的参数写入到_data中
               * 接着调用transact()方法，发起RPC（远程过程调用）请求，同时当前线程挂起；
               * 然后服务端的onTransact()方法会被调用，当RPC过程返回后，当前线程继续执行，从_reply中取出RPC过程返回的数据。
               * @return
               * @throws android.os.RemoteException
               */
              @Override
              public java.util.List<com.dashingqi.aidlproject.aidl.Book> getBookList() throws android.os.RemoteException {
                  android.os.Parcel _data = android.os.Parcel.obtain();
                  android.os.Parcel _reply = android.os.Parcel.obtain();
                  java.util.List<com.dashingqi.aidlproject.aidl.Book> _result;
                  try {
                      _data.writeInterfaceToken(DESCRIPTOR);
                      boolean _status = mRemote.transact(Stub.TRANSACTION_getBookList, _data, _reply, 0);
                      if (!_status && getDefaultImpl() != null) {
                          return getDefaultImpl().getBookList();
                      }
                      _reply.readException();
                      _result = _reply.createTypedArrayList(com.dashingqi.aidlproject.aidl.Book.CREATOR);
                  } finally {
                      _reply.recycle();
                      _data.recycle();
                  }
                  return _result;
              }
  
              @Override
              public void addBook(com.dashingqi.aidlproject.aidl.Book book) throws android.os.RemoteException {
                  android.os.Parcel _data = android.os.Parcel.obtain();
                  android.os.Parcel _reply = android.os.Parcel.obtain();
                  try {
                      _data.writeInterfaceToken(DESCRIPTOR);
                      if ((book != null)) {
                          _data.writeInt(1);
                          book.writeToParcel(_data, 0);
                      } else {
                          _data.writeInt(0);
                      }
                      boolean _status = mRemote.transact(Stub.TRANSACTION_addBook, _data, _reply, 0);
                      if (!_status && getDefaultImpl() != null) {
                          getDefaultImpl().addBook(book);
                          return;
                      }
                      _reply.readException();
                  } finally {
                      _reply.recycle();
                      _data.recycle();
                  }
              }
  
              public static com.dashingqi.aidlproject.aidl.IBookManger sDefaultImpl;
          }
  
          static final int TRANSACTION_basicTypes = (android.os.IBinder.FIRST_CALL_TRANSACTION + 0);
          //声明了两个ID，分别用于标识这个两个方法，在transact过程中客户端所请求到底是那个方法
          static final int TRANSACTION_getBookList = (android.os.IBinder.FIRST_CALL_TRANSACTION + 1);
          static final int TRANSACTION_addBook = (android.os.IBinder.FIRST_CALL_TRANSACTION + 2);
  
          public static boolean setDefaultImpl(com.dashingqi.aidlproject.aidl.IBookManger impl) {
              if (Stub.Proxy.sDefaultImpl == null && impl != null) {
                  Stub.Proxy.sDefaultImpl = impl;
                  return true;
              }
              return false;
          }
  
          public static com.dashingqi.aidlproject.aidl.IBookManger getDefaultImpl() {
              return Stub.Proxy.sDefaultImpl;
          }
      }
  
      /**
       * Demonstrates some basic types that you can use as parameters
       * and return values in AIDL.
       */
      public void basicTypes(int anInt, long aLong, boolean aBoolean, float aFloat, double aDouble, java.lang.String aString) throws android.os.RemoteException;
  
  
      /**
       * getBookList()和addBook()是我们在IBookManager.aidl中所声明的方法
       * @return
       * @throws android.os.RemoteException
       */
      public java.util.List<com.dashingqi.aidlproject.aidl.Book> getBookList() throws android.os.RemoteException;
  
      public void addBook(com.dashingqi.aidlproject.aidl.Book book) throws android.os.RemoteException;
  }
  
  ```

  ###### Binder原理流程图

  - <img src="https://upload-images.jianshu.io/upload_images/4997216-52a19ba625572111.png?imageMogr2/auto-orient/strip|imageView2/2/w/1240" style="zoom:200%;" />

### Android中的IPC方式

###### Bundle

- Bundle实现了Parcelable接口，可以在不同的进程间传输。
- ProcessA中启动ProcessB中的Activity、Service、Receiver，可以在Bundle中附加信息通过Intent传递过去。

###### 使用文件共享

- 两个进程通过读/写同一个文件来交换数据。
- 局限性：就是并发的读写的问题。
- SharedPreferences：/data/data/package name/shared_prefs目录下；底层实现采用xml文件来存储。也属于文件的一种，不建议在进程间通信使用。

###### Messenger

- 信使，在不同进程中传递Message对象。

- 轻量级的IPC方案，底层实现是AIDL。

- 一次处理一个请求，服务端不用考虑线程同步的问题。

  ###### 使用代码

  - 客户端

    ```java
    public class MainActivity extends AppCompatActivity {
    
        private static final String TAG = "MainActivity";
    
    
        private ServiceConnection mServiceConnection = new ServiceConnection() {
            @Override
            public void onServiceConnected(ComponentName name, IBinder service) {
                //获取到服务端的Messneger
                Messenger messenger = new Messenger(service);
                //创建一个Message对象
                Message message = Message.obtain();
                message.what = 10001;
                message.replyTo = clientMessenger;
                Bundle bundle = new Bundle();
                bundle.putString("client_message", "i am from client");
                //设置要传递的数据
                message.setData(bundle);
    
                try {
                    //发送消息到服务端
                    messenger.send(message);
                } catch (RemoteException e) {
                    e.printStackTrace();
                }
            }
    
            @Override
            public void onServiceDisconnected(ComponentName name) {
    
            }
        };
    
    
        private static class ClientHandler extends Handler {
            @Override
            public void handleMessage(@NonNull Message msg) {
                switch (msg.what) {
                    case 10002: {
                        String service_message = msg.getData().getString("service_message");
                        Log.d(TAG, "handleMessage: service message " + service_message);
                    }
                    break;
                    default:
                        super.handleMessage(msg);
                }
    
            }
        }
    
        /**
         * 客户端的Messenger
         */
        private Messenger clientMessenger = new Messenger(new ClientHandler());
    
        @Override
        protected void onCreate(Bundle savedInstanceState) {
            super.onCreate(savedInstanceState);
            setContentView(R.layout.activity_main);
    
            findViewById(R.id.conn_service).setOnClickListener(new View.OnClickListener() {
                @Override
                public void onClick(View v) {
                    Intent intent = new Intent(MainActivity.this, MessengerService.class);
                    bindService(intent, mServiceConnection, BIND_AUTO_CREATE);
                }
            });
        }
    
        @Override
        protected void onDestroy() {
            unbindService(mServiceConnection);
            super.onDestroy();
        }
    
    
    }
    
    ```

    

  - 服务端

    ```java
    public class MessengerService extends Service {
    
    
        private static final String TAG = "MessengerService";
    
        public MessengerService() {
        }
    
        public static class MessengerHandler extends Handler {
            @Override
            public void handleMessage(@NonNull Message msg) {
                switch (msg.what) {
                    case 10001: {
                        String client_message = msg.getData().getString("client_message");
                        Log.d(TAG, "client " + client_message);
    
                        //当收到客户端发来的消息的时候，给客户端一个回应。
                        //获取到客户端的Messenger
                        Messenger clientMessenger = msg.replyTo;
                        Message serviceMessage = Message.obtain();
                        serviceMessage.what = 10002;
                        Bundle bundle = new Bundle();
                        bundle.putString("service_message","ok,receive");
                        serviceMessage.setData(bundle);
    
                        try {
                            clientMessenger.send(serviceMessage);
                        } catch (RemoteException e) {
                            e.printStackTrace();
                        }
                    }
                    break;
    
                    default:
                        super.handleMessage(msg);
                }
    
            }
        }
    
        //服务端的Messenger
        private Messenger serviceMessenger = new Messenger(new MessengerHandler());
    
        @Override
        public IBinder onBind(Intent intent) {
            // TODO: Return the communication channel to the service.
            return serviceMessenger.getBinder();
        }
    }
    
    ```

  - Messenger的工作原理图

    ![Messenger原理图.png](https://upload-images.jianshu.io/upload_images/4997216-ae9798e7f8652578.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

  

###### 使用AIDL

**在AIDL文件中支持的数据类型**

- 基本数据类型
- String和CharSequence
- List：仅仅支持ArrayList，里面的每个元素必须能够被AIDL支持
- Map：仅仅支持HashMap，里面每个元素必须被AIDL支持，包括key和value
- Parcelable：所有实现Parcelable接口的对象
- AIDL：所有AIDL接口本身也可以在AIDL文件中使用

**使用AIDL的注意**

- 自定义的Parcelable对象和AIDL对象必须要显式import进来。

- AIDL文件中用到了自定义的Parcelable对象，必须新建一个和它同名的AIDL文件，并在其中声明它为Parcelable类型。（上文中IBookManger.aidl中使用到了Book这个类 ，我们新建了Book.aidl,然后在里面添加如下代码）

  ```java
  // Book.aidl.aidl
  package com.dashingqi.aidlproject.aidl;
  
  parcelable Book
  
  ```

- 除了基本数据类型，其他类型的必须标上方向：in（输入类参数）、out（输出型参数）、inout（输入输出型）

- 为了开发方便，建议把所有和AIDL相关的类和文件放在同一个包中。

**具体代码实现**

- 远程服务端Service的实现

  ```java
  public class BookManageService extends Service {
      public BookManageService() {
      }
  
      /**
       * 支持并发读/写
       * AIDL文件中的方法是在Binder线程池中执行，需要处理并发读/写
       */
      private CopyOnWriteArrayList mCopyOnWriteArrayList = new CopyOnWriteArrayList<Book>();
  
      private Binder mBinder = new IBookManager.Stub() {
  
          @Override
          public void basicTypes(int anInt, long aLong, boolean aBoolean, float aFloat, double aDouble, String aString) throws RemoteException {
  
          }
  
          @Override
          public void addBook(Book book) throws RemoteException {
              mCopyOnWriteArrayList.add(book);
  
          }
  
          @Override
          public List<Book> getBookList() throws RemoteException {
              return mCopyOnWriteArrayList;
          }
      };
  
      @Override
      public IBinder onBind(Intent intent) {
          return mBinder;
      }
  }
  ```

- 客户端的实现

  ```java
  public class MainActivity extends AppCompatActivity {
  
      private static final String TAG = "MainActivity";
  
      private ServiceConnection mServiceConnection = new ServiceConnection() {
          @Override
          public void onServiceConnected(ComponentName name, IBinder service) {
              IBookManager iBookManager = IBookManager.Stub.asInterface(service);
  
              try {
                  iBookManager.addBook(new Book(1, "Android开发艺术探索"));
  
                  List<Book> bookList = iBookManager.getBookList();
  
                  Log.d(TAG, "book list : " + bookList.toString());
  
              } catch (RemoteException e) {
                  e.printStackTrace();
              }
  
          }
  
          @Override
          public void onServiceDisconnected(ComponentName name) {
  
          }
      };
  
      @Override
      protected void onCreate(Bundle savedInstanceState) {
          super.onCreate(savedInstanceState);
          setContentView(R.layout.activity_main);
  
          Intent intent = new Intent(MainActivity.this, BookManageService.class);
          bindService(intent, mServiceConnection, BIND_AUTO_CREATE);
      }
  
      @Override
      protected void onDestroy() {
          unbindService(mServiceConnection);
          super.onDestroy();
      }
  }
  
  ```

  ###### RemoteCallBackList

  - 对象本质是不能夸进程通信的，实质都是反序列的过程

  - 是系统专门提供的用于删除跨进程listener的接口，是一个泛型，支持管理任意的IInterface接口。

  - 内部由ArrayMap来管理,其中key是IBinder，value是Callback 内部封装了真正远程的listener。

    ``````java
    ArrayMap<IBinder, Callback> mCallbacks
                = new ArrayMap<IBinder, Callback>();
    //	其中 key 和 value 分别这样获取
    IBinder binder = callback.asBinder();
    Callback cb = new Callback(callback, cookie);
    ``````

  - 虽然在夸进程中传递的listener会产生两个不同的对象，但是底层对应的Binder是同一个，所以在解注册只需要找到对应相同的binder对象，然后从中移除即可达到解注册的效果。

  - 客户端进程终止后，它能够自动移除客户端所注册的listener。

  - 内部做了线程同步的工作。



	###### 避免ANR

**客户端**

- 客户端调用远程服务端的方法，该方法是运行在服务端的Binder线程池中，客户端线程会被挂起，如果这时候远程的方法是非常耗时的，那么就会导致客户端长时间阻塞在这里，如果是在UI线程中调用会引用ANR。
- ServiceConnection中实现的方法是在UI线程中运行，所以调用远程服务端方法要注意。
- 解决也就是把客户端的调用放在非UI线程中执行，此时需要更新UI的话，需要切换到UI线程上。

**远程服务端**

- 同理，服务端调用客户端的方法是运行在客户端的Binder线程池中，要保证调用客户端耗时的方法不能运行在UI线程中。

###### Binder意外死亡，重新连接服务

- 在 onServiceDisconected()方法中做重连操作

- 给Binder设置DeathRecipient()监听，当binder死亡的时候会收到binderDied方法的回调。

  ```java
   private IBinder.DeathRecipient mDeathRecipient = new IBinder.DeathRecipient() {
          @Override
          public void binderDied() {
              mIBinderPool.asBinder().unlinkToDeath(mDeathRecipient,1);
              
              mIBinderPool = null;
              
              connectionBinderPoolService();
              
  
          }
      };
      
       try {
                  mIBinderPool.asBinder().linkToDeath(mDeathRecipient,0);
              } catch (RemoteException e) {
                  e.printStackTrace();
              }
  
  ```

  

###### 连接远程服务权限校验

- onBind()进行权限的验证

  ```java
  //采用Permission，在AndroidMenifest中声明所需的权限
  <permission
          android:name="com.dashingqi.check_permission"
          android:protectionLevel="normal" />
  
  //当一个应用绑定到我们的服务中，需要在它的AndroidMenifest文件中采用乳腺癌方式使用permission
   <uses-permission android:name="com.dashingqi.check_permission"/>         
  
  ```

  

- onTransact()方法中进行权限验证

- service的permission属性

###### 使用ContentProvider

###### 使用Socket

## Binder连接池

###### AIDL的流程回顾

- 远程服务端：首先需要创建一个Service和一个AIDL接口文件，然乎创建一个extends至AIDL文件中的Stub类并且实现相关的抽象方法，然后在Service的onBind()方法中返回这个类的对象。
- 客户端：就可以调用bindService()来去绑定远程服务了，获取到binder对象，与远程服务进行通信。

##### 问题分析

- 项目中有100个业务中需要使用进程间通信，难道需要创建100个service？
- Service是Android中的四大组件之一，是系统的一种资源。
- 太多的Service会使得应用看起来很重量级。
- 我们需要减少Service，将AIDL放到一个Service中去管理。

###### 具体流程

> BinderPool 在多个AIDL文件的情况下由一个Service管理 

- 多个AIDL文件

  ```java
  //提供加密解密
  interface ISecurityCenter {
  
  
      String encrypt(String content);
  
      String decrypt(String password);
  }
  
  //提供加法算法
  interface ICompute {
  
     int add(int a,int b);
  
  }
  
  ```

- 为Binder连接池创建AIDL接口文件,为Binder连接池创建远程Service，创建BinderPool

  ```java
  interface IBinderPool {
  
     IBinder queryBinder(int binderCode);
  }
  
  // BinderPool
  public class BinderPool {
      private static final String TAG = "BinderPool";
  
      private Context mContext;
      private static BinderPool mInstance;
      private IBinderPool mIBinderPool;
  
      private BinderPool(Context context) {
          mContext = context.getApplicationContext();
          connectionBinderPoolService();
      }
  
      public static BinderPool getInstance(Context context) {
          if (mInstance == null) {
              synchronized (BinderPool.class) {
                  if (mInstance == null) {
                      mInstance = new BinderPool(context);
                  }
              }
          }
  
          return mInstance;
      }
  
  
      private synchronized void connectionBinderPoolService() {
          Log.d(TAG, "connectionBinderPoolService: ");
        	mCountDownLatch = new CountDownLatch(1);
          Intent intent = new Intent(mContext, BinderPoolService.class);
          mContext.bindService(intent, mBinderPoolSerCon, Context.BIND_AUTO_CREATE);
        
        try {
              //客户端线程挂起
              mCountDownLatch.await();
          } catch (InterruptedException e) {
              e.printStackTrace();
          }
      }
  
  
      private ServiceConnection mBinderPoolSerCon = new ServiceConnection() {
          @Override
          public void onServiceConnected(ComponentName name, IBinder service) {
  
              Log.d(TAG, "onServiceConnected: ");
              mIBinderPool = IBinderPool.Stub.asInterface(service);
            
            //唤醒客户端线程
              mCountDownLatch.countDown();
  
          }
  
          @Override
          public void onServiceDisconnected(ComponentName name) {
  
          }
      };
  
  
      public IBinder queryBinder(int queryCode) {
          IBinder iBinder = null;
  
          try {
              if (mIBinderPool != null) {
                  iBinder = mIBinderPool.queryBinder(queryCode);
              }
          } catch (RemoteException e) {
              e.printStackTrace();
          }
  
          return iBinder;
  
      }
  
      public static class MyBinderPool extends IBinderPool.Stub {
  
          @Override
          public IBinder queryBinder(int binderCode) throws RemoteException {
              IBinder mIBinder = null;
              switch (binderCode) {
                  case 1: {
                      mIBinder = new ISecurityCenterImpl();
  
                  }
                  break;
                  case 2: {
                      mIBinder = new IComputeImpl();
                  }
                  break;
              }
  
              return mIBinder;
          }
      }
  }
  
  // BinderPoolService 
  public class BinderPoolService extends Service {
      public BinderPoolService() {
      }
  
      private Binder mBinderPool = new BinderPool.MyBinderPool();
  
      @Override
      public IBinder onBind(Intent intent) {
          return mBinderPool;
      }
  }
  ```

- 调用

  ```java
   new Thread(new Runnable() {
              @Override
              public void run() {
                  work();
              }
          }).start();
  
  /**
       * 需要在客户端的子线程中调用这个方法
       * 调用服务端的方法有可能是耗时的，
       * bindService()从异步变成同步
       * 建议在子线程中调用。
       */
      private void work() {
          //获取到BinderPool对象，同时连接到远程服务上
          mBinderPool = BinderPool.getInstance(MainActivity.this);
  
          if (mBinderPool != null) {
  
              //根据code 获取对应Binder对象
              IBinder securityCenterBinder = mBinderPool.queryBinder(1);
  
              ISecurityCenter iSecurityCenter = ISecurityCenterImpl.Stub.asInterface(securityCenterBinder);
              try {
                  String dashingqi = iSecurityCenter.decrypt("dashingqi");
                  Log.d(TAG, "work: " + dashingqi);
  
                  //iSecurityCenter.encrypt("zhangqi");
              } catch (RemoteException e) {
                  e.printStackTrace();
              }
  
          }
      }
  ```

  