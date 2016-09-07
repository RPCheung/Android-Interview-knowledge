# Android 面试过的知识点：
### Activity的生命周期：
	
	onCreate ——>onStar ——> onResume
	onStop ——> onRestart ——> onStar（Activity未被销毁的情况  onStop ——> onCreate  （Activity 被销毁的情况）
	onPause ——> onResume  （新的打开的Activity 被设置成透明主题）
	
	onDestroy<——onStop <——onPause### Fragment 的泄漏 ：** 原因 ：当Fragment的对象引用了静态变量或者被定义为静态对象时造成内存泄漏 **
** 解决方法 ： **
1. 尽量减少对Context和Fragment等有生命周期管理的对象使用static关键字
2.  如果Fragment要引用到静态对象，则可以对Fragment使用弱引用 ### MVP ：
** 优点：为了明确每个模块的功能，实现模块间的高内聚低耦合 **
	
	M 为 Model 是对数据的查找、储存、增加、删除；
	V 为 View 是与用户交互的 （一般都由Activity 、Fragment来充当）；
	P 为 Presenter 是View 与Model的纽带 （提供 View Interface让View实现，提供Model Interface让Model实现）；### 布局的优化 ：1. ** ``` 布局重用<include/> ``` **
2. **  ``` 减少视图层级<merge/>  ：``` **多用于替换FrameLayout或者当一个布局包含另一个时，<merge/>标签消除视图层次结构中多余的视图组。例如你的主布局文件是垂直布局，引入了一个垂直布局的include，这是如果include布局使用的LinearLayout就没意义了，使用的话反而减慢你的UI表现。这时可以使用<merge/>标签优化 

