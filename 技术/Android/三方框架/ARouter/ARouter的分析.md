

## ARouter的分析

- arouter-annotation :ARouter路由框架所使用的全部注解，及其相关类
- arouter-compiler:注解处理器，在编译期间把注解相关的目标类生成映射文件。
- arouter-api:实现路由控制
- 总的说来就是 arouter-annotations实现了路由表结构的定义，arouter-compiler在编译期间完成了路由表逻辑的创建，arouter-api在运行期间加载装载路由表逻辑，实现路由的控制



###### 页面注册

- 注解处理器扫描出被标注的类文件
- 按照不同种类的源文件进行分类
- 按照固定的命名格式生成映射文件
- 初始化的时候通过固定的包名加载映射文件

#### ARouter中的注解

###### @Route路由注解

- 代码如下

  ```java
  @Target({ElementType.TYPE})
  @Retention(RetentionPolicy.CLASS)
  public @interface Route {
  
      /**
       * 路径的地址
       */
      String path();
  
      /**
       * 组名，默认为以及路径名字；
       */
      String group() default "";
  
      /**
       * 该路径的名字，用于产生JavaDoc
       */
      String name() default "";
  
      /**
       * 额外配置开关信息，譬如某些页面是否需要网络教研、登录教研
       */
      int extras() default Integer.MIN_VALUE;
  
      /**
       * 该路径的优先级
       */
      int priority() default -1;
  }
  ```

###### @Interceptor拦截器注解

- 代码如下

  ```java
  @Target({ElementType.TYPE})
  @Retention(RetentionPolicy.CLASS)
  public @interface Interceptor {
      /**
       *该拦截器的优先级
       */
      int priority();
  
      /**
       * 该拦截器的名称，用于产生JavaDOC
       */
      String name() default "Default";
  }
  
  ```

###### @Autowired自动装载注解

- 该注解是用于页面跳转的时候传递参数用的。目标页面中，使用该注解标志变量，同时需要调用inject()，在打开页面的时候会自动赋值的

- 代码如下

  ```java
  @Target({ElementType.FIELD})
  @Retention(RetentionPolicy.CLASS)
  public @interface Autowired {
  
      // Mark param's name or service name.
      String name() default "";
  
      // If required, app will be crash when value is null.
      // Primitive type wont be check!
      boolean required() default false;
  
      // Description of the field
      String desc() default "";
  }
    
  ```

#### aroute-compiler注解编译器

###### 一般来说，arouter-compiler所产生的class文件有以下几种

- routes目录下

  ```xml
  1. 工程名$$Group$$分组名【组内的路由清单列表】
  		- 该文件中保存了 路由Url与目标Class的映射关系
  		- 如果@Route注解中没有指定Group，就拿路由地址的一级地址
  2. 工程名$$Root$$模块名【组的清单】
  		- 保存组名与 【工程名$$Group$$分组名】的映射关系
  		- ARouter在初始化的时候会一次性地加载所有root节点，而不会加载任何一个Group节点，这样就会极大地降低初始化时加载节点的数量
  		- 当某一个分组下的某一个页面第一次被访问的时候，整个分组的全部页面都会被加载进去，这就是ARouter的按需加载
  3. 工程名$$Providers$$模块名【Ioc的动作路由清单列表】
  	- provider类型的路由节点的清单列表
  	- 保存了实现IProviders接口的直接子类的Class的路由URL与IPrviders接口子类的全类名
  4. 工程名$$Interceptors$$模块名
  	- 保存了某个模块下拦截器与优先级映射的关系
  ```

- 针对实现IProviders接口的直接子类（Ioc），获取方式

  - ARouter.getInstance().navigation(TestProviders::class.java).sayHello()
  - ARouter.getInstance().build("/provider/test").navigation().sayHello()

#### ARouter的初始化流程

> 通常我都会在应用启动的入口Application的onCreate()方法中调用init()方法进行ARouter的初始化，我们看下init()发生了什么。

###### init()

