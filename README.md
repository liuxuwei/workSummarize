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
   
         ``` int expandSpec = MeasureSpec.makeMeasureSpec(Integer.MAX_VALUE >> 2, MeasureSpec.AT_MOST);
         
              super.onMeasure(widthMeasureSpec,expandSpec);```
