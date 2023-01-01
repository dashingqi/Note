## Navigation

#### 存在的目的

- 通过声明式编程，来确保 “应用内导航” 的一致性
- 通过可视化编程，来直观地反映页面的路由关系
- 通过抽象，来整合 Activity 和 Fragment 的路由跳转代码

#### 运行的机制

###### 定义声明式编程协议

- navigation

- fragment
  - action
  - argument

```xml
<?xml version="1.0" encoding="utf-8"?>
<navigation xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:id="@+id/navigation"
    app:startDestination="@id/settingFragment">

    <!--  startDestination  指定的是   Navigation 整个页面跳转管理栈的最根级页面-->

    <!--
             fragment 对应一个 Fragment
             其中 name属性 对应一个Fragment的地址
             id属性 用来作为一个标示，能让其他fragment的action能找到自己
             <action/>元素  用来指明跳转的事件
                    指明切换的动画
                    事件的id
                    事件的目标（destination）
             <argument/>元素 用来指明接受的参数
    -->
    <fragment
        android:id="@+id/myFragment"
        android:name="com.dashingqi.module.navigation.MyFragment"
        android:label="MyFragment"
        tools:layout="@layout/fragment_my_layout">
        <!--        事件-->
        <action
            android:id="@+id/action_myFragment_to_settingFragment"
            app:destination="@id/settingFragment" />
    </fragment>

    <fragment
        android:id="@+id/settingFragment"
        android:name="com.dashingqi.module.navigation.SettingFragment"
        android:label="SettingFragment"
        tools:layout="@layout/fragment_setting_layout">
        <argument
            android:name="name"
            android:defaultValue="0"
            app:argType="string"
            app:nullable="true" />
    </fragment>

    <fragment
        android:id="@+id/homeFragment"
        android:name="com.dashingqi.module.navigation.HomeFragment"
        android:label="HomeFragment"
        tools:layout="@layout/fragment_home_layout">
        <action
            android:id="@+id/action_homeFragment_to_myFragment"
            app:destination="@id/myFragment" />
    </fragment>

</navigation>
```

