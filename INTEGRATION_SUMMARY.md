# BLE Peripheral + I2C Sensor Integration Summary

## 整合完成概述

已成功将I2C软件驱动集成到simpleBlePeripheral项目中，实现了通过I2C读取传感器数据并通过BLE传输的完整功能。

## 完成的工作

### 1. 文件复制和添加
- ✅ 将 `i2c_soft_driver.h` 和 `i2c_soft_driver.c` 复制到 `simpleBlePeripheral/source/` 目录
- ✅ 保持了原始I2C驱动的完整功能，包括：
  - 软件I2C协议实现（start, stop, read, write）
  - ACK/NAK处理
  - 支持8位和16位寄存器地址模式
  - 完整的GPIO时序控制

### 2. 头文件修改 (`simpleBLEPeripheral.h`)
- ✅ 添加了I2C相关事件定义：
  ```c
  #define SBP_I2C_READ_SENSOR_EVT        0x1000  // Read sensor data via I2C
  #define SBP_I2C_SENSOR_NOTIFY_EVT      0x2000  // Send sensor data via BLE notification
  ```

### 3. OSAL任务管理修改 (`OSAL_SimpleBLEPeripheral.c`)
- ✅ 添加了I2C驱动头文件包含
- ✅ 在任务数组中添加了 `HalPeripheral_ProcessEvent` 任务
- ✅ 在任务初始化函数中添加了 `HalPeripheral_Init` 调用
- ✅ 确保了任务顺序的一致性

### 4. 主应用文件修改 (`simpleBLEPeripheral.c`)
- ✅ 添加了I2C驱动头文件包含
- ✅ 添加了I2C传感器配置常量：
  ```c
  #define I2C_SENSOR_SCL_PIN              GPIO_P7   // P7 as SCL
  #define I2C_SENSOR_SDA_PIN              GPIO_P31  // P31 as SDA
  #define I2C_SENSOR_READ_PERIOD          1000      // Read every 1 second
  #define I2C_SENSOR_DEVICE_ADDR          0x50      // Example sensor address
  #define I2C_SENSOR_REG_ADDR             0x00      // Example register address
  #define I2C_SENSOR_DATA_LEN             8         // Read 8 bytes
  ```
- ✅ 添加了I2C传感器相关变量：
  - 传感器数据缓冲区
  - I2C驱动结构体
  - 初始化状态标志
- ✅ 添加了I2C传感器函数声明：
  - `init_i2c_sensor()` - 初始化I2C传感器
  - `read_i2c_sensor_data()` - 读取传感器数据
  - `send_sensor_data_via_ble()` - 通过BLE发送数据
- ✅ 在 `SimpleBLEPeripheral_Init()` 中添加了I2C初始化调用
- ✅ 在 `SimpleBLEPeripheral_ProcessEvent()` 中添加了I2C事件处理

### 5. 功能实现详情

#### I2C初始化 (`init_i2c_sensor`)
- 配置P7作为SCL，P31作为SDA
- 初始化I2C软件驱动
- 启动周期性传感器读取定时器（1秒间隔）

#### 传感器数据读取 (`read_i2c_sensor_data`)
- 通过I2C从指定设备地址读取数据
- 支持8位寄存器地址模式
- 读取指定长度的数据字节
- 读取完成后触发BLE通知事件

#### BLE数据传输 (`send_sensor_data_via_ble`)
- 使用特征值6发送传感器数据
- 添加计数器确保数据唯一性
- 支持MTU自动适配
- 提供详细的发送状态日志

### 6. 配置参数说明

用户可以根据实际传感器修改以下参数：

```c
#define I2C_SENSOR_SCL_PIN        GPIO_P7   // SCL引脚
#define I2C_SENSOR_SDA_PIN        GPIO_P31  // SDA引脚
#define I2C_SENSOR_DEVICE_ADDR    0x50      // 传感器I2C地址
#define I2C_SENSOR_REG_ADDR       0x00      // 传感器寄存器地址
#define I2C_SENSOR_DATA_LEN       8         // 读取数据长度
#define I2C_SENSOR_READ_PERIOD    1000      // 读取间隔(毫秒)
```

## 使用方法

1. **硬件连接**：
   - 将传感器SCL连接到P7引脚
   - 将传感器SDA连接到P31引脚
   - 确保传感器电源和地线正确连接

2. **软件配置**：
   - 根据实际传感器修改 `I2C_SENSOR_DEVICE_ADDR` 和 `I2C_SENSOR_REG_ADDR`
   - 调整 `I2C_SENSOR_DATA_LEN` 以匹配传感器数据长度

3. **编译和烧录**：
   - 编译整个项目
   - 烧录到BLE模块

4. **手机端通信**：
   - 使用nRF Connect或LightBlue等BLE应用
   - 连接到设备后订阅特征值6的通知
   - 每秒会收到一次传感器数据

## 特性

- ✅ **完整的I2C协议支持**：start/stop条件、ACK/NAK处理、字节读写
- ✅ **GPIO引脚灵活配置**：可轻松修改SCL/SDA引脚
- ✅ **周期性数据采集**：1秒间隔自动读取传感器数据
- ✅ **BLE实时传输**：数据读取后立即通过BLE通知发送
- ✅ **错误处理和日志**：完整的调试信息和错误处理
- ✅ **MTU自适应**：自动适配当前BLE连接的最大传输单元
- ✅ **低功耗设计**：使用OSAL定时器，避免CPU占用

## 注意事项

1. **I2C地址**：确保传感器的I2C地址与配置一致
2. **寄存器地址**：根据传感器数据手册设置正确的寄存器地址
3. **数据长度**：设置合适的数据读取长度，避免超出传感器支持范围
4. **GPIO配置**：P7和P31引脚在某些开发板上可能已被占用，需要检查硬件设计
5. **电源要求**：确保传感器电源电压与BLE模块兼容

## 测试建议

1. 首先使用逻辑分析仪或示波器验证I2C信号是否正确
2. 检查传感器是否有ACK响应
3. 验证BLE连接和通知功能
4. 监控串口日志确认数据流程正常

整合工作已完成，系统现在具备了完整的I2C传感器数据采集和BLE传输功能。