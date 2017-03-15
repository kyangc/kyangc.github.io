title: Android M Design Support Library 学习笔记
date: 2015-07-10 20:00:00
categories: 技术
tags: [MaterialDesign,Android] 
description: Google大*好！Google大*好！Google大*好，重要的事情嗦三遍！
---

随着Android M Preview的发布，新的MD控件库Design Support Library也随之发布。新加入的几个控件方便了我们在APP中应用Material Design。学习的同时记录一下。

<!-- more -->

# 1 FAB(FloatingActionButton)

其实就是小红按钮fab，这次终于有了官方实现。

继承于`ImageView`，所以无法通过`setBackground`改变底色，通过`backgroundTint`设置背景色，通过`src`设置logo，通过`fabsize`设置大小(`Default`/`Mini`)，API21之后可以通过`elevation`设置阴影。

可以通过设置`app:layout_anchor="@id/appbar"`来与`AppBar`联动。

# 2 TextInputLayout

效果是一个始终显示Hint的`EditText`，点击获取焦点会让hint动态的移到`EditText`上方，输入错误之后会有错误信息，是个很漂亮很实用的控件。这个是个布局控件，继承于`LinearLayout`，需要和`EditText`配合使用。

关键方法：

```java
setHint()//设置Hint
setErrorEnabled()//设置是否显示错误
setError()//设置错误信息
```

# 3  Snackbar

一个介于`Dialog`和`Toast`之间的轻量级通知控件。可以提供很方便的通知和用户反馈获取。用法很像`Toast`：

```java
final Snackbar snackbar = Snackbar.make(VIEW,"测试弹出提示",Snackbar.LENGTH_LONG);
                snackbar.show();
                snackbar.setAction("取消",new View.OnClickListener() {
                    @Override
                    public void onClick(View v) {
                        snackbar.dismiss();
                    }
                });
```

VIEW可以是当前布局中任意一个控件，字体颜色使用`colorAccent`，也可以通过`setActionTextColor`改变颜色。

# 4 TabLayout

Tab的官方实现！呼卓拉！终于不用自己在瞎JB写了，感动的想哭。

这个布局继承于`HorizontalScrollView`，常用的属性和可自定义的部分很多，稍微列举一下：

```java
app:tabSelectedTextColor：Tab被选中字体的颜色
app:tabTextColor：Tab未被选中字体的颜色
app:tabIndicatorColor：Tab指示器下标的颜色
addTab(TabLayout.Tab tab, int position, boolean setSelected) 增加选项卡到 layout 中 
getTabAt(int index) 得到选项卡 
getTabCount() 得到选项卡的总个数 
getTabGravity() 得到 tab 的 Gravity 
getTabMode() 得到 tab 的模式 
getTabTextColors() 得到 tab 中文本的颜色 
newTab() 新建个 tab 
removeAllTabs() 移除所有的 tab 
removeTab(TabLayout.Tab tab) 移除指定的 tab 
removeTabAt(int position) 移除指定位置的 tab 
setOnTabSelectedListener(TabLayout.OnTabSelectedListener onTabSelectedListener) 为tab增加选择监听器 
setScrollPosition(int position, float positionOffset, boolean updateSelectedText) 设置滚动位置 
setTabGravity(int gravity) 设置 Gravity ,GRAVATY_CENTER是居中显示，GRAVITY_FILL是把TAB填充至水平位置
setTabMode(int mode) 设置 Mode,有两种值：TabLayout.MODE_SCROLLABLE和TabLayout.MODE_FIXED分别表示当tab的内容超过屏幕宽度是否支持横向水平滑动，第一种支持滑动，第二种不支持，默认不支持水平滑动。 
setTabTextColors(ColorStateList textColor) 设置 tab 中文本的颜色 
setTabTextColors(int normalColor, int selectedColor) 设置 tab 中文本的颜色 默认 选中 
setTabsFromPagerAdapter(PagerAdapter adapter) 设置 PagerAdapter 
setupWithViewPager(ViewPager viewPager) 和 ViewPager 联动
```

用法上面也很简单：

```java
viewPager = findView(R.id.viewPager);//viewPager
tabLayout = findView(R.id.tabs);//tabLayout
List<String> tabList = new ArrayList<>();//tab title list
List<Fragment> fragmentList = new ArrayList<>();//fragment list

tabList.add("Tab1");//填充title list
Fragment f1 = new TabFragment();
Bundle bundle = new Bundle();
bundle.putString("content", "http://blog.csdn.net/feiduclear_up \n CSDN 废墟的树");
f1.setArguments(bundle);
fragmentList.add(f1);//填充fragment list

TabFragmentAdapter fragmentAdapter = new TabFragmentAdapter(getSupportFragmentManager(),fragmentList, tabList);//新建适配器
viewPager.setAdapter(fragmentAdapter);//给ViewPager设置适配器

tabLayout.setTabMode(TabLayout.MODE_FIXED);//设置tab模式，
tabLayout.addTab(tabLayout.newTab().setText(tabList.get(0)));//添加tab选项卡
tabLayout.setupWithViewPager(viewPager);//将TabLayout和ViewPager关联起来。
```

# 5  AppBarLayout

继承于`LinearLayout`，作用是把其包裹的控件作为`AppBar`。多与`Toolbar`和`tabLayout`同时使用。和`CoordinateLayout`配合使用支持手势。

