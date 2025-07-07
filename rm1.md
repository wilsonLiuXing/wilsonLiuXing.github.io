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

# 电池检测器
<font color="red">MissionPlanner\GCSViews\ConfigurationView\ConfigBatteryMonitoring.cs</font>  
## 初始化
<details>
<summary>点击查看代码</summary>  

```C#  

public void Activate()
{
    if (!MainV2.comPort.BaseStream.IsOpen || !MainV2.comPort.MAV.param.ContainsKey("BATT_MONITOR"))
    {
        Enabled = false;
        return;
    }

    startup = true;

    CMB_batmontype.setup(
        ParameterMetaDataRepository.GetParameterOptionsInt("BATT_MONITOR",
            MainV2.comPort.MAV.cs.firmware.ToString()), "BATT_MONITOR", MainV2.comPort.MAV.param);

    if (MainV2.comPort.MAV.param["BATT_CAPACITY"] != null)
        TXT_battcapacity.Text = MainV2.comPort.MAV.param["BATT_CAPACITY"].ToString();

    TXT_voltage.Text = MainV2.comPort.MAV.cs.battery_voltage.ToString();
    TXT_measuredvoltage.Text = TXT_voltage.Text;

    if (MainV2.comPort.MAV.param["BATT_AMP_PERVLT"] != null)
        TXT_AMP_PERVLT.Text = MainV2.comPort.MAV.param["BATT_AMP_PERVLT"].ToString();
    // new
    if (MainV2.comPort.MAV.param["BATT_VOLT_MULT"] != null)
        TXT_divider_VOLT_MULT.Text = MainV2.comPort.MAV.param["BATT_VOLT_MULT"].ToString();

    if (MainV2.comPort.MAV.param["BATT_AMP_PERVOLT"] != null)
        TXT_AMP_PERVLT.Text = MainV2.comPort.MAV.param["BATT_AMP_PERVOLT"].ToString();
    // old
    if (MainV2.comPort.MAV.param["VOLT_DIVIDER"] != null)
        TXT_divider_VOLT_MULT.Text = MainV2.comPort.MAV.param["VOLT_DIVIDER"].ToString();

    if (MainV2.comPort.MAV.param["AMP_PER_VOLT"] != null)
        TXT_AMP_PERVLT.Text = MainV2.comPort.MAV.param["AMP_PER_VOLT"].ToString();

    if (Settings.Instance.GetBoolean("speechbatteryenabled") && Settings.Instance.GetBoolean("speechenable"))
    {
        CHK_speechbattery.Checked = true;
    }
    else
    {
        CHK_speechbattery.Checked = false;
    }

    //http://plane.ardupilot.com/wiki/common-pixhawk-overview/#pixhawk_analog_input_pins_virtual_pin_firmware_mapped_pin_id
    // determine the sensor type
    if (TXT_AMP_PERVLT.Text == (13.6612).ToString() && TXT_divider_VOLT_MULT.Text == (4.127115).ToString())
    {
        CMB_batmonsensortype.SelectedIndex = 1;
    }
    else if (TXT_AMP_PERVLT.Text == (27.3224).ToString() && TXT_divider_VOLT_MULT.Text == (15.70105).ToString())
    {
        CMB_batmonsensortype.SelectedIndex = 2;
    }
    else if (TXT_AMP_PERVLT.Text == (54.64481).ToString() && TXT_divider_VOLT_MULT.Text == (15.70105).ToString())
    {
        CMB_batmonsensortype.SelectedIndex = 3;
    }
    else if (TXT_AMP_PERVLT.Text == (18.0018).ToString() && TXT_divider_VOLT_MULT.Text == (10.10101).ToString())
    {
        CMB_batmonsensortype.SelectedIndex = 4;
    }
    else if (TXT_AMP_PERVLT.Text == (17).ToString() && TXT_divider_VOLT_MULT.Text == (12.02).ToString())
    {
        CMB_batmonsensortype.SelectedIndex = 5;
    }
    else if (TXT_AMP_PERVLT.Text == (24).ToString() && TXT_divider_VOLT_MULT.Text == (12.02).ToString())
    {
        CMB_batmonsensortype.SelectedIndex = 6;
    }
    else if (TXT_AMP_PERVLT.Text == (39.877).ToString() && TXT_divider_VOLT_MULT.Text == (12.02).ToString())
    {
        CMB_batmonsensortype.SelectedIndex = 7;
    }
    else if (TXT_AMP_PERVLT.Text == (24).ToString() && TXT_divider_VOLT_MULT.Text == (18).ToString())
    {
        CMB_batmonsensortype.SelectedIndex = 8;
    }
    else if (TXT_AMP_PERVLT.Text == (36.364).ToString() && TXT_divider_VOLT_MULT.Text == (18.182).ToString())
    {
        CMB_batmonsensortype.SelectedIndex = 9;
    }
    else
    {
        CMB_batmonsensortype.SelectedIndex = 0;
    }

    // determine the board type
    if (MainV2.comPort.MAV.param["BATT_VOLT_PIN"] != null)
    {
        CMB_HWVersion.Enabled = true;

        var value = (double)MainV2.comPort.MAV.param["BATT_VOLT_PIN"];
        if (value == 0) // apm1
        {
            CMB_HWVersion.SelectedIndex = 0;
        }
        else if (value == 1) // apm2
        {
            CMB_HWVersion.SelectedIndex = 1;
        }
        else if (value == 13) // apm2.5
        {
            CMB_HWVersion.SelectedIndex = 2;
        }
        else if (value == 100) // px4
        {
            CMB_HWVersion.SelectedIndex = 3;
        }
        else if (value == 2)
        {
            // pixhawk
            CMB_HWVersion.SelectedIndex = 4;
        }
        else if (value == 6)
        {
            // vrbrain4
            CMB_HWVersion.SelectedIndex = 7;
        }
        else if (value == 10)
        {
            // vrbrain 5 or micro
            if ((double)MainV2.comPort.MAV.param["BATT_CURR_PIN"] == 11)
            {
                CMB_HWVersion.SelectedIndex = 5;
            }
            else
            {
                CMB_HWVersion.SelectedIndex = 6;
            }
        }         
        else if (value == 14)
        {
            // cubeorange
            CMB_HWVersion.SelectedIndex = 8;
        }
        else if (value == 16)
        {
            // durandal
            CMB_HWVersion.SelectedIndex = 9;
        }
         else if (value == 8)
        {
            // Pixhawk 6C/Pix32 v6
            CMB_HWVersion.SelectedIndex = 10;
        }
    }
    else
    {
        CMB_HWVersion.Enabled = false;
    }

    startup = false;

    CMB_batmontype_SelectedIndexChanged(null, null);
    CMB_batmonsensortype_SelectedIndexChanged(null, null);

    timer1.Start();
}
```
</details>  

