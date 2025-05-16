# 校准加速度计
 
<font color="red">MissionPlanner\GCSViews\ConfigurationView\ConfigAccelerometerCalibration.cs: 39</font>

> 前置条件，在_incalibrate(校准中)状态下执行。  

1、开始校准
<details>  
<summary>点击查看代码</summary>  

```C#
 try
 {
     count = 0;

     Log.Info("Sending accel command (mavlink 1.0)");

     if (MainV2.comPort.doCommand((byte) MainV2.comPort.sysidcurrent, (byte) MainV2.comPort.compidcurrent,
         MAVLink.MAV_CMD.PREFLIGHT_CALIBRATION, 0, 0, 0, 0, 1, 0, 0))
     {
         _incalibrate = true;

         sub1 = MainV2.comPort.SubscribeToPacketType(MAVLink.MAVLINK_MSG_ID.STATUSTEXT, receivedPacket, (byte)MainV2.comPort.sysidcurrent, (byte)MainV2.comPort.compidcurrent);
         sub2 = MainV2.comPort.SubscribeToPacketType(MAVLink.MAVLINK_MSG_ID.COMMAND_LONG, receivedPacket, (byte)MainV2.comPort.sysidcurrent, (byte)MainV2.comPort.compidcurrent);

         BUT_calib_accell.Text = Strings.Click_when_Done;
     }
     else
     {
         CustomMessageBox.Show(Strings.CommandFailed, Strings.ERROR);
     }
 }
 catch (Exception ex)
 {
     _incalibrate = false;
     Log.Error("Exception on level", ex);
     CustomMessageBox.Show("Failed to level", Strings.ERROR);
 }
```
</details>  


| 参数名   | 对应值 | 说明|  
|----------|-------|-------|
| MAVLink.MAV_CMD.PREFLIGHT_CALIBRATION   | 241  | |  

```C#
MainV2.comPort.sendPacket(new MAVLink.mavlink_command_long_t { 
  param1 = (float)pos, 
  command = (ushort)MAVLink.MAV_CMD.ACCELCAL_VEHICLE_POS },
    MainV2.comPort.sysidcurrent, MainV2.comPort.compidcurrent
);
```

参数表：

| 参数名   | 对应值 |
|----------|-------|
| param1   | `ACCELCAL_VEHICLE` 对应值，祥见下方     |
| command  | 42429 |
| sysidcurrent    | 1    |
| compidcurrent   | 1 |

### 不同状态完成(pos值)
```C#
  ["ACCELCAL_VEHICLE_POS"] = {
      // 水平方式
      [1] = "ACCELCAL_VEHICLE_POS_LEVEL", 
      // 左侧朝下
      [2] = "ACCELCAL_VEHICLE_POS_LEFT", 
      // 右侧朝下
      [3] = "ACCELCAL_VEHICLE_POS_RIGHT",
      // 机头朝下
      [4] = "ACCELCAL_VEHICLE_POS_NOSEDOWN", 
      // 机头朝上
      [5] = "ACCELCAL_VEHICLE_POS_NOSEUP",
      // 机尾朝下
      [6] = "ACCELCAL_VEHICLE_POS_BACK", 
      // 校准成功
      [16777215] = "ACCELCAL_VEHICLE_POS_SUCCESS",
      // 校准失败 
      [16777216] = "ACCELCAL_VEHICLE_POS_FAILED", 
  },
```
### 数据监听
``` C#
if (arg.msgid == (uint)MAVLink.MAVLINK_MSG_ID.STATUSTEXT)
{
    var message = Encoding.ASCII.GetString(arg.ToStructure<MAVLink.mavlink_statustext_t>().text);

    UpdateUserMessage(message);

    if (message.ToLower().Contains("calibration successful") ||
     message.ToLower().Contains("calibration failed"))
    {
        try
        {
            Invoke((MethodInvoker)delegate
            {
                BUT_calib_accell.Text = Strings.Done;
                BUT_calib_accell.Enabled = false;
            });

            _incalibrate = false;
            MainV2.comPort.UnSubscribeToPacketType(sub1);
            MainV2.comPort.UnSubscribeToPacketType(sub2);
        }
        catch
        {
        }
    }
}
```  

> 这里当响应成功/失败指令后，停止继续校准。
  
# 水平校准
<font colot="red">MissionPlanner\GCSViews\ConfigurationView\ConfigAccelerometerCalibration.cs</font>        

```C#
private void BUT_level_Click(object sender, EventArgs e)
{
    try
    {
        Log.Info("Sending level command (mavlink 1.0)");
        if (MainV2.comPort.doCommand((byte) MainV2.comPort.sysidcurrent, (byte) MainV2.comPort.compidcurrent,
            MAVLink.MAV_CMD.PREFLIGHT_CALIBRATION, 0, 0, 0, 0, 2, 0, 0))
        {
            BUT_level.Text = Strings.Completed;
        }
        else
        {
            CustomMessageBox.Show(Strings.CommandFailed, Strings.ERROR);
        }
    }
    catch (Exception ex)
    {
        Log.Error("Exception on level", ex);
        CustomMessageBox.Show("Failed to level", Strings.ERROR);
    }
}
```
| 参数名         | 类型       | 描述                                   |
|----------------|------------|----------------------------------------|
| MAVLink.MAV_CMD.PREFLIGHT_CALIBRATION   | `int`     | 241     |

> 水平校准只发一条命令，直接返回执行成功还是执行失败。 <strong>地面站测试，大概过了几秒返回 `false` </strong>


# 指南针

<font color="red">MissionPlanner\GCSViews\ConfigurationView\ConfigHWCompass2.cs</font>
## 获取参数命令
1、获取设备 `DEV_ID` :  `"COMPASS_DEV_ID"`、`"COMPASS_DEV_ID2"`、`"COMPASS_DEV_ID3"`。  
2、获取设备是否外置 `externa` :  `"COMPASS_EXTERNAL"`、`"COMPASS_EXTERN2"`、`"COMPASS_EXTERN3"`  
2、获取设备方向 `oriention` :  `"COMPASS_ORIENT"`、`"COMPASS_ORIENT2"`、`"COMPASS_ORIENT3"`

