---
layout: post
title: ArcGIS Engine + C# 实例开发教程：第四讲 状态栏信息的添加与实现
category: arcgis
tags: [GIS, ArcGIS, ArcGIS Engine, AE, C#]
---

在上一讲中，我们完成了MapControl和PageLayoutControl两种视图的同步工作，本讲我们将完成状态栏信息的添加与实现。

应用程序的状态栏一般用来显示程序的当前状态，当前所使用的工具。GIS应用程序一般也在状态栏显示当前光标的坐标、比例尺等信息。

学习完本讲内容，您将学会状态栏编程的基本方法，并且能够在我们的程序的状态栏中添加且显示以下信息：

1. 当前所用工具信息  
2. 当前比例尺  
3. 当前坐标  

###添加状态栏项目

在设计视图中，点击窗体中的状态栏，在其属性面板中找到“Items”项，单击其右边的按钮，在下拉框中选择“StatusLabel”，单击“添加按钮”，依次添加四个StatusLabel，依次修改属性参数如下表所示：

<table class="table">
    <tr>
        <th>序号</th>
        <th>Name属性</th>
        <th>Text属性</th>
        <th>Spring属性</th>
        <th>说明</th>
    </tr>
    <tr>
        <td>1</td>
        <td>MessageLabel</td>
        <td>就绪</td>
        <td>False</td>
        <td>当前所用工具信息</td>
    </tr>
    <tr>
        <td>2</td>
        <td>Blank</td>
        <td></td>
        <td>True</td>
        <td>占位</td>
    </tr>
    <tr>
        <td>3</td>
        <td>ScaleLabel</td>
        <td>比例尺</td>
        <td>False</td>
        <td>当前比例尺</td>
    </tr>
    <tr>
        <td>4</td>
        <td>CoordinateLabel</td>
        <td>当前坐标</td>
        <td>False</td>
        <td>当前坐标</td>
    </tr>
</table>

设置好之后如下图所示：

![状态栏编辑](/images/ae/4-1.jpg)

<div class="alert alert-info">

我们设计出的状态栏最终如下所示：

<table class="table table-bordered">
    <tr>
        <td>就绪</td>
        <td>（Blank）</td>
        <td>比例尺</td>
        <td>当前坐标</td>
    </tr>
</table>

Spring属性表示可以按状态栏剩余空间自动伸缩。所以加入Blank项目，只是为了占个位子，以达到ScaleLabel和CoordinateLabel项目右对齐而MessageLabel项目左对齐的目的。
</div>

<div class="alert alert-info">
为了防止闪烁，我们可以固定状态栏中的比例尺和当前坐标项目的宽度，方法如下：
选中状态栏中的比例尺或当前坐标项目，把其autoSize属性设为False，再在Size属性里设置宽度。经测试，比例尺宽度为150，当前坐标宽度为400比较合适。
</div>

###显示当前所用工具信息

首先添加axToolbarControl1的OnMouseMove事件(相信大家看了以上的教程，已经知道怎么添加事件了吧，还不知道的建议再温习下前几讲的内容)。在其事件响应函数代码如下：

	private void axToolbarControl1_OnMouseMove(object sender, IToolbarControlEvents_OnMouseMoveEvent e)
    {
        //取得鼠标所在工具的索引号
        int index = axToolbarControl1.HitTest(e.x, e.y, false);
        if (index != -1)
        {
            //取得鼠标所在工具的ToolbarItem
            IToolbarItem toolbarItem = axToolbarControl1.GetItem(index);
            //设置状态栏信息
            MessageLabel.Text = toolbarItem.Command.Message;
        }
        else
        {
            MessageLabel.Text = "就绪";
        }
    }
        
###显示当前比例尺

添加axMapControl1的OnMouseMove事件，其代码如下：

	private void axMapControl1_OnMouseMove(object sender, IMapControlEvents2_OnMouseMoveEvent e)
	{
	//显示当前比例尺
	ScaleLabel.Text = "比例尺1:" + ((long)this.axMapControl1.MapScale).ToString();  
	}
	
###显示当前坐标

显示当前坐标也是axMapControl1的OnMouseMove事件中响应，故只要在axMapControl1_OnMouseMove函数中添加如下代码即可：

	//显示当前坐标
	CoordinateLabel.Text = " 当前坐标 X = " + e.mapX.ToString() + " Y = " + e.mapY.ToString() + " " + this.axMapControl1.MapUnits.ToString().Substring(4);

<div class="alert alert-info">
this.axMapControl1.MapUnits的坐标单位带有“esri”前缀，如“esriUnknownUnits”或“esriMeters”，this.axMapControl1.MapUnits.ToString().Substring(4)是为了去除前缀“esri”。当然，你也可以写一个转换函数，将其转换为中文。
</div>
    
###编译运行

按F5编译运行程序。如果你足够细心的话，相信你已经成功了！

在本讲中，介绍中StatusStrip控件的基本使用方法和AE中当所用工具信息、当前比例尺和当前坐标的显示调用方法。

在下一讲，我将给大家带来的是激动人心的鹰眼功能的实现，敬请关注！
