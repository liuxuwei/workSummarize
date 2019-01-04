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
 ![报错图片](data:image/png;base64,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)
 
 解决方法：在Module模块下的 build.gradle中android 节点下加入
 ``` groove
 packagingOptions {
        exclude 'META-INF/rxjava.properties'
    }
 ```



