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
       
    adjustPan：当前窗口的内容将自动移动以便当前焦点从不被键盘覆盖，用户总能看到输入内容的部分。
