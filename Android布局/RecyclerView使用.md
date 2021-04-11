#### 横向滑动

```java
public static void scrollToMiddleHorizontal(View childView, int clickedPosition, int firstVisibleItemPosition, RecyclerView recyclerView) {
        scrollToMiddleHorizontal(childView, clickedPosition, firstVisibleItemPosition, recyclerView, false);
    }

    /**
     * RecyclerView 点击条目，水平滑动 向左滑动（默认）
     *
     * @param childView
     * @param clickedPosition
     * @param firstVisibleItemPosition
     */
    public static void scrollToMiddleHorizontal(View childView, int clickedPosition, int firstVisibleItemPosition, RecyclerView recyclerView, boolean isScrollToRight) {
        int viewWidth = childView.getWidth();
        Rect rect = new Rect();
        recyclerView.getGlobalVisibleRect(rect);
        int reWidth = rect.right - rect.left - viewWidth;
        int left = recyclerView.getChildAt(clickedPosition - firstVisibleItemPosition).getLeft();
        int half = reWidth / 2;
        recyclerView.smoothScrollBy(isScrollToRight ? half - left : left - half, 0);
    }
```

- getGlobalVisibleRect(rect)
  - 会将当前RecyclerView的可见区域的 left、right、top、bottom的值保存到一个Rect对象中。

#### 纵向滑动

```java
/**
     * RecyclerView垂直滑动
     *
     * @param childView
     * @param clickedPosition
     * @param firstVisibleItemPosition
     */
    public static void scrollToMiddleVertical(View childView, int clickedPosition, int firstVisibleItemPosition, RecyclerView recyclerView) {
        int vHeight = childView.getHeight();
        Rect rect = new Rect();
        recyclerView.getGlobalVisibleRect(rect);
        int reHeight = rect.bottom - rect.top - vHeight;
        int top = recyclerView.getChildAt(clickedPosition - firstVisibleItemPosition).getTop();
        int half = reHeight / 2;
        recyclerView.smoothScrollBy(0, top - half);
    }
```

