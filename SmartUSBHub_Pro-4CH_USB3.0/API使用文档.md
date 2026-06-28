# SmartUSBHub Pro-4CH USB3.0 API 使用文档

**版本**: v1.0  
**日期**: 2026-01-10  
**适用产品**: SmartUSBHub Pro-4CH USB3.0

---

## 目录

- [1. 快速开始](#1-快速开始)
- [2. Python SDK](#2-python-sdk)
- [3. 通道控制 API](#3-通道控制-api)
  - [3.1 电源控制](#31-电源控制)
  - [3.2 USB2.0 数据线控制](#32-usb20-数据线控制)
  - [3.3 USB3.0 数据线控制](#33-usb30-数据线控制)
  - [3.4 慢充/快充模式](#34-慢充快充模式)
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
> 目前develop分支未正式发布，未合并入pip包，请使用源码安装方法。

### 1.2 第一个程序

```python
from smartusbhub import SmartUSBHub
import time

# 自动扫描并连接设备
hub = SmartUSBHub.scan_and_connect()

if hub is None:
    print("未找到设备")
    exit(1)

# 获取设备信息
device_info = hub.get_device_info()
print(f"产品名称: {device_info['product_type']}")
print(f"固件版本: {device_info['firmware_version']}")
print(f"序列号: {device_info['serial_no']}")
print(f"最大通道数: {device_info['max_channels']}")

# 打开通道1的电源
result = hub.set_channel_power(1, state=1)
if result:
    print("通道1电源已打开")
    time.sleep(0.5)
    
    # 查询通道1电源状态
    status = hub.get_channel_power_status(1)
    if status == 1:
        print("验证成功: 通道1电源已打开")
else:
    print("打开通道1电源失败")

# 断开连接
hub.disconnect()
```

### 1.3 连接设备

#### 方式1: 自动扫描连接（推荐）

```python
hub = SmartUSBHub.scan_and_connect()
```

自动扫描所有可用的串口，找到第一个 SmartUSBHub 设备并连接。

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
ports = SmartUSBHub.scan_available_ports()
for port in ports:
    print(f"找到设备: {port}")

# 连接第一个设备
if ports:
    hub = SmartUSBHub(ports[0])
```

---

## 2. Python SDK

### SmartUSBHub 类

#### `SmartUSBHub(port)`

- **描述**: 创建 SmartUSBHub 实例。

- **参数**:
  - `port` (str): 串口设备路径，Windows 为 "COM3"，Linux/macOS 为 "/dev/ttyACM0"。

- **示例**:
  ```python
  # 指定端口连接
  hub = SmartUSBHub("/dev/ttyACM0")
  
  # 自动扫描连接（需要先调用 scan_and_connect）
  hub = SmartUSBHub.scan_and_connect()
  ```

#### `scan_available_ports()`

- **描述**: 扫描所有可用的 SmartUSBHub 设备端口。

- **返回值**:
  - `list`: 端口列表，例如 `['/dev/ttyACM0', '/dev/ttyACM1']`。

- **示例**:
  ```python
  ports = SmartUSBHub.scan_available_ports()
  for port in ports:
      print(f"找到设备: {port}")
  ```

#### `scan_and_connect(exclude_ports=None, device_address=None)`

- **描述**: 扫描可用的 SmartUSBHub 设备，并连接到第一个有效设备。

- **参数**:
  - `exclude_ports` (set, 可选): 要排除的端口集合（已连接的端口）。如果为 `None`，自动排除已连接的端口。
  - `device_address` (int, 可选): 要连接的设备地址。如果指定，会尝试连接所有未连接的设备，直到找到匹配地址的设备。
    **注意**: 由于设备地址默认为0，多个设备可能地址相同，建议使用端口号来区分设备。

- **返回值**:
  - SmartUSBHub 实例（如果找到设备），否则返回 `None`。

- **示例**:
  ```python
  # 自动扫描连接
  hub = SmartUSBHub.scan_and_connect()
  if hub is None:
      print("未找到设备")
  
  # 连接指定地址的设备
  hub = SmartUSBHub.scan_and_connect(device_address=0x0001)
  ```

#### `disconnect()`

- **描述**: 断开当前的 SmartUSBHub 设备。

- **示例**:
  ```python
  hub.disconnect()
  ```

## 3. 通道控制 API

### 3.1 电源控制

#### `set_channel_power(*channels, state)`

- **描述**: 设置一个或多个通道的电源状态。

- **参数**:
  - `*channels` (int): 通道编号（1-4），可以指定多个通道。
  - `state` (int): `1` 打开电源，`0` 关闭电源。

- **返回值**:
  - `bool`: 如果命令设置成功返回 `True`，否则返回 `False`。

- **示例**:
  ```python
  # 打开通道1的电源
  hub.set_channel_power(1, state=1)
  
  # 同时打开通道1、2、3、4的电源
  hub.set_channel_power(1, 2, 3, 4, state=1)
  
  # 关闭通道1的电源
  hub.set_channel_power(1, state=0)
  ```

#### `get_channel_power_status(*channels)`

- **描述**: 查询一个或多个通道的电源状态。

- **参数**:
  - `*channels` (int): 通道编号（1-4），可以指定多个通道。

- **返回值**:
  - `dict` 或 `int` 或 `None`: 
    - 查询多个通道时，返回字典 `{1: 1, 2: 0, ...}`，键为通道号，值为状态（1=开，0=关）。
    - 查询单个通道时，返回 `int`（1=开，0=关）。
    - 若超时则返回 `None`。

- **示例**:
  ```python
  # 查询单个通道
  status = hub.get_channel_power_status(1)
  if status == 1:
      print("通道1电源已打开")
  
  # 查询多个通道
  status_dict = hub.get_channel_power_status(1, 2, 3, 4)
  for ch, state in status_dict.items():
      print(f"通道{ch}: {'开' if state == 1 else '关'}")
  ```

---

### 3.2 USB2.0 数据线控制

#### `set_channel_usb2_dataline(*channels, state)`

- **描述**: 设置一个或多个通道的 USB2.0 数据线连接状态。

- **参数**:
  - `*channels` (int): 通道编号（1-4），可以指定多个通道。
  - `state` (int): `1` 连接数据线，`0` 断开数据线。

- **返回值**:
  - `bool`: 如果命令设置成功返回 `True`，否则返回 `False`。

- **示例**:
  ```python
  # 确保通道1电源已打开
  if hub.get_channel_power_status(1) == 0:
      hub.set_channel_power(1, state=1)
      time.sleep(0.1)
  
  # 断开通道1的USB2.0数据线（保持电源打开）
  hub.set_channel_usb2_dataline(1, state=0)
  
  # 重新连接通道1的USB2.0数据线
  hub.set_channel_usb2_dataline(1, state=1)
  
  # 同时控制多个通道
  hub.set_channel_usb2_dataline(1, 2, 3, state=0)
  ```

> [!NOTE]
>
> - 断开数据线时，设备会断开USB连接，但电源保持打开。
> - 重新连接数据线时，设备会重新枚举。

#### `get_channel_usb2_dataline_status(*channels)`

- **描述**: 查询一个或多个通道的 USB2.0 数据线连接状态。

- **参数**:
  - `*channels` (int): 通道编号（1-4），可以指定多个通道。

- **返回值**:
  - `dict` 或 `None`: 字典 `{1: 1, 2: 0, ...}`，键为通道号，值为状态（1=连接，0=断开），若超时则返回 `None`。

- **示例**:
  ```python
  status_dict = hub.get_channel_usb2_dataline_status(1, 2, 3, 4)
  for ch, state in status_dict.items():
      print(f"通道{ch} USB2.0数据线: {'连接' if state == 1 else '断开'}")
  ```

---

### 3.3 USB3.0 数据线控制

#### `set_channel_usb3_dataline(*channels, state)`

- **描述**: 设置一个或多个通道的 USB3.0 数据线连接状态。

- **参数**:
  - `*channels` (int): 通道编号（1-4），可以指定多个通道。
  - `state` (int): `1` 连接数据线，`0` 断开数据线。

- **返回值**:
  - `bool`: 如果命令设置成功返回 `True`，否则返回 `False`。

- **示例**:
  ```python
  # 确保通道1电源已打开
  if hub.get_channel_power_status(1) == 0:
      hub.set_channel_power(1, state=1)
      time.sleep(0.1)
  
  # 断开通道1的USB3.0数据线（保持USB2.0和电源打开）
  hub.set_channel_usb3_dataline(1, state=0)
  
  # 重新连接通道1的USB3.0数据线
  # 注意：需要先关闭再打开电源以触发重新枚举
  hub.set_channel_power(1, state=0)
  time.sleep(1)
  hub.set_channel_power(1, state=1)
  hub.set_channel_usb3_dataline(1, state=1)
  ```

> [!NOTE]
>
> - USB3.0 数据线独立于 USB2.0 数据线控制。
> - 断开 USB3.0 数据线后，设备会降级到 USB2.0 速度。
> - 重新连接 USB3.0 数据线时，建议先关闭再打开电源以触发重新枚举。

#### `get_channel_usb3_dataline_status(*channels)`

- **描述**: 查询一个或多个通道的 USB3.0 数据线连接状态。

- **参数**:
  - `*channels` (int): 通道编号（1-4），可以指定多个通道。

- **返回值**:
  - `dict` 或 `None`: 字典 `{1: 1, 2: 0, ...}`，键为通道号，值为状态（1=连接，0=断开），若超时则返回 `None`。

- **示例**:
  ```python
  status_dict = hub.get_channel_usb3_dataline_status(1, 2, 3, 4)
  for ch, state in status_dict.items():
      print(f"通道{ch} USB3.0数据线: {'连接' if state == 1 else '断开'}")
  ```

---

### 3.4 慢充/快充模式

#### `set_channel_slow_charge(*channels, disconnect_before_switch=False)`

- **描述**: 启用一个或多个通道的慢充模式（限流模式）。

- **参数**:
  - `*channels` (int): 通道编号（1-4），可以指定多个通道。
  - `disconnect_before_switch` (bool, 可选): 如果为 `True`，在切换前先断开通道3秒。默认为 `False`。

- **返回值**:
  - `bool`: 如果命令设置成功返回 `True`，否则返回 `False`。

- **示例**:
  ```python
  # 启用通道1的慢充模式
  hub.set_channel_slow_charge(1)
  
  # 启用多个通道的慢充模式
  hub.set_channel_slow_charge(1, 2, 3, 4)
  
  # 先断开再切换到慢充模式（适用于从快充切换）
  hub.set_channel_slow_charge(1, disconnect_before_switch=True)
  ```

> [!NOTE]
>
> - **慢充模式**：限制充电电流（启用限流开关），适合需要限制功率的场景。
> - 如果通道之前是关闭状态，会先切换到快充模式3秒，然后再切换到慢充模式，以确保数据连接不断开。
> - 如果通道之前是打开状态（快充或慢充），直接切换不会断开连接。

#### `set_channel_fast_charge(*channels, disconnect_before_switch=True)`

- **描述**: 启用一个或多个通道的快充模式（全功率模式）。

- **参数**:
  - `*channels` (int): 通道编号（1-4），可以指定多个通道。
  - `disconnect_before_switch` (bool, 可选): 如果为 `True`，在切换前先断开通道1秒。默认为 `True`。

- **返回值**:
  - `bool`: 如果命令设置成功返回 `True`，否则返回 `False`。

- **示例**:
  ```python
  # 启用通道1的快充模式（默认会先断开）
  hub.set_channel_fast_charge(1)
  
  # 启用多个通道的快充模式
  hub.set_channel_fast_charge(1, 2, 3, 4)
  
  # 不先断开，直接切换到快充模式
  hub.set_channel_fast_charge(1, disconnect_before_switch=False)
  ```

> [!NOTE]
>
> - **快充模式**：提供全功率（禁用限流开关），适合需要最大功率的场景。
> - 默认会先断开连接1秒再切换，以确保硬件状态稳定。

#### `get_channel_charge_mode(*channels)`

- **描述**: 查询一个或多个通道的充电模式。方法内部会自动等待硬件完成切换并重试，确保返回稳定的状态。

- **参数**:
  - `*channels` (int): 通道编号（1-4），可以指定多个通道。

- **返回值**:
  - `dict` 或 `None`: 字典 `{1: 1, 2: 2, ...}`，键为通道号，值为模式：
    - `0`: 关闭
    - `1`: 快充模式
    - `2`: 慢充模式
    - 若超时则返回 `None`。

- **示例**:
  ```python
  # 查询单个通道（返回字典 {1: 1}）
  mode = hub.get_channel_charge_mode(1)
  if mode and 1 in mode:
      mode_val = mode[1]
      mode_str = {0: "关闭", 1: "快充", 2: "慢充"}.get(mode_val, "未知")
      print(f"通道1充电模式: {mode_str}")
  
  # 查询多个通道
  modes = hub.get_channel_charge_mode(1, 2, 3, 4)
  if modes:
      for ch, mode_val in modes.items():
          mode_str = {0: "关闭", 1: "快充", 2: "慢充"}.get(mode_val, "未知")
          print(f"通道{ch}充电模式: {mode_str}")
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
        'id': 'usbmodem132301',                    # 设备ID（端口后缀）
        'address': 0x0000,                         # 设备地址
        'hardware_version': 1,                     # 硬件版本
        'firmware_version': 5,                     # 固件版本
        'product_type': 'HBP_USB3_4CH',           # 产品类型名称
        'max_channels': 4,                         # 最大通道数
        'serial_no': 'XXXXXXXX-XXXXXXXX-XXXXXXXX', # 序列号
        'operate_mode': 'normal',                  # 工作模式
        'auto_restore': 'enabled',                 # 掉电恢复状态
        'button_control_status': 'enabled'         # 按键控制状态
    }
    ```

- **示例**:
  ```python
  device_info = hub.get_device_info()
  print(f"产品类型: {device_info['product_type']}")
  print(f"硬件版本: V1.{device_info['hardware_version']}")
  print(f"固件版本: V1.{device_info['firmware_version']}")
  print(f"序列号: {device_info['serial_no']}")
  print(f"最大通道数: {device_info['max_channels']}")
  ```

#### `get_product_type()`

- **描述**: 查询设备的产品类型。

- **返回值**:
  - `int` 或 `None`: 产品类型，USB3.0 4通道为 `0x02`，若无响应则返回 `None`。

- **示例**:
  ```python
  product_type = hub.get_product_type()
  # 返回: 0x02 (HBP_USB3_4CH)
  ```

#### `get_product_name()`

- **描述**: 获取产品名称。

- **返回值**:
  - `str` 或 `None`: 产品名称，例如 `"HBP_USB3_4CH"`，若无响应则返回 `None`。

- **示例**:
  ```python
  product_name = hub.get_product_name()
  # 返回: "HBP_USB3_4CH"
  ```

#### `get_hardware_version()`

- **描述**: 查询设备的硬件版本。

- **返回值**:
  - `int` 或 `None`: 硬件版本号，若无响应则返回 `None`。

- **示例**:
  ```python
  hw_ver = hub.get_hardware_version()
  # 返回: 1
  print(f"硬件版本: V1.{hw_ver}")
  ```

#### `get_firmware_version()`

- **描述**: 查询设备的固件版本。

- **返回值**:
  - `int` 或 `None`: 固件版本号，若无响应则返回 `None`。

- **示例**:
  ```python
  sw_ver = hub.get_firmware_version()
  # 返回: 5
  print(f"固件版本: V1.{sw_ver}")
  ```

#### `get_serial_no()`

- **描述**: 查询设备的序列号。

- **返回值**:
  - `str` 或 `None`: 序列号，格式为 `"XXXXXXXX-XXXXXXXX-XXXXXXXX"`，若无响应则返回 `None`。

- **示例**:
  ```python
  serial_no = hub.get_serial_no()
  # 返回: "XXXXXXXX-XXXXXXXX-XXXXXXXX"
  ```

#### `get_max_channels()`

- **描述**: 查询设备的最大通道数。

- **返回值**:
  - `int` 或 `None`: 最大通道数，若无响应则返回 `None`。

- **示例**:
  ```python
  max_channels = hub.get_max_channels()
  # 返回: 4
  ```

---

### 工作模式控制

#### `set_operate_mode(mode)`

- **描述**: 设置设备的工作模式。

- **参数**:
  - `mode` (int): 工作模式
    - `OPERATE_MODE_NORMAL` (0): 普通模式，可以同时打开多个通道。

- **返回值**:
  - `bool`: 如果命令设置成功返回 `True`，否则返回 `False`。

- **示例**:
  ```python
  from smartusbhub import SmartUSBHub, OPERATE_MODE_NORMAL
  
  # 设置为普通模式
  hub.set_operate_mode(OPERATE_MODE_NORMAL)
  ```

#### `get_operate_mode()`

- **描述**: 查询设备的工作模式。

- **返回值**:
  - `int` 或 `None`: 工作模式（0=普通模式），若无响应则返回 `None`。

- **示例**:
  ```python
  mode = hub.get_operate_mode()
  if mode == 0:
      print("当前模式: 普通模式")
  ```

---

### 默认状态配置

#### `set_default_power_status(*channels, enable, status=None)`

- **描述**: 设置一个或多个通道的默认电源状态。

- **参数**:
  - `*channels` (int): 通道编号（1-4），可以指定多个通道。
  - `enable` (int): `1` 启用默认电源状态，`0` 禁用。
  - `status` (int, 可选): 默认电源状态，`1` 为打开，`0` 为关闭。默认为 `0`。

- **返回值**:
  - `bool`: 如果命令设置成功返回 `True`，否则返回 `False`。

- **示例**:
  ```python
  # 设置通道1上电时默认打开电源
  hub.set_default_power_status(1, enable=1, status=1)
  
  # 设置通道2上电时默认关闭电源
  hub.set_default_power_status(2, enable=1, status=0)
  
  # 禁用通道1的默认电源状态
  hub.set_default_power_status(1, enable=0)
  ```

> [!NOTE]
>
> 默认电源状态在设备上电时生效，如果启用了掉电恢复功能，则优先使用掉电恢复。

#### `get_default_power_status(*channels)`

- **描述**: 查询一个或多个通道的默认电源状态配置。

- **参数**:
  - `*channels` (int): 通道编号（1-4），可以指定多个通道。

- **返回值**:
  - `dict` 或 `None`: 字典，键为通道号，值为包含 `enabled` 和 `value` 的字典，若超时则返回 `None`。
    ```python
    {
        1: {'enabled': 1, 'value': 1},  # 启用，默认打开
        2: {'enabled': 0, 'value': 0}   # 禁用
    }
    ```

- **示例**:
  ```python
  status = hub.get_default_power_status(1, 2, 3, 4)
  if status:
      for ch, config in status.items():
          if config['enabled']:
              print(f"通道{ch}: 默认电源{'打开' if config['value'] == 1 else '关闭'}")
          else:
              print(f"通道{ch}: 默认电源状态已禁用")
  ```

#### `set_default_dataline_status(*channels, enable, status=None)`

- **描述**: 设置一个或多个通道的默认数据线状态。

- **参数**:
  - `*channels` (int): 通道编号（1-4），可以指定多个通道。
  - `enable` (int): `1` 启用默认数据线状态，`0` 禁用。
  - `status` (int, 可选): 默认数据线状态，`1` 为连接，`0` 为断开。默认为 `0`。

- **返回值**:
  - `bool`: 如果命令设置成功返回 `True`，否则返回 `False`。

- **示例**:
  ```python
  # 设置通道1上电时默认连接数据线
  hub.set_default_dataline_status(1, enable=1, status=1)
  
  # 禁用通道1的默认数据线状态
  hub.set_default_dataline_status(1, enable=0)
  ```

#### `get_default_dataline_status(*channels)`

- **描述**: 查询一个或多个通道的默认数据线状态配置。

- **参数**:
  - `*channels` (int): 通道编号（1-4），可以指定多个通道。

- **返回值**:
  - `dict` 或 `None`: 字典，键为通道号，值为包含 `enabled` 和 `value` 的字典，若超时则返回 `None`。

- **示例**:
  ```python
  status = hub.get_default_dataline_status(1, 2, 3, 4)
  if status:
      for ch, config in status.items():
          if config['enabled']:
              print(f"通道{ch}: 默认数据线{'连接' if config['value'] == 1 else '断开'}")
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
> - **启用**: 断电重启后恢复到上次的通道电源状态
> - **禁用**: 断电重启后使用默认电源状态

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
> - `auto_restore` → `禁用 (0)`
> - `button_enable` → `启用 (1)`
> - 所有通道的默认状态 → `禁用`
> - 工作模式 → `普通模式`

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

- **描述**: 设置设备地址，用于在多台集线器连接的场景中，标识和区分各个集线器。

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

#### 示例：打开通道1电源

```
发送: 55 5A 01 01 01 03
      ↑   ↑  ↑  ↑  ↑  ↑
      头  头 CMD CH VAL CS

解释:
- 0x55 0x5A: 协议头
- 0x01: CMD_SET_CHANNEL_POWER
- 0x01: 通道1 (CHANNEL_1 = 0x01)
- 0x01: 打开电源
- 0x03: 校验和 (0x01+0x01+0x01=0x03)

响应: 55 5A 01 01 01 03 (ACK)
```

### 5.2 命令码定义

#### 通道控制命令

| 命令码 | 命令名 | 说明 |
|--------|--------|------|
| 0x01 | CMD_SET_CHANNEL_POWER | 设置通道电源 |
| 0x00 | CMD_GET_CHANNEL_POWER_STATUS | 获取通道电源状态 |
| 0x05 | CMD_SET_CHANNEL_DATALINE | 设置USB2.0数据线 |
| 0x08 | CMD_GET_CHANNEL_DATALINE_STATUS | 获取USB2.0数据线状态 |
| 0x15 | CMD_SET_CHANNEL_USB3_DATALINE | 设置USB3.0数据线 |
| 0x16 | CMD_GET_CHANNEL_USB3_DATALINE_STATUS | 获取USB3.0数据线状态 |
| 0x17 | CMD_SET_CHANNEL_FAST_CHARGE | 设置快充模式 |
| 0x13 | CMD_SET_CHANNEL_SLOW_CHARGE | 设置慢充模式 |
| 0x19 | CMD_GET_CHANNEL_CHARGE_MODE | 获取充电模式 |

#### 配置命令

| 命令码 | 命令名 | 说明 |
|--------|--------|------|
| 0x0B | CMD_SET_DEFAULT_POWER_STATUS | 设置默认电源状态 |
| 0x0C | CMD_GET_DEFAULT_POWER_STATUS | 获取默认电源状态 |
| 0x0D | CMD_SET_DEFAULT_DATALINE_STATUS | 设置默认数据线状态 |
| 0x0E | CMD_GET_DEFAULT_DATALINE_STATUS | 获取默认数据线状态 |
| 0x06 | CMD_SET_OPERATE_MODE | 设置工作模式（仅支持普通模式） |
| 0x07 | CMD_GET_OPERATE_MODE | 获取工作模式 |

#### 公共命令

| 命令码 | 命令名 | 说明 |
|--------|--------|------|
| 0x09 | CMD_SET_BUTTON_CONTROL | 设置按键使能 |
| 0x0A | CMD_GET_BUTTON_CONTROL_STATUS | 获取按键状态 |
| 0x0F | CMD_SET_AUTO_RESTORE_ENABLE | 设置掉电恢复 |
| 0x10 | CMD_GET_AUTO_RESTORE_STATUS | 获取掉电恢复状态 |
| 0xF0 | CMD_GET_PRODUCT_TYPE | 获取产品类型 |
| 0xF7 | CMD_REBOOT_MCU | 重启设备 |
| 0xFC | CMD_FACTORY_RESET | 恢复出厂设置 |
| 0xFD | CMD_GET_FIRMWARE_VERSION | 获取固件版本 |
| 0xFE | CMD_GET_HARDWARE_VERSION | 获取硬件版本 |
| 0xF1 | CMD_GET_MAX_CHANNELS | 获取最大通道数 |
| 0xF9 | CMD_GET_SERIAL_NO | 获取序列号 |
| 0x11 | CMD_SET_DEVICE_ADDRESS | 设置设备地址 |
| 0x12 | CMD_GET_DEVICE_ADDRESS | 获取设备地址 |

### 5.3 协议示例

#### 示例1: 获取通道1电源状态

```
发送: 55 5A 00 01 00 01
响应: 55 5A 00 01 01 02  (通道1电源已打开)
                    ↑
                  0x01 = 电源打开
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

#### 示例3: 设置USB3.0数据线

```
发送: 55 5A 15 01 01 17
      ↑   ↑  ↑  ↑  ↑  ↑
      头  头 CMD CH VAL CS

CMD = 0x15 (SET_CHANNEL_USB3_DATALINE)
CH = 0x01 (通道1)
VALUE = 0x01 (连接)

响应: 55 5A 15 01 01 17 (ACK)
```

## 8. 常见问题

### 8.1 设备连接问题

#### Q: `scan_and_connect()` 返回 None

#### A: 可能的原因

1. 设备未连接或未上电
2. 权限不足（Linux 需要 udev 规则）
3. 串口被其他程序占用

#### Q: 连接后设备无响应

#### A: 尝试以下步骤

1. 断开重连
2. 重启设备
3. 检查固件版本
4. 增加超时时间

```python
hub = SmartUSBHub("/dev/ttyACM0")
```

### 8.2 多设备场景

#### Q: 如何连接多个设备

#### A: 使用设备地址区分

```python
# 扫描并连接所有设备
hubs = []
exclude_ports = set()

while True:
    hub = SmartUSBHub.scan_and_connect(exclude_ports=exclude_ports)
    if hub is None:
        break
    hubs.append(hub)
    exclude_ports.add(hub.port)
    print(f"设备 {len(hubs)}: 地址 0x{hub.device_address:04X}")

# 控制不同设备
if len(hubs) >= 2:
    hubs[0].set_channel_power(1, state=1)  # 设备1，通道1
    hubs[1].set_channel_power(1, state=1)  # 设备2，通道1
```

---

## 9. 附录

### 9.1 示例代码

#### Python 库示例

示例代码位于 `smartusbhub_ng/examples/SmartUSBHub_Pro/` 目录：

| 文件 | 说明 |
|------|------|
| `power_control_example.py` | 电源控制示例，演示单个和多个通道的电源控制 |
| `dataline_control_example.py` | 数据线控制示例，演示USB2.0和USB3.0数据线控制 |
| `auto_charge_mode_switch.py` | 自动充电模式切换示例，循环切换慢充/快充模式 |
| `multi_device_channel_control.py` | 多设备通道控制示例，演示多设备场景 |
| `setting_example.py` | 配置示例，演示默认状态、掉电恢复等配置 |

### 9.2 相关文档

- [产品规格书](../SmartUSBHub_Pro-4CH_USB3.0/产品规格书.md)
- [GitHub 仓库](https://github.com/mixedsignal-labs/smartusbhub)

---

## 10. 修订历史

| 版本 | 日期 | 修订内容 | 作者 |
|------|------|---------|------|
| v1.0 | 2026-01-10 | 初版发布 | Makerlabtools |

---

**© 2026 makerlabtools. All Rights Reserved.**