## 解析参数
> 在罗盘中，`Bus`、`BusType`、`Address`、`DevType`参数，是通过`DEV_ID`进行拆解得到。
<font color="red">MissionPlanner\ExtLibs\Utilities\Device.cs</font>  
```C#
public BusType bus_type { get { return (BusType)(devid & 0x7); } } // : 3;
public byte bus { get { return (byte)((devid >> 3) & 0x1f); } } //: 5;    // which instance of the bus type
public byte address { get { return (byte)((devid >> 8) & 0xff); } } // address on the bus (eg. I2C address)
public byte devtype { get { return (byte)((devid >> 16) & 0xff); } }
````

## 一、排序
<details>  
<summary>点击展开代码</summary>  

```C#
 private async Task UpdateFirst3()
 {
     if (myDataGridView1.Rows.Count >= 1)
     {
         list[0]._index = 0;
         bool p1 = await MainV2.comPort.setParamAsync((byte)MainV2.comPort.sysidcurrent,
             (byte)MainV2.comPort.compidcurrent,
             "COMPASS_PRIO1_ID",
             int.Parse(myDataGridView1.Rows[0].Cells[devIDDataGridViewTextBoxColumn.Index].Value.ToString()));

         if (!p1)
             CustomMessageBox.Show(Strings.ErrorSettingParameter, Strings.ERROR);
     }

     if (myDataGridView1.Rows.Count >= 2)
     {
         list[1]._index = 1;
         bool p2 = await MainV2.comPort.setParamAsync((byte)MainV2.comPort.sysidcurrent,
             (byte)MainV2.comPort.compidcurrent,
             "COMPASS_PRIO2_ID",
             int.Parse(myDataGridView1.Rows[1].Cells[devIDDataGridViewTextBoxColumn.Index].Value.ToString()));

         if (!p2)
             CustomMessageBox.Show(Strings.ErrorSettingParameter, Strings.ERROR);
     }
     else
     {
         // clear it
         await MainV2.comPort.setParamAsync((byte)MainV2.comPort.sysidcurrent,
             (byte)MainV2.comPort.compidcurrent,
             "COMPASS_PRIO2_ID",
             0);
     }

     if (myDataGridView1.Rows.Count >= 3)
     {
         list[2]._index = 2;
         bool p3 = await MainV2.comPort.setParamAsync((byte)MainV2.comPort.sysidcurrent,
             (byte)MainV2.comPort.compidcurrent,
             "COMPASS_PRIO3_ID",
             int.Parse(myDataGridView1.Rows[2].Cells[devIDDataGridViewTextBoxColumn.Index].Value.ToString()));

         if (!p3)
             CustomMessageBox.Show(Strings.ErrorSettingParameter, Strings.ERROR);
     }
     else
     {
         //clear it
         await MainV2.comPort.setParamAsync((byte)MainV2.comPort.sysidcurrent,
             (byte)MainV2.comPort.compidcurrent,
             "COMPASS_PRIO3_ID",
             0);
     }

     rebootrequired = true;

     myDataGridView1.Invalidate();
 }
```  
</details>

>  这里是针对每个指南针进行单独设置排序
主要代码：
```C#
 await MainV2.comPort.setParamAsync((byte)MainV2.comPort.sysidcurrent,
             (byte)MainV2.comPort.compidcurrent,
             "COMPASS_PRIO3_ID",
             int.Parse(myDataGridView1.Rows[2].Cells[devIDDataGridViewTextBoxColumn.Index].Value.ToString()));
```
| 参数名         | 类型       | 描述                                   |
|----------------|------------|----------------------------------------|
| sysidcurrent   | `byte`     | 系统 ID，固定值     |
| compidcurrent  | `byte`     | 组件 ID，固定值     |
| paramname       | `string`   |`"COMPASS_PRIO1_ID"`：表示第一行，`"COMPASS_PRIO2_ID"`：表示第二行， `"COMPASS_PRIO3_ID"`：表示第三行|
| value         | `int`      | 对应的 `DevId`     |

> 当只有一个指南针时，`COMPASS_PRIO2_ID`、`COMPASS_PRIO3_ID`的 `value`值要设置为 `0`。

> 支持最大指南针数量为3个。

## 二、指南针启用/禁用
<font color="red">MissionPlanner\Controls\MavlinkCheckBox.cs</font>
```C#
 bool ans = MainV2.comPort.setParam((byte)MainV2.comPort.sysidcurrent, (byte)MainV2.comPort.compidcurrent, ParamName, OnValue);
```
| 参数名         | 类型       | 描述                                   |
|----------------|------------|----------------------------------------|
| sysidcurrent   | `byte`     | 系统 ID，固定值     |
| compidcurrent  | `byte`     | 组件 ID，固定值     |
| ParamName       | `string`   |`"COMPASS_USE"`：表示第一个指南针，`"COMPASS_USE2"`：表示第二个指南针， `"COMPASS_USE3"`：表示第三个指南针|
| OnValue         | `int`      | 对应的开关状态，`OffValue = 0D`,`OnValue = 1D`    |

> 注意：`"COMPASS_USE"` 并没有 `1` , MissionPlanner中就是如此定义的。

## 三、安装方向(地面站没有下拉选项，具体无法debug)
>目前下拉框是禁用状态，没有响应时间。  

数据源
<font color="red">MissionPlanner\GCSViews\ConfigurationView\ConfigHWCompass2.cs:133-138</font>
```C#
var source = ParameterMetaDataRepository.GetParameterOptionsInt(orient_param.Name,
        MainV2.comPort.MAV.cs.firmware.ToString())
    .Select(a => new KeyValuePair<string, string>(a.Key.ToString(), a.Value)).ToList();