- 代码如下

  ```java
  public static void init(Application application) {
    			
          if (!hasInit) {
            	// 持有日志打印的全局静态变量
              logger = _ARouter.logger;
            	// 打印ARouter初始化日志
              _ARouter.logger.info(Consts.TAG, "ARouter init start.");
            	// 分析 ----> 1
              hasInit = _ARouter.init(application);
  
              if (hasInit) {
                  _ARouter.afterInit();
              }
  						// 打印ARouter初始化完成日志
              _ARouter.logger.info(Consts.TAG, "ARouter init over.");
          }
    }
  ```

  - 分析1：如果没有进行过初始化，在内部调用了 _ARouter.init()
  
  ```java
  // _ARouter.init()方法
  protected static synchronized boolean init(Application application) {
    			//初始化上下文环境
          mContext = application;
    			// 分析 ------> 2
          LogisticsCenter.init(mContext, executor);
          logger.info(Consts.TAG, "ARouter init success!");
    			//标示是否初始化完成
          hasInit = true;
    			//创建一个与主线程Looper绑定的handler对象
          mHandler = new Handler(Looper.getMainLooper());
  
        return true;
      }
  ```
```
- 分析2：可以推断出，LogisticsCenter.init(mContext, executor);方法中是初始化的核心，我们看下
  
  ```java
  public synchronized static void init(Context context, ThreadPoolExecutor tpe) throws HandlerException {
    			//Application的上下文
          mContext = context;
    			//线程池
          executor = tpe;
  
          try {
              long startInit = System.currentTimeMillis();
              //billy.qi modified at 2017-12-06
              //load by plugin first
              loadRouterMap();
              if (registerByPlugin) {
                  logger.info(TAG, "Load router map by arouter-auto-register plugin.");
              } else {
                  Set<String> routerMap;
  
                  // debug版本或者版本更新的时候，将会重新加载router信息
                  if (ARouter.debuggable() || PackageUtils.isNewVersion(context)) {
                      logger.info(TAG, "Run with debug mode or new install, rebuild router map.");
                      // 通过指定包名，扫描包下面包含的所有的ClassName
                    	// 也就是加载 com.alibaba.android.arouter.routes 包下的
                      routerMap = ClassUtils.getFileNameByPackageName(mContext, ROUTE_ROOT_PAKCAGE);
                      if (!routerMap.isEmpty()) {
                          context.getSharedPreferences(AROUTER_SP_CACHE_KEY, Context.MODE_PRIVATE).edit().putStringSet(AROUTER_SP_KEY_MAP, routerMap).apply();
                      }
  
                      PackageUtils.updateVersion(context);    // Save new version name when router map update finishes.
                  } else {
                      logger.info(TAG, "Load router map from cache.");
                      routerMap = new HashSet<>(context.getSharedPreferences(AROUTER_SP_CACHE_KEY, Context.MODE_PRIVATE).getStringSet(AROUTER_SP_KEY_MAP, new HashSet<String>()));
                  }
  
                  logger.info(TAG, "Find router map finished, map size = " + routerMap.size() + ", cost " + (System.currentTimeMillis() - startInit) + " ms.");
                  startInit = System.currentTimeMillis();
  								// 循环遍历从集合中拿出包下的类名
                  for (String className : routerMap) {
              
                      if (className.startsWith(ROUTE_ROOT_PAKCAGE + DOT + SDK_NAME + SEPARATOR + SUFFIX_ROOT)) {
                       		//存储路中一级路径的映射关系 （第一次只会加载一级路径对应的映射关系）
                          ((IRouteRoot) (Class.forName(className).getConstructor().newInstance())).loadInto(Warehouse.groupsIndex);
                      } else if (className.startsWith(ROUTE_ROOT_PAKCAGE + DOT + SDK_NAME + SEPARATOR + SUFFIX_INTERCEPTORS)) {
                          // 存储拦截器的映射关系啊
                          ((IInterceptorGroup) (Class.forName(className).getConstructor().newInstance())).loadInto(Warehouse.interceptorsIndex);
                      } else if (className.startsWith(ROUTE_ROOT_PAKCAGE + DOT + SDK_NAME + SEPARATOR + SUFFIX_PROVIDERS)) {
                          // 存储实现Providers直接子类的映射关系
                          ((IProviderGroup) (Class.forName(className).getConstructor().newInstance())).loadInto(Warehouse.providersIndex);
                      }
                  }
              }
  
              logger.info(TAG, "Load root element finished, cost " + (System.currentTimeMillis() - startInit) + " ms.");
  
              if (Warehouse.groupsIndex.size() == 0) {
                  logger.error(TAG, "No mapping files were found, check your configuration please!");
              }
  
              if (ARouter.debuggable()) {
                  logger.debug(TAG, String.format(Locale.getDefault(), "LogisticsCenter has already been loaded, GroupIndex[%d], InterceptorIndex[%d], ProviderIndex[%d]", Warehouse.groupsIndex.size(), Warehouse.interceptorsIndex.size(), Warehouse.providersIndex.size()));
            }
          } catch (Exception e) {
              throw new HandlerException(TAG + "ARouter init logistics center exception! [" + e.getMessage() + "]");
          }
      }
