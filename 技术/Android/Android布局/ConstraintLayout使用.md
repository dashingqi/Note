#### 水平方向的权重:app:layout_constraintHorizontal_weight="1"

```xml
 <androidx.constraintlayout.widget.ConstraintLayout
            android:id="@+id/layoutShopCar"
            android:layout_width="0dp"
            android:layout_height="@dimen/qb_px_60"
            app:layout_constraintBottom_toBottomOf="parent"
            app:layout_constraintHorizontal_weight="1"
            app:layout_constraintLeft_toLeftOf="parent"
            app:layout_constraintRight_toLeftOf="@+id/tvBuyNow">

        </androidx.constraintlayout.widget.ConstraintLayout>

        <TextView
            android:id="@+id/tvBuyNow"
            android:layout_width="0dp"
            android:layout_height="@dimen/qb_px_60"
            android:background="#EFFFED"
            android:gravity="center"
            android:text="@string/home_commodity_detail_buy_now"
            android:textColor="#00A762"
            android:textSize="@dimen/qb_px_18"
            app:layout_constraintBottom_toBottomOf="parent"
            app:layout_constraintHorizontal_weight="2"
            app:layout_constraintLeft_toRightOf="@+id/layoutShopCar"
            app:layout_constraintRight_toLeftOf="@+id/tvAddShopCar" />

        <TextView
            android:id="@+id/tvAddShopCar"
            android:layout_width="0dp"
            android:layout_height="@dimen/qb_px_60"
            android:background="#00A762"
            android:gravity="center"
            android:text="@string/home_commodity_detail_add_shopping_car"
            android:textColor="#ffffff"
            android:textSize="@dimen/qb_px_18"
            app:layout_constraintBottom_toBottomOf="parent"
            app:layout_constraintHorizontal_weight="2"
            app:layout_constraintLeft_toRightOf="@+id/tvBuyNow"
            app:layout_constraintRight_toRightOf="parent" />
```

#### Constralayout中Group标签的使用

###### 比如我们可以通过该标签来统一组控件是否显示

