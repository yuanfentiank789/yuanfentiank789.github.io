---
layout: post
title:  "RecyclerView 详解"
date:   2016-06-25 1:05:00
catalog:  true
tags:
    - RecyclerView
    - LayoutManager
    

---

# 概述
RecyclerView更灵活的实现了列表数据的展现，只负责View复用这块，把原来ListView中的布局方式抽象成LayoutManager，提供了一种插拔式的体验，高度的解耦，异常的灵活，通过设置它提供的不同LayoutManager，ItemDecoration , ItemAnimator实现令人瞠目的效果。

**你想要控制其显示的方式，请通过布局管理器LayoutManager**

**你想要控制Item间的间隔（可绘制），请通过ItemDecoration**

**你想要控制Item增删的动画，请通过ItemAnimator**

# 基本使用

RecyclerView包含在android的support v7包中，因此使用前需要在build.gradle中导入：

      compile 'com.android.support:recyclerview-v7:23.2.0'

使用步骤一般如下：


        recyclerView = (RecyclerView) findViewById(R.id.recycler);
        recyclerView.setLayoutManager(new LinearLayoutManager(this, LinearLayoutManager.VERTICAL, false));//设置竖直方向的线性manager
        //recyclerView.addItemDecoration(new MyItemDecoration());//添加divider，非必须
        recyclerView.setAdapter(new MyRecycleAdapter());//设置adapter
        
   值得一提的是，原来我们熟悉的ViewHolder终于被v7库封装了，adapter实现类如下：
   
     class MyRecycleAdapter extends RecyclerView.Adapter {
        @Override
        public RecyclerView.ViewHolder onCreateViewHolder(ViewGroup parent, int viewType) {

            MyViewHolder holder = new MyViewHolder(LayoutInflater.from(RecyclerViewDemo.this).inflate(R.layout.recycler_item, parent, false));
            return holder;
        }

        @Override
        public void onBindViewHolder(RecyclerView.ViewHolder holder, int position) {//绑定数据

        }

        @Override
        public int getItemCount() {
            return 20;
        }

    }
    
recycler_item布局文件如下：

    <?xml version="1.0" encoding="utf-8"?>
    <LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:gravity="center"
        android:background="#00ff00"
        android:orientation="vertical">

        <ImageView
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:src="@drawable/logo" />

    </LinearLayout>
    
默认显示效果如下：

![image](/images/recyclerview/Snip20160625_3.png)

可以看到是没有divider分割线的，需要我们自己实现,下边研究如何实现divider。

## Divider实现
我们可以通过该方法添加分割线： 
mRecyclerView.addItemDecoration() 
该方法的参数为RecyclerView.ItemDecoration，该类为抽象类，官方目前并没有提供默认的实现类（我觉得最好能提供几个）。

