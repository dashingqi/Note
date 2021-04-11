```java
public interface NestedScrollingChild {
    /**
     * Enable or disable nested scrolling for this view.
     * 设置当前View能否支持嵌套滑动
     */
    void setNestedScrollingEnabled(boolean enabled);

    /**
     * Returns true if nested scrolling is enabled for this view.
     * 返回当前View是否支持嵌套滑动
     */
    boolean isNestedScrollingEnabled();

    /**
     * Begin a nestable scroll operation along the given axes.
     * 顺着滑动的方向开始滑动
     * axes：方向
     * return：找到是否有配合滑动的NestedScrollingParent
     */
    boolean startNestedScroll(@ScrollAxis int axes);

    /**
     * Stop a nested scroll in progress.
     * 停止当前滑动
     */
    void stopNestedScroll();

    /**
     * Returns true if this view has a nested scrolling parent.
     * 获取到嵌套试图中是否有NestedScrollingParent
     */
    boolean hasNestedScrollingParent();

    /**
     * 滑动完成之后将已经消费和未消费的传给NestedScrollParent
     * @param dxConsumed：水平方向消费的距离
     * @param dyConsumed ：垂直方向消费的距离
     * @param dxUnconsumed 水平方向剩余的距离
     * @param dyUnconsumed 垂直方向剩余的距离
     * @param offsetInWindow Optional. 
     * @return 返回事件是否被分发成功
     * @see #dispatchNestedPreScroll(int, int, int[], int[])
     */
    boolean dispatchNestedScroll(int dxConsumed, int dyConsumed,
            int dxUnconsumed, int dyUnconsumed, @Nullable int[] offsetInWindow);

    /**
     * 在滑动之前将事件分发给NestedScrollParent
     * @param dx 水平方向消费的距离
     * @param dy 垂直方向消费的距离
     * @param consumed Output. 消费的坐标输出，consumed[0]是NestedScrollParent在水平方向的消费 consumed[1]是NestedScrollParent在垂直方向的    消费		
     * @param offsetInWindow Optional. 包含当前View 调用该方法之前到调用完该方法之后在屏幕上的滑动坐标
     * @return true if the parent consumed some or all of the scroll delta
     * @see #dispatchNestedScroll(int, int, int, int, int[])
     */
    boolean dispatchNestedPreScroll(int dx, int dy, @Nullable int[] consumed,
            @Nullable int[] offsetInWindow);

    /**
     * 将惯性滑动的速度和NestedScrollChild是否消费这个惯性滑动传递给NestedScrollParent
     * @param velocityX 水平方向的滑动速度
     * @param velocityY 垂直方向的互动速度
     * @param consumed 返回是否由子View消费这个惯性滑动
     */
    boolean dispatchNestedFling(float velocityX, float velocityY, boolean consumed);

   	/*
   	 * 在惯性滑动之前将事件传递给NestedScrollParnet
     * @param velocityX 水平方向的速度
     * @param velocityY 垂直方向的速度
     * @return 返回NestedScrollParent是否消费全部的惯性滑动
     */
    boolean dispatchNestedPreFling(float velocityX, float velocityY);
}
```

