---
layout: post
title: 第一讲 桌面GIS应用程序框架的建立
tagline: ArcGIS Engine + C# 实例开发教程
small: 系统教程
category: arcgis
tags: [GIS, ArcGIS, ArcGIS Engine, AE, C#]
---

本讲主要是使用MapControl、PageLayoutControl、ToolbarControl、TOCControl四个控件建立起基本的桌面GIS应用程序框架。最终成果预览如下：

![最终成果预览图](/images/ae/1-1.jpg)

###新建项目

启动VS2005，选择“文件|新建|项目”，在项目类型中选择Visual C#，再选择Windows应用程序模板，输入名称“3sdnMap”，点击确定。

![](/images/ae/1-2.jpg)

在解决方案管理器中将“Form1.cs”重命名为“3sdnMap.cs”，在设计视图中，选中窗体，将其属性中的“Text”改为“3sdnMap”。

###添加控件

选择工具箱中的“菜单和工具栏|MenuStrip”，将其拖入窗体。

选择工具箱中的“ArcGIS Windows Forms”节，将“ToolbarControl”控件拖入窗体，并将其属性中的Dock设置为Top。

选择工具箱中的“菜单和工具栏|StatusStrip”，将其拖入到窗体。

选择工具箱中的“容器|SplitContainer”容器拖入窗体，并将其属性中的Dock设置为Fill。

将TabControl控件拖入Panel1，将Alignment属性设置为Bottom，Dock属性设置为Fill。点击TabPages属性右边的按钮，弹出TabPage集合编辑器，将tabPage1的Name设置为tabPageLayer，Text设置为图层，将tabPage2的Name设置为tabPageProperty，Text设置为属性。如下所示。

![](/images/ae/1-3.jpg)

选择“图层”选项卡，拖入TOCControl控件，设置Dock属性为Fill。

选择“属性”选项卡，拖入DataGridView控件，设置Dock属性为Fill。

拖入TabControl控件到Panel2，设置Dock属性为Fill。并上述类似的方法，将两个选项卡的Name和Text分别设置为：（tabPageMap、地图），（tabPageLayout，制版）。

选择“地图”选项卡，拖入MapControl控件，设置Dock属性为Fill。

选择“制版”选项卡，拖入PageLayoutControl控件，设置Dock属性为Fill。

最后将LicenseControl控件拖入到窗体的任意地方。

按F5编译运行，可以看到刚才布局好的程序界面了。

###控件绑定

通过以上步骤添加的控件还只是单独存在，而我们的程序需要各控件间协同工作，因此要进行控件绑定。

分别右击ToolbarControl、TOCControl控件，将Buddy设置为axMapControl1，如下图所示。

![](/images/ae/1-4.jpg)

这样，工具条和图层控件就与地图控件关联了。

###添加工具

此时，工具条中还没有任何工具，添加的方法也很简单。右击ToolbarControl，选择“属性|Items”，点击Add，选择Commands选项卡中的Generic，双击Open、SaveAs、Redo、Undo即可将相应工具添加到工具条。
<div class="alert alert-info">
常见的工具有：
Map Navigation中的导航工具，Map Inquiry中的查询工具，Feature Selection中的选择工具，你可以根据需要酌情添加工具。
</div>
###编译运行

按F5即可编译运行程序，至此桌面GIS应用程序框架基本框架已经搭建好了(虽然还没写一行代码)，你可以通过工具条的工具打开地图文档，浏览地图了，效果如开篇所示。

下一讲中将大家带来的是菜单的添加及其实现，敬请关注。