```


#### ARouter页面的跳转流程

###### 实例代码

```java
ARouter.getInstance().build("/test/page").navigation()
```

###### 流程分析

- ARouter # getInstance()

  ```java
  public static ARouter getInstance() {
    // 检查ARouter是否经过初始化
          if (!hasInit) {
              throw new InitException("ARouter::Init::Invoke init(context) first!");
          } else {
            // 双重检查模式创建ARouter的实例
              if (instance == null) {
                  synchronized (ARouter.class) {
                      if (instance == null) {
                          instance = new ARouter();
                      }
                  }
              }
              return instance;
          }
      }
  ```

- ARouter # build()

  ```java
  // 返回一个Postcard类型的对象 
  public Postcard build(String path) {
     			// 内部调用了 _ARouter 的一系列方法
          return _ARouter.getInstance().build(path);
      }
  
  // _ARouter好比是一个代理类 ARouter的全部逻辑都是交由_ARouter来执行
  ```

- _ARouter # getInstance() 类比 ARouter # getInstance() 不做分析了

- _ARouter $ build()

  ```java
      /**
       * Build postcard by path and default group
       */
  protected Postcard build(String path) {
    		//当传如的路径为null 就抛出异常
       if (TextUtils.isEmpty(path)) {
           throw new HandlerException(Consts.TAG + "Parameter is invalid!");
        } else {
         		// PathReplaceService 是继承 IProviders
         		// 如果全局有实现了PathReplaceService 则执行 “运行期动态修改路由”逻辑。生成转换后的路由
            PathReplaceService pService = ARouter.getInstance().navigation(PathReplaceService.class);//注释1
            if (null != pService) {
                path = pService.forString(path);
            }
         		// 执行正常的路由逻辑执行
         		// extractGroup() 函数就是用来获取一级路径并且校验一级路径
            return build(path, extractGroup(path));
        }
   }
  
  private String extractGroup(String path) {
    			// 验证传递的path
          if (TextUtils.isEmpty(path) || !path.startsWith("/")) {
              throw new HandlerException(Consts.TAG + "Extract the default group failed, the path must be start with '/' and contain more than 2 '/'!");
          }
  
          try {
            	// 拿到路由的一级路径
              String defaultGroup = path.substring(1, path.indexOf("/", 1));
            	// 判断这个一级路径
              if (TextUtils.isEmpty(defaultGroup)) {
                  throw new HandlerException(Consts.TAG + "Extract the default group failed! There's nothing between 2 '/'!");
              } else {
                  return defaultGroup;
              }
          } catch (Exception e) {
              logger.warning(Consts.TAG, "Failed to extract default group! " + e.getMessage());
              return null;
          }
      }
  ```

- ARouter.getInstance().navigation(PathReplaceService.class); ----> _ARouter.getInstance().navigation(PathReplaceService.class);

- 看下_ARouter # navigation() （注释1）

  ```java
  protected <T> T navigation(Class<? extends T> service) {
          try {
            	// 执行了buildProvider
              Postcard postcard = LogisticsCenter.buildProvider(service.getName());
  
              // Compatible 1.0.5 compiler sdk.
              // Earlier versions did not use the fully qualified name to get the service
              if (null == postcard) {
                  // No service, or this service in old version.
                  postcard = LogisticsCenter.buildProvider(service.getSimpleName());
              }
  
              if (null == postcard) {
                  return null;
              }
  
              LogisticsCenter.completion(postcard);
              return (T) postcard.getProvider();
          } catch (NoRouteFoundException ex) {
              logger.warning(Consts.TAG, ex.getMessage());
              return null;
          }
      }
  	
  // LogisticsCenter # buildProvider() 
      public static Postcard buildProvider(String serviceName) {
        	// 从仓库的 Ioc路由清单列表中 拿到存储的RouteMeata数据
          RouteMeta meta = Warehouse.providersIndex.get(serviceName);
  				// 为 null 说明没有实现PathReplaceService接口的类
          if (null == meta) {
              return null;
          } else {
            	// 有的话 组装一个Postcard对象返回
              return new Postcard(meta.getPath(), meta.getGroup());
          }
      }
  ```

- 看下 _ARouter # build(String path,String group)

  ```java
  protected Postcard build(String path, String group) {
          if (TextUtils.isEmpty(path) || TextUtils.isEmpty(group)) {
              throw new HandlerException(Consts.TAG + "Parameter is invalid!");
          } else {
              PathReplaceService pService = ARouter.getInstance().navigation(PathReplaceService.class);
              if (null != pService) {
                  path = pService.forString(path);
              }
            	// 构建一个新的Postcard对象
              return new Postcard(path, group);
          }
      }
  ```

- ARouter # build()返回的是一个 Postcard对象，调用了Postcard # navigation()方法 看下 Postcard # navigation()

  - Postcard类是继承RouteMeta
  - RouteMeta在编译期，通常作为value存储在map中，比如我们组中的路由清单中文件中

  ```java
      public Object navigation() {
          return navigation(null);
      }
  
      public Object navigation(Context context) {
          return navigation(context, null);
      }
  
      public Object navigation(Context context, NavigationCallback callback) {
          return ARouter.getInstance().navigation(context, this, -1, callback);
      }
  ```

  - 最终还是调用了ARouter # navigation()方法

- ARouter # navigation()

  ```java
  public Object navigation(Context mContext, Postcard postcard, int requestCode, NavigationCallback callback) {
          return _ARouter.getInstance().navigation(mContext, postcard, requestCode, callback);
      }
  ```

  - 同样不难猜出 调用了_ARouter # navigation()

- _ARouter # navigation()

  ```java
   protected Object navigation(final Context context, final Postcard postcard, final int requestCode, final NavigationCallback callback) {
          PretreatmentService pretreatmentService = ARouter.getInstance().navigation(PretreatmentService.class);
          if (null != pretreatmentService && !pretreatmentService.onPretreatment(context, postcard)) {
              // Pretreatment failed, navigation canceled.
              return null;
          }
  
          try {
            	//完善Postcard 当前只有path和group
              LogisticsCenter.completion(postcard);
          } catch (NoRouteFoundException ex) {
              logger.warning(Consts.TAG, ex.getMessage());
  
              if (debuggable()) {
                  // Show friendly tips for user.
                  runInMainThread(new Runnable() {
                      @Override
                      public void run() {
                          Toast.makeText(mContext, "There's no route matched!\n" +
                                  " Path = [" + postcard.getPath() + "]\n" +
                                  " Group = [" + postcard.getGroup() + "]", Toast.LENGTH_LONG).show();
                      }
                  });
              }
  
              if (null != callback) {
                //执行到这里，触发查找失败
                  callback.onLost(postcard);
              } else {
                  //执行到这里，使用IOC.byType()的方式 全局降级策略的实现
                  DegradeService degradeService = ARouter.getInstance().navigation(DegradeService.class);
                  if (null != degradeService) {
                      degradeService.onLost(context, postcard);
                  }
              }
  
              return null;
          }
  
          if (null != callback) {
            	// 执行到这里说明找到了路由的信息，触发路由查找的回调
              callback.onFound(postcard);
          }
  				
     			// 绿色通道校验
          if (!postcard.isGreenChannel()) {  // It must be run in async thread, maybe interceptor cost too mush time made ANR.
            	// 调用拦截器的方法
              interceptorService.doInterceptions(postcard, new InterceptorCallback() {
                  /**
                   * Continue process
                   *
                   * @param postcard route meta
                   */
                  @Override
                  public void onContinue(Postcard postcard) {
                    	//根据路由类型执行具体的路由逻辑
                      _navigation(context, postcard, requestCode, callback);
                  }
  
                  @Override
                  public void onInterrupt(Throwable exception) {
                      if (null != callback) {
                          callback.onInterrupt(postcard);
                      }
  
                      logger.info(Consts.TAG, "Navigation failed, termination by interceptor : " + exception.getMessage());
                  }
              });
          } else {
            	// 根据路由类型执行具体的路由逻辑
              return _navigation(context, postcard, requestCode, callback);
          }
  
          return null;
      }
  ```

- _ARouter # navigation() 路由的具体执行

  ```java
   private Object _navigation(final Context context, final Postcard postcard, final int requestCode, final NavigationCallback callback) {
          final Context currentContext = null == context ? mContext : context;
  
          switch (postcard.getType()) {
              // 当是Activity类型的。就拼装Intent，然后切换线程到UI线程中进行跳转
              case ACTIVITY:
                  // Build intent
                  final Intent intent = new Intent(currentContext, postcard.getDestination());
                  intent.putExtras(postcard.getExtras());
  
                  // Set flags.
                  int flags = postcard.getFlags();
                  if (-1 != flags) {
                      intent.setFlags(flags);
                  } else if (!(currentContext instanceof Activity)) {    // Non activity, need less one flag.
                      intent.setFlags(Intent.FLAG_ACTIVITY_NEW_TASK);
                  }
  
                  // Set Actions
                  String action = postcard.getAction();
                  if (!TextUtils.isEmpty(action)) {
                      intent.setAction(action);
                  }
  
                  // Navigation in main looper.
                  runInMainThread(new Runnable() {
                      @Override
                      public void run() {
                          startActivity(requestCode, currentContext, intent, postcard, callback);
                      }
                  });
  
                  break;
              case PROVIDER:
              		//如果是Ioc，就返回对应的实例
                  return postcard.getProvider();
              case BOARDCAST:
              case CONTENT_PROVIDER:
              case FRAGMENT:
              		
                  Class fragmentMeta = postcard.getDestination();
                  try {
                    	// 通过反射获取到实例
                      Object instance = fragmentMeta.getConstructor().newInstance();
                    	// 如果是Fragement类型的，获取一下Bundle，并设置给Fragment
                      if (instance instanceof Fragment) {
                          ((Fragment) instance).setArguments(postcard.getExtras());
                      } else if (instance instanceof android.support.v4.app.Fragment) {
                          ((android.support.v4.app.Fragment) instance).setArguments(postcard.getExtras());
                      }
  										// 并且返回这个实例
                      return instance;
                  } catch (Exception ex) {
                      logger.error(Consts.TAG, "Fetch fragment instance error, " + TextUtils.formatStackTrace(ex.getStackTrace()));
                  }
              case METHOD:
              case SERVICE:
              default:
                  return null;
          }
  
          return null;
      }
  ```

  - 到这里ARouter的基本跳转就完成了。

###### 说下ARouter中的Postcard和RouteMeta

> Postcard是继承 RouteMeta的

- Postcard

  ```java
  public final class Postcard extends RouteMeta {
      // Base
      private Uri uri;                // 当以Uri形式的时候才有值
      private Object tag;             // 拦截器中出现错误的tag
      private Bundle mBundle;         // bundle中存储着用于传递的数据
      private int flags = -1;         // 用于存储Intent要设置的flag
      private int timeout = 300;      // 拦截器消耗时间的阈值
      private IProvider provider;     // 当是IProvider类型的时候，该值对应着实现的直接子类
      private boolean greenChannel;   // 是否跳过拦截器
      private SerializationService serializationService; // 用于对象的json转换的工具，在构建参数withObject是会被ByType方式赋值进来
  
      // Animation 转场动画
      private Bundle optionsCompat;    // The transition animation of activity
      private int enterAnim = -1; 
      private int exitAnim = -1;
  }
  ```

- RouteMeta

  ```java
  public class RouteMeta {
      private RouteType type;         // 类型 路由的类型 可能是Activity，可能是Fragment
      private Element rawType;        // Raw type of route
      private Class<?> destination;   // 目标Clas
      private String path;            // 路由地址
      private String group;           // 路由的组（默认对应着一级地址）
      private int priority = -1;      // 路由的等级
      private int extra;              // 目标页面的一个配置项，比如该页面需要登录后才能打开 一般配合着拦截器使用
      private Map<String, Integer> paramsType;  // 目标页面需要注入的参数的参数名称
      private String name;
      private Map<String, Autowired> injectConfig; 
    }
  
  // RouteType 路由类型 是一个枚举类型
  public enum RouteType {
      ACTIVITY(0, "android.app.Activity"),
      SERVICE(1, "android.app.Service"),
      PROVIDER(2, "com.alibaba.android.arouter.facade.template.IProvider"),
      CONTENT_PROVIDER(-1, "android.app.ContentProvider"),
      BOARDCAST(-1, ""),
      METHOD(-1, ""),
      FRAGMENT(-1, "android.app.Fragment"),
      UNKNOWN(-1, "Unknown route type");
  }
  ```

###### ARouter的IoC与依赖注入

- 在编译期间扫描出需要自动装配的字段
- 将自动装配的字段注册到映射的文件中
- 跳转的时候按照预先配置从URL中提取参数并按照类型放入到Intent中
- 目标页面在初始化的时候调用ARouter.getInstance().inject(this)
- ARouter会查找在编译期为调用调用方法生成的注入辅助类
- 实例化辅助类后，调用其中的inject方法完成字段的赋值。