---
layout: post
title: ArcGIS Engine + C# 实例开发教程：第七讲 图层符号选择器的实现
category: arcgis
tags: [GIS, ArcGIS, ArcGIS Engine, AE, C#]
---

在上一讲中，我们实现了右键菜单（ContextMenu）的添加与实现，在最后我预留给下一讲的问题是TOCControl控件图层拖拽的实现。后来发现此功能的实现异常简单，只要在TOCControl的属性页中，勾选“Enable Layer Drag and Drop”即可。

这一讲，我们要实现的是图层符号选择器，与ArcMap中的Symbol Selector的类似。本讲较前几讲而言，些许有些复杂，不过只要仔细琢磨，认真操作，你就很容易实现如下所示的符号选择器：

![符号选择器](/images/ae/7-1.jpg)

图1 

在AE开发中，符号选择器有两种实现方式。
一是在程序中直接调用ArcMap中的符号选择器，如下所示：

![符号选择器](/images/ae/7-2.jpg)

图2

二是自定义符号选择器，如图1所示。

由于第一种方式前提是必须安装ArcGIS Desktop，其界面还是英文的，而对二次开发来说，大部分用户希望应该是中文界面。因此开发人员通常选择第二种方式，本讲也着重讲解第二种方式。

<div class="alert alert-info">
通过对前六讲的学习，我已经假定你已经基本熟悉C#语言和VS2005的操作，故在下面的教程中，我不准备说明每一步骤的具体操作方法，而只是说明操作步骤，以节省时间和篇幅。
</div>

###直接调用ArcMap中的符号选择器

（1）添加ESRI.ArcGIS.DisplayUI的引用。

分别在解决方案管理器和代码中添加引用。

（2）添加TOCControl的Double_Click事件。

（3）实现TOCControl的Double_Click事件。

因为种方法不是本讲的重点，故不对代码进行分析，有兴趣的读者请自行理解或结合后面的内容理解。代码如下：

	private void axTOCControl1_OnDoubleClick(object sender, ITOCControlEvents_OnDoubleClickEvent e)
	{
		esriTOCControlItem toccItem = esriTOCControlItem.esriTOCControlItemNone;
		ILayer iLayer = null;
		IBasicMap iBasicMap = null;
		object unk = null;
		object data = null;
		if (e.button == 1)
		{
		    axTOCControl1.HitTest(e.x, e.y, ref toccItem, ref iBasicMap, ref iLayer, ref unk,
		        ref data);
		    System.Drawing.Point pos = new System.Drawing.Point(e.x, e.y);
		    if (toccItem == esriTOCControlItem.esriTOCControlItemLegendClass)
		    {
		        ESRI.ArcGIS.Carto.ILegendClass pLC = new LegendClassClass();
		        ESRI.ArcGIS.Carto.ILegendGroup pLG = new LegendGroupClass();
		        if (unk is ILegendGroup)
		        {
		            pLG = (ILegendGroup)unk;
		        }
		        pLC = pLG.get_Class((int)data);
		        ISymbol pSym;
		        pSym = pLC.Symbol;
		        ESRI.ArcGIS.DisplayUI.ISymbolSelector pSS = new
		            ESRI.ArcGIS.DisplayUI.SymbolSelectorClass();
		        bool bOK = false;
		        pSS.AddSymbol(pSym);
		        bOK = pSS.SelectSymbol(0);
		        if (bOK)
		        {
		            pLC.Symbol = pSS.GetSymbolAt(0);
		        }
		        this.axMapControl1.ActiveView.Refresh();
		        this.axTOCControl1.Refresh();
		    }
		}
	}
	
（4）编译运行即可。

###自定义符号选择器

AE9.2提供了SymbologyControl控件，极大的方便了图层符号选择器的制作。本讲实现的符号选择器有如下功能。

用户双击TOCControl控件中图层的符号时，弹出选择符号对话框，对话框能够根据图层类型自动加载相应的符号，如点、线、面。用户可以调整符号的颜色、线宽、角度等参数。还可以打开自定义的符号文件（\*.ServerStyle），加载更多的符号。

####新建符号选择器窗体

新建Winodws窗体，命名为SymbolSelectorFrm，修改窗体的Text属性为“选择符号”。并添加SymboloryControl、PictureBox、Button、Label、NumericUpDown、GroupBox、ColorDialog、OpenFileDialog、ContextMenuStrip控件。控件布局如下所示：

![符号选择器控件布局](/images/ae/7-3.jpg)

####设置控件属性

设置相应控件的相关属性，如下表所示(空则不用修改)：

<table class="table table-bordered">
    <tr>
        <th>控件</th>
        <th>Name属性</th>
        <th>Text属性</th>
        <th>其它</th>
    </tr>
    <tr>
        <td>SymbologyControl</td>
        <td>axSymbologyControl</td>
        <td></td>
        <td></td>
    </tr>
    <tr>
        <td>PictureBox</td>
        <td>ptbPreview</td>
        <td></td>
        <td></td>
    </tr>
    <tr>
        <td>Label</td>
        <td>lblColor</td>
        <td>颜色</td>
        <td></td>
    </tr>
    <tr>
        <td>Label</td>
        <td>lblSize</td>
        <td>大小</td>
        <td></td>
    </tr>
    <tr>
        <td>Label</td>
        <td>lblWidth</td>
        <td>线宽</td>
        <td></td>
    </tr>
    <tr>
        <td>Label</td>
        <td>lblAngle</td>
        <td>角度</td>
        <td></td>
    </tr>
    <tr>
        <td>Label</td>
        <td>lblOutlineColor</td>
        <td>外框颜色</td>
        <td></td>
    </tr>
    <tr>
        <td>NumericUpDown</td>
        <td>nudSize</td>
        <td></td>
        <td></td>
    </tr>
    <tr>
        <td>NumericUpDown</td>
        <td>nudWidth</td>
        <td></td>
        <td></td>
    </tr>
    <tr>
        <td>NumericUpDown</td>
        <td>nudAngle</td>
        <td></td>
        <td></td>
    </tr>
    <tr>
        <td>Button</td>
        <td>btnColor</td>
        <td>（设置为空）</td>
        <td></td>
    </tr>
    <tr>
        <td>Button</td>
        <td>btnOutlineColor</td>
        <td>（设置为空）</td>
        <td></td>
    </tr>
    <tr>
        <td>Button</td>
        <td>btnMoreSymbols</td>
        <td>更多符号</td>
        <td></td>
    </tr>
    <tr>
        <td>Button</td>
        <td>btnOK</td>
        <td>确定</td>
        <td>DialogResult属性设为OK</td>
    </tr>
    <tr>
        <td>Button</td>
        <td>btnCancel</td>
        <td>取消</td>
        <td></td>
    </tr>
    <tr>
        <td>GroupBox</td>
        <td>groupBox1</td>
        <td>预览</td>
        <td></td>
    </tr>
    <tr>
        <td>GroupBox</td>
        <td>groupBox2</td>
        <td>设置</td>
        <td></td>
    </tr>
    <tr>
        <td>ColorDialog</td>
        <td>colorDialog</td>
        <td></td>
        <td></td>
    </tr>
    <tr>
        <td>OpenFileDialog</td>
        <td>openFileDialog</td>
        <td></td>
        <td>Filter属性设置为：Styles 文件|*.ServerStyle</td>
    </tr>
    <tr>
        <td>ContextMenuStrip</td>
        <td>contextMenuStripMoreSymbol</td>
        <td></td>
        <td></td>
    </tr>
</table>

####添加引用

在解决方案资源管理器中添加ArcGIS Engine的ESRI.ArcGIS.Geodatabase引用，在SymbolSelectorFrm.cs文件中添加如下引用代码：

	using ESRI.ArcGIS.Carto;
	using ESRI.ArcGIS.Display;
	using ESRI.ArcGIS.esriSystem;
	using ESRI.ArcGIS.SystemUI;
	using ESRI.ArcGIS.Controls;
	using ESRI.ArcGIS.Geodatabase;
	
####初始化

（1） 添加SymbolSelectorFrm的全局变量，代码如下：

	private IStyleGalleryItem pStyleGalleryItem;
	private ILegendClass pLegendClass;
	private ILayer pLayer;
	public ISymbol pSymbol;
	public Image pSymbolImage;
	
（2） 修改SymbolSelectorFrm的构造函数，传入图层和图例接口。代码如下：

	/// <summary>
	/// 构造函数,初始化全局变量
	/// </summary>
	/// <param name="tempLegendClass">TOC图例</param>
	/// <param name="tempLayer">图层</param>
	public SymbolSelectorFrm(ILegendClass tempLegendClass, ILayer tempLayer)
	{
		InitializeComponent();
		this.pLegendClass = tempLegendClass;
		this.pLayer = tempLayer;
	}
	
（3） 添加SymbolControl的SymbologyStyleClass设置函数SetFeatureClassStyle()，代码如下：

	/// <summary>
	/// 初始化SymbologyControl的StyleClass,图层如果已有符号,则把符号添加到SymbologyControl中的第一个符号,并选中
	/// </summary>
	/// <param name="symbologyStyleClass"></param>
	private void SetFeatureClassStyle(esriSymbologyStyleClass symbologyStyleClass)
	{
		this.axSymbologyControl.StyleClass = symbologyStyleClass;
		ISymbologyStyleClass pSymbologyStyleClass = this.axSymbologyControl.GetStyleClass(symbologyStyleClass);
		if (this.pLegendClass != null)
		{
		    IStyleGalleryItem currentStyleGalleryItem = new ServerStyleGalleryItem();
		    currentStyleGalleryItem.Name = "当前符号";
		    currentStyleGalleryItem.Item = pLegendClass.Symbol;
		    pSymbologyStyleClass.AddItem(currentStyleGalleryItem,0);
		    this.pStyleGalleryItem = currentStyleGalleryItem;
		}
		pSymbologyStyleClass.SelectItem(0);
	}
	
（4） 添加注册表读取函数ReadRegistry()，此函数从注册表中读取ArcGIS的安装路径，代码如下：

	/// <summary>
	/// 从注册表中取得指定软件的路径
	/// </summary>
	/// <param name="sKey"></param>
	/// <returns></returns>
	private string ReadRegistry(string sKey)
	{
		//Open the subkey for reading
		Microsoft.Win32.RegistryKey rk = Microsoft.Win32.Registry.LocalMachine.OpenSubKey(sKey, true);
		if (rk == null) return "";
		// Get the data from a specified item in the key.
		return (string)rk.GetValue("InstallDir");
	}
	
（5） 添加SymbolSelectorFrm的Load事件。根据图层类型为SymbologyControl导入相应的符号样式文件，如点、线、面，并设置控件的可视性。代码如下：

	private void SymbolSelectorFrm_Load(object sender, EventArgs e)
	{
		//取得ArcGIS安装路径
		string sInstall = ReadRegistry("SOFTWARE\\ESRI\\CoreRuntime");

		//载入ESRI.ServerStyle文件到SymbologyControl
		this.axSymbologyControl.LoadStyleFile(sInstall + "\\Styles\\ESRI.ServerStyle");

		//确定图层的类型(点线面),设置好SymbologyControl的StyleClass,设置好各控件的可见性(visible)
		IGeoFeatureLayer pGeoFeatureLayer = (IGeoFeatureLayer)pLayer;
		switch (((IFeatureLayer)pLayer).FeatureClass.ShapeType)
		{
			case ESRI.ArcGIS.Geometry.esriGeometryType.esriGeometryPoint:
				this.SetFeatureClassStyle(esriSymbologyStyleClass.esriStyleClassMarkerSymbols);
				this.lblAngle.Visible = true;
				this.nudAngle.Visible = true;
				this.lblSize.Visible = true;
				this.nudSize.Visible = true;
				this.lblWidth.Visible = false;
				this.nudWidth.Visible = false;
				this.lblOutlineColor.Visible = false;
				this.btnOutlineColor.Visible = false;
				break;
			case ESRI.ArcGIS.Geometry.esriGeometryType.esriGeometryPolyline:
				this.SetFeatureClassStyle(esriSymbologyStyleClass.esriStyleClassLineSymbols);
				this.lblAngle.Visible = false;
				this.nudAngle.Visible = false;
				this.lblSize.Visible = false;
				this.nudSize.Visible = false;
				this.lblWidth.Visible = true;
				this.nudWidth.Visible = true;
				this.lblOutlineColor.Visible = false;
				this.btnOutlineColor.Visible = false;
				break;
			case ESRI.ArcGIS.Geometry.esriGeometryType.esriGeometryPolygon:
				this.SetFeatureClassStyle(esriSymbologyStyleClass.esriStyleClassFillSymbols);
				this.lblAngle.Visible = false;
				this.nudAngle.Visible = false;
				this.lblSize.Visible = false;
				this.nudSize.Visible = false;
				this.lblWidth.Visible = true;
				this.nudWidth.Visible = true;
				this.lblOutlineColor.Visible = true;
				this.btnOutlineColor.Visible = true;
				break;
			case ESRI.ArcGIS.Geometry.esriGeometryType.esriGeometryMultiPatch:
				this.SetFeatureClassStyle(esriSymbologyStyleClass.esriStyleClassFillSymbols);
				this.lblAngle.Visible = false;
				this.nudAngle.Visible = false;
				this.lblSize.Visible = false;
				this.nudSize.Visible = false;
				this.lblWidth.Visible = true;
				this.nudWidth.Visible = true;
				this.lblOutlineColor.Visible = true;
				this.btnOutlineColor.Visible = true;
				break;
			default:
				this.Close();
			 break;
		}
	}
	
（6） 双击确定按钮和取消按钮，分别添加如下代码：

	/// <summary>
	/// 确定按钮
	/// </summary>
	/// <param name="sender"></param>
	/// <param name="e"></param>
	private void btnOK_Click(object sender, EventArgs e)
	{
		//取得选定的符号
		this.pSymbol = (ISymbol)pStyleGalleryItem.Item;
		//更新预览图像
		this.pSymbolImage = this.ptbPreview.Image;
		//关闭窗体
		this.Close();
	}

	/// <summary>
	/// 取消按钮
	/// </summary>
	/// <param name="sender"></param>
	/// <param name="e"></param>
	private void btnCancel_Click(object sender, EventArgs e)
	{
		this.Close();
	}
	
（7） 为了操作上的方便，我们添加SymbologyControl的DoubleClick事件，当双击符号时同按下确定按钮一样，选定符号并关闭符号选择器窗体。代码如下：

	/// <summary>
	/// 双击符号同单击确定按钮，关闭符号选择器。
	/// </summary>
	/// <param name="sender"></param>
	/// <param name="e"></param>
	private void axSymbologyControl_OnDoubleClick(object sender, ESRI.ArcGIS.Controls.ISymbologyControlEvents_OnDoubleClickEvent e)
	{
		this.btnOK.PerformClick();
	}
	
（8） 再添加符号预览函数，当用户选定某一符号时，符号可以显示在PictureBox控件中，方便预览，函数代码如下：

	/// <summary>
	/// 把选中并设置好的符号在picturebox控件中预览
	/// </summary>
	private void PreviewImage()
	{
		stdole.IPictureDisp picture = this.axSymbologyControl.GetStyleClass(this.axSymbologyControl.StyleClass).PreviewItem(pStyleGalleryItem, this.ptbPreview.Width, this.ptbPreview.Height);
		System.Drawing.Image image = System.Drawing.Image.FromHbitmap(new System.IntPtr(picture.Handle));
		this.ptbPreview.Image = image;
	}
	
（9） 当SymbologyControl的样式改变时，需要重新设置符号参数调整控件的可视性，故要添加SymbologyControl的OnStyleClassChanged事件，事件代码与Load事件类似，如下：

	/// <summary>
	/// 当样式（Style）改变时，重新设置符号类型和控件的可视性
	/// </summary>
	/// <param name="sender"></param>
	/// <param name="e"></param>
	private void axSymbologyControl_OnStyleClassChanged(object sender, ESRI.ArcGIS.Controls.ISymbologyControlEvents_OnStyleClassChangedEvent e)
	{
		switch ((esriSymbologyStyleClass)(e.symbologyStyleClass))
		{
		    case esriSymbologyStyleClass.esriStyleClassMarkerSymbols:
		        this.lblAngle.Visible = true;
		        this.nudAngle.Visible = true;
		        this.lblSize.Visible = true;
		        this.nudSize.Visible = true;
		        this.lblWidth.Visible = false;
		        this.nudWidth.Visible = false;
		        this.lblOutlineColor.Visible = false;
		        this.btnOutlineColor.Visible = false;
		        break;
		    case esriSymbologyStyleClass.esriStyleClassLineSymbols:
		        this.lblAngle.Visible = false;
		        this.nudAngle.Visible = false;
		        this.lblSize.Visible = false;
		        this.nudSize.Visible = false;
		        this.lblWidth.Visible = true;
		        this.nudWidth.Visible = true;
		        this.lblOutlineColor.Visible = false;
		        this.btnOutlineColor.Visible = false;
		        break;
		    case esriSymbologyStyleClass.esriStyleClassFillSymbols:
		        this.lblAngle.Visible = false;
		        this.nudAngle.Visible = false;
		        this.lblSize.Visible = false;
		        this.nudSize.Visible = false;
		        this.lblWidth.Visible = true;
		        this.nudWidth.Visible = true;
		        this.lblOutlineColor.Visible = true;
		        this.btnOutlineColor.Visible = true;
		        break;
		}
	}
	
###调用自定义符号选择器

通过以上操作，本符号选择器雏形已经完成，我们可以3sdnMap主窗体中调用并进行测试。如果您已经完成“直接调用ArcMap中的符号选择器”这一节，请注释axTOCControl1_OnDoubleClick事件响应函数里的代码，并添加如下代码。如果您是直接学习自定义符号选择器这一节的，请先添加axTOCControl1控件的OnDoubleClick事件，再添加如下事件响应函数代码：

	/// <summary>
	/// 双击TOCControl控件时触发的事件
	/// </summary>
	/// <param name="sender"></param>
	/// <param name="e"></param>
	private void axTOCControl1_OnDoubleClick(object sender, ITOCControlEvents_OnDoubleClickEvent e)
	{
		esriTOCControlItem itemType = esriTOCControlItem.esriTOCControlItemNone;
		IBasicMap basicMap = null;
		ILayer layer = null;
		object unk = null;
		object data = null; 
		axTOCControl1.HitTest(e.x, e.y, ref itemType, ref basicMap, ref layer, ref unk, ref data); 
		if (e.button == 1)
		{
		    if(itemType==esriTOCControlItem.esriTOCControlItemLegendClass)
		    {
		        //取得图例
		        ILegendClass pLegendClass = ((ILegendGroup)unk).get_Class((int)data);
		        //创建符号选择器SymbolSelector实例
		        SymbolSelectorFrm SymbolSelectorFrm = new SymbolSelectorFrm(pLegendClass, layer);
		        if (SymbolSelectorFrm.ShowDialog() == DialogResult.OK)
		        {
		            //局部更新主Map控件
		            m_mapControl.ActiveView.PartialRefresh(esriViewDrawPhase.esriViewGeography, null, null);
		            //设置新的符号
		            pLegendClass.Symbol = SymbolSelectorFrm.pSymbol;
		            //更新主Map控件和图层控件
		            this.axMapControl1.ActiveView.Refresh();
		            this.axTOCControl1.Refresh();
		        }
		    }
		}
	}
	
按F5编译运行，相信你已经看到自己新手打造的符号选择器已经出现在眼前了。当然，它还比较简陋，下面我们将一起把它做得更完美些。

###符号参数调整

在地图整饰中，符号参数的调整是必须的功能。下面我们将实现符号颜色、外框颜色、线宽、角度等参数的调整。

（1） 添加SymbologyControl的OnItemSelected事件，此事件在鼠标选中符号时触发，此时显示出选定符号的初始参数，事件响应函数代码如下：

	/// <summary>
	/// 选中符号时触发的事件
	/// </summary>
	/// <param name="sender"></param>
	/// <param name="e"></param>
	private void axSymbologyControl_OnItemSelected(object sender, ESRI.ArcGIS.Controls.ISymbologyControlEvents_OnItemSelectedEvent e)
	{
		pStyleGalleryItem = (IStyleGalleryItem)e.styleGalleryItem;
		Color color;
		switch (this.axSymbologyControl.StyleClass)
		{
			//点符号
			case esriSymbologyStyleClass.esriStyleClassMarkerSymbols:
				color = this.ConvertIRgbColorToColor(((IMarkerSymbol)pStyleGalleryItem.Item).Color as IRgbColor);
				//设置点符号角度和大小初始值
				this.nudAngle.Value = (decimal)((IMarkerSymbol)this.pStyleGalleryItem.Item).Angle;
				this.nudSize.Value = (decimal)((IMarkerSymbol)this.pStyleGalleryItem.Item).Size;
				break;
			//线符号
			case esriSymbologyStyleClass.esriStyleClassLineSymbols:
				color = this.ConvertIRgbColorToColor(((ILineSymbol)pStyleGalleryItem.Item).Color as IRgbColor);
				//设置线宽初始值
				this.nudWidth.Value = (decimal)((ILineSymbol)this.pStyleGalleryItem.Item).Width;
				break;
			//面符号
			case esriSymbologyStyleClass.esriStyleClassFillSymbols:
				color = this.ConvertIRgbColorToColor(((IFillSymbol)pStyleGalleryItem.Item).Color as IRgbColor);
				this.btnOutlineColor.BackColor = this.ConvertIRgbColorToColor(((IFillSymbol)pStyleGalleryItem.Item).Outline.Color as IRgbColor);
				//设置外框线宽度初始值
				this.nudWidth.Value = (decimal)((IFillSymbol)this.pStyleGalleryItem.Item).Outline.Width;
				break;
			default:
				color = Color.Black;
				break;
		}
		//设置按钮背景色
		this.btnColor.BackColor = color;
		//预览符号
		this.PreviewImage();
	}
	
（2） 调整点符号的大小

添加nudSize控件的ValueChanged事件，即在控件的值改变时响应此事件，然后重新设置点符号的大小。代码如下：

	/// <summary>
	/// 调整符号大小-点符号
	/// </summary>
	/// <param name="sender"></param>
	/// <param name="e"></param>
	private void nudSize_ValueChanged(object sender, EventArgs e)
	{
		((IMarkerSymbol)this.pStyleGalleryItem.Item).Size = (double)this.nudSize.Value;
		this.PreviewImage();
	}
	
（3） 调整点符号的角度

添加nudAngle控件的ValueChanged事件，以重新设置点符号的角度。代码如下：

	/// <summary>
	/// 调整符号角度-点符号
	/// </summary>
	/// <param name="sender"></param>
	/// <param name="e"></param>
	private void nudAngle_ValueChanged(object sender, EventArgs e)
	{
		((IMarkerSymbol)this.pStyleGalleryItem.Item).Angle = (double)this.nudAngle.Value;
		this.PreviewImage();
	}

（4） 调整线符号和面符号的线宽

添加nudWidth控件的ValueChanged事件，以重新设置线符号的线宽和面符号的外框线的线宽。代码如下：

	/// <summary>
	/// 调整符号宽度-限于线符号和面符号
	/// </summary>
	/// <param name="sender"></param>
	/// <param name="e"></param>
	private void nudWidth_ValueChanged(object sender, EventArgs e)
	{
		switch (this.axSymbologyControl.StyleClass)
		{
			case esriSymbologyStyleClass.esriStyleClassLineSymbols:
				((ILineSymbol)this.pStyleGalleryItem.Item).Width = Convert.ToDouble(this.nudWidth.Value);
				break;
			case esriSymbologyStyleClass.esriStyleClassFillSymbols:
				//取得面符号的轮廓线符号
				ILineSymbol pLineSymbol = ((IFillSymbol)this.pStyleGalleryItem.Item).Outline;
				pLineSymbol.Width = Convert.ToDouble(this.nudWidth.Value);
				((IFillSymbol)this.pStyleGalleryItem.Item).Outline = pLineSymbol;
				break;
		}
		this.PreviewImage();
	}

（5） 颜色转换

在ArcGIS Engine中，颜色由IRgbColor接口实现，而在.NET框架中，颜色则由Color结构表示。故在调整颜色参数之前，我们必须完成以上两种不同颜色表示方式的转换。关于这两种颜色结构的具体信息，请大家自行查阅相关资料。下面添加两个颜色转换函数。

ArcGIS Engine中的IRgbColor接口转换至.NET中的Color结构的函数：

	/// <summary>
	/// 将ArcGIS Engine中的IRgbColor接口转换至.NET中的Color结构
	/// </summary>
	/// <param name="pRgbColor">IRgbColor</param>
	/// <returns>.NET中的System.Drawing.Color结构表示ARGB颜色</returns>
	public Color ConvertIRgbColorToColor(IRgbColor pRgbColor)
	{
		return ColorTranslator.FromOle(pRgbColor.RGB);
	}
	.NET中的Color结构转换至于ArcGIS Engine中的IColor接口的函数：
	/// <summary>
	/// 将.NET中的Color结构转换至于ArcGIS Engine中的IColor接口
	/// </summary>
	/// <param name="color">.NET中的System.Drawing.Color结构表示ARGB颜色</param>
	/// <returns>IColor</returns>
	public IColor ConvertColorToIColor(Color color)
	{
		IColor pColor = new RgbColorClass();
		pColor.RGB = color.B * 65536 + color.G * 256 + color.R;
		return pColor;
	}
（6） 调整所有符号的颜色

选择颜色时，我们调用.NET的颜色对话框ColorDialog，选定颜色后，修改颜色按钮的背景色为选定的颜色，以方便预览。双击btnColor按钮，添加如下代码：

	/// <summary>
	/// 颜色按钮
	/// </summary>
	/// <param name="sender"></param>
	/// <param name="e"></param>
	private void btnColor_Click(object sender, EventArgs e)
	{
		//调用系统颜色对话框
		if (this.colorDialog.ShowDialog() == DialogResult.OK)
		{
			//将颜色按钮的背景颜色设置为用户选定的颜色
			this.btnColor.BackColor = this.colorDialog.Color;
			//设置符号颜色为用户选定的颜色
			switch (this.axSymbologyControl.StyleClass)
			{
				//点符号
				case esriSymbologyStyleClass.esriStyleClassMarkerSymbols:
					((IMarkerSymbol)this.pStyleGalleryItem.Item).Color = this.ConvertColorToIColor(this.colorDialog.Color);
					break;
				//线符号
				case esriSymbologyStyleClass.esriStyleClassLineSymbols:
					((ILineSymbol)this.pStyleGalleryItem.Item).Color = this.ConvertColorToIColor(this.colorDialog.Color);
					break;
				//面符号
				case esriSymbologyStyleClass.esriStyleClassFillSymbols:
					((IFillSymbol)this.pStyleGalleryItem.Item).Color = this.ConvertColorToIColor(this.colorDialog.Color);
					break;
			}
			//更新符号预览
			this.PreviewImage();
		}
	}

（7） 调整面符号的外框线颜色

同上一步类似，双击btnOutlineColor按钮，添加如下代码：

	/// <summary>
	/// 外框颜色按钮
	/// </summary>
	/// <param name="sender"></param>
	/// <param name="e"></param>
	private void btnOutlineColor_Click(object sender, EventArgs e)
	{
		if (this.colorDialog.ShowDialog() == DialogResult.OK)
		{
		    //取得面符号中的外框线符号
		    ILineSymbol pLineSymbol = ((IFillSymbol)this.pStyleGalleryItem.Item).Outline;
		    //设置外框线颜色
		    pLineSymbol.Color = this.ConvertColorToIColor(this.colorDialog.Color);
		    //重新设置面符号中的外框线符号
		    ((IFillSymbol)this.pStyleGalleryItem.Item).Outline = pLineSymbol;
		    //设置按钮背景颜色
		    this.btnOutlineColor.BackColor = this.colorDialog.Color;
		    //更新符号预览
		    this.PreviewImage();
		}
	}
	
至此，你可以编译运行程序，看看效果如何，是不是感觉很不错了？我们已经能够修改符号的参数，自定义符号了。

但是，SymbologyControl默认加载的是ESRI.ServerStyle文件的样式，用过ArcMap的你可能已经注意到，ArcMap中的Symbol Selector有一个“More Symbols”按钮，可以加载其它的符号和ServerStyle文件。3sdnMap当然“一个都不能少”。

###添加更多符号菜单

还记得我们在开始的时候添加了ContextMenuStrip控件吗？现在它终于派上用场了。我们要实现的功能是：单击“更多符号”弹出菜单（ContextMenu），菜单中列出了ArcGIS自带的其它符号，勾选相应的菜单项就可以在SymbologyControl中增加相应的符号。在菜单的最后一项是“添加符号”，选择这一项时，将弹出打开文件对话框，我们可以由此选择其它的ServerStyle文件，以加载更多的符号。

（1） 定义全局变量

在SymbolSelectorFrm中定义如下全局变量，用于判断菜单是否已经初始化。

	//菜单是否已经初始化标志
	bool contextMenuMoreSymbolInitiated = false;
	
（2） 双击“更多符号”按钮，添加Click事件

在此事件响应函数中，我们要完成ServerStyle文件的读取，将其文件名作为菜单项名称生成菜单并显示菜单。代码如下：

	/// <summary>
	/// “更多符号”按下时触发的事件
	/// </summary>
	/// <param name="sender"></param>
	/// <param name="e"></param>
	private void btnMoreSymbols_Click(object sender, EventArgs e)
	{
		if (this.contextMenuMoreSymbolInitiated == false)
		{
		    string sInstall = ReadRegistry("SOFTWARE\\ESRI\\CoreRuntime");
		    string path = System.IO.Path.Combine(sInstall, "Styles");
		    //取得菜单项数量
		    string[] styleNames = System.IO.Directory.GetFiles(path, "*.ServerStyle");
		    ToolStripMenuItem[] symbolContextMenuItem = new ToolStripMenuItem[styleNames.Length + 1];
		    //循环添加其它符号菜单项到菜单
		    for (int i = 0; i < styleNames.Length; i++)
		    {
		        symbolContextMenuItem[i] = new ToolStripMenuItem();
		        symbolContextMenuItem[i].CheckOnClick = true;
		        symbolContextMenuItem[i].Text = System.IO.Path.GetFileNameWithoutExtension(styleNames[i]);
		        if (symbolContextMenuItem[i].Text == "ESRI")
		        {
		            symbolContextMenuItem[i].Checked = true;
		        }
		        symbolContextMenuItem[i].Name = styleNames[i];
		    }
		    //添加“更多符号”菜单项到菜单最后一项
		    symbolContextMenuItem[styleNames.Length] = new ToolStripMenuItem();
		    symbolContextMenuItem[styleNames.Length].Text = "添加符号";
		    symbolContextMenuItem[styleNames.Length].Name = "AddMoreSymbol";

		    //添加所有的菜单项到菜单
		    this.contextMenuStripMoreSymbol.Items.AddRange(symbolContextMenuItem);
		    this.contextMenuMoreSymbolInitiated = true;
		}
		//显示菜单
		this.contextMenuStripMoreSymbol.Show(this.btnMoreSymbols.Location);
	}
	
（3） 添加contextMenuStripMoreSymbol控件的ItemClicked事件

当单击某一菜单项时响应ItemClicked事件，将选中的ServerStyle文件导入到SymbologyControl中并刷新。当用户单击“添加符号”菜单项时，弹出打开文件对话框，供用户选择其它的ServerStyle文件。代码如下：

	/// <summary>
	/// “更多符号”按钮弹出的菜单项单击事件
	/// </summary>
	/// <param name="sender"></param>
	/// <param name="e"></param>
	private void contextMenuStripMoreSymbol_ItemClicked(object sender, ToolStripItemClickedEventArgs e)
	{
		ToolStripMenuItem pToolStripMenuItem = (ToolStripMenuItem)e.ClickedItem;
		//如果单击的是“添加符号”
		if (pToolStripMenuItem.Name == "AddMoreSymbol")
		{
		    //弹出打开文件对话框
		    if (this.openFileDialog.ShowDialog() == DialogResult.OK)
		    { 
		        //导入style file到SymbologyControl
		        this.axSymbologyControl.LoadStyleFile(this.openFileDialog.FileName);
		        //刷新axSymbologyControl控件
		        this.axSymbologyControl.Refresh();
		    }
		}
		else//如果是其它选项
		{
		    if (pToolStripMenuItem.Checked == false)
		    {
		        this.axSymbologyControl.LoadStyleFile(pToolStripMenuItem.Name);
		        this.axSymbologyControl.Refresh();
		    }
		    else
		    {
		        this.axSymbologyControl.RemoveFile(pToolStripMenuItem.Name);
		        this.axSymbologyControl.Refresh();
		    }
		}
	}

###编译运行

相信你已经盼这一步很久了吧，按照惯例，按下F5吧！大功造成。

以上代码在AE9.2+VS2005+XP中编译通过。

致谢：在本讲的实现过程中，参考了GIS-Lin的Lin-GIS，在此特别感谢！
	
下一讲中，我将给大家带来的是属性数据表的查询显示，敬请关注3SDN.Net！
