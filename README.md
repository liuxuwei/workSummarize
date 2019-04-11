# workSummarize
用于工作中遇到的问题的一些总结，以及一些个人觉得好用的工具类

- 遇到的问题

#### 1.布局中EditText抢占焦点的解决办法：  在外层父布局中添加属性

` android:focusable = "true"       android:focusableInTouchMode = "true" //可以通过触摸获得焦点`

#### 2.ListView子项中包含CheckBox 点击事件失效的问题解决办法： 

  【1】在item根布局添加 ` android:descendantFocusability = "blocksDescendants" `

       descendantFocusability 有三个属性：
  
          beforeDescendants : viewGroup会优先于子控件获取到焦点
      
          afterDescendants : viewGroup只有当子控件不需要获取焦点时才获取焦点
       
          blocksDescendants : viewGroup 会覆盖子类控件直接获取焦点
  
  【2】checkBox 设置  android:focusable = "false"

#### 3.对于ListView 或 RecyclerView 中子项有两种状态的控件， if/else 必须把两种状态都指定，如果缺少else，可能在ViewHolder复用的时候出现状态混乱。

#### 4.android:windowSoftInputMode  主窗口与软键盘的交互模式，可以用来避免输入法面板遮挡问题。这是Android 1.5之后的一个新特性，这个属性能影响两件事情： 

  【一】当有焦点产生时，软键盘是隐藏还是显示
  
  【二】是否减少活动主窗口以便腾出空间放软键盘
  
    这个属性中包含的值：
   
    stateUnspecified：软键盘的状态没有指定，系统将选择一个合适的状态或者依赖于主题的设置。
    
    stateUnchanged：当新的Activity出现时，软键盘将一直保持在上一个Activity中的状态，无论是隐藏还是显示。
       
    stateHidden：用户选择Activity时，软键盘总是隐藏。
       
    stateAlwaysHidden：当该Activity主窗口获取焦点时，软键盘也总是隐藏。
       
    stateVisible：软键盘通常是可见的。
       
    stateAlwaysVisible：当用户选择Activity时，软键盘总是可见的。
       
    adjustUnspecified：默认设置，通常由系统自行决定隐藏还是显示。
       
    adjustResize：该Activity总是调整屏幕大小以便留出软键盘的空间。
       
    adjustPan：当前窗口的内容将自动移动以便当前焦点从不被键盘覆盖，用户总能看到输入内容的部分。、
    
#### 5.adjustResize 失效问题

在Activity中设置了

` getWindow().getDecorView().setSystemUiVisibility(View.SYSTEM_UI_FLAG_LAYOUT_FULLSCREEN)`  

后adjustResize会失效（Android 5497 Bug）。

解决方法： Android5497BugWorkaround.java 见代码文件。 在Activity的onCreate()方法中使用

` Android5497BugWorkaround.assistActivity(this); `

#### 6.ScrollView 嵌套 ListView或者 GridView 的问题。
 
 ScrollView嵌套 ListView 或 GridView，ListView 和 GridView 会出现只显示一条数据的情况。
    
 解决办法：继承ListView或GridView，重写onMesure() 方法。
     
 自定义view 或者 viewGroup时，使用int型的MesureSpec表示一个组件的大小。这个MesureSpec里不仅有组件的大小，还有大小的测量模式。系统组件的大小模式中有三种：
   
   精确模式（MesureSpec.EXACTLY） 这种模式下，尺寸的值是多少，这个组件的长或宽就是多少。
      
   最大模式（MesureSpec.AT_MOST） 这个也就是父控件能够给出的最大空间，当前组件的长或宽最大只能为这么大，也可以比这个小。
      
   未指定模式（MesureSpec.UNSPECIFIED）当前控件可以随便使用空间 不受限制。
      
   一个int类型有32位，测量模式有3种，要表示三种状态，至少得2位二进制位。系统采用最高的2位表示模式。
   
   最高两位是00代表未指定模式， 01代表精确模式，11代表最大模式。

   重写onMeasure()方法关键代码：
   
   ``` java 
   int expandSpec = MeasureSpec.makeMeasureSpec(Integer.MAX_VALUE >> 2, MeasureSpec.AT_MOST);  
   super.onMeasure(widthMeasureSpec,expandSpec);
   ```

#### 7.DrawerLayout穿透问题：

  问题描述：弹出侧滑菜单后，点击空白地方可以穿透点击到被遮挡的内容。 
  
  解决办法：如果侧滑菜单布局在xml中是通过include引入的，在include引用的xml文件根节点加 ` android:clickable = "true"`
  