## 监控器
<details>   
<summary>点击查看代码</summary>  

```C#  
this.CMB_batmontype.DropDownStyle = System.Windows.Forms.ComboBoxStyle.DropDownList;
this.CMB_batmontype.DropDownWidth = 200;
resources.ApplyResources(this.CMB_batmontype, "CMB_batmontype");
this.CMB_batmontype.FormattingEnabled = true;
this.CMB_batmontype.Items.AddRange(new object[] {
resources.GetString("CMB_batmontype.Items"), // 
resources.GetString("CMB_batmontype.Items1"),
resources.GetString("CMB_batmontype.Items2")});
this.CMB_batmontype.Name = "CMB_batmontype";
this.CMB_batmontype.ParamName = null;
this.CMB_batmontype.SubControl = null;
this.CMB_batmontype.SelectedIndexChanged += new System.EventHandler(this.CMB_batmontype_SelectedIndexChanged);
/*  <data name="CMB_batmontype.Items" xml:space="preserve">
    <value>0: Disabled</value>
  </data>
  <data name="CMB_batmontype.Items1" xml:space="preserve">
    <value>3: Battery Volts</value>
  </data>
  <data name="CMB_batmontype.Items2" xml:space="preserve">
    <value>4: Voltage and Current</value>
  </data>*/
// CMB_batmontype_SelectedIndexChanged
  private void CMB_batmontype_SelectedIndexChanged(object sender, EventArgs e)
  {
      if (startup)
          return;
      try
      {
          if (MainV2.comPort.MAV.param["BATT_MONITOR"] == null)
          {
              CustomMessageBox.Show(Strings.ErrorFeatureNotEnabled, Strings.ERROR);
          }
          else
          {
              // 选中的值
              var selection = (int)CMB_batmontype.SelectedValue;
              //传感器
              CMB_batmonsensortype.Enabled = true;
              // 电池电压（计算过）
              TXT_voltage.Enabled = false;

              if (selection == 0)
              {
                  CMB_batmonsensortype.Enabled = false;
                  /// APM 版本
                  CMB_HWVersion.Enabled = false;
                  // 校准
                  groupBox4.Enabled = false;
                  MainV2.comPort.setParam((byte)MainV2.comPort.sysidcurrent, (byte)MainV2.comPort.compidcurrent, "BATT_VOLT_PIN", -1);
                  MainV2.comPort.setParam((byte)MainV2.comPort.sysidcurrent, (byte)MainV2.comPort.compidcurrent, "BATT_CURR_PIN", -1);
              }
              else if (selection == 4)
              {
                  CMB_batmonsensortype.Enabled = true;
                  CMB_HWVersion.Enabled = true;
                  groupBox4.Enabled = true;
                  // 安培每伏
                  TXT_AMP_PERVLT.Enabled = true;
              }
              else if (selection == 3)
              {
                  groupBox4.Enabled = true;
                  CMB_batmonsensortype.Enabled = false;
                  CMB_HWVersion.Enabled = true;
                  TXT_AMP_PERVLT.Enabled = false;
                  TXT_measuredvoltage.Enabled = true;
                  TXT_divider_VOLT_MULT.Enabled = true;
              }

              if (MainV2.comPort.MAV.param.ContainsKey("BATT_MONITOR") &&
                  MainV2.comPort.MAV.param["BATT_MONITOR"].Value == 0 &&
                  selection != 0)
              {
                  MainV2.comPort.setParam((byte)MainV2.comPort.sysidcurrent, (byte)MainV2.comPort.compidcurrent, "BATT_MONITOR", selection);
                  MainV2.comPort.getParamList();
                  this.Activate();
              }
              else
              {
                  MainV2.comPort.setParam((byte)MainV2.comPort.sysidcurrent, (byte)MainV2.comPort.compidcurrent, "BATT_MONITOR", selection);
              }
          }
      }
      catch
      {
          CustomMessageBox.Show("Set BATT_MONITOR,BATT_VOLT_PIN,BATT_CURR_PIN Failed", Strings.ERROR);
      }
  }
```
</details>  

