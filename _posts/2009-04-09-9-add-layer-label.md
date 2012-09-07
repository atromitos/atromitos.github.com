---
layout: post
title: ArcGIS Engine + C# 实例开发教程：第九讲 图层标注
category: arcgis
tags: [GIS, ArcGIS, ArcGIS Engine, AE, C#]
---

上一讲中，我们实现了图层属性表的浏览功能，这一讲给大家讲解图层标注的实现方法。本文实现的最终效果如下：

![图层标注](/images/ae/9-1.jpg)

图层标注实现起来并不复杂，本例仅做一个简单示范，只加载AE的样式库，标注选定的字段，旨在抛砖引玉。更高级的功能，如自定义样式和修改样式，由读者自己实现。

主要思路：

	加载图层字段 –> 加载文本样式 -> 设置文本样式
	
实现过程：

	创建标注设置窗体 -> 创建图层标注的Command -> 添加Command到图层右键菜单
	
###创建标注设置窗体

（1）添加一个Windows窗体，命名为LabelLayerFrm.cs。添加控件如下：
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
        <td>ComboBox</td>
        <td>cbbField</td>
        <td></td>
        <td></td>
    </tr>
    <tr>
        <td>Button</td>
        <td>btnOK</td>
        <td>确定</td>
        <td>DialogResult设为OK</td>
    </tr>
    <tr>
        <td>Button</td>
        <td>btnCancel</td>
        <td>取消</td>
        <td>DialogResult设为Cancel</td>
    </tr>
    <tr>
        <td>GroupBox</td>
        <td>groupBox1</td>
        <td>字段</td>
        <td></td>
    </tr>
    <tr>
        <td>GroupBox</td>
        <td>groupBox2</td>
        <td>符号</td>
        <td></td>
    </tr>
</table>

（2）为LabelLayerFrm类添加两个成员变量：

	public ILayer pLayer;
	private IStyleGalleryItem pStyleGalleryItem;

（3）重载一个构造函数：

	public LabelLayerFrm(ILayer layer)
	{
		InitializeComponent();
		pLayer = layer;
	}
	
(4) 添加成员函数ReadRegistry，用于从注册表中读取ArcGIS的安装路径。

	/// <summary>
	/// 读取注册表中的制定软件的路径
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

（5）添加LabelLayerFrm窗体的Load事件，以加载图层字段到下拉模型，加载文本样式到SymbologyControl控件。

	private void LabelLayerFrm_Load(object sender, EventArgs e)
	{
		//加载图层字段
		ITable pTable = pLayer as ITable;
		IField pField = null;
		for (int i = 0; i < pTable.Fields.FieldCount; i++)
		{
		    pField = pTable.Fields.get_Field(i);
		    cbbField.Items.Add(pField.AliasName);
		}
		cbbField.SelectedIndex = 0;
		//获得ArcGIS的安装路径
		string sInstall = ReadRegistry("SOFTWARE\\ESRI\\CoreRuntime");
		//加载ESRI.ServerStyle 样式文件到SymbologyControl
		this.axSymbologyControl1.LoadStyleFile(sInstall + "\\Styles\\ESRI.ServerStyle");
		this.axSymbologyControl1.GetStyleClass(esriSymbologyStyleClass.esriStyleClassTextSymbols).SelectItem(0);
	}

（6）添加axSymbologyControl1控件的OnItemSelected事件，以设置选定的样式。

	private void axSymbologyControl1_OnItemSelected(object sender, ISymbologyControlEvents_OnItemSelectedEvent e)
	{
		pStyleGalleryItem = (IStyleGalleryItem)e.styleGalleryItem;
	}
	
（7）添加确定按扭的Click事件，为选定图层中的选定的字段以选定的样式标注。

	private void btnOK_Click(object sender, EventArgs e)
	{
		IGeoFeatureLayer pGeoFeatureLayer = pLayer as IGeoFeatureLayer;
		pGeoFeatureLayer.AnnotationProperties.Clear();//必须执行，因为里面有一个默认的
		IBasicOverposterLayerProperties pBasic = new BasicOverposterLayerPropertiesClass();
		ILabelEngineLayerProperties pLableEngine = new LabelEngineLayerPropertiesClass();
		ITextSymbol pTextSymbol = new TextSymbolClass();            
		pTextSymbol = (ITextSymbol)pStyleGalleryItem.Item;
		//你可以在这里修改样式的颜色和字体等属性，本文从略
		//pTextSymbol.Color
		//pTextSymbol.Font 
		string pLable = "[" + (string)cbbField .SelectedItem + "]";
		pLableEngine.Expression = pLable;
		pLableEngine.IsExpressionSimple = true;
		pBasic.NumLabelsOption = esriBasicNumLabelsOption.esriOneLabelPerShape;
		pLableEngine.BasicOverposterLayerProperties = pBasic;
		pLableEngine.Symbol = pTextSymbol;
		pGeoFeatureLayer.AnnotationProperties.Add(pLableEngine as IAnnotateLayerProperties);
		pGeoFeatureLayer.DisplayAnnotation = true;
	}

至此，标注设置窗体已经完成，如果你编译通不过，看看是不是忘了添加相关引用了。

###创建图层标注的Command

（1）创建一个新类，以ArcGIS的BaseCommand为模板，命名为LabelLayerCmd.cs。
<div class="alert alert-warn">
注意：在新建Base Command模板时，会弹出一个对话框让我们选择模板适用对象，这时我们要选择MapControl、PageLayoutControl，即选择第二项或者倒数第二项。
</div>

（2）添加LabelLayerCmd类的成员变量。

	private ILayer pLayer = null;
	IMapControl3 pMap;

(3) 修改默认构造函数如下：

	public LabelLayerCmd(ILayer lyr,IMapControl3 map)
	{
		//
		// TODO: Define values for the public properties
		//
		base.m_category = ""; //localizable text
		base.m_caption = "标注";  //localizable text 
		base.m_message = "标注";  //localizable text
		base.m_toolTip = "标注";  //localizable text
		base.m_name = "标注";   //unique id, non-localizable (e.g. "MyCategory_MyCommand")
		pLayer = lyr;
		pMap = map;
		try
		{
		    //
		    // TODO: change bitmap name if necessary
		    //
		    string bitmapResourceName = GetType().Name + ".bmp";
		    base.m_bitmap = new Bitmap(GetType(), bitmapResourceName);
		}
		catch (Exception ex)
		{
		    System.Diagnostics.Trace.WriteLine(ex.Message, "Invalid Bitmap");
		}
	}

(4) 修改OnClick函数为：

	/// <summary>
	/// Occurs when this command is clicked
	/// </summary>
	public override void OnClick()
	{
		// TODO: Add LabelLayerCmd.OnClick implementation
		LabelLayerFrm labelLyrFrm = new LabelLayerFrm(pLayer);
		labelLyrFrm.ShowDialog();
		pMap.Refresh(esriViewDrawPhase.esriViewGraphics, null, null);
	}
	
###添加Command到图层右键菜单

回到3sdnMap主窗体类，找到axTOCControl1_OnMouseDown事件响应函数，修改如下代码片断：

	//弹出右键菜单
	if (item == esriTOCControlItem.esriTOCControlItemMap)
		m_menuMap.PopupMenu(e.x, e.y, m_tocControl.hWnd);
	if (item == esriTOCControlItem.esriTOCControlItemLayer)
	{
	m_menuLayer.AddItem(new OpenAttributeTable(layer), -1, 2, true , esriCommandStyles.esriCommandStyleTextOnly);
	//动态添加图层标注的Command到图层右键菜单
		m_menuLayer.AddItem(new LabelLayerCmd(layer, m_mapControl), -1, 3, false, esriCommandStyles.esriCommandStyleTextOnly);
		//弹出图层右键菜单 
		m_menuLayer.PopupMenu(e.x, e.y, m_tocControl.hWnd);
		//移除菜单项
		m_menuLayer.Remove(3);
		m_menuLayer.Remove(2);
	}
	
至此，已经完成图层文本标注，编译运行吧，是不是看到开篇的效果了？

以上代码在Windows XP Sp3 + VS2005 + AE9.2/9.3环境下编译通过。

*(THE END)*