#### 8.防止双击重复跳转:

  解决办法：在BaseActivity中重写startActivityForResult()方法。
  
  ``` java
     private String mActivityJumpTag;
     private long mActivityJumpTime;
     
     @Override
     public void startActivityForResult(Intent intent, int requestCode, @Nullable Bundle options){
             if(startActivitySelfCheck(intent)){
                   //查看源码得知 startActivity 最终也会调用 startActivityForResult
                   super.startActivityForResult(intent,requestCode,options);
             }
     }

    protected boolean startActivitySelfCheck(Intent intent){
            //默认检查通过
            boolean result = true; 
            //标记对象
            String tag;
            if (intent.getComponent() != null){      //显式跳转
                    tag = intent.getComponent().getClassName();
            } else if (intent.getAction() != null){     //隐式跳转
                    tag = intent.getAction();     
            } else {
                    return result;
            } 
                 
                   if (tag.equals(mActivityJumpTag) && mActivityJumpTime
                                                              >= SystemClock.uptimeMills() - 500) {
                      return false;
             }
             //记录启动标记和时间
             mActivityJumpTag = tag;
             mActivityJumpTime = SystemClock.uptimeMills(); 
                   return result;       
    }
    
 ```
 
#### 9.防止修改系统字体大小，app字体大小改变。

   解决办法：在Application中重写getResources()方法
   
   ``` java
     @Override 
     public Resources getResources() {
           Resources res = super.getResourcesf();
           if (res.getConfiguration().fontScale != 1){
                Configuration  newConfig = new Configuration();
                newConfig.setToDefaults();
                res.updateConfiguration(newConfiguration, res.getDisplayMetrics());
           }
           return res;
     }
   ```
   
#### 10.设置禁止横竖屏切换 在代码中设置会有小问题。

   问题描述：在代码中设置 ` setRequestedOrientation(ActivityInfo.SCREEN_ORIENTATION_PORTRAIT)` 后，虽然打开Activity后旋转手机不会导致横竖屏切换，但是将手机先横屏再进行跳转还是会发生横竖屏切换（在代码中设置Activity为竖屏）。
   
   解决方法： 在AndroidManifest.xml中设置 ` screenOrientation = "portrait"`
<<<<<<< HEAD

