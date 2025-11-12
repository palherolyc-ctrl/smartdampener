# MPU9250 + BLE Peripheral Integration Summary

## 整合完成概述

已成功将MPU9250九轴传感器驱动集成到simpleBlePeripheral项目中，实现了通过I2C读取MPU9250的加速度、陀螺仪、磁力计和温度数据，并通过BLE传输到移动设备的完整功能。

## 完成的工作

### 1. 文件复制和添加
- ✅ 将MPU9250驱动文件复制到 `simpleBlePeripheral/source/` 目录：
  - `driver_mpu9250.h` - MPU9250主驱动头文件
  - `driver_mpu9250.c` - MPU9250主驱动实现
  - `driver_mpu9250_interface.h` - 接口头文件
  - `driver_mpu9250_interface.c` - 接口实现（适配现有I2C驱动）
  - `driver_mpu9250_basic.h` - Basic API头文件
  - `driver_mpu9250_basic.c` - Basic API实现
  - `driver_mpu9250_code.h` - 驱动代码头文件

### 2. 接口适配
- ✅ 修改了 `driver_mpu9250_interface.c`，使其使用现有的I2C软件驱动：
  - I2C初始化和反初始化
  - I2C读写操作（适配现有驱动接口）
  - 延迟函数（使用OSAL延迟）
  - 调试打印功能（使用LOG模块）

### 3. 主应用文件修改 (`simpleBLEPeripheral.c`)
- ✅ 添加了MPU9250相关头文件包含：
  ```c
  #include "driver_mpu9250.h"
  #include "driver_mpu9250_basic.h"
  ```
- ✅ 添加了MPU9250传感器配置常量：
  ```c
  #define MPU9250_SENSOR_SCL_PIN                    GPIO_P7   // P7 as SCL
  #define MPU9250_SENSOR_SDA_PIN                    GPIO_P31  // P31 as SDA
  #define MPU9250_SENSOR_READ_PERIOD                1000      // Read every 1 second
  #define MPU9250_SENSOR_DEVICE_ADDR                0x68      // MPU9250 I2C address
  ```
- ✅ 添加了MPU9250传感器数据结构：
  ```c
  typedef struct {
      float accel_x, accel_y, accel_z;    // Accelerometer data (g)
      float gyro_x, gyro_y, gyro_z;      // Gyroscope data (dps)
      float mag_x, mag_y, mag_z;         // Magnetometer data (uT)
      float temperature;                  // Temperature (Celsius)
  } mpu9250_sensor_data_t;
  ```
- ✅ 添加了MPU9250相关变量：
  - 传感器数据缓冲区
  - 初始化状态标志
- ✅ 添加了MPU9250函数声明：
  - `init_mpu9250_sensor()` - 初始化MPU9250传感器
  - `read_mpu9250_sensor_data()` - 读取传感器数据
  - `send_mpu9250_data_via_ble()` - 通过BLE发送数据
- ✅ 在 `SimpleBLEPeripheral_Init()` 中添加了MPU9250初始化调用
- ✅ 在 `SimpleBLEPeripheral_ProcessEvent()` 中添加了MPU9250事件处理

### 4. 功能实现详情

#### MPU9250初始化 (`init_mpu9250_sensor`)
- 使用MPU9250 Basic API初始化传感器
- 配置I2C接口和设备地址
- 启动周期性传感器读取定时器（1秒间隔）

#### 传感器数据读取 (`read_mpu9250_sensor_data`)
- 使用MPU9250 Basic API读取所有传感器数据：
  - 加速度计数据（X、Y、Z轴，单位：g）
  - 陀螺仪数据（X、Y、Z轴，单位：度/秒）
  - 磁力计数据（X、Y、Z轴，单位：微特斯拉）
  - 温度数据（摄氏度）
- 存储数据到全局结构体
- 读取完成后触发BLE通知事件

#### BLE数据传输 (`send_mpu9250_data_via_ble`)
- 使用特征值6发送传感器数据
- 数据格式：计数器(2字节) + 9轴数据 + 温度数据 = 42字节
- 支持MTU自动适配
- 提供详细的发送状态日志

### 5. 数据格式说明