Orientation.DataSource = source;
Orientation.DisplayMember = "Value";
Orientation.ValueMember = "Key";
```
配置参数：
```xml
<COMPASS_ORIENT>
  <DisplayName>Compass orientation</DisplayName>
  <Description>The orientation of the first external compass relative to the vehicle frame. This value will be ignored unless this compass is set as an external compass. When set correctly in the northern hemisphere, pointing the nose and right side down should increase the MagX and MagY values respectively. Rolling the vehicle upside down should decrease the MagZ value. For southern hemisphere, switch increase and decrease. NOTE: For internal compasses, AHRS_ORIENT is used.</Description>
  <Values>0:None,1:Yaw45,2:Yaw90,3:Yaw135,4:Yaw180,5:Yaw225,6:Yaw270,7:Yaw315,8:Roll180,9:Roll180Yaw45,10:Roll180Yaw90,11:Roll180Yaw135,12:Pitch180,13:Roll180Yaw225,14:Roll180Yaw270,15:Roll180Yaw315,16:Roll90,17:Roll90Yaw45,18:Roll90Yaw90,19:Roll90Yaw135,20:Roll270,21:Roll270Yaw45,22:Roll270Yaw90,23:Roll270Yaw135,24:Pitch90,25:Pitch270,26:Pitch180Yaw90,27:Pitch180Yaw270,28:Roll90Pitch90,29:Roll180Pitch90,30:Roll270Pitch90,31:Roll90Pitch180,32:Roll270Pitch180,33:Roll90Pitch270,34:Roll180Pitch270,35:Roll270Pitch270,36:Roll90Pitch180Yaw90,37:Roll90Yaw270,38:Yaw293Pitch68Roll180,39:Pitch315,40:Roll90Pitch315</Values>
  <User>Advanced</User>
</COMPASS_ORIENT>
```


## 四、自动调整偏移量（实现与指南针启用/禁用一致）  

<font color="red">MissionPlanner\Controls\MavlinkCheckBox.cs</font>  

```C#
 bool ans = MainV2.comPort.setParam((byte)MainV2.comPort.sysidcurrent, (byte)MainV2.comPort.compidcurrent, ParamName, OnValue);
```
| 参数名         | 类型       | 描述                                   |
|----------------|------------|----------------------------------------|
| sysidcurrent   | `byte`     | 系统 ID，固定值     |
| compidcurrent  | `byte`     | 组件 ID，固定值     |
| ParamName       | `string`   |`"COMPASS_LEARN"`|
| OnValue         | `int`      | 对应的开关状态，`OffValue = 0D`,`OnValue = 1D`    |

## 五、重启
```C#
MainV2.comPort.doReboot(bootloadermode, currentvehicle);
```
| 参数名         | 类型       | 描述                                   |
|----------------|------------|----------------------------------------|
| bootloadermode   | `boolen`     | 是否重启并进入Bootloader模式     |
| currentvehicle  | `boolen`     | 使用当前sysid/compid或扫描它 |

## 五、指南针校准
<font color="red">MissionPlanner\GCSViews\ConfigurationView\ConfigHWCompass2.cs</font>
### 开始校准
<details>  
<summary>点击查看代码</summary>  

```C#
private void BUT_OBmagcalstart_Click(object sender, EventArgs e)
{
    if (rebootrequired && !CheckReboot())
    {
        return;
    }

    try
    {
        /// 发送开始指令
        MainV2.comPort.doCommand((byte)MainV2.comPort.sysidcurrent, (byte)MainV2.comPort.compidcurrent, MAVLink.MAV_CMD.DO_START_MAG_CAL, 0, 1, 1, 0, 0, 0, 0);
    }
    catch (Exception ex)
    {
        this.LogError(ex);
        CustomMessageBox.Show("Failed to start MAG CAL, check the autopilot is still responding.\n" + ex.ToString(), Strings.ERROR);
        return;
    }
    /// 重置数据
    mprog.Clear();
    mrep.Clear();
    horizontalProgressBar1.Value = 0;
    horizontalProgressBar2.Value = 0;
    horizontalProgressBar3.Value = 0;

    /// 订阅数据包
    packetsub1 = MainV2.comPort.SubscribeToPacketType(MAVLink.MAVLINK_MSG_ID.MAG_CAL_PROGRESS, ReceviedPacket, (byte)MainV2.comPort.sysidcurrent, (byte)MainV2.comPort.compidcurrent);
    packetsub2 = MainV2.comPort.SubscribeToPacketType(MAVLink.MAVLINK_MSG_ID.MAG_CAL_REPORT, ReceviedPacket, (byte)MainV2.comPort.sysidcurrent, (byte)MainV2.comPort.compidcurrent);
    BUT_OBmagcalaccept.Enabled = true;
    BUT_OBmagcalcancel.Enabled = true;
    timer1.Start();
}
```
</details>  

| 参数名         | 类型       | 对应值                                   |  
|----------------|------------|----------------------------------------|  
| MAVLink.MAV_CMD.DO_START_MAG_CAL   | `int`     | 42424     |
| MAVLink.MAVLINK_MSG_ID.MAG_CAL_PROGRESS  | `int`     | 191 |
| MAVLink.MAVLINK_MSG_ID.MAG_CAL_REPORT  | `int`     | 192 |

### 完成校准

<details>  
<summary>点击查看代码</summary>  

```C#
private void BUT_OBmagcalaccept_Click(object sender, EventArgs e)
{
    try
    {
        /// 完成校准指令
        MainV2.comPort.doCommand((byte)MainV2.comPort.sysidcurrent, (byte)MainV2.comPort.compidcurrent, MAVLink.MAV_CMD.DO_ACCEPT_MAG_CAL, 0, 0, 1, 0, 0, 0, 0);

    }
    catch (Exception ex)
    {
        CustomMessageBox.Show(ex.ToString(), Strings.ERROR, MessageBoxButtons.OK);
    }
    /// 取消数据订阅
    MainV2.comPort.UnSubscribeToPacketType(packetsub1);
    MainV2.comPort.UnSubscribeToPacketType(packetsub2);

    timer1.Stop();
}
```
</details>  

| 参数名         | 类型       | 对应值                                   |  
|----------------|------------|----------------------------------------|  
| MAVLink.MAV_CMD.DO_ACCEPT_MAG_CAL   | `int`     | 42425     |


### 取消校准

<details>  
<summary>点击查看代码</summary>  

```C#
private void BUT_OBmagcalcancel_Click(object sender, EventArgs e)
{
    try
    {
        /// 取消校准
        MainV2.comPort.doCommand((byte)MainV2.comPort.sysidcurrent, (byte)MainV2.comPort.compidcurrent, MAVLink.MAV_CMD.DO_CANCEL_MAG_CAL, 0, 0, 1, 0, 0, 0, 0);
    }
    catch (Exception ex)
    {
        CustomMessageBox.Show(ex.ToString(), Strings.ERROR, MessageBoxButtons.OK);
    }
    /// 取消订阅数据包
    MainV2.comPort.UnSubscribeToPacketType(packetsub1);
    MainV2.comPort.UnSubscribeToPacketType(packetsub2);

    timer1.Stop();
}
```
</details>  

| 参数名         | 类型       | 对应值                                   |  
|----------------|------------|----------------------------------------|  
| MAVLink.MAV_CMD.DO_CANCEL_MAG_CAL   | `int`     | 42426     |


### 校准模式
1、配置参数（下拉框数据）
<font color="red">DMissionPlanner\GCSViews\ConfigurationView\ConfigHWCompass2.cs：121-122</font>  
```C#
mavlinkComboBoxfitness.setup(ParameterMetaDataRepository.GetParameterOptionsInt("COMPASS_CAL_FIT",
    MainV2.comPort.MAV.cs.firmware.ToString()), "COMPASS_CAL_FIT", MainV2.comPort.MAV.param);