#### 11.学习RxJava时遇到的编译错误
 ![报错图片](https://github.com/liuxuwei/workSummarize/raw/master/rxjavaError.png)
 
 解决方法：在Module模块下的 build.gradle中android 节点下加入
 ``` groove
 packagingOptions {
        exclude 'META-INF/rxjava.properties'
    }
 ```
 
 #### 12.三星手机拍照后，照片可能被旋转的问题
 
 #### 13.配置config.gradle时候，粗心犯下的一个错误
 
 ![groovy错误](https://raw.githubusercontent.com/liuxuwei/workSummarize/master/groovyError.png)
 
 原因：配置config.gradle时，格式应该为： compileSdkVersion : 28      自己粗心写成了 = 号

#### 14.git本地仓库和远程仓库合并时提示： refusing to merge unrelated histories

 错误重现步骤：在本地先创建了一个本地仓库，然后把自己的项目建好。 同时github上也建立了一个远程仓库。 
 
 在将本地代码提交的时候提示：
 
 > Updates were rejected because the tip of your current branch is behind its remote counterpart. 
 
 然后使用`git pull origin master` 时提示 > refusing to merge unrelated histories. 
 
 解决方法： 先使用`git pull origin master --allow-unrelated-histories`
           然后使用 `git push origin master: master`  (意思为：将本地的master分支推送到远程master分支)
           
#### 15.配置了config.gradle后，引入某个库编译报错

![报错图片](https://github.com/liuxuwei/workSummarize/blob/master/iconError.png?raw=true)

解决方法： 在manifest文件中，application节点下添加 `tools:replace="android:icon"`

#### 16.在kotlin中使用单元测试，引入自定义rule报错：

![报错图片](https://github.com/liuxuwei/workSummarize/blob/master/junitError.png?raw=true)

解决方法： 在 @rule 注解后添加 @JvmField

#### 17.在junit单元测试中，使用RxJava2切换线程时报错，如图：

![报错图片](https://github.com/liuxuwei/workSummarize/blob/master/junitError2.png?raw=true)

在stackoverflow上找到的解决办法：在 @Before 标记的方法中添加 

`RxAndroidPlugins.setInitMainThreadSchedulerHandler(scheduler -> Schedulers.trampoline());`

#### 18.Android P禁止明文传输，使用明文传输会crash：

![报错图片](https://github.com/liuxuwei/workSummarize/blob/master/clearTextError.png?raw=true)

解决办法：在AndroidManifest文件application节点添加

`android:usesCleartextTraffic="true" `

#### 19.Android在App中吊起QQ进行联系

实现代码：

``` kotlin
   try {
      val url = "mqqwpa://im/chat?chat_type=wpa&uin=$qqNum"
      startActivity(Intent(Intent.ACTION_VIEW,Uri.parse(url)))
   } catch (e: Exception) {
      toast("请先安装QQ")
   }
```

#### 20.ViewPager结合Fragment，在Fragment中弹出输入法，切换Fragment时，输入法无法隐藏

解决方法：

在KeyBoardUtil中强制隐藏输入法的函数：

``` kotlin
    fun hideInputKeyBoard(context: Context){
	val activity = context as Activity
	activity.runOnUiThread{
	    val mInputManager = context.getSystemService(Context.INPUT_METHOD_SERVICE) as InputMethodManager
	    if (activity.currentFocus != null) {
		mInputManager.hideSoftInputFromWindow(activity.currentFocus.windowToken,InputMethodManager.HIDE_NOT_ALWAYS)
		activity.window.setSoftInputMode(WindowManager.LayoutParams.SOFT_INPUT_STATE_HIDDEN)
	    }
	}
    }
```

然后在ViewPager的addOnPageChangeListener()方法的 onPageScrolled()方法中进行调用。

#### 21.WebView设置了 webview.getSettings().setBuiltInZoomControls(true)以后，用户触摸屏幕会出现放大缩小图标，如果在图标自动消失之前退出当前的Activity，会报异常。

异常如图：
       ![报错图片](https://github.com/liuxuwei/workSummarize/blob/master/webViewZoomButtonError.png?raw=true)

解决办法：自定义WebView（尝试网上的其他办法：比如在onDestroy中设置View.GONE， 在finish方法中removeAllViews() 在我的程序中不起作用）

自定义WebView的代码（来自tackOverflow）

``` java
public class CustomWebView extends WebView{

    private ZoomButtonsController zoom_controll = null;

    public CustomWebView(Context context) {
        this(context, null);
    }

    public CustomWebView(Context context, AttributeSet attrs) {
        this(context, attrs, android.R.attr.webViewStyle);
    }

    public CustomWebView(Context context, AttributeSet attrs, int defStyleAttr) {
        super(context, attrs, defStyleAttr);
        initWebSettings();
    }

    @SuppressLint("NewApi")
    private void initWebSettings() {
        WebSettings webSettings = getSettings();

        webSettings.setSupportZoom(true);
        webSettings.setBuiltInZoomControls(true);
        if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.HONEYCOMB) {
            webSettings.setAllowContentAccess(true);
            webSettings.setDisplayZoomControls(false);

        } else {
            try {
                Class webview = Class.forName("android.webkit.WebView");
                Method method = webview.getMethod("getZoomButtonsController");
                zoom_controll = (ZoomButtonsController) method.invoke(this, null);
            } catch (Exception e) {
                e.printStackTrace();
            }
        }
    }


    @Override
    public boolean onTouchEvent(MotionEvent event) {
        if(zoom_controll != null)
            zoom_controll.getZoomControls().setVisibility(View.GONE);
        return super.onTouchEvent(event);
    }

}
```

#### 22.Service是运行在主线程中的
 
 在启动Service的Activity的onCreate()方法， 和Service的onCreate()方法中同时打印日志： 
 
 ```java
 Log.i(TAG,"onCreate ThreaId:"+android.os.Process.myTid());
 ```
 
 会发现线程id是一样的，由此证实Service是运行在主线程中的

#### 23.使用bindService将Activity与Service绑定，退出Activity时，如果未调用unBindService()方法，会报错：

![报错图片](https://github.com/liuxuwei/workSummarize/blob/master/ServiceConnectionLeaked.png?raw=true)

#### 24.报错信息：Could not determine artifacts for com.umeng.umsdk:common:1.5.4: No cached version available for offline mode
 
解决方法：打开Settings-> Build,Execution,Deployment->Gradle，把offlin work的勾去掉。

#### 25.生成APK时，统一APK名称的方式 ， 在build.gradle文件中 Android{}中添加以下代码：

```groove
  applicationVariants.all { variant -> 
	variant.outputs.all {
		outputFileName = "项目名称_${variant.versionName}_${variant.name}.apk"
	}
  }
```

 


