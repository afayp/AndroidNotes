---
layout:     post
title:      "ViewPager知识点"
date:       2016-03-19 20:33:07
author:     "afayp"
catalog:    true
tags:
    - Android
---




# 基本使用

ViewPager这个类可以让用户左右切换当前的view。ViewPager类直接继承ViewGroup类，是一个容器，可以往其中装view对象或者fragment。
<!--more-->

ViewPager类需要一个适配器类给它提供数据。有三种：
- PageAdapter--数据源：List<view>
- FragmentPagerAdapter--数据源：List<Fragment>
- FragmentStatePagerAdapter--数据源：List<Fragment>



先看一个简单使用的栗子：

1.引入布局xml代码
```java
<android.support.v4.view.ViewPager  
    android:id="@+id/viewpager"  
    android:layout_width="wrap_content"  
    android:layout_height="wrap_content"  
    android:layout_gravity="center" />  
```
2.Activity代码
```java
public class MainActivity extends Activity {  
  
    private View view1, view2, view3;  
    private ViewPager viewPager;  //对应的viewPager  
      
    private List<View> viewList;//view数组  
     
     
    @Override  
    protected void onCreate(Bundle savedInstanceState) {  
        super.onCreate(savedInstanceState);  
        setContentView(R.layout.activity_main);  
          
        viewPager = (ViewPager) findViewById(R.id.viewpager);  
        LayoutInflater inflater=getLayoutInflater();  
        view1 = inflater.inflate(R.layout.layout1, null);  
        view2 = inflater.inflate(R.layout.layout2,null);  
        view3 = inflater.inflate(R.layout.layout3, null);  
          
        viewList = new ArrayList<View>();// 将要分页显示的View装入数组中  
        viewList.add(view1);  
        viewList.add(view2);  
        viewList.add(view3);  
          
          
        PagerAdapter pagerAdapter = new PagerAdapter() {  
              
            @Override  
            public boolean isViewFromObject(View arg0, Object arg1) {  
                return arg0 == arg1;  //就直接这么写
            }  
              
            @Override  
            public int getCount() {  
                return viewList.size();//数据源大小  
            }  
              
            @Override  
            public void destroyItem(ViewGroup container, int position,  
                    Object object) {  
                container.removeView(viewList.get(position));  
            }  
              
            @Override  
            public Object instantiateItem(ViewGroup container, int position) {  
                // TODO Auto-generated method stub  
                container.addView(viewList.get(position));  
                return viewList.get(position);  
            }  
        };           
        viewPager.setAdapter(pagerAdapter);          
    }    
}  
```

对于PageAdapter，必须重写的四个函数：

> boolean isViewFromObject(View arg0, Object arg1)  
int getCount()   
void destroyItem(ViewGroup container, int position,Object object)  
Object instantiateItem(ViewGroup container, int position)  

至于每个函数怎么实现，参考上面的栗子。



至于FragmentPagerAdapter，只需要重写getItem(int)和getCount()就可以了。  
如果要处理大量的页面切换，建议使用FragmentStatePagerAdapter。
```java
	mFragments = new ArrayList<Fragment>();
	Fragment mTab01 = new Fragment01();
	Fragment mTab02 = new Fragment02();
	Fragment mTab03 = new Fragment03();
	Fragment mTab04 = new Fragment04();
	mFragments.add(mTab01);
	mFragments.add(mTab02);
	mFragments.add(mTab03);
	mFragments.add(mTab04);
	mAdapter = new FragmentPagerAdapter(getSupportFragmentManager()) {//或者getFragmentManager 
		@Override
		public int getCount() {
			return mFragments.size();
		}
		@Override
		public Fragment getItem(int arg0) {
			return mFragments.get(arg0);
		}
	};
	mViewPager.setAdapter(mAdapter);
```
这里要注意统一导android.app.Fragment或者android.support.v4.app.Fragment包，前者用getFragmentManager，后者用getSupportFragmentManager


常用的两个方法：

```java
mViewPager.getCurrentItem();
mViewPager.setCurrentItem(i);//定位到指定item
```

# 监听


