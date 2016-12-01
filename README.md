# 为什么要屏幕适配？ #
![碎片化](img/device_framentation.png)

## 统计 ##
* [OpenSignal](https://opensignal.com/reports/2015/08/android-fragmentation/)
* [友盟统计](http://www.umindex.com/)

## 碎片化 ##

* 品牌机型碎片化
* 屏幕尺寸碎片化
* 操作系统碎片化

为了保证用户获得一致的用户体验效果，使得某一元素在Android不同尺寸、不同分辨率的手机上具备相同的
显示效果，则需要我们进行屏幕适配。

# 基础概念 #

## 屏幕尺寸 ##
屏幕尺寸是指屏幕对角线的长度，单位是英寸，1 inch=2.54 cm

## 屏幕分辨率 ##
手机在横向和纵向上的像素点数总和，单位是像素（pixel)，1px = 1像素点，
举个栗子，1080x1920，即宽度方向上有1080个像素点，在高度方向上有1920个像素点。

## 屏幕像素密度 ##
每英寸像素点个数，单位是dpi，dots per inch。
为简便起见，Android 将所有屏幕密度分组为六种通用密度： 低、中、高、超高、超超高和超超超高。

* ldpi（低）~120dpi
* mdpi（中）~160dpi
* hdpi（高）~240dpi
* xhdpi（超高）~320dpi
* xxhdpi（超超高）~480dpi
* xxxhdpi（超超超高）~640dpi

![屏幕密度计算](img/dpi_example.png)

## 屏幕密度无关像素dp(dip) ##
Density Independent Pixels，即密度无关像素

* 160dpi, 1dp = 1px
* 240dpi, 1dp = 1.5px
* 320dpi, 1dp = 2px
* 480dpi, 1dp = 3px
* 640dpi, 1dp = 4px

### 使用px在低、中、高屏幕密度下的效果###
![不支持不同密度](img/density-test-bad.png)

### 使用dp在低、中、高屏幕密度下的效果 ###
![支持不同密度](img/density-test-good.png)

## 独立比例像素sp ##
Scale Independent Pixels, 即sp或sip。
Android开发时用此单位设置文字大小，可根据字体大小首选项进行缩放，推荐使用12sp、14sp、18sp、22sp作为字体设置的大小，不推荐使用奇数和小数，容易造成精度的丢失问题,
小于12sp的字体会太小导致用户看不清。

# 屏幕适配之图片适配 #
![屏幕密度比例](img/screens-densities.png)

在设计图标时，对于5种主流的像素密度(mdpi,hdpi,xhdpi,xxhdpi和xxxdpi)应按照
2:3:4:6:8的比例进行缩放。例如一个启动图片ic_launcher.png,它在各个像素密度文件夹下大小为：

* ldpi（低）
* mdpi（中）48*48
* hdpi（高）72*72
* xhdpi（超高）96*96
* xxhdpi（超超高）144*144
* xxxhdpi（超超超高）192*192

## 存在的问题 ##
* 每套分辨率出一套图，为美工或者设计增加了许多工作量
* 对Android工程文件的apk包变的很大

## 解决方法 ##
### Android SDK加载图片流程 ###
1. Android SDK会根据屏幕密度自动选择对应的资源文件进行渲染加载，比如说，SDK检测到你手机的分辨率是xhdpi，会优先到xhdpi文件夹下找对应的图片资源；
2. 如果xhdpi文件夹下没有图片资源，那么就会去分辨率高的文件夹下查找，比如xxhdpi，直到找到同名图片资源，将它按比例缩小成xhpi图片；
3. 如果往上查找图片还是没有找到，那么就会往低分辨率的文件夹查找，比如hdpi，直到找到同名图片资源，将它按比例放大成xhpi图片。

**根据加载图片的流程，可以得出理论上提供一套图片就可以了。**

###那么应该提供哪种分辨率规格呢？###

**原则上越高越好，同时结合当前主流分辨率屏幕**

## 自动拉伸图片 ##
![.9图片设计](img/ninepatch_raw.png)

![,9图片效果](img/ninepatch_examples.png)

# 屏幕适配之布局适配 #
## 布局参数 ##
使用wrap_content, match_parent, layout_weight。

### weight的使用 ###
![weight](img/weight_examples.png)

* 当layout_width为0dp，layout_weight分别是1和2	

		<LinearLayout
	        android:layout_width="match_parent"
	        android:layout_height="wrap_content"
	        android:orientation="horizontal">
	
	        <Button
	            android:layout_width="0dp"
	            android:layout_height="wrap_content"
	            android:layout_weight="1"
	            android:text="weight = 1"/>
	
	        <Button
	            android:layout_width="0dp"
	            android:layout_height="wrap_content"
	            android:layout_weight="2"
	            android:text="weight = 2"/>
	    </LinearLayout>

* 当layout_width为match_parent,layout_weight分别为1和2

	    <LinearLayout
	        android:layout_width="match_parent"
	        android:layout_height="wrap_content"
	        android:orientation="horizontal">
	
	        <Button
	            android:layout_width="match_parent"
	            android:layout_height="wrap_content"
	            android:layout_weight="1"
	            android:text="weight = 1"/>
	
	        <Button
	            android:layout_width="match_parent"
	            android:layout_height="wrap_content"
	            android:layout_weight="2"
	            android:text="weight = 2"/>
	    </LinearLayout>

### weight的计算 ###
宽度 = 原来宽度 + 权重比值 * 剩余宽度

* 当layout_width为0dp，layout_weight分别是1和2	

	第一个按钮：宽度 = 0 + 1/3 * 屏宽 = 1/3屏宽

	第二个按钮：宽度 = 0 + 2/3 * 屏宽 = 2/3屏宽
* 当layout_width为match_parent, layout_weight分别是1和2
	
	第一个按钮：宽度 = 屏宽 + 1/3 * (屏宽 - 2 * 屏宽) = 2/3屏宽

	第二个按钮：宽度 = 屏宽 + 2/3 * (屏宽 - 2 * 屏宽) = 1/3屏宽

## 布局使用 ##
使用相对布局，禁用绝对布局。

## 限定符 ##

### 尺寸限定符 ###
* 在手机较小的屏幕上，加载layout文件夹布局
* 在平板电脑和电视的屏幕（>7英寸）上， 加载layout-large文件夹的布局
* Android3.2版本之前

### 最小宽度限定符 ###
* 在手机较小的屏幕上，加载layout文件夹布局
* 标准7英寸平板（其最小宽度为 600 dp），加载layout-sw600dp文件夹的布局
* 在Android3.2版本及之后版本

### 布局别名 ###

* 适配手机的单面板（默认）布局：res/layout/activity_main.xml
* 适配尺寸>7寸平板的双面板布局（Android 3.2前）：res/layout-large/activity_main.xml
* 适配尺寸>7寸平板的双面板布局（Android 3.2后）：res/layout-sw600dp/activity_main.xml

最后的两个文件的xml内容是完全相同的，这会带来：文件名的重复从而带来一些列后期维护的问题，修改一个文件，
可能忘记修改另外一个。**于是为了要解决这种重复问题，我们引入了布局别名**。

* 适配手机的单面板（默认）布局：res/layout/activity_main.xml
* 适配尺寸>7寸平板的双面板布局：res/layout/activity_twopanes.xml


* res/values/layout.xml

		<?xml version="1.0" encoding="utf-8"?>
		<resources>
		    <item name="main" type="layout">@layout/activity_main</item>
		</resources>

* res/values-large/layout.xml

	
		<?xml version="1.0" encoding="utf-8"?>
		<resources>
		    <item name="main" type="layout">@layout/activity_twopanes</item>
		</resources>

* res/values-sw600dp/layout.xml

		<?xml version="1.0" encoding="utf-8"?>
		<resources>
		    <item name="main" type="layout">@layout/activity_twopanes</item>
		</resources>

* setContentView(R.layout.main);


### 屏幕方向限定符 ###
* res/layout-land
* res/layout-port
* res/layout-sw600dp-land
* res/layout-sw600dp-port

# 屏幕适配之dimen适配 #
* Nexus 4 (4.7英寸 768x1280:xhdpi)

	![dimen1](img/dimen_example1.png)

* Nexus S (4英寸 480x800:hdpi)

	![dimen1](img/dimen_example2.png)

即使使用dp，依然不能解决屏幕分辨率的适配问题，我们可以针对不同的屏幕创建不同的dimen值。

* res/values/dimens.xml

		<resources>
	    <dimen name="button_length_1">180dp</dimen>
	    <dimen name="button_length_2">160dp</dimen>
		</resources>

* res/values-480x800/dimens.xml

		<resources>
		    <dimen name="button_length_1">113dp</dimen>
		    <dimen name="button_length_2">100dp</dimen>
		</resources>


# 屏幕适配之百分比布局 #
* [官方文档](https://developer.android.com/topic/libraries/support-library/features.html#percent)
* [Github Sample](https://github.com/JulienGenoud/android-percent-support-lib-sample)

		<?xml version="1.0" encoding="utf-8"?>
		<android.support.percent.PercentRelativeLayout
		    xmlns:android="http://schemas.android.com/apk/res/android"
		    xmlns:app="http://schemas.android.com/apk/res-auto"
		    android:layout_width="match_parent"
		    android:layout_height="wrap_content">
		
		    <Button
		        android:layout_width="0dp"
		        android:layout_height="wrap_content"
		        android:text="30%"
		        app:layout_widthPercent="30%"/>
		
		    <Button
		        android:layout_width="0dp"
		        android:layout_height="wrap_content"
		        android:layout_alignParentRight="true"
		        android:text="20%"
		        app:layout_widthPercent="20%"/>
		
		</android.support.percent.PercentRelativeLayout>


# 屏幕适配之自适应用户界面 #
![NewsReader横屏](img/newsreader_land.png)

![NewsReader竖屏](img/newsreader_port.png)

当NewsReader在横屏时是双面板，左侧是HeadLinesFragment, 右侧是ArticleFragment, 点击新闻标题, 切换ArticleFragment的内容。
当NewsReader在竖屏时是单面板，只有个HeadLinesFragment, 点击新闻标题，跳转到ArticleActivity去显示新闻内容。所以，要实现这样的横竖屏适配，只是通过布局是完成不了的，
不同业务逻辑的处理，还需要写代码来完成，这就是我们的自适应用户界面。

## 使用布局别名 ##
* res/values/layouts.xml

		<resources>
	    <item name="main_layout" type="layout">@layout/onepane_with_bar</item>
	    <bool name="has_two_panes">false</bool>
		</resources>

* res/values-sw600dp-land/layouts.xml
	
		<resources>
	    <item name="main_layout" type="layout">@layout/twopanes</item>
	    <bool name="has_two_panes">true</bool>
		</resources>

* res/values-sw600dp-port/layouts.xml

		<resources>
		    <item name="main_layout" type="layout">@layout/onepane</item>
		    <bool name="has_two_panes">false</bool>
		</resources>

## 判断是单面板还是双面板 ##
	View articleView = findViewById(R.id.article);
	mIsDualPane = articleView != null && articleView.getVisibility() == View.VISIBLE;//如果能够找到ArticleFragment则是双面板

## 单双面板的不同业务逻辑 ##
    public void onHeadlineSelected(int index) {
        mArtIndex = index;
        if (mIsDualPane) {
            // display it on the article fragment
            mArticleFragment.displayArticle(mCurrentCat.getArticle(index));
        }
        else {
            // use separate activity
            Intent i = new Intent(this, ArticleActivity.class);
            i.putExtra("catIndex", mCatIndex);
            i.putExtra("artIndex", index);
            startActivity(i);
        }
    }

# 参考 #
本文部分文字和图片直接摘录自以下内容：

* [Android官网](https://developer.android.com/guide/practices/screens_support.html)
* [Android开发：最全面、最易懂的Android屏幕适配解决方案](http://www.jianshu.com/p/ec5a1a30694b)