通过BLE传输的数据格式：
```
字节 0-1:  计数器（2字节）
字节 2-5:  加速度计X轴（4字节float）
字节 6-9:  加速度计Y轴（4字节float）
字节 10-13: 加速度计Z轴（4字节float）
字节 14-17: 陀螺仪X轴（4字节float）
字节 18-21: 陀螺仪Y轴（4字节float）
字节 22-25: 陀螺仪Z轴（4字节float）
字节 26-29: 磁力计X轴（4字节float）
字节 30-33: 磁力计Y轴（4字节float）
字节 34-37: 磁力计Z轴（4字节float）
字节 38-41: 温度（4字节float）
```

## 使用方法

### 1. 硬件连接
```
MPU9250模块    ->  BLE模块
VCC            ->  3.3V
GND            ->  GND
SCL            ->  P7引脚
SDA            ->  P31引脚
```

### 2. 软件配置
- MPU9250地址默认设置为0x68（AD0_LOW）
- 如果您的MPU9250模块AD0引脚连接到VCC，请使用0x69地址
- 读取间隔设置为1000ms（1秒）

### 3. 编译和烧录
- 编译整个项目
- 烧录到BLE模块

### 4. 手机端通信
- 使用nRF Connect或LightBlue等BLE应用
- 连接到设备后订阅特征值6的通知
- 每秒会收到一次包含9轴数据和温度的42字节数据包

## 特性

- ✅ **完整的MPU9250支持**：加速度计、陀螺仪、磁力计、温度传感器
- ✅ **高精度数据**：浮点数格式，保留小数精度
- ✅ **实时数据采集**：1秒间隔自动读取传感器数据
- ✅ **BLE实时传输**：数据读取后立即通过BLE通知发送
- ✅ **错误处理和日志**：完整的调试信息和错误处理
- ✅ **MTU自适应**：自动适配当前BLE连接的最大传输单元
- ✅ **低功耗设计**：使用OSAL定时器，避免CPU占用

## 注意事项

1. **I2C地址**：MPU9250默认地址为0x68，如果AD0引脚接高电平则为0x69
2. **电源要求**：确保MPU9250电源电压为3.3V
3. **校准**：MPU9250可能需要校准以获得准确数据
4. **数据处理**：接收到的数据为32位浮点数，需要按小端序解析
5. **GPIO配置**：P7和P31引脚已配置为上拉模式，符合I2C协议要求

## 测试建议

1. 首先使用逻辑分析仪或示波器验证I2C信号是否正确
2. 检查MPU9250是否有正确的ACK响应
3. 验证BLE连接和通知功能
4. 监控串口日志确认数据流程正常
5. 测试不同方向的运动以验证传感器数据变化

## 数据解析示例（Python）

```python
import struct
import time

def parse_mpu9250_data(data):
    """解析MPU9250 BLE通知数据"""
    if len(data) != 42:
        return None
    
    counter = struct.unpack('<H', data[0:2])[0]
    
    accel_x = struct.unpack('<f', data[2:6])[0]
    accel_y = struct.unpack('<f', data[6:10])[0]
    accel_z = struct.unpack('<f', data[10:14])[0]
    
    gyro_x = struct.unpack('<f', data[14:18])[0]
    gyro_y = struct.unpack('<f', data[18:22])[0]
    gyro_z = struct.unpack('<f', data[22:26])[0]
    
    mag_x = struct.unpack('<f', data[26:30])[0]
    mag_y = struct.unpack('<f', data[30:34])[0]
    mag_z = struct.unpack('<f', data[34:38])[0]
    
    temperature = struct.unpack('<f', data[38:42])[0]
    
    return {
        'counter': counter,
        'accel': {'x': accel_x, 'y': accel_y, 'z': accel_z},
        'gyro': {'x': gyro_x, 'y': gyro_y, 'z': gyro_z},
        'mag': {'x': mag_x, 'y': mag_y, 'z': mag_z},
        'temperature': temperature
    }

# 使用示例
# data = received_ble_notification_data
# sensor_data = parse_mpu9250_data(data)
# print(f"Accel: {sensor_data['accel']}")
# print(f"Gyro: {sensor_data['gyro']}")
# print(f"Mag: {sensor_data['mag']}")
# print(f"Temp: {sensor_data['temperature']}°C")
```

整合工作已完成，系统现在具备了完整的MPU9250九轴传感器数据采集和BLE传输功能。