# 6  CoordinateLayout

`CoordinatorLayout`是一个增强型的`FrameLayout`。它的作用有两个：作为一个布局的根布局、最为一个为子视图之间相互协调手势效果的一个协调布局。

```xml
app:layout_scrollFlags="scroll|enterAlways|enterAlwaysCollapsed|exitUntilCollapsed"
```

通过该属性来确定哪个组件是可滑动的，设置的`layout_scrollFlags`有如下几种选项：

```java
scroll//所有想滚动出屏幕的view都需要设置这个flag- 没有设置这个flag的view将被固定在屏幕顶部。
enterAlways//这个flag让任意向下的滚动都会导致该view变为可见，启用快速“返回模式”。
enterAlwaysCollapsed//当你的视图已经设置minHeight属性又使用此标志时，你的视图只能已最小高度进入，只有当滚动视图到达顶部时才扩大到完整高度。
exitUntilCollapsed//滚动退出屏幕，最后折叠在顶端。
```

为了使得`Toolbar`可以滑动，我们必须还得有个条件，就是`CoordinatorLayout`布局下包裹一个可以滑动的布局，比如 `RecyclerView`，`NestedScrollView`(经过测试，`ListView`，`ScrollView`不支持)具有滑动效果的组件。并且给这些组件设置如下属性来告诉`CoordinatorLayout`，该组件是带有滑动行为的组件，然后`CoordinatorLayout`在接受到滑动时会通知`AppBarLayout`中可滑动的`Toolbar`可以滑出屏幕了。

```xml
app:layout_behavior="@string/appbar_scrolling_view_behavior"
```

**总结： 为了使得Toolbar有滑动效果，必须做到如下三点：**


>1. `CoordinatorLayout` 必须作为整个布局的父布局容器。
>2. 给需要滑动的组件设置 `app:layout_scrollFlags="scroll|enterAlways"`属性。
>3. 给你的可滑动的组件，也就是`RecyclerView`或者`NestedScrollView`设置如下属性：`app:layout_behavior="@string/appbar_scrolling_view_behavior"`


# 7  CollapsingToolbarLayout

一个可折叠的`Toolbar`，通常作为`AppbarLayout`的子视图使用。在该布局中放置`Toolbar`和`CollapsingToolbarLayout` 提供以下属性和方法是用：

```java
Collapsing title//ToolBar的标题，当CollapsingToolbarLayout全屏没有折叠时，title显示的是大字体，在折叠的过程中，title不断变小到一定大小的效果。你可以调用setTitle(CharSequence)方法设置title。
Content scrim//ToolBar被折叠到顶部固定时候的背景，你可以调用setContentScrim(Drawable)方法改变背景或者 在属性中使用 app:contentScrim=”?attr/colorPrimary”来改变背景。
Status bar scrim//状态栏的背景，调用方法setStatusBarScrim(Drawable)。还没研究明白，不过这个只能在Android5.0以上系统有效果。
Parallax scrolling children//CollapsingToolbarLayout滑动时，子视图的视觉差，可以通过属性app:layout_collapseParallaxMultiplier=”0.6”改变。
CollapseMode//子视图的折叠模式，有两种:
   pin：固定模式，在折叠的时候最后固定在顶端；
   parallax：视差模式，在折叠的时候会有个视差折叠的效果。我们可以在布局中使用属性app:layout_collapseMode来改变。
```

**总结:** `CollapsingToolbarLayout`主要是提供一个可折叠的`Toolbar`容器，对容器中的不同视图设置`layout_collapseMode`折叠模式，来达到不同的折叠效果。


>1. Toolbar 的高度`layout_height`必须固定，不能`wrap_content`，否则Toolbar不会滑动，也没有折叠效果。 
>2. 为了能让`FloatingActionButton`也能折叠且消失出现，我们必须给FAB设置锚点属性 `app:layout_anchor="@id/appbar" `意思是FAB浮动按钮显示在哪个布局区域。 
且设置当前锚点的位置 `app:layout_anchorGravity="bottom|end|right"`意思FAB浮动按钮在这个布局区域的具体位置。两个属性共同作用才是的FAB浮动按钮也能折叠消失，出现。
>3. 给需要有折叠效果的组件设置`layout_collapseMode`属性。


# 8 NavigationView

这个其实是对于以前的`DrawerLayout`中的`ListView`的整合实现。外侧布局和以前一样是`DrawerLayout`，而后直接使用这个布局作为抽屉布局。

```xml
<android.support.v4.widget.DrawerLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    android:id="@+id/drawer_layout"
    android:layout_width="match_parent"
    android:layout_height="match_parent">

    <!-- the main content view -->

    <include layout="@layout/layout_content" />

    <!-- the navigetion view -->

    <android.support.design.widget.NavigationView
        android:id="@+id/navigationView"
        android:layout_width="wrap_content"
        android:layout_height="match_parent"
        android:layout_gravity="left"
        android:fitsSystemWindows="true"
        app:headerLayout="@layout/layout_header"
        app:menu="@layout/layout_menu">

    </android.support.design.widget.NavigationView>

</android.support.v4.widget.DrawerLayout> 
```

自定义的部分比较简单，这里就不详细记录了。