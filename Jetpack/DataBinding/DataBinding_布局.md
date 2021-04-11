## 定义

数据绑定库是一种支持库，借助该库我们可以使用声明性格式将布局中的界面组件绑定到应用中的数据源。

## 布局

#### layout

- 布局的根节点必须是<layout> 同时layout中只能包含一个View标签。不能直接包含<merge>标签

#### data

- <data>标签的内容也就是DataBinding的数据，data标签只能存在一个。

#### variable

- 通过variable标签可以指定类，然后在控件的属性值中就可以使用

  ```xml
  <data>
  	<variable name="user" type="com.liangfeizc.databindingsamples.basic.User" />
  </data>
  ```

  - 其中name值不能包含下划线。

#### import

- 导入需要使用的类，可以直接使用类中的静态方法

  ```xml
  <data>
    <!--导入类-->
      <import type="com.liangfeizc.databindingsamples.basic.User" />
    <!--因为User已经导入, 所以可以简写类名-->
      <variable name="user" type="User" />
  </data>
  ```

#### class

- <data> 标签有个属性 <class>可以自定义DataBinding生成的类名和路径

  ```xml
  <!--自定义类名-->
  <data class="CustomDataBinding"></data>
  
  <!--自定义生成路径以及类型-->
  <data class=".CustomDataBinding"></data> <!--自动在包名下生成包以及类-->
  ```

#### alias

- <variable>标签如果需要导入两个同名的类时可以使用alias属性

  ```xml
  <import type="com.example.home.data.User" />
  <import type="com.examle.detail.data.User" alias="DetailUser" />
  <variable name="user" type="DetailUser" />
  ```

#### include

- 在include其他布局的时候可能需要传递变量值过去

  ```xml
  <variable
            name="userName"
            type="String"/>
  
  ....
  
  <include
           layout="@layout/include_demo"
           bind:userName="@{userName}"/>
  
  
  // include_demo
  
    <data>
  
          <variable
              name="userName"
              type="String"/>
      </data>
  
  ...
  
  android:text="@{userName}"
  
  ```

  - 两个布局通过include的bind:<变量名>值来传递。并且两者必须有同一个变量

#### 自动布局属性

- DataBinding对于自定义属性支持非常好，只要View中包含setter方法就可以直接在布局中使用该属性

  ```java
  public void setCustomName(@NonNull final String customName) {
      mLastName.setText("吴彦祖");
    }
    
   // 在布局中
   app:customName="@{@string/wuyanzu}"
  ```

  