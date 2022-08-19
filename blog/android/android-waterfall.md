<center>android实现瀑布流</center>

# 前言
瀑布流，又称瀑布流式布局。是比较流行的一种页面布局，视觉表现为参差不齐的多栏布局，随着页面滚动条向下滚动，这种布局还会不断加载数据块并附加至当前尾部。最早采用此布局的是Pinterest，逐渐在国内流行开来。国内大多数清新站基本为这类风格。
特点:
- 1、琳琅满目：整版以图片为主，大小不一的图片按照一定的规律排列。
- 2、唯美：图片的风格以唯美的图片为主。
- 3、操作简单：在浏览网站的时候只需要轻轻滑动一下鼠标滚轮，一切的美妙的图片精彩便可呈现在你面前。
 

## 效果
<center><img src="../../images/android-waterfall.png" width="350"/></center>

## 布局
```
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical">

    <com.scwang.smart.refresh.layout.SmartRefreshLayout
        android:id="@+id/waterfallLayout"
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        android:layout_weight="1"
        app:srlEnablePreviewInEditMode="true"
        app:srlEnableLoadMoreWhenContentNotFull="false">

        <androidx.recyclerview.widget.RecyclerView
            android:id="@+id/waterfall_recycler_view"
            android:layout_width="match_parent"
            android:layout_height="match_parent"
            android:overScrollMode="never"
            android:background="@color/white"/>
    </com.scwang.smart.refresh.layout.SmartRefreshLayout>

</LinearLayout>
```
## WaterfallListAdapter
```
open class WaterfallListAdapter(layoutResId: Int, data: MutableList<GoodsBean>): BaseQuickAdapter<GoodsBean, BaseViewHolder>(layoutResId, data) {
    private var imageLoader: ImageLoader = CoilUtil.getImageLoader()
    private var imageWidth: Int = 0

    override fun onAttachedToRecyclerView(recyclerView: RecyclerView) {
        super.onAttachedToRecyclerView(recyclerView)
        imageWidth = ScreenUtil.getWindowWidth(context) / 2 - 4
    }

    override fun convert(holder: BaseViewHolder, item: GoodsBean) {
        var height = item.height * imageWidth / item.width
        holder.getView<ImageView>(R.id.waterfall_item_img).layoutParams = LinearLayout.LayoutParams(imageWidth, height)
        holder.getView<ImageView>(R.id.waterfall_item_img).load(item.thumb, imageLoader ) {
            crossfade(true)
            placeholder(R.drawable.default_img)
            error(R.drawable.default_img)
        }
        holder.setText(R.id.waterfall_item_title, item.name)
    }
}
```
## WaterfallListActivity
```
class WaterfallListActivity: BaseActivity(R.layout.layout_waterfall), MavericksView {
    private lateinit var mCheckForGapMethod: Method
    private lateinit var mMarkItemDecorInsetsDirtyMethod: Method

    private val adapter: WaterfallListAdapter by lazy {
        WaterfallListAdapter(R.layout.layout_waterfall_item, arrayListOf())
    }
    private val staggeredGridLayoutManager: StaggeredGridLayoutManager by lazy {
        StaggeredGridLayoutManager(2, StaggeredGridLayoutManager.VERTICAL)
    }
    private val loadingDialog: LoadingDialog by lazy { LoadingDialog(this) }
    private val previewPicture: PreviewPictureDialog by lazy { PreviewPictureDialog(this) }

    private val viewModel: WaterfallViewModel by viewModel()

    override fun initView() {
        ImmersionBar.with(this).transparentStatusBar().statusBarDarkFont(false).init()

        //下拉刷新
        waterfallLayout.setOnRefreshListener {
            run {
                viewModel.refresh(true)
            }
        }

        //瀑布列表
        staggeredGridLayoutManager.gapStrategy = StaggeredGridLayoutManager.GAP_HANDLING_NONE //解决加载下一页后重新排列的问题
        waterfall_recycler_view.layoutManager = staggeredGridLayoutManager
        waterfall_recycler_view.itemAnimator = null

        mCheckForGapMethod = StaggeredGridLayoutManager::class.java.getDeclaredMethod("checkForGaps")
        mMarkItemDecorInsetsDirtyMethod = RecyclerView::class.java.getDeclaredMethod("markItemDecorInsetsDirty")
        mCheckForGapMethod.isAccessible = true
        mMarkItemDecorInsetsDirtyMethod.isAccessible = true

        waterfall_recycler_view.addOnScrollListener(object: RecyclerView.OnScrollListener() {
            override fun onScrolled(recyclerView: RecyclerView, dx: Int, dy: Int) {
                super.onScrolled(recyclerView, dx, dy)
                val result = mCheckForGapMethod.invoke(waterfall_recycler_view.layoutManager) as Boolean
                //如果发生了重新排序，刷新itemDecoration
                if(result) {
                    mMarkItemDecorInsetsDirtyMethod.invoke(waterfall_recycler_view)
                }
            }
        })
        adapter.setOnItemClickListener{_, view, position ->
            if (view.id == R.id.waterfallItemLayout) {
                val imgUrls = adapter.data.map { v-> v.thumb }
                previewPicture.show(imgUrls, position)
            }
        }
        waterfall_recycler_view.adapter = adapter
        val space = resources.getDimension(R.dimen.waterfall_space)
        waterfall_recycler_view.addItemDecoration(SpacesItemDecoration(space.toInt()))

        //上拉加载更多
        waterfallLayout.setOnLoadMoreListener {
            run {
                viewModel.loadMore()
            }
        }
        waterfallLayout.setEnableAutoLoadMore(true)

        addStateChangeListener()
    }

    override fun initData() {
        viewModel.refresh(false)
    }

    private fun addStateChangeListener() {
        viewModel.onEach {
            when (it.isLoading) {
                true -> loadingDialog.show()
                false -> {
                    loadingDialog.dismiss()

                    when (it.fetchType) {
                        ActionType.INIT -> {
                            adapter.setList(it.dataList)
                        }
                        ActionType.REFRESH -> {
                            adapter.setList(it.dataList)
                            waterfallLayout.run {
                                finishRefresh()
                            }
                        }
                        ActionType.LOADMORE -> {
                            adapter.addData(adapter.data.size, it.newList)
                            if (it.currentPage <= it.totalPage)
                                waterfallLayout.run {
                                    finishLoadMore()
                                }
                            else
                                waterfallLayout.run {
                                    finishLoadMoreWithNoMoreData()
                                }
                        }
                    }
                }
            }
        }
    }
}
```

## 结束语
- 👀 目前专注于前端
- ⚙️ 在react、react-native开发方面有丰富的经验
- 🔭 最近在学习安卓，有自己的开源安卓项目，集成react-native热更新功能
- 我❤️ 思考、学习、编码和健身
- 如果文章对您有帮助，三连支持一下～O(∩_∩)O谢谢！



