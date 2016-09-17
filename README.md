# Android 面试过的知识点：
### Activity的生命周期：
	
	onCreate ——>onStar ——> onResume
	onStop ——> onRestart ——> onStar（Activity未被销毁的情况  onStop ——> onCreate  （Activity 被销毁的情况）
	onPause ——> onResume  （新的打开的Activity 被设置成透明主题）
	
	onDestroy<——onStop <——onPause### Fragment 的泄漏 ：** 原因 ：当Fragment的对象引用了静态变量或者被定义为静态对象时造成内存泄漏 **
** 解决方法 ： **
	
	1. 尽量减少对Context和Fragment等有生命周期管理的对象使用static关键字
	2. 如果Fragment要引用到静态对象，则可以对Fragment使用弱引用 
### Activity启动模式应用场景 :
singleTop的应用场景 ：

	登录成功跳转到主页，按下两次登录按钮，生成了两个主页。一些有启动延迟的页面（往往是动画，网络造成）也会有这样的情况，为了防止两个重复的Activity，把Activity设置成singleTop
	
		还有一种场景 从activity A启动了个service进行耗时操作，或者某种监听，这个时候你home键了，service收集到信息，要返回activityA了，就用singleTop启动，实际不会创建新的activityA，只是resume了。不过使用standard又会创造2个A的实例。
		
singleTask的应用场景 ：

	singleTask适合作为程序入口点。例如浏览器的主界面。不管从多少个应用启动浏览器，只会启动主界面一次，其余情况都会走onNewIntent，并且会清空主界面上面的其他页面。
	
singleInstance的应用场景 ：
		
		singleInstance适合需要与程序分离开的页面。例如闹铃提醒，将闹铃提醒与闹铃设置分离。singleInstance不要用于中间页面，如果用于中间页面，跳转会有问题，比如：A -> B (singleInstance) -> C，完全退出后，在此启动，首先打开的是B。

### Activity的onNewIntent()方法何时会被调用 ：

		当ActivityA的LaunchMode为SingleTop时，如果ActivityA在栈顶,且现在要再启动ActivityA，这时会调用onNewIntent()方法

	当ActivityA的LaunchMode为SingleInstance,SingleTask时,如果已经ActivityA已经在堆栈中，那么此时会调用onNewIntent()方法

	当ActivityA的LaunchMode为Standard时，由于每次启动ActivityA都是启动新的实例，和原来启动的没关系，所以不会调用原来ActivityA的onNewIntent方法
	### MVP ：
** 优点：为了明确每个模块的功能，实现模块间的高内聚低耦合 **
	
	M 为 Model 是对数据的查找、储存、增加、删除；
	V 为 View 是与用户交互的 （一般都由Activity 、Fragment来充当）；
	P 为 Presenter 是View 与Model的纽带 （提供 View Interface让View实现，提供Model Interface让Model实现）；Model层抽象接口：
	public interface IInfoModel {
    	//从数据提供者获取数据方法
    	InfoBean getInfo();
    	//存入数据提供者方法
    	void setInfo(InfoBean info);
	}
	
Model层抽象实现：

	public class InfoModelImpl implements IInfoModel {
    	//模拟存储数据
    	private InfoBean infoBean = new InfoBean();

    	@Override
    	public InfoBean getInfo() {
        //模拟存储数据，真实有很多操作
        	return infoBean;
    	}

    	@Override
    	public void setInfo(InfoBean info) {
        	//模拟存储数据，真实有很多操作
        	infoBean = info;
    	}
	}
	
View层的抽象接口：
	
	public interface IInfoView {
    	//给UI显示数据的方法
    	void setInfo(InfoBean info);
    	//从UI取数据的方法
    	InfoBean getInfo();
	}
	
Presenter的实现：
	
	public class Presenter {
    	private IInfoModel infoModel;
    	private IInfoView infoView;

    	public Presenter(IInfoView infoView) {
        	this.infoView = infoView;

        	infoModel = new InfoModelImpl();
    	}
    	//供UI调运
    	public void saveInfo(InfoBean bean) {
        	infoModel.setInfo(bean);
    	}
    	//供UI调运
    	public void getInfo() {
        	//通过调用IInfoView的方法来更新显示，设计模式运用
        	//类似回调监听处理
        	infoView.setInfo(infoModel.getInfo());
    	}
	}