```
| 参数名         | 类型       | 说明                                   |  
|----------------|------------|----------------------------------------|  
| source   | `List<KeyValuePair<int, string>>`     | 下拉列表数据     |
| paramname   | `String`     | `"COMPASS_CAL_FIT"`     |
| paramlist   | `MAVLink.MAVLinkParamList`     | 当前选中值(不确定，有所怀疑)  |

2、改变校验模式  

<font color="red">MissionPlanner\Controls\MavlinkComboBox.cs：180-183</font>  

```C#
if (!MainV2.comPort.setParam((byte)MainV2.comPort.sysidcurrent, (byte)MainV2.comPort.compidcurrent, ParamName, (float)(int)((MavlinkComboBox)sender).SelectedValue))
{
    CustomMessageBox.Show("Set " + ParamName + " Failed!", Strings.ERROR);
}
```
| 参数名         | 类型       | 说明                                   |  
|----------------|------------|----------------------------------------|  
| paramname   | `String`     | `"COMPASS_CAL_FIT"`      |
| value   | `double`     | 选择的值  |    

3、下拉数据参数  

<font color="red">MissionPlanner\ExtLibs\wasm\wwwroot\ParameterMetaDataBackup.xml： 1943-1950</font>  
```xml
<COMPASS_CAL_FIT>
      <DisplayName>Compass calibration fitness</DisplayName>
      <Description>This controls the fitness level required for a successful compass calibration. A lower value makes for a stricter fit (less likely to pass). This is the value used for the primary magnetometer. Other magnetometers get double the value.</Description>
      <Range>4 32</Range>
      <!-- 这里是对应配置 -->
      <Values>4:Very Strict,8:Strict,16:Default,32:Relaxed</Values>
      <Increment>0.1</Increment>
      <User>Advanced</User>
    </COMPASS_CAL_FIT>
    <COMPASS_DEC>
```

# 设置安装飞控方向  

<font color="red">MissionPlanner\GCSViews\ConfigurationView\ConfigRawParams.cs</font>

### 一、参数
```xml
 <AHRS_ORIENTATION>
   <DisplayName>Board Orientation</DisplayName>
   <Description>Overall board orientation relative to the standard orientation for the board type. This rotates the IMU and compass readings to allow the board to be oriented in your vehicle at any 90 or 45 degree angle. This option takes affect on next boot. After changing you will need to re-level your vehicle.</Description>
   <Values>0:None,1:Yaw45,2:Yaw90,3:Yaw135,4:Yaw180,5:Yaw225,6:Yaw270,7:Yaw315,8:Roll180,9:Roll180Yaw45,10:Roll180Yaw90,11:Roll180Yaw135,12:Pitch180,13:Roll180Yaw225,14:Roll180Yaw270,15:Roll180Yaw315,16:Roll90,17:Roll90Yaw45,18:Roll90Yaw90,19:Roll90Yaw135,20:Roll270,21:Roll270Yaw45,22:Roll270Yaw90,23:Roll270Yaw135,24:Pitch90,25:Pitch270,26:Pitch180Yaw90,27:Pitch180Yaw270,28:Roll90Pitch90,29:Roll180Pitch90,30:Roll270Pitch90,31:Roll90Pitch180,32:Roll270Pitch180,33:Roll90Pitch270,34:Roll180Pitch270,35:Roll270Pitch270,36:Roll90Pitch180Yaw90,37:Roll90Yaw270,38:Yaw293Pitch68Roll180,39:Pitch315,40:Roll90Pitch315,100:Custom</Values>
   <User>Advanced</User>
 </AHRS_ORIENTATION>
```
### 二、设置指令  
<details>
 <summary>点击查看代码</summary>  
 
 ```C#