## 传感器
<details>  
<summary>点击查看代码</summary>  

```C#  
this.CMB_batmonsensortype.DropDownWidth = 200;
this.CMB_batmonsensortype.FormattingEnabled = true;
this.CMB_batmonsensortype.Items.AddRange(new object[] {
resources.GetString("CMB_batmonsensortype.Items"),
resources.GetString("CMB_batmonsensortype.Items1"),
resources.GetString("CMB_batmonsensortype.Items2"),
resources.GetString("CMB_batmonsensortype.Items3"),
resources.GetString("CMB_batmonsensortype.Items4"),
resources.GetString("CMB_batmonsensortype.Items5"),
resources.GetString("CMB_batmonsensortype.Items6"),
resources.GetString("CMB_batmonsensortype.Items7"),
resources.GetString("CMB_batmonsensortype.Items8"),
resources.GetString("CMB_batmonsensortype.Items9")});
resources.ApplyResources(this.CMB_batmonsensortype, "CMB_batmonsensortype");
this.CMB_batmonsensortype.Name = "CMB_batmonsensortype";
this.CMB_batmonsensortype.SelectedIndexChanged += new System.EventHandler(this.CMB_batmonsensortype_SelectedIndexChanged);

// CMB_batmonsensortype_SelectedIndexChanged
private void CMB_batmonsensortype_SelectedIndexChanged(object sender, EventArgs e)
{
    var selection = int.Parse(CMB_batmonsensortype.Text.Substring(0, 1));

    if (selection == 1) // atto 45
    {
        var maxvolt = 13.6f;
        var maxamps = 44.7f;
        var mvpervolt = 242.3f;
        var mvperamp = 73.20f;

        // ~ 3.295v
        var topvolt = (maxvolt * mvpervolt) / 1000;
        // ~ 3.294v
        var topamps = (maxamps * mvperamp) / 1000;

        TXT_divider_VOLT_MULT.Text = (maxvolt / topvolt).ToString();
        TXT_AMP_PERVLT.Text = (maxamps / topamps).ToString();
    }
    else if (selection == 2) // atto 90
    {
        var maxvolt = 50f;
        var maxamps = 89.4f;
        var mvpervolt = 63.69f;
        var mvperamp = 36.60f;

        var topvolt = (maxvolt * mvpervolt) / 1000;
        var topamps = (maxamps * mvperamp) / 1000;

        TXT_divider_VOLT_MULT.Text = (maxvolt / topvolt).ToString();
        TXT_AMP_PERVLT.Text = (maxamps / topamps).ToString();
    }
    else if (selection == 3) // atto 180
    {
        var maxvolt = 50f;
        var maxamps = 178.8f;
        var mvpervolt = 63.69f;
        var mvperamp = 18.30f;

        var topvolt = (maxvolt * mvpervolt) / 1000;
        var topamps = (maxamps * mvperamp) / 1000;

        TXT_divider_VOLT_MULT.Text = (maxvolt / topvolt).ToString();
        TXT_AMP_PERVLT.Text = (maxamps / topamps).ToString();
    }
    else if (selection == 4) // 3dr iv
    {
        var maxvolt = 50f;
        var maxamps = 90f;
        var mvpervolt = 99f;
        var mvperamp = 55.55f;

        var topvolt = (maxvolt * mvpervolt) / 1000;
        var topamps = (maxamps * mvperamp) / 1000;

        TXT_divider_VOLT_MULT.Text = (maxvolt / topvolt).ToString();
        TXT_AMP_PERVLT.Text = (maxamps / topamps).ToString();
    }
    else if (selection == 5) // 3dr 4 in one esc
    {
        TXT_divider_VOLT_MULT.Text = (12.02).ToString();
        TXT_AMP_PERVLT.Text = (17).ToString();
    }
    else if (selection == 6) // hv 3dr apm - what i have
    {
        TXT_divider_VOLT_MULT.Text = (12.02).ToString();
        TXT_AMP_PERVLT.Text = (24).ToString();
    }
    else if (selection == 7) // hv 3dr px4 cube
    {
        TXT_divider_VOLT_MULT.Text = (12.02).ToString();
        TXT_AMP_PERVLT.Text = (39.877).ToString();
    }
    else if (selection == 8) // pixhack
    {
        TXT_divider_VOLT_MULT.Text = (18).ToString();
        TXT_AMP_PERVLT.Text = (24).ToString();
    }
    else if (selection == 9) // Holybro Pixhawk4
    {
        TXT_divider_VOLT_MULT.Text = (18.182).ToString();
        TXT_AMP_PERVLT.Text = (36.364).ToString();
    }

    // enable to update
    TXT_divider_VOLT_MULT.Enabled = true;
    TXT_AMP_PERVLT.Enabled = true;
    TXT_measuredvoltage.Enabled = true;

    // update
    TXT_ampspervolt_Validated(TXT_AMP_PERVLT, null);

    TXT_divider_Validated(TXT_divider_VOLT_MULT, null);

    // disable
    TXT_divider_VOLT_MULT.Enabled = false;
    TXT_AMP_PERVLT.Enabled = false;
    TXT_measuredvoltage.Enabled = false;

    //reenable if needed
    if (selection == 0)
    {
        TXT_divider_VOLT_MULT.Enabled = true;
        TXT_AMP_PERVLT.Enabled = true;
        TXT_measuredvoltage.Enabled = true;
    }
}
```
</details>  
  
