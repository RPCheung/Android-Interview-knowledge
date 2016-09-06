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

### 软件版本号的更新 ：
	首先调用getPackageManager()获取PackageManager	然后PackageManager对象调用getPackageInfo()获得PackageInfo对象	接着PackageInfo对象获取versionCode属性	最后获取服务器最新版本号与versionCode属性作比较