View层的实现 ：
	
	public class MainActivity extends ActionBarActivity implements IInfoView,View.OnClickListener{
	
		private void initData() {
        	presenter = new Presenter(this);
        	
        	infoTxt = (TextView) findViewById(R.id.show);(各种初始化控件)
        	saveBtn.setOnClickListener(this);（各种监听的设置）
        	....
        }
        
        @Override
    	public void setInfo(InfoBean info) {
    		
    		设置数据...
    	}
    	
    	@Override
    	public InfoBean getInfo() {
    	
    		获取数据...
       	}
       	
       	@Override
    	public void onClick(View v) {
    	
    		 switch (v.getId()) {
            	case R.id.XXXX:
                	presenter.saveInfo(getInfo());
                break;
            	case R.id.XXXXX:
                	presenter.getInfo();
               	break;
        	}
    	}
	}
	### 布局的优化 ：1. ** ``` 布局重用<include/> ``` **
2. **  ``` 减少视图层级<merge/>  ：``` **多用于替换FrameLayout或者当一个布局包含另一个时，<merge/> 标签消除视图层次结构中多余的视图组。例如你的主布局文件是垂直布局，引入了一个垂直布局的include，这是如果include布局使用的LinearLayout就没意义了，使用的话反而减慢你的UI表现。这时可以使用<merge/>标签优化 

3. ** ``` 需要时使用<ViewStub/> ： ``` **
： <ViewStub/> 标签最大的优点是当你需要时才会加载，使用他并不会影响UI初始化时的性能。各种不常用的布局想进度条、显示错误消息等可以使用<ViewStub />标签，以减少内存使用量，加快渲染速度。<ViewStub />是一个不可见的，大小为0的View。
** ``` 注：ViewStub目前有个缺陷就是还不支持 <merge/> 标签。``` **
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
    
#### 属性动画 ：
*** ``` 属性动画的原理： ``` ***
	
		属性动画要求动画作用的对象提供该属性的get和set方法，属性动画根据你传递的该熟悉的初始值和最终值，以动画的效果多次去调用set方法，每次传递给set方法的值都不一样，确切来说是随着时间的推移，所传递的值越来越接近最终值。
** 总结一下，你对object的属性xxx做动画，如果想让动画生效，要同时满足两个条件： **

*** ``` 1. object必须要提供setXxx方法，如果动画的时候没有传递初始值，那么还要提供getXxx方法，因为系统要去拿xxx属性的初始值（如果这条不满足，程序直接Crash）``` ***

*** ```2. object的setXxx对属性xxx所做的改变必须能够通过某种方法反映出来，比如会带来ui的改变啥的（如果这条不满足，动画无效果但不会Crash） ```  ***

** 以上条件缺一不可 **

##### ``` 实现: ```

** 1. 用一个类来包装原始对象，间接为其提供get和set方法 : **

		private void performAnimate() {  
    		ViewWrapper wrapper = new ViewWrapper(mButton);  
    		ObjectAnimator.ofInt(wrapper, "width", 500).setDuration(5000).start();  
    	}  
    	
    	@Override 
    	public void onClick(View v) {  
    		if (v == mButton) {  
        		performAnimate();  
    		}  
    	}  
    	private static class ViewWrapper {  
    		private View mTarget;  
  
    		public ViewWrapper(View target) {  
        		mTarget = target;  
    		}  
  
    		public int getWidth() {  
        		return mTarget.getLayoutParams().width;  
    		}  
  
    		public void setWidth(int width) {  
        		mTarget.getLayoutParams().width = width;  
        		mTarget.requestLayout();  
    		}  
    	}  
** 2. 采用ValueAnimator，监听动画过程，自己实现属性的改变 : **

	private void performAnimate(final View target, final int start, final int end) {  
    ValueAnimator valueAnimator = ValueAnimator.ofInt(1, 100);  
  
    valueAnimator.addUpdateListener(new AnimatorUpdateListener() {  
  
        //持有一个IntEvaluator对象，方便下面估值的时候使用  
        private IntEvaluator mEvaluator = new IntEvaluator();  
  
        @Override  
        public void onAnimationUpdate(ValueAnimator animator) {  
            //获得当前动画的进度值，整型，1-100之间  
            int currentValue = (Integer)animator.getAnimatedValue();  
            Log.d(TAG, "current value: " + currentValue);  
  
            //计算当前进度占整个动画过程的比例，浮点型，0-1之间  
            float fraction = currentValue / 100f;  
  
            //这里我偷懒了，不过有现成的干吗不用呢  
            //直接调用整型估值器通过比例计算出宽度，然后再设给Button  
            target.getLayoutParams().width = mEvaluator.evaluate(fraction, start, end);  
            target.requestLayout();  
        }  
    });  
  
    	valueAnimator.setDuration(5000).start();  
    
    }
      
    @Override  
    public void onClick(View v) {  
    	if (v == mButton) {  
        	performAnimate(mButton, mButton.getWidth(), 500);  
    	}  
    }  

