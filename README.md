# workSummarize
用于工作中遇到的问题的一些总结，以及一些个人觉得好用的工具类

- 遇到的问题
1.布局中EditText抢占焦点的解决办法：  在外层父布局中添加属性

` android:focusable = "true"       android:focusableInTouchMode = "true" //可以通过触摸获得焦点`

2.ListView子项中包含CheckBox 点击事件失效的问题解决办法： 

  【1】在item根布局添加 ` android:descendantFocusability = "blocksDescendants" `

descendantFocusability 有三个属性：
  
  beforeDescendants : viewGroup会优先于子控件获取到焦点
     
  afterDescendants : viewGroup只有当子控件不需要获取焦点时才获取焦点
       
  blocksDescendants : viewGroup 会覆盖子类控件直接获取焦点
  
  【2】checkBox 设置  android:focusable = "false"
