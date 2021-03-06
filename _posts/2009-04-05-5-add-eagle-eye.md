---
layout: post
title: ArcGIS Engine + C# 实例开发教程：第五讲 鹰眼的实现
category: arcgis
tags: [GIS, ArcGIS, ArcGIS Engine, AE, C#]
---

在上一讲中，我们实现了状态栏的相关信息显示，在这一讲中我们将要实现鹰眼功能。

所谓的鹰眼，就是一个缩略地图，上面有一个矩形框，矩形框区域就是当前显示的地图区域，拖动矩形框可以改变当前地图显示的位置，改变矩形框的大小，可以改变当前地图的显示区域大小，从起到导航的作用。鹰眼是地图浏览中常用的功能之一。

关于鹰眼的实现方式，最常用的是用一个MapControl控件显示地图全图，并在上面画一个红色矩形框表示当前地图的显示范围，并实现鹰眼MapControl与主窗体的MapControl互动。本讲最终效果如下所示：

![鹰眼的最终效果图](/images/ae/5-4.jpg)

图1 鹰眼效果

###添加鹰眼控件

由于本教程在第一讲中没有预先考虑到鹰眼所放的位置，故我们要先稍微调整一下程序框架，并添加一个MapControl用于显示鹰眼。

在本教程中，我们将鹰眼放在图层控件的下方，调整方法如下：

（1）在设计视图中，选择tabControl1控件，即放图层和属性的那个容器，将其Dock属性设为None，并用鼠标拖拽将其缩小。把工具箱中的SplitContainer控件拖到窗体的左窗格，即放在tabControl1控件的旁边。并将其Orientation属性设置为Horizontal。

（2）选中tabControl1控件，按Ctrl+X剪切，再选中刚才粘贴到SplitContainer2的Panel1中，如图2所示。操作完成后效果如图3所示。

![鹰眼](/images/ae/5-1.jpg)

图2 

![鹰眼](/images/ae/5-2.jpg)

图3

（3）再选中SplitContainer2控件（如果不好选中，直接以属性面板中选择SplitContainer2），将其Dock属性设置为Fill。再选中tabControl1，将其Dock属性也设置为Fill。

（4）从工具箱中选择MapControl控件并拖到SplitContainer2的Panel2，作为鹰眼控件。最终效果如图4所示。

![鹰眼](/images/ae/5-3.jpg)

图4

###鹰眼的实现

####载入地图到鹰眼控件

当地图载入到主Map控件时，同时也载入到鹰眼控件，在axMapControl1_OnMapReplaced事件响应函数（此函数上一讲中已经添加了）中添加如下代码：

	private void axMapControl1_OnMapReplaced(object sender, IMapControlEvents2_OnMapReplacedEvent e)
	{
		//前面代码省略

		//当主地图显示控件的地图更换时，鹰眼中的地图也跟随更换
		this.axMapControl2.Map = new MapClass();
		//添加主地图控件中的所有图层到鹰眼控件中
		for (int i = 1; i <= this.axMapControl1.LayerCount; i++)
		{
			this.axMapControl2.AddLayer(this.axMapControl1.get_Layer(this.axMapControl1.LayerCount - i));
		}
		//设置MapControl显示范围至数据的全局范围
		this.axMapControl2.Extent = this.axMapControl1.FullExtent;
		//刷新鹰眼控件地图
		this.axMapControl2.Refresh();
	}
	
####绘制鹰眼矩形框

为鹰眼控件MapControl1添加OnExtentUpdated事件，此事件是在主Map控件的显示范围改变时响应，从而相应更新鹰眼控件中的矩形框。其响应函数代码如下：
    
    private void axMapControl1_OnExtentUpdated(object sender, IMapControlEvents2_OnExtentUpdatedEvent e)
    {
        // 得到新范围
        IEnvelope pEnv = (IEnvelope)e.newEnvelope;
        IGraphicsContainer pGra = axMapControl2.Map as IGraphicsContainer;
        IActiveView pAv = pGra as IActiveView;
        //在绘制前，清除axMapControl2中的任何图形元素
        pGra.DeleteAllElements();
        IRectangleElement pRectangleEle = new RectangleElementClass();
        IElement pEle = pRectangleEle as IElement;
        pEle.Geometry = pEnv;
        //设置鹰眼图中的红线框
        IRgbColor pColor = new RgbColorClass();
        pColor.Red = 255;
        pColor.Green = 0;
        pColor.Blue = 0;
        pColor.Transparency = 255;
        //产生一个线符号对象
        ILineSymbol pOutline = new SimpleLineSymbolClass();
        pOutline.Width = 2;
        pOutline.Color = pColor;
        //设置颜色属性
        pColor = new RgbColorClass();
        pColor.Red = 255;
        pColor.Green = 0;
        pColor.Blue = 0;
        pColor.Transparency = 0;
        //设置填充符号的属性
        IFillSymbol pFillSymbol = new SimpleFillSymbolClass();
        pFillSymbol.Color = pColor;
        pFillSymbol.Outline = pOutline;
        IFillShapeElement pFillShapeEle = pEle as IFillShapeElement;
        pFillShapeEle.Symbol = pFillSymbol;
        pGra.AddElement((IElement)pFillShapeEle, 0);
        //刷新
        pAv.PartialRefresh(esriViewDrawPhase.esriViewGraphics, null, null);
    }
    
####鹰眼与主Map控件互动

为鹰眼控件MapControl2添加OnMouseDown事件，代码如下：

	private void axMapControl2_OnMouseDown(object sender, IMapControlEvents2_OnMouseDownEvent e)
    {
        if (this.axMapControl2.Map.LayerCount != 0)
        {
			//按下鼠标左键移动矩形框
            if (e.button == 1)
            {
                IPoint pPoint = new PointClass();
                pPoint.PutCoords(e.mapX, e.mapY);
                IEnvelope pEnvelope = this.axMapControl1.Extent;
                pEnvelope.CenterAt(pPoint);
                this.axMapControl1.Extent = pEnvelope;
				this.axMapControl1.ActiveView.PartialRefresh(esriViewDrawPhase.esriViewGeography, null, null);
            }
			//按下鼠标右键绘制矩形框
            else if (e.button == 2)
            {
                IEnvelope pEnvelop = this.axMapControl2.TrackRectangle();
                this.axMapControl1.Extent = pEnvelop;
                this.axMapControl1.ActiveView.PartialRefresh(esriViewDrawPhase.esriViewGeography, null, null);
            }
        }
    }
    
为鹰眼控件MapControl2添加OnMouseMove事件，主要实现按下鼠标左键的时候移动矩形框，同时也改变主的图控件的显示范围。代码如下：

	private void axMapControl2_OnMouseMove(object sender, IMapControlEvents2_OnMouseMoveEvent e)
	{
		//如果不是左键按下就直接返回
		if (e.button != 1) return;
		IPoint pPoint = new PointClass();
		pPoint.PutCoords(e.mapX, e.mapY);
		this.axMapControl1.CenterAt(pPoint);
		this.axMapControl1.ActiveView.PartialRefresh(esriViewDrawPhase.esriViewGeography, null, null);
	}
	
###编译运行

按F5编译运行程序。

期待的鹰眼功能你已经实现了，按下左键在鹰眼窗口中移动，或者按下右键在鹰眼窗口中画一个矩形，主地图窗口的显示范围都会跟着变化。主地图窗口中的地图经放大缩小等操作后，鹰眼窗口的矩形框大小也会随着改变。

在下一讲中，我将给大家带的是TOCControl控件右键菜单的实现，敬请关注！
