# 端口设置
<font color="red">MissionPlanner\GCSViews\ConfigurationView\ConfigSerial.cs</font>

## 端口数量
```C#
 serialPorts = 0;
 foreach (var param in MainV2.comPort.MAV.param.Keys)
 {
     if (param.StartsWith("SERIAL") && param.EndsWith("_BAUD"))
     {
         int port;
         Int32.TryParse(param.Substring(6, 1), out port); //if unable to parse, then port = 0
         if (port > serialPorts)
         {
             serialPorts = port;
         }
     }
 }

 if (serialPorts == 0)
 {
     //No serial ports found
     return;
 }
```
## uartName 设置
```C#
 string uartName = "";

 if (_uartNames.ContainsKey(i))
 {
     uartName = _uartNames[i];
 }

 //Get the RTS/CTS options if there is any
 string ctsrtsParamName = "BRD_SER" +i.ToString() + "_RTSCTS";
 if (MainV2.comPort.MAV.param.ContainsKey(ctsrtsParamName))
 {
     var ctsValue = MainV2.comPort.MAV.param[ctsrtsParamName].Value;
     if (ctsValue == 1)
     {
         uartName = uartName + " (RTS/CTS)";
     }
     if (ctsValue == 2)
     {
         uartName = uartName + " (RTS/CTS Auto)";
     }
 }
```

## Protocol  

> <code>portName</code>为<code>"SERIALx"</cdoe>  

```C#  
string protParamName = portName + "_PROTOCOL";
var protOptions = ParameterMetaDataRepository.GetParameterOptionsInt(protParamName, MainV2.comPort.MAV.cs.firmware.ToString());
if (protOptions.Count > 0)
{
    ComboBox cmb = new ComboBox()
    {
        Dock = DockStyle.Fill,
        DropDownStyle = ComboBoxStyle.DropDownList,
        DataSource = protOptions,
        DisplayMember = "Value",
        ValueMember = "Key",
        Anchor = AnchorStyles.None,
        Name = protParamName
    };

    widenComboBox(cmb);
    ThemeManager.ApplyThemeTo(cmb);
    tableLayoutPanel1.GetControlFromPosition(2, i)?.Dispose();
    tableLayoutPanel1.Controls.Add(cmb, 2, i);

    // Populate the current selection from the cell value, if it's valid
    int val = -1;
    if (int.TryParse(MainV2.comPort.MAV.param[protParamName].Value.ToString(), out val))
    {
        cmb.SelectedValue = val;
    }
    else
    {
        cmb.SelectedIndex = -1;
    }
    cmb.SelectedIndexChanged += (s, a) =>
    {
        setParam(cmb.Name, cmb.SelectedValue.ToString());
        //Apply rules
        doApplyRules(portName, cmb.SelectedValue.ToString());
    };
}
```  
## OPTIONS