### 下拉项  

```xml
<data name="CMB_batmonsensortype.Items" xml:space="preserve">
  <value>0: Other</value>
</data>
<data name="CMB_batmonsensortype.Items1" xml:space="preserve">
  <value>1: AttoPilot 45A</value>
</data>
<data name="CMB_batmonsensortype.Items2" xml:space="preserve">
  <value>2: AttoPilot 90A</value>
</data>
<data name="CMB_batmonsensortype.Items3" xml:space="preserve">
  <value>3: AttoPilot 180A</value>
</data>
<data name="CMB_batmonsensortype.Items4" xml:space="preserve">
  <value>4: 3DR Power Module</value>
</data>
<data name="CMB_batmonsensortype.Items5" xml:space="preserve">
  <value>5: 3DR 4 in 1 ESC</value>
</data>
<data name="CMB_batmonsensortype.Items6" xml:space="preserve">
  <value>6: 3DR HV Power Module APM</value>
</data>
<data name="CMB_batmonsensortype.Items7" xml:space="preserve">
  <value>7: Cube HV Power Module</value>
</data>
<data name="CMB_batmonsensortype.Items8" xml:space="preserve">
  <value>8: CUAV HV PM</value>
</data>
<data name="CMB_batmonsensortype.Items9" xml:space="preserve">
  <value>9: Holybro Power Module</value>
</data>
```
## 电池容量

