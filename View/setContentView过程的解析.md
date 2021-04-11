## 本文主要从如下几点学习LayoutInflater

- LayoutInflater是啥
- LayoutInflater的获取
- LayoutInflater的inflate方法
- 总结

## LayoutInflater是啥

###### 源码定义

```java
Instantiates a layout XML file into its corresponding {@link android.view.View}
objects. It is never used directly. Instead, use
{@link android.app.Activity#getLayoutInflater()} or
{@link Context#getSystemService} to retrieve a standard LayoutInflater instance
that is already hooked up to the current context and correctly configured
for the device you are running on.

// 说的意思就是
  
 LayoutInflater是将XML布局文件实例化为View的对象。不要单独使用它，需要使用
 Activity.getLayoutInflater 或者使用 Context.getSystemService()来获取于
 当前Context相关联并且正确配置的LayoutInflater
```

- 一般我们叫它布局加载器

## LayutInflater的获取

- Activity.getLayoutInflater()
- Context.getSystemService(Context.LAYOUT_INFLATER_SERVICE)
- LayoutInflater.from(Context)

###### 这三种方式的对比

- Activity.getLayoutInflater()方法最终是调用了PhoneWindow中的getLayoutInflater的方法，PhoneWindow的构造方法中，是使用方式三来进行LayoutInflater的获取，所以方式一是调用了方式三来获取LayoutInflater对象的。
- 我们点开LayoutInflater.from方法我们会看见又调用了方式二获取到了LayoutInflater对象。

## LayoutInflater的inflate方法

> LayoutInflater类中有多个重载的inflate方法，但是这些方法最终都会调用下面这个方法

- 代码如下

  ```java
  public View inflate(XmlPullParser parser, @Nullable ViewGroup root, boolean attachToRoot) {
          synchronized (mConstructorArgs) {
              Trace.traceBegin(Trace.TRACE_TAG_VIEW, "inflate");
  
              final Context inflaterContext = mContext;
              //获取到Xml文件中属性信息集合
              final AttributeSet attrs = Xml.asAttributeSet(parser);
              Context lastContext = (Context) mConstructorArgs[0];
              mConstructorArgs[0] = inflaterContext;
              View result = root;
  
              try {
                  // 用来检查xml文件的根布局是否合理
                  advanceToRootNode(parser);
                  final String name = parser.getName();
  
                  if (DEBUG) {
                      System.out.println("**************************");
                      System.out.println("Creating root view: "
                              + name);
                      System.out.println("**************************");
                  }
  
                  // 如果是Merge标签的话，必须依附一个RootView，否则的话就抛出异常
                  if (TAG_MERGE.equals(name)) {
                      if (root == null || !attachToRoot) {
                          throw new InflateException("<merge /> can be used only with a valid "
                                  + "ViewGroup root and attachToRoot=true");
                      }
  	
                      rInflate(parser, root, inflaterContext, attrs, false);
                  } else {
                      // Temp is the root view that was found in the xml
                      //如果不是Merge标签，就通过标签的名字创建一个View对象
                      final View temp = createViewFromTag(root, name, inflaterContext, attrs);
  
                      ViewGroup.LayoutParams params = null;
  
                      if (root != null) {
                          if (DEBUG) {
                              System.out.println("Creating params from root: " +
                                      root);
                          }
                          // Create layout params that match root, if supplied
                          //如果Root不为null，那么根据标签的参数生成LayoutParams
                          params = root.generateLayoutParams(attrs);
                          if (!attachToRoot) {
                              // Set the layout params for temp if we are not
                              // attaching. (If we are, we use addView, below)
                              //如果不是attachToRoot 那么就将这个Tag创建出来的View设置LayoutParams
                              temp.setLayoutParams(params);
                          }
                      }
  
                      if (DEBUG) {
                          System.out.println("-----> start inflating children");
                      }
  
                      // Inflate all children under temp against its context.
                      rInflateChildren(parser, temp, attrs, true);
  
                      if (DEBUG) {
                          System.out.println("-----> done inflating children");
                      }
  
                      // We are supposed to attach all the views we found (int temp)
                      // to root. Do that now.
                      //如果root不为null并且是attachToRoot那么就将创建好的View添加到Root中
                      if (root != null && attachToRoot) {
                          root.addView(temp, params);
                      }
  
                      // Decide whether to return the root that was passed in or the
                      // top view found in xml.
                      if (root == null || !attachToRoot) {
                          result = temp;
                      }
                  }
  
              } catch (XmlPullParserException e) {
                  final InflateException ie = new InflateException(e.getMessage(), e);
                  ie.setStackTrace(EMPTY_STACK_TRACE);
                  throw ie;
              } catch (Exception e) {
                  final InflateException ie = new InflateException(
                          getParserStateDescription(inflaterContext, attrs)
                          + ": " + e.getMessage(), e);
                  ie.setStackTrace(EMPTY_STACK_TRACE);
                  throw ie;
              } finally {
                  // Don't retain static reference on context.
                  mConstructorArgs[0] = lastContext;
                  mConstructorArgs[1] = null;
  
                  Trace.traceEnd(Trace.TRACE_TAG_VIEW);
              }
  
              return result;
          }
      }
  ```