```java
mViewPager.setOnPageChangeListener(new ViewPager.OnPageChangeListener() {
    @Override
    public void onPageScrolled(int position, float positionOffset, int positionOffsetPixels) {
    }
    @Override
    public void onPageSelected(int position) {
        int currentItem = mViewPager.getCurrentItem();
        switch (currentItem) {
            case 0:
					//doSomething
            break;
			case 1:
					//doSomething
            break;
        }
    }
    @Override
    public void onPageScrollStateChanged(int state) {

    }
});
```

onPageScrolled中的三个参数比较重要。第一个参数是position。它的含义是表示当前显示的界面中的第一个界面。意思就是的当滑动的时候，有可能出现两个界面，position指的是左边的界面。第二个参数是positionOffset指的是偏移量的比例，取值范围是[0, 1)。第三个参数是positionOffsetPixels是指偏移的像素值。后两个参数都相对页面（一个page）来说的。



# PagerTitleStrip/PagerTabStrip


另外设计到的两个类分别是PagerTitleStrip类(viewpager的标题 )和PagerTabStrip类(一个viewpager的指示器，效果就是一个横的粗的下划线)。  
PagerTitleStrip类直接继承自ViewGroup类，而PagerTabStrip类继承PagerTitleStrip类，所以这两个类也是容器类。  
但是有一点需要注意，在定义XML的layout的时候，这两个类必须是ViewPager标签的子标签，不然会出错。如果两个都写了，那么用了title，tab会失效。
PagerTitleStrip没有指示器（下面的一条粗线），只有标题，且标题没有响应事件（无法点击跳转）；而PagerTabStrip是带有指示器的，当然也有标题，具有相应事件。

两个类用法完全相同，下面举个例子：

xml代码：
```java
<android.support.v4.view.ViewPager  
    android:id="@+id/viewpager"  
    android:layout_width="wrap_content"  
    android:layout_height="200dip"  
    android:layout_gravity="center">  
      
    <android.support.v4.view.PagerTitleStrip  
        android:id="@+id/pagertitle"    
        android:layout_width="wrap_content"    
        android:layout_height="wrap_content"    
        android:layout_gravity="top"  
        />  
      
</android.support.v4.view.ViewPager>  
```
我们将PagerTitleStrip将其作为ViewPager的子控件直接嵌入其中；这是第一步；当然android:layout_gravity=""的值要设置为top或bottom。将标题栏显示在顶部或底部。

Activity代码（重点是重写适配器的getPageTitle（）函数）：
```java
	@Override  
    public CharSequence getPageTitle(int position) {  
        // TODO Auto-generated method stub  
        return titleList.get(position);  
    }  
```
跟第一个类似，只是在PagerAdapter中多实现了一个方法。

## 更改PagerTabStrip样式

主要靠PagerTabStrip的setXxx()方法；
```java
tab = (PagerTabStrip) findViewById(R.id.pagertab);  
tab.setTabIndicatorColorResource(R.color.green); //下划线颜色
tab.setTabIndicatorColor(Color.BLUE);
tab.setBackgroundColor(Color.YELLOW); //背景
tab.setDrawFullUnderline(false);//取消tab下面的长横线
tab.setTextColor(Color.RED);//字体颜色
```

## 在Title前添加图片

重写适配器CharSequence getPageTitle(int)方法时，我们不返回String对象，而是返回一个SpannableStringBuilder对象。



用PagerTitleStrip/PagerTabStrip实现的效果太丑，开发中一般不用。。

# 自定义指示条

待撸<http://blog.csdn.net/harvic880925/article/details/38557517>



# 切换动画

详细看<http://blog.csdn.net/lmj623565791/article/details/40411921/>


ViewPager有个方法叫做：
setPageTransformer(boolean reverseDrawingOrder, PageTransformer transformer) 用于设置ViewPager切换时的动画效果。

# TabLayout和ViewPager联用

在设配器里重写getPageTitle()方法，然后tabLayout.setupWithViewPager(viewPager)即可



# Fragment中嵌套ViewPager，ViewPager里多个Fragment

<http://blog.csdn.net/mybook1122/article/details/24003343>

# ViewPager实现引导页



# 参考链接

<http://blog.csdn.net/harvic880925/article/details/38453725>