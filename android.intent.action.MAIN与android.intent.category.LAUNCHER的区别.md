## android.intent.action.MAIN 与 android.intent.category.LAUNCHER的区别

- android.intent.action.MAIN决定一个应用程序最先启动那个组件
- android.intent.category.LAUNCHER决定应用程序是否显示在程序列表里(说白了就是是否在桌面上显示一个图标)

这两个属性组合情况：

- 第一种情况：有MAIN,无LAUNCHER，程序列表中无图标

	原因：android.intent.category.LAUNCHER决定应用程序是否显示在程序列表里

- 第二种情况：无MAIN,有LAUNCHER，程序列表中无图标

	原因：android.intent.action.MAIN决定应用程序最先启动的Activity，如果没有Main，则不知启动哪个Activity，故也不会有图标出现

所以这两个属性一般成对出现。
如果一个应用中有两个组件intent-filter都添加了android.intent.action.MAIN和
android.intent.category.LAUNCHER这两个属性，则这个应用将会显示两个图标，写在前面的组件先运行。