- infalte方法总结
  - 如果是Merge标签，必须要依附于一个RootView，之后调用了rInflate方法
  - 否则的话就调用了createViewFromTag()创建View的对象。

###### rInflate方法

- 代码如下

  ```java
  void rInflate(XmlPullParser parser, View parent, Context context,
              AttributeSet attrs, boolean finishInflate) throws XmlPullParserException, IOException {
  
          final int depth = parser.getDepth();
          int type;
          boolean pendingRequestFocus = false;
          
          // 循环去解析View下面的子View
          while (((type = parser.next()) != XmlPullParser.END_TAG ||
                  parser.getDepth() > depth) && type != XmlPullParser.END_DOCUMENT) {
  
              // XmlPullParse.START_DOCUMENT 也符合 while循环的条件
              // 也符合下面代码的条件
              if (type != XmlPullParser.START_TAG) {
                  continue;
              }
  
              //获取到标签的名字
              final String name = parser.getName();
  
              if (TAG_REQUEST_FOCUS.equals(name)) {
                  pendingRequestFocus = true;
                  consumeChildElements(parser);
              } else if (TAG_TAG.equals(name)) {
                  parseViewTag(parser, parent, attrs);
              } else if (TAG_INCLUDE.equals(name)) {
                  // include标签不能作为布局文件的根布局
                  if (parser.getDepth() == 0) {
                      throw new InflateException("<include /> cannot be the root element");
                  }
                  parseInclude(parser, context, parent, attrs);
              } else if (TAG_MERGE.equals(name)) {
                  //Merge标签必须放到布局文件的根节点上
                  throw new InflateException("<merge /> must be the root element");
              } else {
                  //调用createViewFromTag 创建View对象
                  final View view = createViewFromTag(parent, name, context, attrs);
                  final ViewGroup viewGroup = (ViewGroup) parent;
                  final ViewGroup.LayoutParams params = viewGroup.generateLayoutParams(attrs);
                  //循环遍历创建View下的子View，该方法最终会调用到 rInflate()方法
                  rInflateChildren(parser, view, attrs, true);
                  //将每次创建的View添加到父布局中，这样就形成了完整的DOM树
                  viewGroup.addView(view, params);
              }
          }
  
          if (pendingRequestFocus) {
              parent.restoreDefaultFocus();
          }
  
          if (finishInflate) {
              //如果inflate结束，那么回调parent的onFinishInflate方法
              parent.onFinishInflate();
          }
      }
  ```

  - rInflate方法总结：该方法内主要就是循环View下面的子View，拿到表标签的名字

  - 如果是include标签，不能把这个标签作为布局文件的根布局
  - 如果是merge标签必须放到布局文件的根布局上
  - 其他标签通过createViewFromTag()方法创建View，然后调用rInflateChildren()继续循环当前获取到View下面的子View重复rInflate方法所做的事情，之后把获取到的View添加到父布局中，这样就形成了成型的DOM树了。

###### createViewFromTag方法

