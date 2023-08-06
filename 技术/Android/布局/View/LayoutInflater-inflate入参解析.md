#### LayoutInflater # inflate

```java
 public View inflate(@LayoutRes int resource, @Nullable ViewGroup root, boolean attachToRoot) {
   // ......
 }
```

##### 参数入参组合

###### root != null && attachToRoot = true

###### resource布局被添加到父布局root中，此时inflate返回的View 是root对应的View；	

###### root != null && attachToRoot = true

reousrce布局的根节点布局参数生效，不指定父布局，root的作用是用于使得resource布局参数生效，此时返回的view对应的是resource布局

###### root == null && attachToRoot = false

resource布局根节点布局参数不生效（layout开头的布局参数），非布局参数如bg依然生效，此时返回的 View是resource对应的布局