```java
public interface NestedScrollingParent {
    /**
     * React to a descendant view initiating a nestable scroll operation, claiming the
     * nested scroll operation if appropriate.
     * 对NestedScrollChild的滑动做出反应
		 * child：
		 */
    boolean onStartNestedScroll(@NonNull View child, @NonNull View target, @ScrollAxis int axes);

    /**
     * React to the successful claiming of a nested scroll operation.
     *
     * <p>This method will be called after
     * {@link #onStartNestedScroll(View, View, int) onStartNestedScroll} returns true. It offers
     * an opportunity for the view and its superclasses to perform initial configuration
     * for the nested scroll. Implementations of this method should always call their superclass's
     * implementation of this method if one is present.</p>
     *
     * @param child Direct child of this ViewParent containing target
     * @param target View that initiated the nested scroll
     * @param axes Flags consisting of {@link ViewCompat#SCROLL_AXIS_HORIZONTAL},
     *                         {@link ViewCompat#SCROLL_AXIS_VERTICAL} or both
     * @see #onStartNestedScroll(View, View, int)
     * @see #onStopNestedScroll(View)
     */
    void onNestedScrollAccepted(@NonNull View child, @NonNull View target, @ScrollAxis int axes);

    /**
     * React to a nested scroll operation ending.
     *
     * <p>Perform cleanup after a nested scrolling operation.
     * This method will be called when a nested scroll stops, for example when a nested touch
     * scroll ends with a {@link MotionEvent#ACTION_UP} or {@link MotionEvent#ACTION_CANCEL} event.
     * Implementations of this method should always call their superclass's implementation of this
     * method if one is present.</p>
     *
     * @param target View that initiated the nested scroll
     */
    void onStopNestedScroll(@NonNull View target);

    /**
     * React to a nested scroll in progress.
     *
     * <p>This method will be called when the ViewParent's current nested scrolling child view
     * dispatches a nested scroll event. To receive calls to this method the ViewParent must have
     * previously returned <code>true</code> for a call to
     * {@link #onStartNestedScroll(View, View, int)}.</p>
     *
     * <p>Both the consumed and unconsumed portions of the scroll distance are reported to the
     * ViewParent. An implementation may choose to use the consumed portion to match or chase scroll
     * position of multiple child elements, for example. The unconsumed portion may be used to
     * allow continuous dragging of multiple scrolling or draggable elements, such as scrolling
     * a list within a vertical drawer where the drawer begins dragging once the edge of inner
     * scrolling content is reached.</p>
     *
     * @param target The descendent view controlling the nested scroll
     * @param dxConsumed Horizontal scroll distance in pixels already consumed by target
     * @param dyConsumed Vertical scroll distance in pixels already consumed by target
     * @param dxUnconsumed Horizontal scroll distance in pixels not consumed by target
     * @param dyUnconsumed Vertical scroll distance in pixels not consumed by target
     */
    void onNestedScroll(@NonNull View target, int dxConsumed, int dyConsumed,
            int dxUnconsumed, int dyUnconsumed);

    /**
     * React to a nested scroll in progress before the target view consumes a portion of the scroll.
     *
     * <p>When working with nested scrolling often the parent view may want an opportunity
     * to consume the scroll before the nested scrolling child does. An example of this is a
     * drawer that contains a scrollable list. The user will want to be able to scroll the list
     * fully into view before the list itself begins scrolling.</p>
     *
     * <p><code>onNestedPreScroll</code> is called when a nested scrolling child invokes
     * {@link View#dispatchNestedPreScroll(int, int, int[], int[])}. The implementation should
     * report how any pixels of the scroll reported by dx, dy were consumed in the
     * <code>consumed</code> array. Index 0 corresponds to dx and index 1 corresponds to dy.
     * This parameter will never be null. Initial values for consumed[0] and consumed[1]
     * will always be 0.</p>
     *
     * @param target View that initiated the nested scroll
     * @param dx Horizontal scroll distance in pixels
     * @param dy Vertical scroll distance in pixels
     * @param consumed Output. The horizontal and vertical scroll distance consumed by this parent
     */
    void onNestedPreScroll(@NonNull View target, int dx, int dy, @NonNull int[] consumed);

    /**
     * Request a fling from a nested scroll.
     *
     * <p>This method signifies that a nested scrolling child has detected suitable conditions
     * for a fling. Generally this means that a touch scroll has ended with a
     * {@link VelocityTracker velocity} in the direction of scrolling that meets or exceeds
     * the {@link ViewConfiguration#getScaledMinimumFlingVelocity() minimum fling velocity}
     * along a scrollable axis.</p>
     *
     * <p>If a nested scrolling child view would normally fling but it is at the edge of
     * its own content, it can use this method to delegate the fling to its nested scrolling
     * parent instead. The parent may optionally consume the fling or observe a child fling.</p>
     *
     * @param target View that initiated the nested scroll
     * @param velocityX Horizontal velocity in pixels per second
     * @param velocityY Vertical velocity in pixels per second
     * @param consumed true if the child consumed the fling, false otherwise
     * @return true if this parent consumed or otherwise reacted to the fling
     */
    boolean onNestedFling(@NonNull View target, float velocityX, float velocityY, boolean consumed);

    /**
     * React to a nested fling before the target view consumes it.
     *
     * <p>This method siginfies that a nested scrolling child has detected a fling with the given
     * velocity along each axis. Generally this means that a touch scroll has ended with a
     * {@link VelocityTracker velocity} in the direction of scrolling that meets or exceeds
     * the {@link ViewConfiguration#getScaledMinimumFlingVelocity() minimum fling velocity}
     * along a scrollable axis.</p>
     *
     * <p>If a nested scrolling parent is consuming motion as part of a
     * {@link #onNestedPreScroll(View, int, int, int[]) pre-scroll}, it may be appropriate for
     * it to also consume the pre-fling to complete that same motion. By returning
     * <code>true</code> from this method, the parent indicates that the child should not
     * fling its own internal content as well.</p>
     *
     * @param target View that initiated the nested scroll
     * @param velocityX Horizontal velocity in pixels per second
     * @param velocityY Vertical velocity in pixels per second
     * @return true if this parent consumed the fling ahead of the target view
     */
    boolean onNestedPreFling(@NonNull View target, float velocityX, float velocityY);

    /**
     * 返回当前NestedScrollingParent滑动的方向
     */
    @ScrollAxis
    int getNestedScrollAxes();
}
```