```C#
//Options label
string param_name = portName + "_OPTIONS";

var binlist = ParameterMetaDataRepository.GetParameterBitMaskInt(param_name, MainV2.comPort.MAV.cs.firmware.ToString());
var param_value = MainV2.comPort.MAV.param[param_name].Value;

Label label1 = new Label()
{
    Text = "",
    Anchor = AnchorStyles.None,
    Dock = DockStyle.Fill,
    AutoSize = true,
    MaximumSize = new Size(300, 200),
    Location = new Point(0, 0),
    Size = new Size(100, 20),
    TextAlign = ContentAlignment.MiddleLeft
};
//populate the label based on the options value
setLabelOptions(param_name, label1);
ThemeManager.ApplyThemeTo(label1);
tableLayoutPanel1.GetControlFromPosition(3, i)?.Dispose();
tableLayoutPanel1.Controls.Add(label1, 3, i);

//Bit setting button and form
var bitmask = ParameterMetaDataRepository.GetParameterBitMaskInt(param_name, MainV2.comPort.MAV.cs.firmware.ToString());
if (bitmask.Count > 0)
{
    MyButton optionsControl = new MyButton() { Text = "Set Bitmask" };
    optionsControl.Click += (s, a) =>
    {
        var mcb = new MavlinkCheckBoxBitMask();
        var list = new MAVLink.MAVLinkParamList();

        // Try and get type so the correct bitmask to value conversion is done
        var type = MAVLink.MAV_PARAM_TYPE.INT32;
        if (MainV2.comPort.MAV.param.ContainsKey(param_name))
        {
            type = MainV2.comPort.MAV.param[param_name].TypeAP;
        }

        list.Add(new MAVLink.MAVLinkParam(param_name, double.Parse(MainV2.comPort.MAV.param[param_name].Value.ToString(), CultureInfo.InvariantCulture),
            type));
        mcb.setup(param_name, list);
        mcb.ValueChanged += (o, x, value) =>
        {
            setParam(param_name, value.ToString());
            setLabelOptions(param_name, label1);
            mcb.Focus();
        };
        var frm = mcb.ShowUserControl();
        frm.TopMost = true;
        //set the location of the form to center of the screen
        frm.Location = new Point((Screen.PrimaryScreen.WorkingArea.Width - frm.Width) / 2,
                                       (Screen.PrimaryScreen.WorkingArea.Height - frm.Height) / 2);
    };

    ThemeManager.ApplyThemeTo(optionsControl);
    tableLayoutPanel1.GetControlFromPosition(4, i)?.Dispose();
    tableLayoutPanel1.Controls.Add(optionsControl, 4, i);
}
```  
## Speed
```C#  
string baudParamName = portName + "_BAUD";
var baudOptions = ParameterMetaDataRepository.GetParameterOptionsInt(baudParamName, MainV2.comPort.MAV.cs.firmware.ToString());
if (baudOptions.Count > 0)
{
    ComboBox cmb = new ComboBox() { Dock = DockStyle.Fill };
    cmb.DropDownStyle = ComboBoxStyle.DropDownList;
    cmb.DataSource = baudOptions;
    cmb.DisplayMember = "Value";
    cmb.ValueMember = "Key";
    cmb.Anchor = AnchorStyles.None;
    cmb.Name = baudParamName;
    widenComboBox(cmb);
    ThemeManager.ApplyThemeTo(cmb);
    tableLayoutPanel1.GetControlFromPosition(1, i)?.Dispose();
    tableLayoutPanel1.Controls.Add(cmb, 1, i);
    int val = -1;
    if (int.TryParse(MainV2.comPort.MAV.param[baudParamName].Value.ToString(), out val))
    {
        cmb.SelectedValue = val;
    }
    else
    {
        cmb.SelectedIndex = -1;
    }
    cmb.SelectedIndexChanged += (s, a) =>
    {
        setParam(cmb.Name, cmb.SelectedValue.ToString());
    };
}
```

# 基本调参
<font color="red">MissionPlanner\GCSViews\ConfigurationView\ConfigArduplane.Designer.cs</font>


## 舵机 Roll Pid
```C#
 // 
// groupBox8
// INT_MAX
this.groupBox8.Controls.Add(this.RLL2SRV_IMAX);
this.groupBox8.Controls.Add(this.label49);
// D
this.groupBox8.Controls.Add(this.RLL2SRV_D);
this.groupBox8.Controls.Add(this.label50);
// I
this.groupBox8.Controls.Add(this.RLL2SRV_I);
this.groupBox8.Controls.Add(this.label51);
// P
this.groupBox8.Controls.Add(this.RLL2SRV_P);
this.groupBox8.Controls.Add(this.label52);
resources.ApplyResources(this.groupBox8, "groupBox8");
this.groupBox8.Name = "groupBox8";
this.groupBox8.TabStop = false;
```
### 参数
```C#
RLL2SRV_IMAX.setup(0, 0, 100, 0, new String[] {"RLL2SRV_IMAX","RLL_RATE_IMAX"}, MainV2.comPort.MAV.param);
RLL2SRV_D.setup(0, 0, 1, 0, new String[] {"RLL2SRV_D","RLL_RATE_D"}, MainV2.comPort.MAV.param);
RLL2SRV_I.setup(0, 0, 1, 0, new String[] {"RLL2SRV_I","RLL_RATE_I"}, MainV2.comPort.MAV.param);
RLL2SRV_P.setup(0, 0, 1, 0, new String[] {"RLL2SRV_P","RLL_RATE_P"}, MainV2.comPort.MAV.param);
```

## L1 控制 - 转向控制
```C#
// 阻尼
this.groupBox4.Controls.Add(this.NAVL1_DAMPING);
this.groupBox4.Controls.Add(this.label9);
// 周期
this.groupBox4.Controls.Add(this.NAVL1_PERIOD);
this.groupBox4.Controls.Add(this.label10);
resources.ApplyResources(this.groupBox4, "groupBox4");
this.groupBox4.Name = "groupBox4";
this.groupBox4.TabStop = false;
```
### 参数
```C#
NAVL1_DAMPING.setup(0, 0, 1, 0, "NAVL1_DAMPING", MainV2.comPort.MAV.param);
NAVL1_PERIOD.setup(0, 0, 1, 0, "NAVL1_PERIOD", MainV2.comPort.MAV.param);
```

