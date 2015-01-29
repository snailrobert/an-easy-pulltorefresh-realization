# an-easy-pulltorefresh-realization
Android PullToRefresh的简易实现原理，主要是分析，便于定制修改和以后自己实现

一、定制PullToRefresh的思路（主要是针对https://github.com/chrisbanes/Android-PullToRefresh）：
  1. 实现自动上拉加载更多：
      实现OnLastItemVisibleListener即可。
      
  2. 若子View是ListView，则当ListView添加header时对手动下拉刷新的影响：
	  当手动下拉刷新时，是否显示loading view由isFirstItemVisible()这个方法决定。而系统默认的条件是ListView的第一个子View的高度判断（top），所以当存在ListView的HeaderView的时候要能够下拉刷新就需要修改该方法。

  3. 若子View是ListView，则当ListView添加footer时对手动上拉加载更多的影响：
	  当手动上拉加载更多时，是否显示loading view由isLastItemVisible()这个方法决定。而系统默认的条件是ListView的最后一个子View的高度判断(bottom)，所以当存在ListView的FooterView的时候要能够下拉刷新就需要修改该方法。

(以上修改均在PullToRefreshBase中即可)

  4. 在PullToRefresh中，所有的header view和footer view都继承自LoadingLayout，所以如果要修改loading view的样式可以到这边修改。比如说在refresh时，loading view上显示一个人正在喝水等各种动画效果。

  5. PullToRefresh也提供了横向的效果，按照这种效果，我们可以很easy的实现左右bounce回弹的效果，当然上下也可以（由LinearLayout的orientation决定）。实现，只要将loading view内容置空即可，设置一定的宽度或者高度。

  6. PullToRefresh还提供了上拉或者下拉时添加音乐效果。。。


二、实现原理：

	1、PullToRefreshBase扩展于LinearLayout，垂直排列
　　2、从上到下的顺序是：Header, Content, Footer
　　3、Content填充满父控件，通过设置top, bottom的padding来使Header和Footer不可见，也就是让它超出屏幕外（通过在PullToRefreshBase中直接调用setPadding(0,-y,0,y)即可，y分别为header view和footer view的高度height）
　　4、下拉时，调用scrollTo方法来将整个布局向下滑动，从而把Header显示出来，上拉正好与下拉相反（主要通过onInceptTouchEvent()和onTouchEvent()捕获move等事件实现，当event为down、move的时候scrollTo，当up的时候onRefresh()，refreshComplete()后scroll回来）。
　　5、派生类需要实现的是：将Content View填充到父容器中，比如，如果你要使用的话，那么你需要把ListView, ScrollView, WebView等添加到容器中。
　　
　　注1：其中第三点主要是在view的onSizeChanged()中实现的。执行顺序:
　　View的构造函数——》finishInflate()——》onMeasure()——》onSizeChanged()——》onLayout()——》onDraw()
　　注2：onSizeChange并不一定会调用，只有View的大小发生变化才会调用，而且也不一定一定从root开始调用。onMeasure在整个界面上需要放置一样东西或拿掉一样东西时会调用。比如addView就是放置，removeview就是拿掉，另外比较特殊的是，child设置为gone会触发onMeasure，但是invisible不会触发onMeasure。一旦执行过onMeasure，往往就会执行onLayout来重新布局
　　注3：从onSizeChanged()想到的，只要Activity设置属性：android:windowSoftInputMode = "adjustResize" ，当软键盘弹出时就会自动调用onSizeChanged()方法，从而确保整体布局上移，避免按钮被遮挡住的问题。

主要参考: http://blog.csdn.net/leehong2005/article/details/12567757

	http://www.tuicool.com/articles/M3qiue