```C#  
 resources.ApplyResources(this.TXT_battcapacity, "TXT_battcapacity");
 this.TXT_battcapacity.Name = "TXT_battcapacity";
 this.TXT_battcapacity.Validated += new System.EventHandler(this.TXT_battcapacity_Validated);
 // TXT_battcapacity_Validated
   private void TXT_battcapacity_Validated(object sender, EventArgs e)
  {
      if (startup || ((TextBox)sender).Enabled == false)
          return;
      try
      {
          if (MainV2.comPort.MAV.param["BATT_CAPACITY"] == null)
          {
              CustomMessageBox.Show(Strings.ErrorFeatureNotEnabled, Strings.ERROR);
          }
          else
          {
              MainV2.comPort.setParam((byte)MainV2.comPort.sysidcurrent, (byte)MainV2.comPort.compidcurrent, "BATT_CAPACITY", float.Parse(TXT_battcapacity.Text));
          }
      }
      catch
      {
          CustomMessageBox.Show("Set BATT_CAPACITY Failed", Strings.ERROR);
      }
  }
```  



## APM版本  
<details>
<summary>点击查看代码</summary>   

```C#
 this.CMB_HWVersion.DropDownWidth = 200;
 this.CMB_HWVersion.FormattingEnabled = true;
 this.CMB_HWVersion.Items.AddRange(new object[] {
 resources.GetString("CMB_HWVersion.Items"),
 resources.GetString("CMB_HWVersion.Items1"),
 resources.GetString("CMB_HWVersion.Items2"),
 resources.GetString("CMB_HWVersion.Items3"),
 resources.GetString("CMB_HWVersion.Items4"),
 resources.GetString("CMB_HWVersion.Items5"),
 resources.GetString("CMB_HWVersion.Items6"),
 resources.GetString("CMB_HWVersion.Items7"),
 resources.GetString("CMB_HWVersion.Items8"),
 resources.GetString("CMB_HWVersion.Items9"),
 resources.GetString("CMB_HWVersion.Items10")});
 resources.ApplyResources(this.CMB_HWVersion, "CMB_HWVersion");
 this.CMB_HWVersion.Name = "CMB_HWVersion";
 this.CMB_HWVersion.SelectedIndexChanged += new System.EventHandler(this.CMB_apmversion_SelectedIndexChanged);

 // CMB_apmversion_SelectedIndexChanged
 private void CMB_apmversion_SelectedIndexChanged(object sender, EventArgs e)
{
    if (startup)
        return;

    var selection = int.Parse(CMB_HWVersion.Text.Substring(0, 2).Replace(":", ""));

    try
    {
        if (selection == 0)
        {
            // apm1
            MainV2.comPort.setParam((byte)MainV2.comPort.sysidcurrent, (byte)MainV2.comPort.compidcurrent, "BATT_VOLT_PIN", 0);
            MainV2.comPort.setParam((byte)MainV2.comPort.sysidcurrent, (byte)MainV2.comPort.compidcurrent, "BATT_CURR_PIN", 1);
        }
        else if (selection == 1)
        {
            // apm2
            MainV2.comPort.setParam((byte)MainV2.comPort.sysidcurrent, (byte)MainV2.comPort.compidcurrent, "BATT_VOLT_PIN", 1);
            MainV2.comPort.setParam((byte)MainV2.comPort.sysidcurrent, (byte)MainV2.comPort.compidcurrent, "BATT_CURR_PIN", 2);
        }
        else if (selection == 2)
        {
            //apm2.5
            MainV2.comPort.setParam((byte)MainV2.comPort.sysidcurrent, (byte)MainV2.comPort.compidcurrent, "BATT_VOLT_PIN", 13);
            MainV2.comPort.setParam((byte)MainV2.comPort.sysidcurrent, (byte)MainV2.comPort.compidcurrent, "BATT_CURR_PIN", 12);
        }
        else if (selection == 3)
        {
            //px4
            MainV2.comPort.setParam((byte)MainV2.comPort.sysidcurrent, (byte)MainV2.comPort.compidcurrent, "BATT_VOLT_PIN", 100);
            MainV2.comPort.setParam((byte)MainV2.comPort.sysidcurrent, (byte)MainV2.comPort.compidcurrent, "BATT_CURR_PIN", 101);
        }
        else if (selection == 4)
        {
            //px4
            MainV2.comPort.setParam((byte)MainV2.comPort.sysidcurrent, (byte)MainV2.comPort.compidcurrent, "BATT_VOLT_PIN", 2);
            MainV2.comPort.setParam((byte)MainV2.comPort.sysidcurrent, (byte)MainV2.comPort.compidcurrent, "BATT_CURR_PIN", 3);
        }
        else if (selection == 5)
        {
            //vrbrain 5
            MainV2.comPort.setParam((byte)MainV2.comPort.sysidcurrent, (byte)MainV2.comPort.compidcurrent, "BATT_VOLT_PIN", 10);
            MainV2.comPort.setParam((byte)MainV2.comPort.sysidcurrent, (byte)MainV2.comPort.compidcurrent, "BATT_CURR_PIN", 11);
        }
        else if (selection == 6)
        {
            //vr micro brain 5
            MainV2.comPort.setParam((byte)MainV2.comPort.sysidcurrent, (byte)MainV2.comPort.compidcurrent, "BATT_VOLT_PIN", 10);
            MainV2.comPort.setParam((byte)MainV2.comPort.sysidcurrent, (byte)MainV2.comPort.compidcurrent, "BATT_CURR_PIN", -1);
        }
        else if (selection == 7)
        {
            //vr brain 4
            MainV2.comPort.setParam((byte)MainV2.comPort.sysidcurrent, (byte)MainV2.comPort.compidcurrent, "BATT_VOLT_PIN", 6);
            MainV2.comPort.setParam((byte)MainV2.comPort.sysidcurrent, (byte)MainV2.comPort.compidcurrent, "BATT_CURR_PIN", 7);
        }
        else if (selection == 8)
        {
            //cube orange
            MainV2.comPort.setParam((byte)MainV2.comPort.sysidcurrent, (byte)MainV2.comPort.compidcurrent, "BATT_VOLT_PIN", 14);
            MainV2.comPort.setParam((byte)MainV2.comPort.sysidcurrent, (byte)MainV2.comPort.compidcurrent, "BATT_CURR_PIN", 15);
        }
        else if (selection == 9)
        {
            //durandal
            MainV2.comPort.setParam((byte)MainV2.comPort.sysidcurrent, (byte)MainV2.comPort.compidcurrent, "BATT_VOLT_PIN", 16);
            MainV2.comPort.setParam((byte)MainV2.comPort.sysidcurrent, (byte)MainV2.comPort.compidcurrent, "BATT_CURR_PIN", 17);
        }
        else if (selection == 10)
        {
            //Pixhawk 6C/Pix32 v6
            MainV2.comPort.setParam((byte)MainV2.comPort.sysidcurrent, (byte)MainV2.comPort.compidcurrent, "BATT_VOLT_PIN", 8);
            MainV2.comPort.setParam((byte)MainV2.comPort.sysidcurrent, (byte)MainV2.comPort.compidcurrent, "BATT_CURR_PIN", 4);
        }
    }
    catch
    {
        CustomMessageBox.Show("Set BATT_????_PIN Failed", Strings.ERROR);
    }
}

```
</details>