该类的源码如下：

    public static abstract class ItemDecoration {

    public void onDraw(Canvas c, RecyclerView parent, State state) {
                onDraw(c, parent);
     }


    public void onDrawOver(Canvas c, RecyclerView parent, State state) {
                onDrawOver(c, parent);
     }

    public void getItemOffsets(Rect outRect, View view, RecyclerView parent, State state) {
                getItemOffsets(outRect, ((LayoutParams)         view.getLayoutParams()).getViewLayoutPosition(),
                    parent);
    }

    @Deprecated
    public void getItemOffsets(Rect outRect, int itemPosition, RecyclerView parent) {
            outRect.set(0, 0, 0, 0);
     }
     
 当我们调用mRecyclerView.addItemDecoration()方法添加decoration的时候，RecyclerView在绘制的时候，会去绘制decorator，即调用该类的onDraw和onDrawOver方法，
 
 onDraw方法先于drawChildren
 
 onDrawOver在drawChildren之后，一般我们选择复写其中一个即可。
 
 getItemOffsets 可以通过outRect.set()为每个Item设置一定的偏移量，主要用于绘制Decorator。
 
 接下来我们看一个RecyclerView.ItemDecoration的实现类，该类很好的实现了RecyclerView添加分割线（当使用LayoutManager为LinearLayoutManager时），来自于v7包的sample中：
 
    public class DividerItemDecoration extends RecyclerView.ItemDecoration {

    private static final int[] ATTRS = new int[]{
            android.R.attr.listDivider
    };

    public static final int HORIZONTAL_LIST = LinearLayoutManager.HORIZONTAL;

    public static final int VERTICAL_LIST = LinearLayoutManager.VERTICAL;

    private Drawable mDivider;

    private int mOrientation;

    public DividerItemDecoration(Context context, int orientation) {
        final TypedArray a = context.obtainStyledAttributes(ATTRS);
        mDivider = a.getDrawable(0);
        a.recycle();
        setOrientation(orientation);
    }

    public void setOrientation(int orientation) {
        if (orientation != HORIZONTAL_LIST && orientation != VERTICAL_LIST) {
            throw new IllegalArgumentException("invalid orientation");
        }
        mOrientation = orientation;
    }

    @Override
    public void onDraw(Canvas c, RecyclerView parent) {
        if (mOrientation == VERTICAL_LIST) {
            drawVertical(c, parent);
        } else {
            drawHorizontal(c, parent);
        }
    }

    public void drawVertical(Canvas c, RecyclerView parent) {
        final int left = parent.getPaddingLeft();
        final int right = parent.getWidth() - parent.getPaddingRight();

        final int childCount = parent.getChildCount();
        for (int i = 0; i < childCount; i++) {
            final View child = parent.getChildAt(i);
            final RecyclerView.LayoutParams params = (RecyclerView.LayoutParams) child
                    .getLayoutParams();
            final int top = child.getBottom() + params.bottomMargin +
                    Math.round(ViewCompat.getTranslationY(child));
            final int bottom = top + mDivider.getIntrinsicHeight();
            mDivider.setBounds(left, top, right, bottom);
            mDivider.draw(c);
        }
    }

    public void drawHorizontal(Canvas c, RecyclerView parent) {
        final int top = parent.getPaddingTop();
        final int bottom = parent.getHeight() - parent.getPaddingBottom();

        final int childCount = parent.getChildCount();
        for (int i = 0; i < childCount; i++) {
            final View child = parent.getChildAt(i);
            final RecyclerView.LayoutParams params = (RecyclerView.LayoutParams) child
                    .getLayoutParams();
            final int left = child.getRight() + params.rightMargin +
                    Math.round(ViewCompat.getTranslationX(child));
            final int right = left + mDivider.getIntrinsicHeight();
            mDivider.setBounds(left, top, right, bottom);
            mDivider.draw(c);
        }
    }

    @Override
    public void getItemOffsets(Rect outRect, int itemPosition, RecyclerView parent) {
        if (mOrientation == VERTICAL_LIST) {
            outRect.set(0, 0, 0, mDivider.getIntrinsicHeight());
        } else {
            outRect.set(0, 0, mDivider.getIntrinsicWidth(), 0);
        }
    }
    }

如果设置的是GridLayoutManager，则以上divider就不能满足需求了。主要是因为它在绘制的时候，比如水平线，针对每个child的取值为：

    final int left = parent.getPaddingLeft();
    final int right = parent.getWidth() - parent.getPaddingRight();
    
因为每个Item一行，这样是没问题的。而GridLayoutManager时，一行有多个childItem，这样就多次绘制了（需要针对child逐一绘制），并且GridLayoutManager时，Item如果为最后一列（则右边无间隔线）或者为最后一行（底部无分割线）。
针对上述，我们编写了DividerGridItemDecoration：

    class DividerGridItemDecoration extends RecyclerView.ItemDecoration {

        private static final int[] ATTRS = new int[]{android.R.attr.listDivider};
        private Drawable mDivider;

        public DividerGridItemDecoration(Context context) {
            final TypedArray a = context.obtainStyledAttributes(ATTRS);
            mDivider = a.getDrawable(0);
            a.recycle();
        }

        @Override
        public void onDraw(Canvas c, RecyclerView parent, RecyclerView.State state) {

            drawHorizontal(c, parent);
            drawVertical(c, parent);

        }

        private int getSpanCount(RecyclerView parent) {
            // 列数
            int spanCount = -1;
            RecyclerView.LayoutManager layoutManager = parent.getLayoutManager();
            if (layoutManager instanceof GridLayoutManager) {

                spanCount = ((GridLayoutManager) layoutManager).getSpanCount();
            } else if (layoutManager instanceof StaggeredGridLayoutManager) {
                spanCount = ((StaggeredGridLayoutManager) layoutManager)
                        .getSpanCount();
            }
            return spanCount;
        }

        public void drawHorizontal(Canvas c, RecyclerView parent) {
            int childCount = parent.getChildCount();
            for (int i = 0; i < childCount; i++) {
                final View child = parent.getChildAt(i);
                final RecyclerView.LayoutParams params = (RecyclerView.LayoutParams) child
                        .getLayoutParams();
                final int left = child.getLeft() - params.leftMargin;
                final int right = child.getRight() + params.rightMargin
                        + mDivider.getIntrinsicWidth();
                final int top = child.getBottom() + params.bottomMargin;
                final int bottom = top + mDivider.getIntrinsicHeight();
                mDivider.setBounds(left, top, right, bottom);
                mDivider.draw(c);
            }
        }

        public void drawVertical(Canvas c, RecyclerView parent) {
            final int childCount = parent.getChildCount();
            for (int i = 0; i < childCount; i++) {
                final View child = parent.getChildAt(i);

                final RecyclerView.LayoutParams params = (RecyclerView.LayoutParams) child
                        .getLayoutParams();
                final int top = child.getTop() - params.topMargin;
                final int bottom = child.getBottom() + params.bottomMargin;
                final int left = child.getRight() + params.rightMargin;
                final int right = left + mDivider.getIntrinsicWidth();

                mDivider.setBounds(left, top, right, bottom);
                mDivider.draw(c);
            }
        }

        private boolean isLastColum(RecyclerView parent, int pos, int spanCount,
                                    int childCount) {
            RecyclerView.LayoutManager layoutManager = parent.getLayoutManager();
            if (layoutManager instanceof GridLayoutManager) {
                if ((pos + 1) % spanCount == 0)// 如果是最后一列，则不需要绘制右边
                {
                    return true;
                }
            } else if (layoutManager instanceof StaggeredGridLayoutManager) {
                int orientation = ((StaggeredGridLayoutManager) layoutManager)
                        .getOrientation();
                if (orientation == StaggeredGridLayoutManager.VERTICAL) {
                    if ((pos + 1) % spanCount == 0)// 如果是最后一列，则不需要绘制右边
                    {
                        return true;
                    }
                } else {
                    childCount = childCount - childCount % spanCount;
                    if (pos >= childCount)// 如果是最后一列，则不需要绘制右边
                        return true;
                }
            }
            return false;
        }

        private boolean isLastRaw(RecyclerView parent, int pos, int spanCount,
                                  int childCount) {
            RecyclerView.LayoutManager layoutManager = parent.getLayoutManager();
            if (layoutManager instanceof GridLayoutManager) {
                childCount = childCount - childCount % spanCount;
                if (pos >= childCount)// 如果是最后一行，则不需要绘制底部
                    return true;
            } else if (layoutManager instanceof StaggeredGridLayoutManager) {
                int orientation = ((StaggeredGridLayoutManager) layoutManager)
                        .getOrientation();
                // StaggeredGridLayoutManager 且纵向滚动
                if (orientation == StaggeredGridLayoutManager.VERTICAL) {
                    childCount = childCount - childCount % spanCount;
                    // 如果是最后一行，则不需要绘制底部
                    if (pos >= childCount)
                        return true;
                } else
                // StaggeredGridLayoutManager 且横向滚动
                {
                    // 如果是最后一行，则不需要绘制底部
                    if ((pos + 1) % spanCount == 0) {
                        return true;
                    }
                }
            }
            return false;
        }

        @Override
        public void getItemOffsets(Rect outRect, int itemPosition,
                                   RecyclerView parent) {
            int spanCount = getSpanCount(parent);
            int childCount = parent.getAdapter().getItemCount();
            if (isLastRaw(parent, itemPosition, spanCount, childCount))// 如果是最后一行，则不需要绘制底部
            {
                outRect.set(0, 0, mDivider.getIntrinsicWidth(), 0);
            } else if (isLastColum(parent, itemPosition, spanCount, childCount))// 如果是最后一列，则不需要绘制右边
            {
                outRect.set(0, 0, 0, mDivider.getIntrinsicHeight());
            } else {
                outRect.set(0, 0, mDivider.getIntrinsicWidth(),
                        mDivider.getIntrinsicHeight());
            }
        }
    }
主要在getItemOffsets方法中，去判断如果是最后一行，则不需要绘制底部；如果是最后一列，则不需要绘制右边，整个判断也考虑到了StaggeredGridLayoutManager的横向和纵向，所以稍稍有些复杂。最重要还是去理解，如何绘制什么的不重要。

一般如果仅仅是希望有空隙，还是去设置item的margin方便。
## Adapter实现
Adapter需要继承RecyclerView内部定义的Adapter，和BaseAdapter类似，最简单情况下实现以下几个方法即可：

    class MyRecycleAdapter extends RecyclerView.Adapter {
        /**
         * 绑定holder中的view
         *
         * @param parent
         * @param viewType
         * @return
         */
        @Override
        public RecyclerView.ViewHolder onCreateViewHolder(ViewGroup parent, int viewType) {

            View item = LayoutInflater.from(RecyclerViewDemo.this).inflate(R.layout.recycler_item, parent, false);
            MyViewHolder holder = new MyViewHolder(item);
            holder.imageView = (ImageView) item.findViewById(R.id.itemimage);
            return holder;
        }

        @Override
        public void onBindViewHolder(RecyclerView.ViewHolder holder, int position) {
            if (position % 2 == 0) {
               ViewGroup.LayoutParams params = holder.itemView.getLayoutParams();
                params.height=params.height/2;
                holder.itemView.setLayoutParams(params);
            }
        }

        @Override
        public int getItemCount() {
            return 20;
        }

    }
 除此之外，还有getItemViewType等方法，大部分方法和BaseAdapter是一一对应的。

## LayoutManage使用
有3个实现类，类继承结构如下：

![image](/images/recyclerview/Snip20160625_2.png)

从名字上很好理解：

**LinearLayoutManger**：线性布局，支持水平和竖直两个方向，可通过参数指定；

**GridLayoutManager**：类似于gridview的布局方式；

    recyclerView.setLayoutManager(new GridLayoutManager(this,3));//3列的gridview

**StaggeredGridLayoutManager**：瀑布流效果的布局方式，其实他可以实现GridLayoutManager一样的功能，仅仅按照下列代码：

    / mRecyclerView.setLayoutManager(new GridLayoutManager(this,3));
        mRecyclerView.setLayoutManager(new StaggeredGridLayoutManager(3,        StaggeredGridLayoutManager.VERTICAL));
        
这两种写法显示的效果是一致的，但是注意StaggeredGridLayoutManager构造的第二个参数传一个orientation，如果传入的是StaggeredGridLayoutManager.VERTICAL代表有多少列；那么传入的如果是StaggeredGridLayoutManager.HORIZONTAL就代表有多少行，比如本例如果改为：

    mRecyclerView.setLayoutManager(new StaggeredGridLayoutManager(4,
        StaggeredGridLayoutManager.HORIZONTAL));

    
 那么效果为： 
  
  ![image](http://img.blog.csdn.net/20150415150125431)
  
  可以看到，固定为4行，变成了左右滑动。有一点需要注意，如果是横向的时候，item的宽度需要注意去设置，毕竟横向的宽度没有约束了，应为控件可以横向滚动了。
  
  如果让你去实现个瀑布流，最起码不是那么随意就可以实现的吧？但是，如果使用RecyclerView，分分钟的事。 
那么如何实现？其实你什么都不用做，只要使用StaggeredGridLayoutManager我们就已经实现了，只是上面的item布局我们使用了固定的高度，下面我们仅仅在适配器的onBindViewHolder方法中为我们的item设置个随机的高度（代码就不贴了，最后会给出源码下载地址），看看效果图：

  ![image](http://img.blog.csdn.net/20150415193645985)
  
通过RecyclerView去实现ListView、GridView、瀑布流的效果基本上没有什么区别，而且可以仅仅通过设置不同的LayoutManager即可实现。还有更nice的地方，就在于item增加、删除的动画也是可配置的。接下来看一下ItemAnimator。

  


## ItemAnimator
ItemAnimator也是一个抽象类，好在系统为我们提供了一种默认的实现类，期待系统多 
添加些默认的实现。

借助默认的实现，当Item添加和移除的时候，添加动画效果很简单:

     // 设置item动画
    mRecyclerView.setItemAnimator(new DefaultItemAnimator());
系统为我们提供了一个默认的实现，我们为我们的瀑布流添加以上一行代码，效果为：

![image](http://img.blog.csdn.net/20150415194114149)

如果是GridLayoutManager呢？动画效果为：

 ![image](http://img.blog.csdn.net/20150415150130083)

注意，这里更新数据集不是用adapter.notifyDataSetChanged()而是 
notifyItemInserted(position)与notifyItemRemoved(position) 
否则没有动画效果。 当然了只提供了一种动画，那么我们肯定可以去自定义各种nice的动画效果。 
高兴的是，github上已经有很多类似的项目了，这里我们直接引用下：RecyclerViewItemAnimators，大家自己下载查看。 
提供了SlideInOutLeftItemAnimator,SlideInOutRightItemAnimator, 
SlideInOutTopItemAnimator,SlideInOutBottomItemAnimator等动画效果。

## Click and LongClick
系统没有提供ClickListener和LongClickListener。 
不过我们也可以自己去添加，只是会多了些代码而已。 
实现的方式比较多，你可以通过mRecyclerView.addOnItemTouchListener去监听然后去判断手势， 
当然你也可以通过adapter中自己去提供回调，这里我们选择后者，前者的方式，大家有兴趣自己去实现。

那么代码也比较简单：

    class HomeAdapter extends RecyclerView.Adapter<HomeAdapter.MyViewHolder>
    {

    //...
    public interface OnItemClickLitener
    {
        void onItemClick(View view, int position);
        void onItemLongClick(View view , int position);
    }

    private OnItemClickLitener mOnItemClickLitener;

    public void setOnItemClickLitener(OnItemClickLitener mOnItemClickLitener)
    {
        this.mOnItemClickLitener = mOnItemClickLitener;
    }

    @Override
    public void onBindViewHolder(final MyViewHolder holder, final int position)
    {
        holder.tv.setText(mDatas.get(position));

        // 如果设置了回调，则设置点击事件
        if (mOnItemClickLitener != null)
        {
            holder.itemView.setOnClickListener(new OnClickListener()
            {
                @Override
                public void onClick(View v)
                {
                    int pos = holder.getLayoutPosition();
                    mOnItemClickLitener.onItemClick(holder.itemView, pos);
                }
            });

            holder.itemView.setOnLongClickListener(new OnLongClickListener()
            {
                @Override
                public boolean onLongClick(View v)
                {
                    int pos = holder.getLayoutPosition();
                    mOnItemClickLitener.onItemLongClick(holder.itemView, pos);
                    return false;
                }
            });
        }
    }
    //...
    }
 adapter中自己定义了个接口，然后在onBindViewHolder中去为holder.itemView去设置相应 
的监听最后回调我们设置的监听。Activity中去设置监听：

    mAdapter.setOnItemClickLitener(new OnItemClickLitener()
        {

            @Override
            public void onItemClick(View view, int position)
            {
                Toast.makeText(HomeActivity.this, position + " click",
                        Toast.LENGTH_SHORT).show();
            }

            @Override
            public void onItemLongClick(View view, int position)
            {
                Toast.makeText(HomeActivity.this, position + " long click",
                        Toast.LENGTH_SHORT).show();
                        mAdapter.removeData(position);
            }
        });
        
整个体验下来，感觉这种插拔式的设计太棒了，如果系统再能提供一些常用的分隔符，多添加些动画效果就更好了。

通过简单改变下LayoutManager，就可以产生不同的效果，那么我们可以根据手机屏幕的宽度去动态设置LayoutManager，屏幕宽度一般的，显示为ListView；宽度稍大的显示两列的GridView或者瀑布流（或者横纵屏幕切换时变化，有点意思~）；显示的列数和宽度成正比。甚至某些特殊屏幕，让其横向滑动~~再选择一个nice的动画效果，相信这种插件式的编码体验一定会让你迅速爱上RecyclerView。



  