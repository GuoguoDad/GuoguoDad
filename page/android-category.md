<center>安卓高仿京东分类</center>
## 前言
网购已经成为我们生活的一般，经常用的京东、淘宝等app，接到一个需求，开发电商app的分类页，这里写了个demo，高仿京东的分类页，我们看下效果

<center><img src="../images/category.png"/></center>

## 思路
    顶部： 封住一个组件，这里不介绍
    左侧： 一个RecyclerView即可，比较简单，这里也不介绍，想了解的直接看源码
    右侧: TabLayout + RecyclerView 布局， 难点:  tabLayout滚动RecyclerView到指定位置，滚动RecyclerView时，默认选中tabLayout

## 右侧布局
``` 
<androidx.appcompat.widget.LinearLayoutCompat
    android:id="@+id/rightLinearLayoutCompat"
    app:layout_behavior="@string/appbar_scrolling_view_behavior"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical">

    <com.google.android.material.tabs.TabLayout
        android:id="@+id/tabLayout"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        app:tabMode="scrollable"
        app:tabIndicatorHeight="0dp"
        app:tabMaxWidth="120dp"
        app:tabMinWidth="60dp"
        app:tabRippleColor="@color/transparent"
        app:tabTextAppearance="@style/tab_title"
        app:tabTextColor="@color/color_404040"
        app:tabSelectedTextColor="@color/color_E33A3C"
        app:tabBackground="@drawable/selector_tab_indicator2"/>

    <androidx.recyclerview.widget.RecyclerView
        android:id="@+id/categoryRecyclerView"
        android:layout_width="match_parent"
        android:layout_height="match_parent"/>

</androidx.appcompat.widget.LinearLayoutCompat>
```

```
R.layout.main_right_grid_header

<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:orientation="vertical"
    android:layout_width="match_parent"
    android:layout_height="wrap_content">

    <TextView
        android:id="@+id/textHeader"
        android:padding="@dimen/dp_4"
        android:textColor="#181818"
        android:textSize="14dp"
        android:textStyle="bold"
        android:text="专属推荐"
        android:layout_width="match_parent"
        android:layout_height="wrap_content" />
</LinearLayout>
```

```
R.layout.main_right_grid

<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:orientation="vertical"
    android:layout_width="match_parent"
    android:layout_height="100dp"
    android:gravity="center">

    <ImageView
        android:id="@+id/thirdCategoryIcon"
        android:layout_width="60dp"
        android:layout_height="60dp"
        android:scaleType="fitXY"/>

    <TextView
        android:id="@+id/text_content"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:textSize="12dp"
        android:textColor="#898989"
        android:text="男装馆"
        android:layout_marginTop="10dp"/>

</LinearLayout>
```
## TabLayout addTab
```
tabLayout.removeAllTabs()
data.cateList.forEach { v -> tabLayout.addTab(tabLayout.newTab().setText(v.categoryName)) }
```

## RecyclerView的adapter
```
package com.aries.category.ui.adapter

import android.widget.ImageView
import coil.load
import com.chad.library.adapter.base.BaseSectionQuickAdapter
import com.chad.library.adapter.base.viewholder.BaseViewHolder
import com.aries.category.R
import com.aries.category.ui.modal.CategoryModal
import com.aries.common.util.CoilUtil

class SectionQuickAdapter(sectionHeadResId: Int, layoutResId: Int, data: MutableList<CategoryModal> ):
    BaseSectionQuickAdapter<CategoryModal, BaseViewHolder>(sectionHeadResId, layoutResId, data) {
    private var imageLoader = CoilUtil.getImageLoader()

    override fun convertHeader(helper: BaseViewHolder, item: CategoryModal) {
        helper.setText(R.id.textHeader, item.categoryName)
    }

    override fun convert(holder: BaseViewHolder, item: CategoryModal) {
        holder.getView<ImageView>(R.id.thirdCategoryIcon).load(item.iconUrl, imageLoader ) {
            crossfade(true)
            placeholder(R.drawable.default_img)
            error(R.drawable.default_img)
        }
        holder.setText(R.id.text_content, item.categoryName)
    }
}
```