private void BUT_writePIDS_Click(object sender, EventArgs e)
{
    if (Common.MessageShowAgain("Write Raw Params", "Are you Sure?") != DialogResult.OK)
        return;

    // sort with enable at the bottom - this ensures params are set before the function is disabled
    // _changes为Map,key为“命令字段”，value为修改值。
    var temp = _changes.Keys.Cast<string>().ToList();

    temp.SortENABLE();

    bool enable = temp.Any(a => a.EndsWith("_ENABLE"));

    int error = 0;
    bool reboot = false;

    foreach (string value in temp)
    {
        try
        {
            if (MainV2.comPort.BaseStream == null || !MainV2.comPort.BaseStream.IsOpen)
            {
                CustomMessageBox.Show("Your are not connected", Strings.ERROR);
                return;
            }

            MainV2.comPort.setParam(value, (double)_changes[value]);
            //check if reboot required
            if (ParameterMetaDataRepository.GetParameterRebootRequired(value, MainV2.comPort.MAV.cs.firmware.ToString()))
            {
                reboot = true;
            }
            try
            {
                // set control as well
                var textControls = Controls.Find(value, true);
                if (textControls.Length > 0)
                {
                    ThemeManager.ApplyThemeTo(textControls[0]);
                }
            }
            catch
            {
            }

            try
            {
                // set param table as well
                foreach (DataGridViewRow row in Params.Rows)
                {
                    if (row.Cells[Command.Index].Value.ToString() == value)
                    {
                        row.Cells[Value.Index].Style.BackColor = ThemeManager.ControlBGColor;
                        _changes.Remove(value);
                        break;
                    }
                }
            }
            catch
            {
            }
        }
        catch
        {
            error++;
            CustomMessageBox.Show("Set " + value + " Failed");
        }
    }
```
</details>

# 遥控器校准

## 一、接收机链接端口
>对应的下拉框数据 <code>UARTx</code>，中的<code>x</code>代码对应的<code>SERIAL</code>,目前地面站分别为<code>SERIAL0-6</code>。  
>1、假如选择了<code>UART1</code>，那么设置<code>SERIAL1_PROTOCOL = 23</code>、<code>SERIAL1_PROTOCOL = 57</code>。  

代码参考：[设置安装飞控方向](#设置安装飞控方向)

## 二、接收机协议(全部参数列表-RC_PROTOCOLS)

1、参数
| 编号 | 协议名称   |
|------|------------|
| 0    | All        |
| 1    | PPM        |
| 2    | IBUS       |
| 3    | SBUS       |
| 4    | SBUS_NI    |
| 5    | DSM        |
| 6    | SUMD       |
| 7    | SRXL       |
| 8    | SRXL2      |
| 9    | CRSF       |
| 10   | ST24       |
| 11   | FPORT      |
| 12   | FPORT2     |
| 13   | FastSBUS   |
| 14   | DroneCAN   |
| 15   | Ghost      |

更多详情请参考：[RC_PROTOCOLS 参数文档](https://ardupilot.org/copter/docs/parameters.html#rc-protocols-rc-protocols-enabled)

2、写入代码参考：[设置安装飞控方向](#设置安装飞控方向)
## 三、通道数据  

<font color="red">MissionPlanner\GCSViews\ConfigurationView\ConfigRadioInput.cs</font>  

一、"反转"按钮
> 是否反转并不与飞控实现交互，纯界面展示。但要检查 <code>SWITCH_ENABLE</code>值是否为 <code>1</code> ,如果是则要设置为 <code>0</code> 。  

```C#
 if (MainV2.comPort.MAV.param["SWITCH_ENABLE"] != null &&
     (float)MainV2.comPort.MAV.param["SWITCH_ENABLE"] == 1)
 {
     try
     {
         MainV2.comPort.setParam((byte)MainV2.comPort.sysidcurrent, (byte)MainV2.comPort.compidcurrent, "SWITCH_ENABLE", 0);
         CustomMessageBox.Show("Disabled Dip Switchs");
     }
     catch
     {
         CustomMessageBox.Show("Error Disableing Dip Switch");
     }
 }
```  
二、通道数据设置  
1、ch1至ch16界面布局
```C#
this.BARpitch.BackgroundColor = System.Drawing.Color.FromArgb(((int)(((byte)(20)))), ((int)(((byte)(20)))), ((int)(((byte)(255)))));
this.BARpitch.BorderColor = System.Drawing.SystemColors.ActiveBorder;
this.BARpitch.DisplayScale = 1F;
this.BARpitch.DrawLabel = true;
this.BARpitch.Label = "Pitch";
resources.ApplyResources(this.BARpitch, "BARpitch");
this.BARpitch.Maximum = 2200;
this.BARpitch.maxline = 0;
this.BARpitch.Minimum = 800;
this.BARpitch.minline = 0;
this.BARpitch.Name = "BARpitch";
this.BARpitch.Value = 1500;
this.BARpitch.ValueColor = System.Drawing.Color.FromArgb(((int)(((byte)(255)))), ((int)(((byte)(0)))), ((int)(((byte)(255)))));
```  
2、数据获取
>在代码中没有找到对应的取值方式，GTP说通过 <code>RC_CHANNELS</code>指令获取。  

三、校准遥控
<details>
<summary>点击查看代码</summary>  

```C#  
// 遥控器校准按钮点击事件处理函数
private void BUT_Calibrateradio_Click(object sender, EventArgs e)
{
    // 如果校准正在进行（run = true），则结束校准
    if (run)
    {
        BUT_Calibrateradio.Text = Strings.Completed; // 更新按钮文本为“完成”
        run = false; // 标记校准结束
        return;
    }

    // 显示安全警告：确保遥控器和接收机已通电，且电机未供电/未安装螺旋桨
    CustomMessageBox.Show(
        "Ensure your transmitter is on and receiver is powered and connected\nEnsure your motor does not have power/no props!!!");

    // 保存当前飞控的数据流速率（用于后续恢复）
    var oldrc = MainV2.comPort.MAV.cs.raterc;          // 遥控器通道数据速率
    var oldatt = MainV2.comPort.MAV.cs.rateattitude;   // 姿态数据速率
    var oldpos = MainV2.comPort.MAV.cs.rateposition;   // 位置数据速率
    var oldstatus = MainV2.comPort.MAV.cs.ratestatus;  // 状态数据速率

    // 临时调整数据流速率：优先接收遥控器通道数据（10Hz），其他数据流暂停
    MainV2.comPort.MAV.cs.raterc = 10;         // 设置遥控器通道数据速率为10Hz
    MainV2.comPort.MAV.cs.rateattitude = 0;    // 禁用姿态数据流
    MainV2.comPort.MAV.cs.rateposition = 0;    // 禁位置数据流
    MainV2.comPort.MAV.cs.ratestatus = 0;      // 禁用状态数据流

    // 请求飞控发送遥控器通道数据流（MAVLink协议）
    try
    {
        MainV2.comPort.requestDatastream(MAVLink.MAV_DATA_STREAM.RC_CHANNELS, 10);
    }
    catch { /* 忽略请求失败 */ }

    // 更新按钮文本为“点击完成”
    BUT_Calibrateradio.Text = Strings.Click_when_Done;

    // 提示用户操作：移动所有摇杆和开关到极限位置
    CustomMessageBox.Show(
        "Click OK and move all RC sticks and switches to their\nextreme positions so the red bars hit the limits.");

    run = true; // 标记校准开始

    // 主校准循环：实时检测通道值并更新最小/最大值
    while (run)
    {
        Application.DoEvents(); // 处理UI事件（避免界面卡死）
        Thread.Sleep(5);       // 短暂延迟（5ms），降低CPU占用

        // 更新当前飞控状态数据（绑定到UI）
        MainV2.comPort.MAV.cs.UpdateCurrentSettings(
            currentStateBindingSource.UpdateDataSource(MainV2.comPort.MAV.cs), 
            true, 
            MainV2.comPort
        );

        // 检查通道1输入是否有效（800-2200μs为合理PWM范围）
        if (MainV2.comPort.MAV.cs.ch1in > 800 && MainV2.comPort.MAV.cs.ch1in < 2200)
        {
            // 遍历所有16个通道，更新每个通道的最小值（rcmin）和最大值（rcmax）
            for (int i = 0; i < 16; i++)
            {
                rcmin[i] = Math.Min(rcmin[i], MainV2.comPort.MAV.cs.GetChannelValue(i)); // 更新最小值
                rcmax[i] = Math.Max(rcmax[i], MainV2.comPort.MAV.cs.GetChannelValue(i)); // 更新最大值
            }

            // 更新主控制通道（横滚、俯仰、油门、偏航）的UI显示
            BARroll.minline = (int)rcmin[chroll - 1];    // 横滚通道最小值
            BARroll.maxline = (int)rcmax[chroll - 1];    // 横滚通道最大值
            BARpitch.minline = (int)rcmin[chpitch - 1];  // 俯仰通道最小值
            BARpitch.maxline = (int)rcmax[chpitch - 1]; // 俯仰通道最大值
            BARthrottle.minline = (int)rcmin[chthro - 1];// 油门通道最小值
            BARthrottle.maxline = (int)rcmax[chthro - 1];// 油门通道最大值
            BARyaw.minline = (int)rcmin[chyaw - 1];      // 偏航通道最小值
            BARyaw.maxline = (int)rcmax[chyaw - 1];      // 偏航通道最大值

            // 更新辅助通道（5-16）的UI显示
            for (int i = 4; i < 16; i++)
            {
                setBARStatus(GetBarControl(i + 1), rcmin[i], rcmax[i]); // 设置通道条形图范围
            }
        }
    }

    // 校验通道1输入是否有效
    if (rcmin[0] <= 800 || rcmin[0] >= 2200)
    {
        CustomMessageBox.Show("Bad channel 1 input, canceling"); // 输入异常提示
        return;
    }

    // 提示用户将摇杆回中并确认
    CustomMessageBox.Show("Ensure all your sticks are centered and throttle is down, and click ok to continue");

    // 更新飞控状态数据（获取当前居中值作为“微调”值）
    MainV2.comPort.MAV.cs.UpdateCurrentSettings(
        currentStateBindingSource.UpdateDataSource(MainV2.comPort.MAV.cs), 
        true, 
        MainV2.comPort
    );

    // 记录每个通道的居中值（Trim值）
    for (int i = 0; i < 16; i++)
    {
        rctrim[i] = Constrain(MainV2.comPort.MAV.cs.GetChannelValue(i), i); // 约束并保存Trim值
    }

    // 生成校准结果日志
    var data = "---------------\n";
    for (var a = 0; a < rctrim.Length; a++)
    {
        BUT_Calibrateradio.Text = Strings.Saving; // 更新按钮文本为“保存中”
        try
        {
            // 校验数据有效性：最小值 < 最大值，且Trim值在范围内
            if (rcmin[a] < rcmax[a] && rcmin[a] != 0 && rcmax[a] != 0 &&
                rctrim[a] <= rcmax[a] && rctrim[a] >= rcmin[a] && rctrim[a] != 0 &&
                rcmin[a] != rcmax[a])
            {
                // 将校准结果写入飞控参数（RCx_MIN, RCx_MAX, RCx_TRIM）
                MainV2.comPort.setParam(
                    (byte)MainV2.comPort.sysidcurrent, 
                    (byte)MainV2.comPort.compidcurrent,
                    "RC" + (a + 1).ToString("0") + "_MIN", rcmin[a], true
                );
                MainV2.comPort.setParam(
                    (byte)MainV2.comPort.sysidcurrent, 
                    (byte)MainV2.comPort.compidcurrent,
                    "RC" + (a + 1).ToString("0") + "_MAX", rcmax[a], true
                );
                MainV2.comPort.setParam(
                    (byte)MainV2.comPort.sysidcurrent, 
                    (byte)MainV2.comPort.compidcurrent,
                    "RC" + (a + 1).ToString("0") + "_TRIM", rctrim[a], true
                );
            }
        }
        catch
        {
            if (MainV2.comPort.MAV.param.ContainsKey("RC" + (a + 1).ToString("0") + "_MIN"))
                CustomMessageBox.Show("Failed to set Channel " + (a + 1)); // 参数写入失败提示
        }

        data = data + "CH" + (a + 1) + " " + rcmin[a] + " | " + rcmax[a] + "\n"; // 记录通道范围
    }

    // 恢复原始数据流速率
    MainV2.comPort.MAV.cs.raterc = oldrc;
    MainV2.comPort.MAV.cs.rateattitude = oldatt;
    MainV2.comPort.MAV.cs.rateposition = oldpos;
    MainV2.comPort.MAV.cs.ratestatus = oldstatus;

    // 请求飞控恢复原始数据流
    try
    {
        MainV2.comPort.requestDatastream(MAVLink.MAV_DATA_STREAM.RC_CHANNELS, oldrc);
    }
    catch { /* 忽略失败 */ }

    // 显示校准结果摘要
    CustomMessageBox.Show(
        "Here are the detected radio options\nNOTE Channels not connected are displayed as 1500 +-2\nNormal values are around 1100 | 1900\nChannel:Min | Max \n" +
        data, "Radio");

    BUT_Calibrateradio.Text = Strings.Completed; // 标记校准完成
}
```
</details>

# 通道功能  
<font color="red">MissionPlanner\GCSViews\ConfigurationView\ConfigUserDefined.cs</font>
## 一、参数

<details>
<summary>点击展开查看通道功能表格</summary>

| 编号 | 功能名称                     |
|------|------------------------------|
| 0    | Do Nothing                  |
| 2    | FLIP Mode                   |
| 3    | Simple Mode                 |
| 4    | RTL                         |
| 5    | Save Trim                   |
| 7    | Save WP                     |
| 9    | Camera Trigger              |
| 10   | RangeFinder Enable          |
| 11   | Fence Enable                |
| 13   | Super Simple Mode           |
| 14   | Acro Trainer                |
| 15   | Sprayer Enable              |
| 16   | AUTO Mode                   |
| 17   | AUTOTUNE Mode               |
| 18   | LAND Mode                   |
| 19   | Gripper Release             |
| 21   | Parachute Enable            |
| 22   | Parachute Release           |
| 23   | Parachute 3pos              |
| 24   | Auto Mission Reset          |
| 25   | AttCon Feed Forward         |
| 26   | AttCon Accel Limits         |
| 27   | Retract Mount1              |
| 28   | Relay1 On/Off               |
| 29   | Landing Gear                |
| 30   | Lost Copter Sound           |
| 31   | Motor Emergency Stop        |
| 32   | Motor Interlock             |
| 33   | BRAKE Mode                  |
| 34   | Relay2 On/Off               |
| 35   | Relay3 On/Off               |
| 36   | Relay4 On/Off               |
| 37   | THROW Mode                  |
| 38   | ADSB Avoidance Enable       |
| 39   | PrecLoiter Enable           |
| 40   | Proximity Avoidance Enable  |
| 41   | ArmDisarm (4.1 and lower)   |
| 42   | SMARTRTL Mode               |
| 43   | InvertedFlight Enable       |
| 44   | Winch Enable                |
| 45   | Winch Control               |
| 46   | RC Override Enable          |
| 47   | User Function 1             |
| 48   | User Function 2             |
| 49   | User Function 3             |
| 52   | ACRO Mode                   |
| 55   | GUIDED Mode                 |
| 56   | LOITER Mode                 |
| 57   | FOLLOW Mode                 |
| 58   | Clear Waypoints             |
| 60   | ZigZag Mode                 |
| 61   | ZigZag SaveWP               |
| 62   | Compass Learn               |
| 65   | GPS Disable                 |
| 66   | Relay5 On/Off               |
| 67   | Relay6 On/Off               |
| 68   | STABILIZE Mode              |
| 69   | POSHOLD Mode                |
| 70   | ALTHOLD Mode                |
| 71   | FLOWHOLD Mode               |
| 72   | CIRCLE Mode                 |
| 73   | DRIFT Mode                  |
| 75   | SurfaceTrackingUpDown       |
| 76   | STANDBY Mode                |
| 78   | RunCam Control              |
| 79   | RunCam OSD Control          |
| 80   | VisOdom Align               |
| 81   | Disarm                      |
| 83   | ZigZag Auto                 |
| 84   | AirMode                     |
| 85   | Generator                   |
| 90   | EKF Source Set              |
| 94   | VTX Power                   |
| 99   | AUTO RTL                    |
| 100  | KillIMU1                    |
| 101  | KillIMU2                    |
| 102  | Camera Mode Toggle          |
| 103  | EKF lane switch attempt     |
| 104  | EKF yaw reset               |
| 105  | GPS Disable Yaw             |
| 109  | use Custom Controller       |
| 110  | KillIMU3                    |
| 112  | SwitchExternalAHRS          |
| 113  | Retract Mount2              |
| 151  | TURTLE Mode                 |
| 152  | SIMPLE heading reset        |
| 153  | ArmDisarm (4.2 and higher)  |
| 154  | ArmDisarm with AirMode (4.2 and higher) |
| 158  | Optflow Calibration         |
| 159  | Force IS_Flying             |
| 161  | Turbine Start(heli)         |
| 162  | FFT Tune                    |
| 163  | Mount Lock                  |
| 164  | Pause Stream Logging        |
| 165  | Arm/Emergency Motor Stop    |
| 166  | Camera Record Video         |
| 167  | Camera Zoom                 |
| 168  | Camera Manual Focus         |
| 169  | Camera Auto Focus           |
| 171  | Calibrate Compasses         |
| 172  | Battery MPPT Enable         |
| 174  | Camera Image Tracking       |
| 175  | Camera Lens                 |
| 177  | Mount LRF enable            |
| 178  | FlightMode Pause/Resume     |
| 180  | Test autotuned gains after tune is complete |
| 182  | AHRS AutoTrim               |
| 212  | Mount1 Roll                 |
| 213  | Mount1 Pitch                |
| 214  | Mount1 Yaw                  |
| 215  | Mount2 Roll                 |
| 216  | Mount2 Pitch                |
| 217  | Mount2 Yaw                  |
| 219  | Transmitter Tuning          |
| 300  | Scripting1                  |
| 301  | Scripting2                  |
| 302  | Scripting3                  |
| 303  | Scripting4                  |
| 304  | Scripting5                  |
| 305  | Scripting6                  |
| 306  | Scripting7                  |
| 307  | Scripting8                  |
| 308  | Scripting9                  |
| 309  | Scripting10                 |
| 310  | Scripting11                 |
| 311  | Scripting12                 |
| 312  | Scripting13                 |
| 313  | Scripting14                 |
| 314  | Scripting15                 |
| 315  | Scripting16                 |

</details>
二、设置参数
<font color="red">MissionPlanner\Controls\MavlinkComboBox.cs</font>
```C#
 if (!MainV2.comPort.setParam((byte)MainV2.comPort.sysidcurrent, (byte)MainV2.comPort.compidcurrent, ParamName, (float)(Int32)Enum.Parse(_source, this.Text)))
 {
     CustomMessageBox.Show(String.Format(Strings.ErrorSetValueFailed, ParamName), Strings.ERROR);
 }
```  

| 字段名 | 说明                     |  
|------|------------------------------|  
|sysidcurrent|默认值|
|compidcurrent|默认值|
|ParamName|修改字段，这里分别为<code>RC6_OPTION</code>-<code>RC16_OPTION</code>|  
|sysidcurrent|修改下拉值|  

三、获取指定参数
发送对应的参数获取如 <code>RC6_OPTION</code> ，发送 <code>RC6_OPTION</code> 获取。  

# 舵机输出
<font color="red">MissionPlanner\GCSViews\ConfigurationView\ConfigRadioOutput.cs</font>  
```C#
 private void setup(int servono)
 {
     var servo = String.Format("SERVO{0}", servono);

     var label = new Label()
         {Text = servono.ToString(), AutoSize = true, TextAlign = System.Drawing.ContentAlignment.MiddleCenter};
     var bAR1 = new HorizontalProgressBar2()
     {
         Minimum = 800, Maximum = 2200, Value = 1500, DrawLabel = true, Name = "BAR" + servono,
         Dock = DockStyle.Fill
     };
     var rev1 = new MissionPlanner.Controls.MavlinkCheckBox()
         {Enabled = false, Dock = DockStyle.Fill, AutoSize = true};
     var func1 = new MavlinkComboBox()
     { Enabled = false, Dock = DockStyle.Fill, DropDownStyle = ComboBoxStyle.DropDownList, Width = 160 };
     var min1 = new MavlinkNumericUpDown() { Minimum = 800, Maximum = 2200, Value = 1500, Enabled = false, Width = 50 };
     var trim1 = new MavlinkNumericUpDown() { Minimum = 800, Maximum = 2200, Value = 1500, Enabled = false, Width = 50 };
     var max1 = new MavlinkNumericUpDown() { Minimum = 800, Maximum = 2200, Value = 1500, Enabled = false, Width = 50 };

     this.tableLayoutPanel1.Controls.Add(label, 0, servono);
     this.tableLayoutPanel1.Controls.Add(bAR1, 1, servono);
     this.tableLayoutPanel1.Controls.Add(rev1, 2, servono);
     this.tableLayoutPanel1.Controls.Add(func1, 3, servono);
     this.tableLayoutPanel1.Controls.Add(min1, 4, servono);
     this.tableLayoutPanel1.Controls.Add(trim1, 5, servono);
     this.tableLayoutPanel1.Controls.Add(max1, 6, servono);

     bAR1.DataBindings.Add("Value", bindingSource1, "ch" + servono + "out");
     rev1.setup(1, 0, servo + "_REVERSED", MainV2.comPort.MAV.param);
     func1.setup(ParameterMetaDataRepository.GetParameterOptionsInt(servo + "_FUNCTION",
             MainV2.comPort.MAV.cs.firmware.ToString()), servo + "_FUNCTION", MainV2.comPort.MAV.param);
     min1.setup(800, 2200, 1, 1, servo + "_MIN", MainV2.comPort.MAV.param);
     trim1.setup(800, 2200, 1, 1, servo + "_TRIM", MainV2.comPort.MAV.param);
     max1.setup(800, 2200, 1, 1, servo + "_MAX", MainV2.comPort.MAV.param);
 }
```
## 一、参数值

| 编号 | 参数名                    |
|------|------------------------------|
| position | ch<code>X</code>out|
| reverse | SERVO<code>X</code>_REVERSED|
| function | SERVO<code>X</code>_FUNCTION|
| mix | SERVO<code>X</code>_MIN|
| trim | SERVO<code>X</code>_TRIM|
| max | SERVO<code>X</code>_MAX|  

> <code>X</code> 表示对应的编号，如<code>ch1out</code>。   

> <code>mix</code>、<code>trim</code>、<code>max</code>的最大值为<code>2200</code>,最小值为<code>800</code>。


### 参数设置
>参数设置与常规的参数设置一之。<code>ParamName</code> 格式为[一、参数值](#一、参数值)中的的格式。
```C#
 try
 {
     bool ans = MainV2.comPort.setParam((byte)MainV2.comPort.sysidcurrent, (byte)MainV2.comPort.compidcurrent, ParamName, OnValue);
     if (ans == false)
         CustomMessageBox.Show(String.Format(Strings.ErrorSetValueFailed, ParamName), Strings.ERROR);
     else
         CallBackOnChange?.Invoke();
 }
 catch
 {
     CustomMessageBox.Show(String.Format(Strings.ErrorSetValueFailed, ParamName), Strings.ERROR);
 }
```