### WebView的缓存 ：
1. 有IO把HTML文件储存到手机 ：
		
		1. 定义一个离线下载的服务Service
		2. 启动后台服务Service来执行异步下载
		3. 存储到本地缓存文件夹或用DiskLruCache
		4. 每一次加载url之前，先判断数据库是否存在缓存内容
		5. 如果存在缓存，优先加载本地缓存，如果不，在，才执行联网请求(mWebView.loadUrl("file:///android_cache/01.html"))
	[DiskLruCache详情请点击](http://blog.csdn.net/guolin_blog/article/details/28863651)
2. 用webView的自带缓存 ：
		
		mWebView.getSettings().setRenderPriority(RenderPriority.HIGH);
        // 建议缓存策略为，判断是否有网络，有的话，使用LOAD_DEFAULT,无网络时，使用LOAD_CACHE_ELSE_NETWORK
 
        mWebView.getSettings().setCacheMode(WebSettings.LOAD_CACHE_ELSE_NETWORK); // 设置缓存模式
        // 开启DOM storage API 功能
        mWebView.getSettings().setDomStorageEnabled(true);
        // 开启database storage API功能
        mWebView.getSettings().setDatabaseEnabled(true);
        String cacheDirPath = getFilesDir().getAbsolutePath()+ APP_CACHE_DIRNAME;
       
        // 设置数据库缓存路径
        mWebView.getSettings().setDatabasePath(cacheDirPath); // API 19 deprecated
        // 设置Application caches缓存目录
        mWebView.getSettings().setAppCachePath(cacheDirPath);
        // 开启Application Cache功能
        mWebView.getSettings().setAppCacheEnabled(true);

### View的拖动 ：
1. 如果你将滑动后的目标位置的坐标传递给layout()，这样子就会把view的位置给重新布置了一下，在视觉上就是view的一个滑动的效果。
public
		
		public class DragView extends View{
		
  			private int lastX;
  			private int lastY;
  			
  			public DragView(Context context, AttributeSet attrs) {
    			super(context, attrs);
  			}
  			
  			public boolean onTouchEvent(MotionEvent event) {
  			
    			//获取到手指处的横坐标和纵坐标
    			int x = (int) event.getX();
    			int y = (int) event.getY();
    			
    			switch(event.getAction()){
    			
      				case MotionEvent.ACTION_DOWN:
      				
        				lastX = x;
        				lastY = y;
        				
      				break;
      				
      				case MotionEvent.ACTION_MOVE:
        			
        				//计算移动的距离
        				int offX = x - lastX;
        				int offY = y - lastY;
        			
        				//调用layout方法来重新放置它的位置
        				layout(getLeft()+offX, getTop()+offY,getRight()+offX  , getBottom()+offY);
          			
      				break;
    			}
    		
    			return true;
  			}
		} 
		先记录一下down的坐标，然后记录move的坐标，相减得到偏移量，再与getLeft()、getTop()、
		getRight()、getBottom() 得到相对位置，调用layout();
		
### 实现自定义全局异常捕捉并重启应用的类 :
	
	1. 自定义CrashHandler类实现UncaughtExceptionHandler接口 （用单例保持唯一对象）
	2. 创建Thread.UncaughtExceptionHandler变量 mDefaultHandler
	
	//获取系统默认的UncaughtException处理器 
	3. mDefaultHandler = Thread.getDefaultUncaughtExceptionHandler();
	
	//设置该CrashHandler为程序的默认处理器  
    4. Thread.setDefaultUncaughtExceptionHandler(this);
   
    5. 自定义uncaughtException(Thread thread, Throwable ex)方法
    
    6. 自定义Application获取 CrashHandler
    
    7.注册Application

### Android子线程更新UI （正常开发不建议）:
	奥秘在于ViewRoot的建立时间，它是在 ActivityThread.java的 final void
	handleResumeActivity(IBinder token, boolean clearHide, boolean isForward)
	里创建的
	
	答案就是在Activity.onResume前，ViewRoot实例没有建立，所以没有ViewRoot.checkThread检查。
	而btn.setText时设定的文本却保留了下来，所以当ViewRoot真正去刷新界面时，
	就把"TestThread2.run"刷了出来！

### Android 五种储存方式 ：
1. 文件储存（文件IO流） ：
		
		FileOutputStream out = openFileOutput("data",Context.MODE_PRIVATE);
		ButteredWriter writer = new ButteredWriter(new OutputSreamWriter(out));
		writer.write(data);
2. SharedPreferences ：
		
		默认存储路径：/data/data/<PackageName>/shared_prefs.xml
		
		获取SharedPreferences对象的方法 :
		
			1. Context的getSharedPreferences()方法，参数一是文件名，参数二是操作模式
			2. Activity的getPreferences()方法，参数为操作模式，使用当前应用程序包名为文件名
			3. PreferenceManager的getDefaultSharedPreferences()静态方法，接收Context参数，使用当前应用程序包名为文件名

		存储数据 :
		
			1. 调用SharedPreferences对象的edit()方法获取一个SharedPreferences.Editor对象
			2. 向Editor对象中添加数据putBoolean、putString等
			3. 调用commit()方法提交数据

3. SQLite数据库存储 :
		
		默认存储路径：/data/data/<PackageName>/databases
		数据类型 : integer 整型 real 浮点 text 文本类型 blob 二进制类型
		使用方法 ：继承SQLiteOpenHelper，重写构造方法和 onCreate
		升级数据库时调用 onUpgrade(SQLiteDatabase db, int oldVersion, int newVersion)方法
		例 ：
		
			@Override  
    		public void onUpgrade(SQLiteDatabase db, int oldVersion, int newVersion) {   
          		db.execSQL("drop table if exists Book");
          		onCreate(db):
    		} 
    	在MainActivity中只需将version改为大于原来版本号即可 ：
    	
    		MyDatabaseHelper helper = new MyDatabaseHelper(this,"BookStore.db",null,2);

4. ContentProvider ：
		
		1. 继承ContentProvider重写增删该查方法(若多表操作用UriMatcher)
			private static final String AUTHORITY = "com.scott.provider.PersonProvider";  
    		private static final int PERSON_ALL = 0;  
    		private static final int PERSON_ONE = 1;
    		
    		matcher = new UriMatcher(UriMatcher.NO_MATCH);  
          
        	matcher.addURI(AUTHORITY, "path（一般是表名）", PERSON_ALL);   //匹配记录集合  
        	matcher.addURI(AUTHORITY, "path", PERSON_ONE);
        	
        	 switch (match) {
        	   
        	case PERSON_ALL:  
            
            break;  
        	case PERSON_ONE:  
                       
            break;  
        default:  
            throw new IllegalArgumentException("Unknown URI: " + uri);  
        }  
	  
		2. 在AndroidManifest.xml注册 ：
		
			 <application android:icon="@drawable/icon" android:label="@string/app_name">
			 		<provider  android:authorities="com.rp.mycontentprovider"
			 					android:name=".MyContentProvider"
			 					android:multiprocess="true"
            					android:exported="true"/>
         	</application>
        3. 用ContentResolver解析数据 ：
        	
        	 private static final Uri PERSON_ALL_URI = Uri.parse("content://" + AUTHORITY + "/persons"); 
        	 增删该查方法中传入PERSON_ALL_URI
5. 网络存储

### 支付SDK ：

### 数组与链表的区别 ：
1. 数组 ：

		1. 数组是将元素在内存中连续存放，由于每个元素占用内存相同，可以通过下标迅速访问数组中任何元素
		2. 如果想删除一个元素，需要移动大量元素去填掉被移动的元素
		3. 所以应用需要快速访问数据，很少或不插入和删除元素，就应该用数组
		
2. 链表 ：
		
		1. 链表恰好相反，链表中的元素在内存中不是顺序存储的，而是通过存在元素中的指针联系到一起
		2. 如果要访问链表中一个元素，需要从第一个元素开始，一直找到需要的元素位置，但是增加和删除一个元素对于链表数据结构就非常简单了，只要修改元素中的指针就可以了
		3. 如果应用需要经常插入和删除元素你就需要用链表数据结构了
		
### CharSequence与String的关系 ：
	CharSequence 是 char 值的一个序列接口，String 继承于CharSequence，除了String实现了CharSequence之外，StringBuffer(线程安全的)和StringBuilder(非线程安全的)也实现了CharSequence接口
	
### HashMap的底层模型 ：
	HashMap底层模型是一个链表的数组，数组的元素是一个Entry拥有一个Entry.next的模拟指针，先通过key的hashcode判断数组位置，为null则插入，不为空则遍历链表插入
	
### Android字体设置 ：
方法一：XML中使用android默认字体 ：
	
	在TextView中使用Android:typeface属性加载 Android 系统默认支持三种字体，分别为：“sans”, “serif”, “monospace"
	
方法二：在Android中可以引入其他字体 ：
	
	首先要将字体文件保存在assets/fonts/目录下
	然后 Typeface typeFace =Typeface.createFromAsset(getAssets(),"fonts/HandmadeTypewriter.ttf")
	最后 textView.setTypeface(typeFace)
	
### Android四层架构 ：
![](https://github.com/RPCheung/Android-Interview-knowledge/blob/master/4.jpg)
 
### 软件版本号的更新 ：
	首先调用getPackageManager()获取PackageManager	然后PackageManager对象调用getPackageInfo()获得PackageInfo对象	接着PackageInfo对象获取versionCode属性	最后获取服务器最新版本号与versionCode属性作比较