## 测量电池电压  
<details>  
<summary>点击查看代码</summary>  

```C#  
 resources.ApplyResources(this.TXT_measuredvoltage, "TXT_measuredvoltage");
 this.TXT_measuredvoltage.Name = "TXT_measuredvoltage";
 this.TXT_measuredvoltage.PreviewKeyDown += new System.Windows.Forms.PreviewKeyDownEventHandler(this.TXT_measuredvoltage_PreviewKeyDown);
 this.TXT_measuredvoltage.Validated += new System.EventHandler(this.TXT_measuredvoltage_Validated);
 // TXT_measuredvoltage_Validated
  private void TXT_measuredvoltage_Validated(object sender, EventArgs e)
 {
     if (startup || ((TextBox)sender).Enabled == false)
         return;
     try
     {
         var measuredvoltage = float.Parse(TXT_measuredvoltage.Text);
         var voltage = float.Parse(TXT_voltage.Text);
         var divider = float.Parse(TXT_divider_VOLT_MULT.Text);
         if (voltage == 0)
             return;
         var new_divider = (measuredvoltage * divider) / voltage;
         TXT_divider_VOLT_MULT.Text = new_divider.ToString();
     }
     catch
     {
         CustomMessageBox.Show(Strings.InvalidNumberEntered, Strings.ERROR);
         return;
     }

     try
     {
         MainV2.comPort.setParam(new[] { "VOLT_DIVIDER", "BATT_VOLT_MULT" }, float.Parse(TXT_divider_VOLT_MULT.Text));
     }
     catch
     {
         if (MainV2.comPort.MAV.param.ContainsKey("BATT_MONITOR") &&
             (MainV2.comPort.MAV.param["BATT_MONITOR"].Value == 3 ||
              MainV2.comPort.MAV.param["BATT_MONITOR"].Value == 4)) {
            CustomMessageBox.Show("Set BATT_VOLT_MULT Failed", Strings.ERROR);
         }
     }
 }
```
</details> 