- 代码如下

  ```java
  View createViewFromTag(View parent, String name, Context context, AttributeSet attrs,
              boolean ignoreThemeAttr) {
          if (name.equals("view")) {
              name = attrs.getAttributeValue(null, "class");
          }
  
          // Apply a theme wrapper, if allowed and one is specified.
          if (!ignoreThemeAttr) {
              final TypedArray ta = context.obtainStyledAttributes(attrs, ATTRS_THEME);
              final int themeResId = ta.getResourceId(0, 0);
              if (themeResId != 0) {
                  context = new ContextThemeWrapper(context, themeResId);
              }
              ta.recycle();
          }
  
          try {
            	// 该方法主要使用Factory创建View，返回创建的View
              View view = tryCreateView(parent, name, context, attrs);
  
              //当通过Factory或者Factory2的onCreateView方法创建的View为null
              if (view == null) {
  
                  final Object lastContext = mConstructorArgs[0];
                  mConstructorArgs[0] = context;
                  try {
                      //就通过onCreateView方法创建View对象
  
                      // 如果View的name中没有"."说明是系统的控件
                      //会在接下来的调用中 在name前面加 android.view.
                      if (-1 == name.indexOf('.')) {
  
                          view = onCreateView(context, parent, name, attrs);
                      } else {
                          //如果name中包含了. 则直接调用了createView方法
                          view = createView(context, name, null, attrs);
                      }
                  } finally {
                      mConstructorArgs[0] = lastContext;
                  }
              }
  
              return view;
          } catch (InflateException e) {
              throw e;
  
          } catch (ClassNotFoundException e) {
              final InflateException ie = new InflateException(
                      getParserStateDescription(context, attrs)
                      + ": Error inflating class " + name, e);
              ie.setStackTrace(EMPTY_STACK_TRACE);
              throw ie;
  
          } catch (Exception e) {
              final InflateException ie = new InflateException(
                      getParserStateDescription(context, attrs)
                      + ": Error inflating class " + name, e);
              ie.setStackTrace(EMPTY_STACK_TRACE);
              throw ie;
          }
      }
  ```

- createViewFromTag方法的总结
  - 先通过tryCreateView创建View，内部主要是调用Factory的onCreateView创建view
  - 如果上述的方式创建的View为null，就使用createView方法创建View
  - 其中onCreateView方法最终调用的是createView()方法

###### createView方法

- 代码如下

  ```java
    public final View createView(@NonNull Context viewContext, @NonNull String name,
              @Nullable String prefix, @Nullable AttributeSet attrs)
              throws ClassNotFoundException, InflateException {
          Objects.requireNonNull(viewContext);
          Objects.requireNonNull(name);
          Constructor<? extends View> constructor = sConstructorMap.get(name);
          if (constructor != null && !verifyClassLoader(constructor)) {
              constructor = null;
              sConstructorMap.remove(name);
          }
          Class<? extends View> clazz = null;
  
          try {
              Trace.traceBegin(Trace.TRACE_TAG_VIEW, name);
  
              if (constructor == null) {
                  // Class not found in the cache, see if it's real, and try to add it
                  // 通过Class.forName() 拿到当前类的Class对象
                  clazz = Class.forName(prefix != null ? (prefix + name) : name, false,
                          mContext.getClassLoader()).asSubclass(View.class);
  
                  if (mFilter != null && clazz != null) {
                      boolean allowed = mFilter.onLoadClass(clazz);
                      if (!allowed) {
                          failNotAllowed(name, prefix, viewContext, attrs);
                      }
                  }
                  // 通过反射getConstructor方法获取到Class对象的Constructor
                  constructor = clazz.getConstructor(mConstructorSignature);
                  constructor.setAccessible(true);
                  sConstructorMap.put(name, constructor);
              } else {
                  // If we have a filter, apply it to cached constructor
                  if (mFilter != null) {
                      // Have we seen this name before?
                      Boolean allowedState = mFilterMap.get(name);
                      if (allowedState == null) {
                          // New class -- remember whether it is allowed
                          clazz = Class.forName(prefix != null ? (prefix + name) : name, false,
                                  mContext.getClassLoader()).asSubclass(View.class);
  
                          boolean allowed = clazz != null && mFilter.onLoadClass(clazz);
                          mFilterMap.put(name, allowed);
                          if (!allowed) {
                              failNotAllowed(name, prefix, viewContext, attrs);
                          }
                      } else if (allowedState.equals(Boolean.FALSE)) {
                          failNotAllowed(name, prefix, viewContext, attrs);
                      }
                  }
              }
  
              Object lastContext = mConstructorArgs[0];
              mConstructorArgs[0] = viewContext;
              Object[] args = mConstructorArgs;
              args[1] = attrs;
  
              try {
                  //通过newInstance获取到View的对象
                  final View view = constructor.newInstance(args);
                  //如果当前View是ViewStub
                  if (view instanceof ViewStub) {
                      // Use the same context when inflating ViewStub later.
                      // 强转成 ViewStub
                      final ViewStub viewStub = (ViewStub) view;
                      viewStub.setLayoutInflater(cloneInContext((Context) args[0]));
                  }
                  return view;
              } finally {
                  mConstructorArgs[0] = lastContext;
              }
          } catch (NoSuchMethodException e) {
              final InflateException ie = new InflateException(
                      getParserStateDescription(viewContext, attrs)
                      + ": Error inflating class " + (prefix != null ? (prefix + name) : name), e);
              ie.setStackTrace(EMPTY_STACK_TRACE);
              throw ie;
  
          } catch (ClassCastException e) {
              // If loaded class is not a View subclass
              final InflateException ie = new InflateException(
                      getParserStateDescription(viewContext, attrs)
                      + ": Class is not a View " + (prefix != null ? (prefix + name) : name), e);
              ie.setStackTrace(EMPTY_STACK_TRACE);
              throw ie;
          } catch (ClassNotFoundException e) {
              // If loadClass fails, we should propagate the exception.
              throw e;
          } catch (Exception e) {
              final InflateException ie = new InflateException(
                      getParserStateDescription(viewContext, attrs) + ": Error inflating class "
                              + (clazz == null ? "<unknown>" : clazz.getName()), e);
              ie.setStackTrace(EMPTY_STACK_TRACE);
              throw ie;
          } finally {
              Trace.traceEnd(Trace.TRACE_TAG_VIEW);
          }
      }
  ```

  - createView方法的核心就是通过反射获取到View的对象

