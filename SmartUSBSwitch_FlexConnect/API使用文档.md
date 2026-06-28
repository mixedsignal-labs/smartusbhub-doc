# SmartUSBSwitch FlexConnect API 使用文档

**版本**: v1.0  
**日期**: 2026-01-06  
**适用产品**: SmartUSBSwitch FlexConnect

---

## 目录

- [1. 快速开始](#1-快速开始)
- [2. Python SDK](#2-python-sdk)
- [3. FlexConnect 专用 API](#3-flexconnect-专用-api)
- [4. 公共 API](#4-公共-api)
- [5. 协议详解](#5-协议详解)
- [6. 错误处理](#6-错误处理)
- [7. 最佳实践](#7-最佳实践)
- [8. 常见问题](#8-常见问题)
- [9. 附录](#9-附录)
  - [9.1 示例代码](#91-示例代码)
  - [9.2 调试技巧](#92-调试技巧)
  - [9.3 相关文档](#93-相关文档)
- [10. 修订历史](#10-修订历史)

---

## 1. 快速开始

### 1.1 安装 Python SDK

#### 设置虚拟环境（推荐）

在开发目录中，创建 Python 虚拟环境：

```bash
python -m venv venv
```

激活虚拟环境：

- **Windows 平台**:
  ```bash
  .\venv\Scripts\activate.bat
  ```

- **Linux/macOS 平台**:
  ```bash
  source ./venv/bin/activate
  ```

#### 从源码安装

```bash
# 克隆 develop 分支
git clone -b develop https://github.com/mixedsignal-labs/smartusbhub.git
cd smartusbhub

# 安装依赖
pip install -r requirements.txt

# 安装 SDK
pip install -e .
```

#### 通过 pip 安装

```bash
pip install smartusbhub
```

#### 依赖项

```
pyserial >= 3.5
```

> [!NOTE]
>
> 目前为合并入pip包，对于flexconnect产品客户，请使用源码安装方法。



### 1.2 第一个程序

```python
from smartusbhub import SmartUSBHub, FLEXCONNECT_MODE_PC, FLEXCONNECT_MODE_UDISK1, FLEXCONNECT_MODE_UDISK2
import time

# 自动扫描并连接设备
hub = SmartUSBHub.scan_and_connect()

if hub is None:
    print("未找到设备")
    exit(1)

# 获取设备信息
device_info = hub.get_device_info()
print(f"产品名称: {device_info['product_name']}")
print(f"固件版本: {device_info['firmware_version']}")
print(f"序列号: {device_info['serial_no']}")

# 获取当前模式
current_mode = hub.get_flexconnect_mode()
mode_names = {
    FLEXCONNECT_MODE_PC: "PC 模式（ADB 调试）",
    FLEXCONNECT_MODE_UDISK1: "U 盘1 模式",
    FLEXCONNECT_MODE_UDISK2: "U 盘2 模式"
}
print(f"当前模式: {mode_names.get(current_mode, '未知')}")

# 切换到 PC 模式
result = hub.set_flexconnect_mode(FLEXCONNECT_MODE_PC)
if result:
    print("切换到 PC 模式成功")
    time.sleep(0.5)  # 等待设备稳定
    
    # 验证模式
    current_mode = hub.get_flexconnect_mode()
    if current_mode == FLEXCONNECT_MODE_PC:
        print("验证成功: 当前为 PC 模式")
else:
    print("切换失败")

# 断开连接
hub.disconnect()
```

### 1.3 连接设备

#### 方式1: 自动扫描连接（推荐）

```python
hub = SmartUSBHub.scan_and_connect()
```

自动扫描所有可用的串口，找到第一个 FlexConnect 设备并连接。

#### 方式2: 指定端口连接

```python
# Windows
hub = SmartUSBHub("COM3")

# Linux
hub = SmartUSBHub("/dev/ttyACM0")

# macOS
hub = SmartUSBHub("/dev/cu.usbmodem132301")
```

#### 方式3: 扫描所有设备

```python
devices = SmartUSBHub.scan_devices()
for device in devices:
    print(f"找到设备: {device['port']} - {device['product_name']}")

# 连接第一个设备
if devices:
    hub = SmartUSBHub(devices[0]['port'])
```

---

## 2. Python SDK

### SmartUSBHub 类

#### `SmartUSBHub(port=None, baudrate=115200, timeout=1.0)`

- **描述**: 创建 SmartUSBHub 实例。

- **参数**:
  - `port` (str, 可选): 串口设备路径，Windows 为 "COM3"，Linux/macOS 为 "/dev/ttyACM0"。如果为 `None`，则自动扫描。
  - `baudrate` (int, 可选): 波特率，CDC 接口无需配置，保持默认即可。
  - `timeout` (float, 可选): 命令响应超时时间（秒），默认为 1.0。

- **示例**:
  ```python
  # 指定端口连接
  hub = SmartUSBHub(port="/dev/ttyACM0", timeout=1.0)
  
  # 自动扫描连接
  hub = SmartUSBHub()
  ```

#### `scan_devices()`

- **描述**: 扫描所有可用的 SmartUSBHub 设备。

- **返回值**:
  - `list`: 设备列表，每个设备包含：
    ```python
    {
        'port': '/dev/ttyACM0',
        'product_name': 'SmartUSBSwitch FlexConnect',
        'product_type': 0x14,
        'hardware_version': 'v1.0',
        'firmware_version': 'v1.0.0'
    }
    ```

- **示例**:
  ```python
  devices = SmartUSBHub.scan_devices()
  for device in devices:
      print(f"找到设备: {device['port']} - {device['product_name']}")
  ```

#### `scan_and_connect()`

- **描述**: 扫描可用的 SmartUSBHub 设备，并连接到第一个有效设备。

- **返回值**:
  - SmartUSBHub 实例（如果找到设备），否则返回 `None`。

- **示例**:
  ```python
  hub = SmartUSBHub.scan_and_connect()
  if hub is None:
      print("未找到设备")
  ```

#### `disconnect()`

- **描述**: 断开当前的 SmartUSBHub 设备。

- **示例**:
  ```python
  hub.disconnect()
  ```

### 常量定义

#### FlexConnect 模式常量

```python
FLEXCONNECT_MODE_PC = 0x00         # PC 模式（ADB 调试）
FLEXCONNECT_MODE_UDISK1 = 0x01     # U 盘1 模式
FLEXCONNECT_MODE_UDISK2 = 0x02     # U 盘2 模式
FLEXCONNECT_MODE_DISCONNECT = 0x03 # 断开所有连接模式
```

#### 产品类型常量

```python
PRODUCT_TYPE_FLEXCONNECT = 0x14    # FlexConnect 产品类型
```

---

## 3. FlexConnect 专用 API

### 设置 FlexConnect 工作模式

#### `set_flexconnect_mode(mode)`

- **描述**: 设置 FlexConnect 工作模式。

- **参数**:
  - `mode` (int): 目标模式
    - `FLEXCONNECT_MODE_PC` (0x00): PC 模式（ADB 调试）
    - `FLEXCONNECT_MODE_UDISK1` (0x01): U 盘1 模式
    - `FLEXCONNECT_MODE_UDISK2` (0x02): U 盘2 模式
    - `FLEXCONNECT_MODE_DISCONNECT` (0x03): 断开所有连接

- **返回值**:
  - `bool`: 如果命令设置成功返回 `True`，否则返回 `False`。

- **异常**:
  - `ValueError`: 设备不是 FlexConnect 产品或模式值无效

- **示例**:
  ```python
  from smartusbhub import SmartUSBHub, FLEXCONNECT_MODE_PC, FLEXCONNECT_MODE_UDISK1, FLEXCONNECT_MODE_UDISK2
  import time
  
  # 切换到 PC 模式
  result = hub.set_flexconnect_mode(FLEXCONNECT_MODE_PC)
  if result:
      print("切换到 PC 模式成功")
      time.sleep(0.5)  # 等待设备稳定
      
      # 验证模式
      current_mode = hub.get_flexconnect_mode()
      if current_mode == FLEXCONNECT_MODE_PC:
          print("验证成功: 当前为 PC 模式")
  else:
      print("切换失败")
  
  # 切换到 U 盘1 模式
  result = hub.set_flexconnect_mode(FLEXCONNECT_MODE_UDISK1)
  if result:
      print("切换到 U 盘1 模式成功")
      time.sleep(0.5)
  ```

> [!NOTE]
>
> - 切换模式后，建议等待 0.3~1 秒，让设备和主机完成枚举
> - 如果启用了掉电恢复，新模式会自动保存到 Flash

---

### 获取 FlexConnect 工作模式

#### `get_flexconnect_mode()`

- **描述**: 获取当前 FlexConnect 工作模式。

- **返回值**:
  - `int` 或 `None`: 当前模式（0x00~0x03），若超时则返回 `None`。

- **异常**:
  - `ValueError`: 设备不是 FlexConnect 产品

- **示例**:
  ```python
  from smartusbhub import SmartUSBHub, FLEXCONNECT_MODE_PC, FLEXCONNECT_MODE_UDISK1, FLEXCONNECT_MODE_UDISK2
  
  current_mode = hub.get_flexconnect_mode()
  mode_names = {
      FLEXCONNECT_MODE_PC: "PC 模式（ADB 调试）",
      FLEXCONNECT_MODE_UDISK1: "U 盘1 模式",
      FLEXCONNECT_MODE_UDISK2: "U 盘2 模式"
  }
  
  if current_mode is not None:
      print(f"当前模式: {mode_names.get(current_mode, '未知')}")
  else:
      print("获取模式失败")
  ```

---

### 获取 FlexConnect 故障状态

#### `get_flexconnect_fault()`

- **描述**: 获取 FlexConnect 故障状态。

- **返回值**:
  - `int` 或 `None`: 故障状态字节（0 = 无故障），若超时则返回 `None`。
    - Bit 0: DUT_VBUS_FAULT (车机端 VBUS 故障)
    - Bit 1: UDISK1_VBUS_FAULT (U盘1 VBUS 故障)
    - Bit 2: UDISK2_VBUS_FAULT (U盘2 VBUS 故障)

- **异常**:
  - `ValueError`: 设备不支持故障检测

- **示例**:
  ```python
  fault = hub.get_flexconnect_fault()
  
  if fault is not None:
      if fault == 0:
          print("无故障")
      else:
          if fault & 0x01:
              print("警告: 车机端 VBUS 故障")
          if fault & 0x02:
              print("警告: U盘1 VBUS 故障")
          if fault & 0x04:
              print("警告: U盘2 VBUS 故障")
  else:
      print("获取故障状态失败")
  ```

---

### 设置 FlexConnect 上电默认模式

#### `set_flexconnect_default_mode(mode)`

- **描述**: 设置 FlexConnect 上电默认模式。

- **参数**:
  - `mode` (int): 上电默认模式
    - `FLEXCONNECT_MODE_PC` (0x00): PC 模式
    - `FLEXCONNECT_MODE_UDISK1` (0x01): U 盘1 模式
    - `FLEXCONNECT_MODE_UDISK2` (0x02): U 盘2 模式

- **返回值**:
  - `bool`: 如果命令设置成功返回 `True`，否则返回 `False`。

- **异常**:
  - `ValueError`: 设备不是 FlexConnect 产品或模式值无效

> [!NOTE]
>
> `FLEXCONNECT_MODE_DISCONNECT` 不能作为默认模式

- **示例**:
  ```python
  from smartusbhub import SmartUSBHub, FLEXCONNECT_MODE_PC, FLEXCONNECT_MODE_UDISK1
  
  # 设置上电默认为 PC 模式
  result = hub.set_flexconnect_default_mode(FLEXCONNECT_MODE_PC)
  if result:
      print("设置默认模式成功")
  else:
      print("设置失败")
      
  # 设置上电默认为 U 盘1 模式
  result = hub.set_flexconnect_default_mode(FLEXCONNECT_MODE_UDISK1)
  if result:
      print("设置默认模式成功")
  ```

---

### 获取 FlexConnect 上电默认模式

#### `get_flexconnect_default_mode()`

- **描述**: 获取 FlexConnect 上电默认模式。

- **返回值**:
  - `int` 或 `None`: 上电默认模式（0x00~0x02），若超时则返回 `None`。

- **异常**:
  - `ValueError`: 设备不是 FlexConnect 产品

- **示例**:
  ```python
  from smartusbhub import SmartUSBHub, FLEXCONNECT_MODE_PC, FLEXCONNECT_MODE_UDISK1, FLEXCONNECT_MODE_UDISK2
  
  default_mode = hub.get_flexconnect_default_mode()
  mode_names = {
      FLEXCONNECT_MODE_PC: "PC 模式",
      FLEXCONNECT_MODE_UDISK1: "U 盘1 模式",
      FLEXCONNECT_MODE_UDISK2: "U 盘2 模式"
  }
  
  if default_mode is not None:
      print(f"上电默认模式: {mode_names.get(default_mode, '未知')}")
  else:
      print("获取默认模式失败")
  ```

---

## 4. 公共 API

### 设备信息

#### `get_device_info()`

- **描述**: 获取设备完整信息。

- **返回值**:
  - `dict`: 包含设备信息的字典。
    ```python
    {
        'product_type': 0x14,                          # 产品类型
        'product_name': 'SmartUSBSwitch FlexConnect',  # 产品名称
        'hardware_version': 'v1.0',                    # 硬件版本
        'firmware_version': 'v1.0.0',                  # 固件版本
        'serial_number': 'XXXXXXXX-XXXXXXXX-XXXXXXXX', # 序列号
        'max_channels': 4                              # 最大通道数
    }
    ```

- **示例**:
  ```python
  device_info = hub.get_device_info()
  print(f"产品名称: {device_info['product_name']}")
  print(f"产品类型ID: 0x{device_info['product_type']:02X}")
  print(f"硬件版本: V1.{device_info['hardware_version']}")
  print(f"固件版本: V1.{device_info['firmware_version']}")
  print(f"序列号: {device_info['serial_no']}")
  ```

#### `get_product_type()`

- **描述**: 查询设备的产品类型。

- **返回值**:
  - `int` 或 `None`: 产品类型，FlexConnect 为 0x14，若无响应则返回 `None`。

- **示例**:
  ```python
  product_type = hub.get_product_type()
  # 返回: 0x14 (FlexConnect)
  ```

#### `get_hw_ver()`

- **描述**: 查询设备的硬件版本。

- **返回值**:
  - `str` 或 `None`: 硬件版本，若无响应则返回 `None`。

- **示例**:
  ```python
  hw_ver = hub.get_hw_ver()
  # 返回: "v1.0"
  ```

#### `get_sw_ver()`

- **描述**: 查询设备的固件版本。

- **返回值**:
  - `str` 或 `None`: 固件版本，若无响应则返回 `None`。

- **示例**:
  ```python
  sw_ver = hub.get_sw_ver()
  # 返回: "v1.0.0"
  ```

#### `get_serial_no()`

- **描述**: 查询设备的序列号。

- **返回值**:
  - `str` 或 `None`: 序列号，若无响应则返回 `None`。

- **示例**:
  ```python
  serial_no = hub.get_serial_no()
  # 返回: "XXXXXXXX-XXXXXXXX-XXXXXXXX"
  ```

---

### 掉电恢复控制

#### `set_auto_restore(enable)`

- **描述**: 启用或禁用掉电恢复功能。

- **参数**:
  - `enable` (bool): `True` 启用，`False` 禁用。

- **返回值**:
  - `bool`: 如果命令设置成功返回 `True`，否则返回 `False`。

- **示例**:
  ```python
  # 启用掉电恢复
  hub.set_auto_restore(True)
  
  # 禁用掉电恢复
  hub.set_auto_restore(False)
  ```

> [!NOTE]
>
> - **启用**: 断电重启后恢复到上次的工作模式
> - **禁用**: 断电重启后使用上电默认模式

#### `get_auto_restore_status()`

- **描述**: 查询是否启用掉电恢复功能。

- **返回值**:
  - `int` 或 `None`: `1` 表示启用，`0` 表示禁用，若无响应则返回 `None`。

- **示例**:
  ```python
  auto_restore = hub.get_auto_restore_status()
  if auto_restore == 1:
      print("掉电恢复: 已启用")
  elif auto_restore == 0:
      print("掉电恢复: 已禁用")
  else:
      print("获取掉电恢复状态失败")
  ```

---

### 按键控制

#### `set_button_control(enable)`

- **描述**: 启用或禁用物理按键。

- **参数**:
  - `enable` (bool): `True` 启用，`False` 禁用。

- **返回值**:
  - `bool`: 如果命令设置成功返回 `True`，否则返回 `False`。

- **示例**:
  ```python
  # 禁用按键（防止误操作）
  hub.set_button_control(False)
  
  # 启用按键
  hub.set_button_control(True)
  ```

#### `get_button_control_status()`

- **描述**: 查询物理按键是否启用。

- **返回值**:
  - `int` 或 `None`: `1` 表示启用，`0` 表示禁用，若无响应则返回 `None`。

- **示例**:
  ```python
  button_status = hub.get_button_control_status()
  if button_status == 1:
      print("按键控制: 已启用")
  elif button_status == 0:
      print("按键控制: 已禁用")
  else:
      print("获取按键控制状态失败")
  ```

---

### 系统控制

#### `factory_reset()`

- **描述**: 恢复出厂设置。

- **返回值**:
  - `bool`: 如果命令设置成功返回 `True`，否则返回 `False`。

- **示例**:
  ```python
  if hub.factory_reset():
      print("恢复出厂设置成功")
      print("设备将在 3 秒后重启...")
      time.sleep(3)
      hub.disconnect()
  ```

> [!NOTE]
>
> 恢复的参数：
> - `flexconnect_default_mode` → `MODE_PC (0)`
> - `auto_restore` → `禁用 (0)`
> - `button_enable` → `启用 (1)`
> - `current_mode` → `MODE_PC`

#### `reboot_mcu()`

- **描述**: 重启 MCU。

- **返回值**:
  - `bool`: 如果命令设置成功返回 `True`，否则返回 `False`。

> [!NOTE]
>
> 重启后需要重新连接设备。

- **示例**:
  ```python
  if hub.reboot_mcu():
      print("设备正在重启...")
      hub.disconnect()
      time.sleep(3)
      
      # 重新连接
      hub = SmartUSBHub.scan_and_connect()
  ```

---

### 设备地址

#### `set_device_address(address)`

- **描述**: 设备地址用于在多台集线器连接的场景中，标识和区分各个集线器。

- **参数**:
  - `address` (int): 地址的编号由用户自定义，其取值范围为: `0x0000 - 0xFFFF`。

- **返回值**:
  - `bool`: 如果命令设置成功返回 `True`，否则返回 `False`。

> [!NOTE]
>
> 创建 SmartUSBHub 实例时会自动获取连接设备的地址，不同 SmartUSBHub 实例可以通过地址区分。

- **示例**:
  ```python
  import time
  
  # 获取原始地址
  original_address = hub.get_device_address()
  print(f"当前设备地址: 0x{original_address:04X}")
  
  # 设置新地址
  new_address = 0x0001
  result = hub.set_device_address(new_address)
  if result:
      print("设置成功")
      time.sleep(0.2)
      
      # 验证地址
      address = hub.get_device_address()
      if address == new_address:
          print(f"验证成功: 当前设备地址为 0x{address:04X}")
      else:
          print(f"验证失败: 期望 0x{new_address:04X}, 实际 0x{address:04X}")
  else:
      print("设置失败")
  ```

#### `get_device_address()`

- **描述**: 获取设备的地址。

- **返回值**:
  - `int` 或 `None`: 设备地址，若无响应则返回 `None`。

- **示例**:
  ```python
  device_address = hub.get_device_address()
  if device_address is not None:
      print(f"设备地址: 0x{device_address:04X}")
  else:
      print("获取设备地址失败")
  ```

---

## 5. 协议详解

### 5.1 协议格式

#### 基本格式

```
发送: [0x55] [0x5A] [CMD] [CH] [VALUE] [CHECKSUM]
响应: [0x55] [0x5A] [CMD] [CH] [VALUE] [CHECKSUM]

CHECKSUM = CMD + CH + VALUE (8位和校验)
```

#### 示例：切换到 PC 模式

```
发送: 55 5A 20 00 00 20
      ↑   ↑  ↑  ↑  ↑  ↑
      头  头 CMD CH VAL CS

解释:
- 0x55 0x5A: 协议头
- 0x20: CMD_SET_FLEXCONNECT_MODE
- 0x00: 通道（FlexConnect 固定为 0）
- 0x00: FLEXCONNECT_MODE_PC
- 0x20: 校验和 (0x20+0x00+0x00=0x20)

响应: 55 5A 20 00 00 20 (ACK)
```

### 5.2 命令码定义

#### FlexConnect 专用命令

| 命令码 | 命令名 | 数据方向 | 说明 |
|--------|--------|---------|------|
| 0x20 | CMD_SET_FLEXCONNECT_MODE | 主机→设备 | 设置工作模式 |
| 0x21 | CMD_GET_FLEXCONNECT_MODE | 主机→设备 | 获取当前模式 |
| 0x22 | CMD_GET_FLEXCONNECT_FAULT | 主机→设备 | 获取故障状态 |
| 0x24 | CMD_SET_FLEXCONNECT_DEFAULT_MODE | 主机→设备 | 设置默认模式 |
| 0x25 | CMD_GET_FLEXCONNECT_DEFAULT_MODE | 主机→设备 | 获取默认模式 |

#### 公共命令

| 命令码 | 命令名 | 数据方向 | 说明 |
|--------|--------|---------|------|
| 0x09 | CMD_SET_BUTTON_CONTROL | 主机→设备 | 设置按键使能 |
| 0x0A | CMD_GET_BUTTON_CONTROL_STATUS | 主机→设备 | 获取按键状态 |
| 0x0F | CMD_SET_AUTO_RESTORE_ENABLE | 主机→设备 | 设置掉电恢复 |
| 0x10 | CMD_GET_AUTO_RESTORE_STATUS | 主机→设备 | 获取掉电恢复状态 |
| 0xF0 | CMD_GET_PRODUCT_TYPE | 主机→设备 | 获取产品类型 |
| 0xF7 | CMD_REBOOT_MCU | 主机→设备 | 重启设备 |
| 0xFC | CMD_FACTORY_RESET | 主机→设备 | 恢复出厂设置 |
| 0xFD | CMD_GET_FIRMWARE_VERSION | 主机→设备 | 获取固件版本 |
| 0xFE | CMD_GET_HARDWARE_VERSION | 主机→设备 | 获取硬件版本 |

### 5.3 协议示例

#### 示例1: 获取当前模式

```
发送: 55 5A 21 00 00 21
响应: 55 5A 21 00 01 22  (当前为 UDISK1 模式)
                    ↑
                  0x01 = MODE_UDISK1
```

#### 示例2: 启用掉电恢复

```
发送: 55 5A 0F 00 01 10
      ↑   ↑  ↑  ↑  ↑  ↑
      头  头 CMD CH VAL CS

CMD = 0x0F (SET_AUTO_RESTORE_ENABLE)
VALUE = 0x01 (启用)

响应: 55 5A 0F 00 01 10 (ACK)
```

#### 示例3: 获取故障状态

```
发送: 55 5A 22 00 00 22
响应: 55 5A 22 00 03 25  (故障状态 = 0x03)
                    ↑
                  Bit0=1: 车机端故障
                  Bit1=1: U盘1故障
```

---

## 6. 错误处理

### 6.1 异常类型

#### ValueError

设备不支持该功能或参数无效时抛出。

**示例**:

```python
try:
    hub.set_flexconnect_mode(FLEXCONNECT_MODE_PC)
except ValueError as e:
    print(f"参数错误: {e}")
```

#### 通信超时

返回 `None` 或 `False`，表示设备无响应。

**示例**:

```python
mode = hub.get_flexconnect_mode()
if mode is None:
    print("通信超时，设备可能未连接或无响应")
```

### 6.2 错误处理模式

#### 方式1: 检查返回值

```python
result = hub.set_flexconnect_mode(FLEXCONNECT_MODE_PC)
if not result:
    print("切换失败")
    # 尝试重新连接或记录错误
```

#### 方式2: 异常捕获

```python
try:
    hub.set_flexconnect_mode(FLEXCONNECT_MODE_UDISK1)
except ValueError as e:
    print(f"参数错误: {e}")
except Exception as e:
    print(f"未知错误: {e}")
```

### 6.3 重试机制

```python
import time

def set_mode_with_retry(hub, mode, max_retries=3):
    for i in range(max_retries):
        try:
            result = hub.set_flexconnect_mode(mode)
            if result:
                return True
            print(f"尝试 {i+1}/{max_retries} 失败，重试...")
            time.sleep(0.5)
        except Exception as e:
            print(f"异常: {e}")
            time.sleep(0.5)
    return False

# 使用
if set_mode_with_retry(hub, FLEXCONNECT_MODE_PC):
    print("切换成功")
else:
    print("切换失败（已重试3次）")
```

---

## 7. 最佳实践

### 7.1 连接与初始化

```python
import time
from smartusbhub import SmartUSBHub, FLEXCONNECT_MODE_PC

def initialize_device():
    """初始化设备并验证连接"""
    # 连接设备
    hub = SmartUSBHub.scan_and_connect()
    if hub is None:
        print("错误: 未找到设备")
        return None
    
    # 获取设备信息
    info = hub.get_device_info()
    print(f"已连接: {info['product_name']}")
    print(f"固件版本: {info['firmware_version']}")
    print(f"序列号: {info['serial_number']}")
    
    # 验证产品类型
    if info['product_type'] != 0x14:
        print("错误: 设备不是 FlexConnect")
        hub.disconnect()
        return None
    
    # 检查当前状态
    mode = hub.get_flexconnect_mode()
    auto_restore = hub.get_auto_restore_status()
    print(f"当前模式: {mode}, 掉电恢复: {auto_restore}")
    
    return hub

# 使用
hub = initialize_device()
if hub:
    # 执行业务逻辑
    pass
```

### 7.2 模式切换与验证

```python
def switch_mode_and_verify(hub, target_mode, mode_name):
    """切换模式并验证"""
    print(f"\n切换到 {mode_name}...")
    
    # 切换模式
    result = hub.set_flexconnect_mode(target_mode)
    if not result:
        print(f"切换失败")
        return False
    
    # 等待设备稳定
    time.sleep(0.3)
    
    # 验证模式
    current_mode = hub.get_flexconnect_mode()
    if current_mode == target_mode:
        print(f"切换成功: {mode_name}")
        return True
    else:
        print(f"验证失败: 期望 {target_mode}, 实际 {current_mode}")
        return False

# 使用
switch_mode_and_verify(hub, FLEXCONNECT_MODE_PC, "PC 模式")
switch_mode_and_verify(hub, FLEXCONNECT_MODE_UDISK1, "U盘1 模式")
```

### 7.3 配置掉电恢复

```python
def configure_power_recovery(hub, enable, default_mode=None):
    """配置掉电恢复功能"""
    print(f"\n配置掉电恢复: {'启用' if enable else '禁用'}")
    
    # 设置掉电恢复
    result = hub.set_auto_restore(enable)
    if not result:
        print("设置掉电恢复失败")
        return False
    
    time.sleep(0.2)
    
    # 验证设置
    status = hub.get_auto_restore_status()
    if status != (1 if enable else 0):
        print(f"验证失败: 期望 {1 if enable else 0}, 实际 {status}")
        return False
    
    print(f"掉电恢复已{'启用' if enable else '禁用'}")
    
    # 如果禁用掉电恢复，设置默认模式
    if not enable and default_mode is not None:
        print(f"设置默认模式为: {default_mode}")
        result = hub.set_flexconnect_default_mode(default_mode)
        if result:
            print("默认模式设置成功")
        else:
            print("默认模式设置失败")
            return False
    
    return True

# 使用
# 启用掉电恢复
configure_power_recovery(hub, True)

# 禁用掉电恢复，使用 PC 模式作为默认
configure_power_recovery(hub, False, FLEXCONNECT_MODE_PC)
```

---

## 8. 常见问题

### 8.1 设备连接问题

#### Q: `scan_and_connect()` 返回 None

#### A: 可能的原因

1. 设备未连接 CMD 接口或未上电
2. 权限不足（Linux 需要 udev 规则）
3. 串口被其他程序占用

#### Q: 连接后设备无响应

#### A: 尝试以下步骤

1. 断开重连
2. 重启设备
3. 检查固件版本
4. 增加超时时间

```python
hub = SmartUSBHub(timeout=2.0)  # 增加超时到 2 秒
```

#### Q: 切换模式后设备未正确识别

#### A: 增加等待时间，让主机完成枚举

```python
hub.set_flexconnect_mode(FLEXCONNECT_MODE_PC)
time.sleep(1.0)  # 等待 1 秒
```

#### Q: ADB 无法识别车机

#### A: 检查以下事项

1. USB 线缆只支持 USB-A to USB-A，不支持 Type-C
2. 确认已切换到 PC 模式
3. 车机 USB 角色是否切换为 Device
4. ADB 驱动是否安装

#### Q: U 盘无法识别

#### A: 检查以下事项

1. 确认已切换到 UDISK 模式
2. 车机 USB 角色是否切换为 Host
3. U 盘是否兼容（建议使用 FAT32 格式）
4. VBUS 供电是否正常

---

## 9. 附录

### 9.1 示例代码

#### Python 库示例

示例代码位于 `smartusbhub_ng/examples/FlexConnect/` 目录：

| 文件 | 说明 |
|------|------|
| `basic_usage.py` | 基础使用入门，演示连接、查询、切换等基本操作 |
| `flexconnect_mode_switch_demo.py` | 交互式模式切换演示，循环切换各种模式 |
| `power_restore_demo.py` | 掉电恢复功能完整演示，包含优先级测试 |
| `automation_suite.py` | 完整的自动化测试套件，含 ADB 和 U 盘测试 |
| `diagnose_params.py` | 参数诊断工具，读取并显示所有相关参数 |

### 9.2 调试技巧

#### 启用详细日志

```python
import logging

# 配置日志
logging.basicConfig(
    level=logging.DEBUG,
    format='%(asctime)s - %(name)s - %(levelname)s - %(message)s'
)

hub = SmartUSBHub.scan_and_connect()
```

#### 使用串口监视工具

- **Windows**: Realterm, Putty
- **Linux/macOS**: minicom, screen

```bash
# Linux/macOS
screen /dev/ttyACM0 115200

# 查看原始数据
minicom -D /dev/ttyACM0
```

---

## 10. 修订历史

| 版本 | 日期 | 修订内容 | 作者 |
|------|------|---------|------|
| v1.0 | 2026-01-06 | 初版发布 | Makerlabtools |

---

**© 2026 makerlabtools. All Rights Reserved.**