## 电池电压（计算过）
```C#
 resources.ApplyResources(this.TXT_voltage, "TXT_voltage");
 this.TXT_voltage.Name = "TXT_voltage";
 this.TXT_voltage.ReadOnly = true;
 // 
 this.timer1.Interval = 1000;
this.timer1.Tick += new System.EventHandler(this.timer1_Tick);
 private void timer1_Tick(object sender, EventArgs e)
 {
     TXT_voltage.Text = MainV2.comPort.MAV.cs.battery_voltage.ToString();
     txt_current.Text = MainV2.comPort.MAV.cs.current.ToString();
 }
``` 
## 电压分压器(计算过)
```C#
resources.ApplyResources(this.TXT_divider_VOLT_MULT, "TXT_divider_VOLT_MULT");
this.TXT_divider_VOLT_MULT.Name = "TXT_divider_VOLT_MULT";
this.TXT_divider_VOLT_MULT.PreviewKeyDown += new System.Windows.Forms.PreviewKeyDownEventHandler(this.TXT_divider_PreviewKeyDown);
this.TXT_divider_VOLT_MULT.Validated += new System.EventHandler(this.TXT_divider_Validated);
// TXT_divider_PreviewKeyDown
private void TXT_divider_PreviewKeyDown(object sender, PreviewKeyDownEventArgs e)
{
    if (e.KeyData == Keys.Enter)
        TXT_divider_Validated(sender, e);
}
// TXT_divider_Validated
private void TXT_divider_Validated(object sender, EventArgs e)
{
    if (startup || ((TextBox)sender).Enabled == false)
        return;
    try
    {
        MainV2.comPort.setParam(new[] { "VOLT_DIVIDER", "BATT_VOLT_MULT" }, float.Parse(TXT_divider_VOLT_MULT.Text));
    }
    catch
    {
        if (MainV2.comPort.MAV.param.ContainsKey("BATT_MONITOR") &&
            (MainV2.comPort.MAV.param["BATT_MONITOR"].Value == 3 ||
             MainV2.comPort.MAV.param["BATT_MONITOR"].Value == 4)) {
          CustomMessageBox.Show("Set BATT_VOLT_MULT Failed", Strings.ERROR);
        }
    }
}
```  
## 测量电流
<details>
<summary>点击查看代码</summary>  

```C#  
resources.ApplyResources(this.txt_meascurrent, "txt_meascurrent");
this.txt_meascurrent.Name = "txt_meascurrent";
this.txt_meascurrent.Validated += new System.EventHandler(this.txt_meascurrent_Validated);
// txt_meascurrent_Validated
 private void txt_meascurrent_Validated(object sender, EventArgs e)
 {
     if (startup || ((TextBox)sender).Enabled == false)
         return;
     try
     {
         var measuredcurrent = float.Parse(txt_meascurrent.Text);
         var current = float.Parse(txt_current.Text);
         var divider = float.Parse(TXT_AMP_PERVLT.Text);
         if (current == 0)
             return;
         var new_divider = (measuredcurrent * divider) / current;
         TXT_AMP_PERVLT.Text = new_divider.ToString();
     }
     catch
     {
         CustomMessageBox.Show(Strings.InvalidNumberEntered, Strings.ERROR);
         return;
     }

     try
     {
         MainV2.comPort.setParam(new[] { "AMP_PER_VOLT", "BATT_AMP_PERVOLT", "BATT_AMP_PERVLT" }, float.Parse(TXT_AMP_PERVLT.Text));
     }
     catch
     {
         if (MainV2.comPort.MAV.param.ContainsKey("BATT_MONITOR") &&
             (MainV2.comPort.MAV.param["BATT_MONITOR"].Value == 3 ||
              MainV2.comPort.MAV.param["BATT_MONITOR"].Value == 4)) {
           CustomMessageBox.Show("Set BATT_AMP_PERVOLT Failed", Strings.ERROR);
         }
     }
 }
```
</detials>   