## 总结

###### infalte重载的方法

```java
 public View inflate(@LayoutRes int resource, @Nullable ViewGroup root) {
   // 当root不为null attachToRoot为true否则为false
        return inflate(resource, root, root != null);
    }
    
 public View inflate(@LayoutRes int resource, @Nullable ViewGroup root, boolean attachToRoot) {
        final Resources res = getContext().getResources();
        if (DEBUG) {
            Log.d(TAG, "INFLATING from resource: \"" + res.getResourceName(resource) + "\" ("
                  + Integer.toHexString(resource) + ")");
        }

        View view = tryInflatePrecompiled(resource, res, root, attachToRoot);
        if (view != null) {
            return view;
        }

        XmlResourceParser parser = res.getLayout(resource);
        try {
            return inflate(parser, root, attachToRoot);
        } finally {
            parser.close();
        }
    }
    
    
    public View inflate(XmlPullParser parser, @Nullable ViewGroup root) {
      // 同样 root不为null attachToRoot为true 否则为false
        return inflate(parser, root, root != null);
    }

    
    //最终都会调用的inflate方法
     public View inflate(XmlPullParser parser, @Nullable ViewGroup root, boolean attachToRoot) {
```

- 通过如下代码我们得出：如果root不为null 并且attachToRoot为true，那么给加载的布局文件指定一个父布局

  ```java
   if (root != null && attachToRoot) {
                          // 如果root不为null 并且attachToRoot为true
                          //那么给加载的布局文件指定一个父布局
                          root.addView(temp, params);
                      }
  ```

  

- 通过如下代码我们得出：如果root不为null，并且attacheToRoot为false，那么就把root的LayoutParams设置给布局文件的最外层。

  ​	

  ```java
   if (root != null) {
                          if (DEBUG) {
                              System.out.println("Creating params from root: " +
                                      root);
                          }
                          // Create layout params that match root, if supplied
                          //如果Root不为null，那么根据标签的参数生成LayoutParams
                          params = root.generateLayoutParams(attrs);
                          if (!attachToRoot) {
                              // Set the layout params for temp if we are not
                              // attaching. (If we are, we use addView, below)
                              //如果不是attachToRoot 那么就将这个Tag创建出来的View设置LayoutParams
                              temp.setLayoutParams(params);
                          }
                      }
  ```

  

- 如果调用的inflate方法中，没有设置attacheToRoot的赋值入口，那么attachToRoot的取值取决于root是否为null
  - 如果root为null，那么attachToRoot为false
  - 如果root不为null，那么attacheToRoot为true