## TECS
```C#
// 最大下降(m/s)
this.groupBox5.Controls.Add(this.TECS_SINK_MAX);
this.groupBox5.Controls.Add(this.label15);
// 时间常数
this.groupBox5.Controls.Add(this.TECS_TIME_CONST);
this.groupBox5.Controls.Add(this.label14);
// Pitch抑制
this.groupBox5.Controls.Add(this.TECS_PTCH_DAMP);
this.groupBox5.Controls.Add(this.label13);
// 最小下降(m/s)
this.groupBox5.Controls.Add(this.TECS_SINK_MIN);
this.groupBox5.Controls.Add(this.label11);
// 最大爬升(m/s)
this.groupBox5.Controls.Add(this.TECS_CLMB_MAX);
this.groupBox5.Controls.Add(this.label12);
resources.ApplyResources(this.groupBox5, "groupBox5");
this.groupBox5.Name = "groupBox5";
this.groupBox5.TabStop = false;
```  
### 参数
```C#
TECS_SINK_MAX.setup(0, 0, 1, 0, "TECS_SINK_MAX", MainV2.comPort.MAV.param);
TECS_TIME_CONST.setup(0, 0, 1, 0, "TECS_TIME_CONST", MainV2.comPort.MAV.param);
TECS_PTCH_DAMP.setup(0, 0, 1, 0, "TECS_PTCH_DAMP", MainV2.comPort.MAV.param);
TECS_SINK_MIN.setup(0, 0, 1, 0, "TECS_SINK_MIN", MainV2.comPort.MAV.param);
TECS_CLMB_MAX.setup(0, 0, 1, 0, "TECS_CLMB_MAX", MainV2.comPort.MAV.param);
```

## 舵机Pitch Pid
```C#
// INT_MAX
this.groupBox9.Controls.Add(this.PTCH2SRV_IMAX);
this.groupBox9.Controls.Add(this.label53);
// D
this.groupBox9.Controls.Add(this.PTCH2SRV_D);
this.groupBox9.Controls.Add(this.label54);
// I
this.groupBox9.Controls.Add(this.PTCH2SRV_I);
this.groupBox9.Controls.Add(this.label55);
// P
this.groupBox9.Controls.Add(this.PTCH2SRV_P);
this.groupBox9.Controls.Add(this.label56);
resources.ApplyResources(this.groupBox9, "groupBox9");
this.groupBox9.Name = "groupBox9";
this.groupBox9.TabStop = false;
```  
### 参数
```C#
PTCH2SRV_IMAX.setup(0, 0, 100, 0, new String[] {"PTCH2SRV_IMAX","PTCH_RATE_IMAX"}, MainV2.comPort.MAV.param);
PTCH2SRV_D.setup(0, 0, 1, 0, new String[] {"PTCH2SRV_D","PTCH_RATE_D"}, MainV2.comPort.MAV.param);
PTCH2SRV_I.setup(0, 0, 1, 0, new String[] {"PTCH2SRV_I","PTCH_RATE_I"}, MainV2.comPort.MAV.param);
PTCH2SRV_P.setup(0, 0, 1, 0, new String[] {"PTCH2SRV_P","PTCH_RATE_P"}, MainV2.comPort.MAV.param);
```
## 其它混合
```C#
// 舵混合
  this.groupBox16.Controls.Add(this.KFF_PTCH2THR);
  this.groupBox16.Controls.Add(this.label83);
  // P 至
  this.groupBox16.Controls.Add(this.KFF_RDDRMIX);
  this.groupBox16.Controls.Add(this.label78);
  resources.ApplyResources(this.groupBox16, "groupBox16");
  this.groupBox16.Name = "groupBox16";
  this.groupBox16.TabStop = false;
```
### 参数
```C#
KFF_PTCH2THR.setup(0, 0, 1, 0, new string[] { "KFF_THR2PTCH","KFF_PTCH2THR"}, MainV2.comPort.MAV.param);
KFF_RDDRMIX.setup(0, 0, 1, 0, "KFF_RDDRMIX", MainV2.comPort.MAV.param);
```
## 导航角度
```C#
// 最小Pitch
this.groupBox2.Controls.Add(this.LIM_PITCH_MIN);
this.groupBox2.Controls.Add(this.label39);
// 最大Pitch
this.groupBox2.Controls.Add(this.LIM_PITCH_MAX);
this.groupBox2.Controls.Add(this.label38);
// 转向最大
this.groupBox2.Controls.Add(this.LIM_ROLL_CD);
this.groupBox2.Controls.Add(this.label37);
resources.ApplyResources(this.groupBox2, "groupBox2");
this.groupBox2.Name = "groupBox2";
this.groupBox2.TabStop = false;
```
### 参数
```C#
 LIM_PITCH_MIN.setup(0, 0, 1, 1, "PTCH_LIM_MIN_DEG", MainV2.comPort.MAV.param);
 LIM_PITCH_MAX.setup(0, 0, 1, 1, "PTCH_LIM_MAX_DEG", MainV2.comPort.MAV.param);
 LIM_ROLL_CD.setup(0, 0, 1, 1, "ROLL_LIMIT_DEG", MainV2.comPort.MAV.param);
```
## 舵机 Yaw  