## 设置RecyclerView的 adapter 和 layoutManager
```
    gridLayoutManager = GridLayoutManager(context, 3)
    sectionQuickAdapter = SectionQuickAdapter(R.layout.main_right_grid_header, R.layout.main_right_grid, arrayListOf())

    val thisTabLayout = tabLayout
    categoryRecyclerView.run {
        layoutManager = gridLayoutManager
        adapter = sectionQuickAdapter
    }
```
## 设置数据
```
val list: ArrayList<CategoryModal> = arrayListOf()
data.cateList.forEach { v ->
    run {
        list.add(CategoryModal(v.iconUrl, v.categoryName, v.categoryCode, true))
        v.cateList?.forEach { m -> list.add(CategoryModal(m.iconUrl, m.categoryName, m.categoryCode, false))}
    }
}
sectionQuickAdapter.setList(list)
```

## 重点设置 tabLayout 和 RecyclerView 联动关系
```
categoryRecyclerView.run {
    setOnTouchListener { _, p1 ->
        if (p1.action == MotionEvent.ACTION_DOWN) {
            isRecyclerScroll = true
        }
        false
    }
    addOnScrollListener(object : RecyclerView.OnScrollListener(){
        override fun onScrolled(recyclerView: RecyclerView, dx: Int, dy: Int) {
            super.onScrolled(recyclerView, dx, dy)
            if (isRecyclerScroll) {
                val position = findHeaderPositionByTab(gridLayoutManager.findFirstVisibleItemPosition())
                if (position != lastPos) {
                    thisTabLayout.setScrollPosition(position, 0F, true)
                }
                lastPos = position
            }
        }
        override fun onScrollStateChanged(recyclerView: RecyclerView, newState: Int) {
            super.onScrollStateChanged(recyclerView, newState)
            if (mShouldScroll) {
                mShouldScroll = false
                scrollTop(mToPosition)
            }
        }
    })
}
tabLayout.run {
    addOnTabSelectedListener(object : TabLayout.OnTabSelectedListener{
        override fun onTabSelected(tab: TabLayout.Tab?) {
            val position = tab?.position
            if (position != null) {
                scrollRecycleView2Top(position)
            }
            isRecyclerScroll = false
        }
        override fun onTabUnselected(tab: TabLayout.Tab?) {}
        override fun onTabReselected(tab: TabLayout.Tab?) {}
    })
}
```

```
private fun scrollTop(position: Int) {
    val firstPosition = gridLayoutManager.findFirstVisibleItemPosition()
    val lastPosition = gridLayoutManager.findLastVisibleItemPosition()

    if (position < firstPosition) {
        // 如果跳转位置在第一个可见位置之前，就smoothScrollToPosition可以直接跳转
        categoryRecyclerView.smoothScrollToPosition(position)
    } else if (position <= lastPosition){
        // 跳转位置在第一个可见项之后，最后一个可见项之前
        // smoothScrollToPosition根本不会动，此时调用smoothScrollBy来滑动到指定位置
        val movePosition = position - firstPosition
        if (movePosition >=0 && movePosition < categoryRecyclerView.childCount) {
            val scrollY = categoryRecyclerView.getChildAt(position - firstPosition).top
            categoryRecyclerView.smoothScrollBy(0, scrollY)
        }
    } else {
        // 如果要跳转的位置在最后可见项之后，则先调用smoothScrollToPosition将要跳转的位置滚动到可见位置
        // 再通过onScrollStateChanged控制再次调用smoothMoveToPosition，执行上一个判断中的方法
        categoryRecyclerView.smoothScrollToPosition(position)
        mToPosition = position
        mShouldScroll = true
    }
}
```

```
 private fun scrollRecycleView2Top(position: Int) {
    val currentItem = dataCopy.cateList[position]
    val currentPosition = sectionQuickAdapter.data.indexOfFirst { v -> v.categoryCode == currentItem.categoryCode }

    scrollTop(currentPosition)
}
```

## 总结
    刚开始学习android，这种复杂的布局花了很长时间，这里的 联动效果也浪费了很长时间，这里记录一下，如果对你有帮助，给个赞，谢谢

## 源码
    https://github.com/GuoguoDad/jd_mall
    
