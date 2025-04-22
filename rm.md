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