3. ** ``` 需要时使用<ViewStub/> ： ``` **
： <ViewStub />标签最大的优点是当你需要时才会加载，使用他并不会影响UI初始化时的性能。各种不常用的布局想进度条、显示错误消息等可以使用<ViewStub />标签，以减少内存使用量，加快渲染速度。<ViewStub />是一个不可见的，大小为0的View。
** ``` 注：ViewStub目前有个缺陷就是还不支持 <merge /> 标签。``` **
[详细代码请点击查看](http://blog.csdn.net/xyz_lmn/article/details/14524567) 

### WebView调用安卓以及数据的传递 :
1. 先启用WebView对JavaScript的支持
2. 使用addJavascriptInterfac方法 传入被调用方法所在的对象 和js调用此方法的对象 
**  ```（@JavascriptInterface  解决安全性的漏洞）  ``` **
3. 创造被js调用的方法

### Android 调用js ：
	WebView使用loadURL方法直接加载js;

### 为什么要用来http传输json :
		http（超文本传输协议）是默认以tcp协议作为传输层的应用层协议，TCP传输数据具有较强稳定性，和数据纠错功能，有着耳熟能详的“三次握手”的特征;

### Gson的底层 ：
	Gson的底层是用java的反射机制对JavaBean进行遍历匹配 最终找出对应得属性
### 三级缓存 ：
		安卓中的三级缓存，在做服务器的人眼里网络根本不算缓存的一级，只能说它内存和磁盘之间存在“层”的关系;

### 线程池 :
###### 使用背景：
1. 	当需要开辟大量线程时，造成系统吞吐量过大，gc频繁出现，造成内存抖动（因频繁创造对象和回收对象产生的），从而导致界面卡顿
2. 线程过多早内存溢出（OOM）

*** ``` 解决以上问题就使用线程池ExecutorService 主要就是对线程的复用和管理线程的创建与消亡 ``` ***

### 保存视频进度的实现 ：
	在onPause阶段里调用getCurrentPosition()方法 来保存当前进度，onStar阶段调用seekTo(mCurrentPosition)方法

### VideoView seekTo不准确的解决方法 ：
1. 替换成满足需求的视频源文件（寻找合格的视频文件）。
2. 对视频源文件进行处理，增加其关键帧数量，比如可以1s设置一个关键帧（基于目前已有的视频文件进行处理）。（如果选择第二种方式，要增加视频的关键帧数量，可以推荐大家使用FFmpeg进行增加关键帧的处理工作。[详情请看ffmpeg官网](http://ffmpeg.org/)

	 *** ``` 原因 ： 其实seekTo跳转的位置其实并不是参数所带的position，而是离position最近的关键帧，而是从一个关键帧的位置开始。``` ***
	 
### Activity与Fragment通讯 ：
	1. 在Fragment里使用getActivity()
	2. 在Activity里使用getFragmentManager()

### 参数传递
	java在传递的参数时，传递的的只是对象的引用的副本，方法改变不了形参的对象，但对象的属性可以改变（副本引用也和引用指向同一对象）
	
### Android屏幕适配
##### dp、dip、dpi、sp、px之间的关系 :
	dip：Density Independent Pixels（密度无关像素）的缩写。以160dpi为基准，1dp=1px 
	dp：同dip dpi：屏幕像素密度的单位，“dot per inch”的缩写 
	px：像素，物理上的绝对单位
	sp：Scale-Independent Pixels的缩写，可以根据文字大小首选项自动进行缩放。Google推荐我们使用12sp以上的大小，通常可以使用12sp，14sp，18sp，	22sp，最好不要使用奇数和小数。

![](https://github.com/RPCheung/Android-Interview-knowledge/blob/master/1.png)
	
*** 说明：如果A设备的参数为480×320，160dpi，B设置的参数为800×480，240dpi。我们要画出一条和屏幕宽度一样长的直线，如果使用px作为单位，必须在A设备上设置为320px，在B设备上设置480px。但是如果我们使用dp作为单位，由于以160dpi为基准，1dp=1px，所以A设备上设置为320dp就等于屏幕宽度（320px），在B设备上设置为320dp就等于320×（240/160）=480px，即B设备的屏幕宽度。这样，使用dp作为单位就可以实现简单的屏幕适配。这知识一种巧合，也有B设备的像素密度不是这样刚刚好的，就需要我们运用别的屏幕适配技术。 ***
##### mdpi、hdpi、xdpi、xxdpi、xxxdpi计算和区分 :

![](https://github.com/RPCheung/Android-Interview-knowledge/blob/master/2.jpg)
![](https://github.com/RPCheung/Android-Interview-knowledge/blob/master/3.png)
*** 在Google官方开发文档中，说明了 mdpi：hdpi：xhdpi：xxhdpi：xxxhdpi=2：3：4：6：8 的尺寸比例进行缩放。例如，一个图标的大小为48×48dp，表示在mdpi上，实际大小为48×48px，在hdpi像素密度上，实际尺寸为mdpi上的1.5倍，即72×72px，以此类推。 ***

### Android 动画 ：
#### 逐帧动画 ：
*** ``` 1、先将图片素材放入res/drawable目录下 ``` ***

*** ``` 2、创建Animation-list帧布局文件，文件存放在res/drawable目录下（animation1.xml） : ``` ***
	
		<animation-list  xmlns:android="http://schemas.android.com/apk/res/android"  
		 android:oneshot="true">  
    		<item android:drawable="@drawable/icon1" android:duration="150"/>
    	</animation-list> 

*** ``` 设置播放动画的ImageView （主要将src设为 ：animation1.xml）： ``` ***

	 	<ImageView android:id="@+id/animationIV"  
            android:layout_width="wrap_content"  
            android:layout_height="wrap_content"  
            android:padding="5px"  
            android:src="@drawable/animation1"/> 
            
*** ``` 在java中初始化ImageView，设置播放动画 ： ``` ***

	animationIV = (ImageView) findViewById(R.id.animationIV);  
    animationDrawable = (AnimationDrawable) animationIV.getDrawable();  
    animationDrawable.start();  
    
#### 补间动画 ：
*** ``` AlphaAnimation：透明度（alpha）渐变效果，对应<alpha/>标签。 (alpha_demo.xml)``` ***

	<alpha xmlns:android="http://schemas.android.com/apk/res/android"  
    android:interpolator="@android:anim/accelerate_decelerate_interpolator"  
    android:fromAlpha="1.0"  
    android:toAlpha="0.1"  
    android:duration="2000"/>
      
 	<!--   fromAlpha :起始透明度  
			toAlpha:结束透明度  
 			1.0表示完全不透明  
 			0.0表示完全透明  -->  
*** ``` TranslateAnimation：位移渐变，需要指定移动点的开始和结束坐标，对应<translate/>标签。 (rotate_demo.xml) ``` ***

	<rotate xmlns:android="http://schemas.android.com/apk/res/android"  
    android:interpolator="@android:anim/accelerate_decelerate_interpolator"  
    android:fromDegrees="0"  
    android:toDegrees="360"  
    android:duration="1000"  
    android:repeatCount="1"  
    android:repeatMode="reverse"/>  
    
	<!--   
	fromDegrees:表示旋转的起始角度  
	toDegrees:表示旋转的结束角度  
	repeatCount:旋转的次数  默认值是0 代表旋转1次  如果值是repeatCount=4 旋转5次，值为-1或者infinite时，表示补间动画永不停止  
	repeatMode 设置重复的模式。默认是restart。当repeatCount的值大于0或者为infinite时才有效。  
	repeatCount=-1 或者infinite 循环了  
	还可以设成reverse，表示偶数次显示动画时会做与动画文件定义的方向相反的方向动行。--> 

*** ```此效果有两种java调用方式 ：(TranslateAnimation)``` ***
		
		TranslateAnimation translateAnimation = new TranslateAnimation(0, 200, 0, 0); 
		translateAnimation.setDuration(2000); 
    	imageView.startAnimation(translateAnimation);  


*** ``` ScaleAnimation：缩放渐变，可以指定缩放的参考点，对应<scale/>标签。 (scale_demo.xml) ``` ***

	<scale xmlns:android="http://schemas.android.com/apk/res/android"  
    android:interpolator="@android:anim/accelerate_interpolator"  
    android:fromXScale="0.2"  
    android:toXScale="1.5"  
    android:fromYScale="0.2"  
    android:toYScale="1.5"  
    android:pivotX="50%"  
    android:pivotY="50%"  
    android:duration="2000"/>  
  
	<!--   
	fromXScale:表示沿着x轴缩放的起始比例  
	toXScale:表示沿着x轴缩放的结束比例  
	fromYScale:表示沿着y轴缩放的起始比例  
	toYScale:表示沿着y轴缩放的结束比例  
	图片中心点：  
	android:pivotX="50%"   
    android:pivotY="50%"  -->  


*** ``` RotateAnimation：旋转渐变，可以指定旋转的参考点，对应<rotate/>  (translate_demo.xml)``` ***

	<translate xmlns:android="http://schemas.android.com/apk/res/android"  
    android:interpolator="@android:anim/accelerate_decelerate_interpolator"  
    android:fromXDelta="0"  
    android:toXDelta="320"  
    android:fromYDelta="0"  
    android:toYDelta="0"  
    android:duration="2000"/>   
      
	<!--   android:interpolator 动画的渲染器  
	1、accelerate_interpolator(动画加速器) 使动画在开始的时候 最慢,然后逐渐加速  
	2、decelerate_interpolator(动画减速器)使动画在开始的时候 最快,然后逐渐减速  
	3、accelerate_decelerate_interpolator（动画加速减速器）  中间位置分层:  使动画在开始的时候 最慢,然后逐渐加速使动画在开始的时候 最快,然后逐渐减速  结束的位置最慢  
	fromXDelta  动画起始位置的横坐标  
	toXDelta    动画起结束位置的横坐标  
	fromYDelta  动画起始位置的纵坐标  
	toYDelta   动画结束位置的纵坐标  
	duration 动画的持续时间  	-->  

*** ``` AnimationSet：组合渐变，支持组合多种渐变效果，对应<set/> ``` ***

	<set xmlns:android="http://schemas.android.com/apk/res/android"  
    android:interpolator="@android:anim/decelerate_interpolator"  
    android:shareInterpolator="true" >  
  
    <scale  
        android:duration="2000"  
        android:fromXScale="0.2"  
        android:fromYScale="0.2"  
        android:pivotX="50%"  
        android:pivotY="50%"  
        android:toXScale="1.5"  
        android:toYScale="1.5" />  
  
    <rotate  
        android:duration="1000"  
        android:fromDegrees="0"  
        android:repeatCount="1"  
        android:repeatMode="reverse"  
        android:toDegrees="360" />  
  
    <translate  
        android:duration="2000"  
        android:fromXDelta="0"  
        android:fromYDelta="0"  
        android:toXDelta="320"  
        android:toYDelta="0" />  
  
    <alpha  
        android:duration="2000"  
        android:fromAlpha="1.0"  
        android:toAlpha="0.1" />  
  
	</set>  
*** ``` 把图片设置到ImageView的src ，最后java中设置ImageView 代码如下 ： ``` ***

	imageView = (ImageView) findViewById(R.id.imageView1);
	Animation animation = AnimationUtils.loadAnimation(this,R.anim.alpha_demo);  
    imageView.startAnimation(animation);

### 软件版本号的更新 ：
	首先调用getPackageManager()获取PackageManager	然后PackageManager对象调用getPackageInfo()获得PackageInfo对象	接着PackageInfo对象获取versionCode属性	最后获取服务器最新版本号与versionCode属性作比较