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