## 电流(计算过)
```C#
 resources.ApplyResources(this.txt_current, "txt_current");
 this.txt_current.Name = "txt_current";
 this.txt_current.ReadOnly = true;
//
this.timer1.Interval = 1000;
this.timer1.Tick += new System.EventHandler(this.timer1_Tick);

 private void timer1_Tick(object sender, EventArgs e)
 {
     TXT_voltage.Text = MainV2.comPort.MAV.cs.battery_voltage.ToString();
     txt_current.Text = MainV2.comPort.MAV.cs.current.ToString();
 }
```
## 安培每伏
```C#
resources.ApplyResources(this.TXT_AMP_PERVLT, "TXT_AMP_PERVLT");
this.TXT_AMP_PERVLT.Name = "TXT_AMP_PERVLT";
this.TXT_AMP_PERVLT.PreviewKeyDown += new System.Windows.Forms.PreviewKeyDownEventHandler(this.TXT_ampspervolt_PreviewKeyDown);
this.TXT_AMP_PERVLT.Validated += new System.EventHandler(this.TXT_ampspervolt_Validated);  

//TXT_ampspervolt_PreviewKeyDown
  private void TXT_ampspervolt_PreviewKeyDown(object sender, PreviewKeyDownEventArgs e)
  {
      if (e.KeyData == Keys.Enter)
          TXT_ampspervolt_Validated(sender, e);
  }
// TXT_ampspervolt_Validated
private void TXT_ampspervolt_Validated(object sender, EventArgs e)
{
    if (startup || ((TextBox)sender).Enabled == false)
        return;
    try
    {
        MainV2.comPort.setParam(new[] { "AMP_PER_VOLT", "BATT_AMP_PERVOLT", "BATT_AMP_PERVLT" }, float.Parse(TXT_AMP_PERVLT.Text));
    }
    catch
    {
        if (MainV2.comPort.MAV.param.ContainsKey("BATT_MONITOR") &&
            (MainV2.comPort.MAV.param["BATT_MONITOR"].Value == 3 ||
             MainV2.comPort.MAV.param["BATT_MONITOR"].Value == 4)) {
          CustomMessageBox.Show("Set BATT_AMP_PERVOLT Failed", Strings.ERROR);
        }
    }
}
```  
## 低电量时MP警告
```C#
 resources.ApplyResources(this.CHK_speechbattery, "CHK_speechbattery");
 this.CHK_speechbattery.Name = "CHK_speechbattery";
 this.CHK_speechbattery.UseVisualStyleBackColor = true;
 this.CHK_speechbattery.CheckedChanged += new System.EventHandler(this.CHK_speechbattery_CheckedChanged);
 // CHK_speechbattery_CheckedChanged
 private void CHK_speechbattery_CheckedChanged(object sender, EventArgs e)
{
    if (startup)
        return;

    // enable the battery event
    Settings.Instance["speechbatteryenabled"] = ((CheckBox)sender).Checked.ToString();
    // enable speech engine
    Settings.Instance["speechenable"] = true.ToString();

    if (((CheckBox)sender).Checked)
    {
        var speechstring = "WARNING, Battery at {batv} Volt, {batp} percent";
        if (Settings.Instance["speechbattery"] != null)
            speechstring = Settings.Instance["speechbattery"].ToString();
        if (DialogResult.Cancel ==
            InputBox.Show("Notification", "What do you want it to say?", ref speechstring))
            return;
        Settings.Instance["speechbattery"] = speechstring;

        speechstring = "9.6";
        if (Settings.Instance["speechbatteryvolt"] != null)
            speechstring = Settings.Instance["speechbatteryvolt"].ToString();
        if (DialogResult.Cancel ==
            InputBox.Show("Battery Level", "What Voltage do you want to warn at?", ref speechstring))
            return;
        Settings.Instance["speechbatteryvolt"] = speechstring;

        speechstring = "20";
        if (Settings.Instance["speechbatterypercent"] != null)
            speechstring = Settings.Instance["speechbatterypercent"].ToString();
        if (DialogResult.Cancel ==
            InputBox.Show("Battery Level", "What percentage do you want to warn at?", ref speechstring))
            return;
        Settings.Instance["speechbatterypercent"] = speechstring;
    }
}
```