```C#
 // 积分器最大
 this.groupBox10.Controls.Add(this.YAW2SRV_IMAX);
 this.groupBox10.Controls.Add(this.label57);
 // 抑制
 this.groupBox10.Controls.Add(this.YAW2SRV_DAMP);
 this.groupBox10.Controls.Add(this.label58);
 // 积分
 this.groupBox10.Controls.Add(this.YAW2SRV_INT);
 this.groupBox10.Controls.Add(this.label59);
 // yaw 至 roll
 this.groupBox10.Controls.Add(this.YAW2SRV_RLL);
 this.groupBox10.Controls.Add(this.label60);
 resources.ApplyResources(this.groupBox10, "groupBox10");
 this.groupBox10.Name = "groupBox10";
 this.groupBox10.TabStop = false;
```
### 参数
```C#
YAW2SRV_IMAX.setup(0, 0, 100, 0, "YAW2SRV_IMAX", MainV2.comPort.MAV.param);
YAW2SRV_DAMP.setup(0, 0, 1, 0, "YAW2SRV_DAMP", MainV2.comPort.MAV.param);
YAW2SRV_INT.setup(0, 0, 1, 0, "YAW2SRV_INT", MainV2.comPort.MAV.param);
YAW2SRV_RLL.setup(0, 0, 1, 0, "YAW2SRV_RLL", MainV2.comPort.MAV.param);
```
## 油门 0-100%  

```C#
// 旋转速度
 this.groupBox3.Controls.Add(this.THR_SLEWRATE);
 this.groupBox3.Controls.Add(this.label5);
 // 最大
 this.groupBox3.Controls.Add(this.THR_MAX);
 this.groupBox3.Controls.Add(this.label6);
 // 最小
 this.groupBox3.Controls.Add(this.THR_MIN);
 this.groupBox3.Controls.Add(this.label7);
 // 巡航
 this.groupBox3.Controls.Add(this.TRIM_THROTTLE);
 this.groupBox3.Controls.Add(this.label8);
 resources.ApplyResources(this.groupBox3, "groupBox3");
 this.groupBox3.Name = "groupBox3";
 this.groupBox3.TabStop = false;
```
### 参数
```C#
THR_SLEWRATE.setup(0, 0, 1, 0, "THR_SLEWRATE", MainV2.comPort.MAV.param);
THR_MAX.setup(0, 0, 1, 0, "THR_MAX", MainV2.comPort.MAV.param);
THR_MIN.setup(0, 0, 1, 0, "THR_MIN", MainV2.comPort.MAV.param);
TRIM_THROTTLE.setup(0, 0, 1, 0, "TRIM_THROTTLE", MainV2.comPort.MAV.param);
```
## 空速 m/s

```C#
// 比例
this.groupBox1.Controls.Add(this.ARSPD_RATIO);
this.groupBox1.Controls.Add(this.label1);
// FBW 最大
this.groupBox1.Controls.Add(this.ARSPD_FBW_MAX);
this.groupBox1.Controls.Add(this.label2);
// FBW 最小
this.groupBox1.Controls.Add(this.ARSPD_FBW_MIN);
this.groupBox1.Controls.Add(this.label3);
// 巡航
this.groupBox1.Controls.Add(this.TRIM_ARSPD_CM);
this.groupBox1.Controls.Add(this.label4);
resources.ApplyResources(this.groupBox1, "groupBox1");
this.groupBox1.Name = "groupBox1";
this.groupBox1.TabStop = false;
 this.groupBox3.TabStop = false;
```
### 参数
```C#
ARSPD_RATIO.setup(0, 2.5f, 1, 0.005f, "ARSPD_RATIO", MainV2.comPort.MAV.param);
ARSPD_FBW_MAX.setup(0, 0, 1, 1, new string[] { "AIRSPEED_MAX", "ARSPD_FBW_MAX" }, MainV2.comPort.MAV.param);
ARSPD_FBW_MIN.setup(0, 0, 1, 1, new string[] { "AIRSPEED_MIN", "ARSPD_FBW_MIN" }, MainV2.comPort.MAV.param);
TRIM_ARSPD_CM.setup(0, 50, 1, 1, "AIRSPEED_CRUISE", MainV2.comPort.MAV.param);
```
