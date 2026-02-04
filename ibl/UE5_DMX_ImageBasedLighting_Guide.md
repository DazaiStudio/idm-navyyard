# Unreal Engine 5.5 - DMX Image Based Lighting Guide

使用 DMX Pixel Mapping 将场景画面转换为 DMX 信号输出至 LED CYC 灯具。

---

## 硬件连接设置

### 设备清单

| 设备 | 型号 | 用途 |
|------|------|------|
| Art-Net 转换器 | Netron EN12 | Art-Net 转 DMX512 |
| DMX Splitter | Obsidian Control Systems | DMX 信号分配 |
| LED CYC | Leviton LWCYC-00B | RGBW 灯具 |

### 连接方式

```
PC (Ethernet) ──► Netron EN12 (Art-Net to DMX) ──► Obsidian Splitter ──► LED CYC
```

![Hardware Connection](screenshot/physical/connection.JPG)

### 网络设置

**Windows 设置路径：** Settings > Network & Internet > Ethernet

| 设置 | 值 |
|------|-----|
| IP assignment | Manual |
| IPv4 address | 10.108.12.51 |
| IPv4 mask | 255.255.255.0 |

> 电脑与 DMX Box 必须在同一网段

![PC Network Settings](screenshot/physical/pc-network.png)

### Netron EN12 设置

| 设置 | 值 |
|------|-----|
| IP Address | 10.108.12.43 |
| Status | No Cue (待机) |

![Netron EN12](screenshot/physical/dmxbox_setting.JPG)

---

## 1. 启用 DMX 插件

**Edit > Plugins** 搜索 "DMX"，启用以下插件：

| 插件名称 | 用途 |
|---------|------|
| DMX Control Console | DMX 测试控制台 |
| DMX Engine | DMX 核心功能 |
| DMX Fixtures | 灯具 Blueprint |
| DMX Pixel Mapping | 像素映射 (核心功能) |
| DMX Protocol | DMX 协议支持 |
| Remote Control Protocol DMX | 远程控制 |

![DMX Plugins](screenshot/dmx-plugin.png)

---

## 2. 创建 DMX Library

**Content Browser > 右键 > DMX > DMX Library**

命名：`DMXLib_CYC`

### 2.1 设置 Output Ports

在 **Library Settings** 标签页：

| 设置 | 值 |
|------|-----|
| Output Port | OutputPort1 |
| Protocol | Art-Net |
| Local Universe | 1 - 10 |
| Extern Universe | 0 - 9 |
| Enabled | ✓ |

![DMX Output Settings](screenshot/dmx_output_setting-1.png)

---

## 3. Project Settings - DMX

**Edit > Project Settings > Plugins > DMX**

| 设置 | 值 |
|------|-----|
| Protocol Type | Art Net |
| Communication Type | Broadcast |
| Network Interface Card IP | 192.168.1.91 (你的网卡 IP) |
| Input into Engine | ✓ |
| Local Universe Start | 1 |
| Amount of Universes | 10 |
| Protocol Universe Remap | (根据需求设置) |
| Send DMX by default | ✓ |
| Receive DMX by default | ✓ |

![DMX Project Settings](screenshot/dmx_output_setting-2.png)

---

## 4. 创建 Fixture Type

在 DMX Library 的 **Fixture Types** 标签页：

### LWCYC (LED Cyclorama)

| 设置 | 值 |
|------|-----|
| Name | LWCYC |
| DMX Category | Other |
| Mode | 4COL |
| Auto Channel Span | ✓ |
| Fixture Matrix Enabled | ✗ |

### Channel 配置 (4COL Mode)

| Ch | Name | Attribute |
|----|------|-----------|
| 1 | `<Function>` | Red |
| 2 | `<Function>` | Green |
| 3 | `<Function>` | Blue |
| 4 | `<Function>` | White |

**Function Settings:**
- Data Type: 8bit
- Attribute Mapping: Red/Green/Blue/White
- Physical From: 0
- Physical To: 1.0

![Fixture Type](screenshot/fix_type.png)

### LWCYC-00B 完整 DMX Channel 表

![Channel Chart](screenshot/physical/ledcyc_channelchart.JPG)

#### 4COL Mode (4 channels)

| Ch | Control | DMX Value | Output |
|----|---------|-----------|--------|
| 1 | Red | 0-255 | Brightness 0-100% |
| 2 | Green | 0-255 | Brightness 0-100% |
| 3 | Blue | 0-255 | Brightness 0-100% |
| 4 | White | 0-255 | Brightness 0-100% |

#### HSI Mode (3 channels)

| Ch | Control | DMX Value | Output |
|----|---------|-----------|--------|
| 1 | Hue | 0-255 | 0-360° |
| 2 | Saturation | 0-255 | 0-100% |
| 3 | Brightness | 0-255 | 0-100% |

#### FIL Mode (2 channels) - 16 色效果

| Ch | Control | DMX Value | Color |
|----|---------|-----------|-------|
| 1 | Brightness | 0-255 | 0-100% |
| 2 | Color Select | 0-15 | Transparent |
| | | 16-31 | Pale gold |
| | | 32-47 | Straw yellow |
| | | 48-63 | Light amber |
| | | 64-79 | Chrome orange |
| | | 80-95 | Hot pink |
| | | 96-111 | Shocking pink |
| | | 112-127 | Magenta |
| | | 128-143 | Bright red |
| | | 144-159 | Rose purple |
| | | 160-175 | Light purple |
| | | 176-191 | Moss green |
| | | 192-207 | Dark green |
| | | 208-223 | Primary blue |
| | | 224-239 | Peacock blue |
| | | 240-255 | Dark blue |

#### CCT Mode (2 channels)

| Ch | Control | DMX Value | Output |
|----|---------|-----------|--------|
| 1 | Brightness | 0-255 | 0-100% |
| 2 | Color Temp | 0-63 | 3200K |
| | | 64-127 | 4300K |
| | | 128-191 | 5600K |
| | | 192-255 | 6500K |

---

## 5. 创建 Fixture Patch

在 **Fixture Patch** 标签页，为每个 CYC 灯具分配 DMX 地址：

| Fixture | Type | Mode | Universe | Starting Ch |
|---------|------|------|----------|-------------|
| LWCYC | LWCYC | 4COL | 1 | 1 |
| LWCYC_001 | LWCYC | 4COL | 1 | 5 |
| LWCYC_002 | LWCYC | 4COL | 1 | 9 |
| LWCYC_003 | LWCYC | 4COL | 1 | 13 |

每个灯具占用 4 channels (RGBW)。

![Fixture Patch](screenshot/fix-patch.png)

---

## 6. 创建 Render Target

**Content Browser > 右键 > Textures > Render Target**

命名：`RT_DownSample_CYC`

| 设置 | 值 |
|------|-----|
| Size X | 128 |
| Size Y | 128 |
| Render Target Format | RTF RGBA16f |
| Clear Color | 0, 0, 0, 1 |
| Address X | Clamp |
| Address Y | Clamp |

![Render Target](screenshot/dmx_pxiel_rendertarget.png)

---

## 7. 创建 Scene Capture 2D

在场景中放置 **BP_DownSampleSceneCapture_CYC**

### 设置

| 属性 | 值 |
|------|-----|
| Texture Target | RT_DownSample_CYC |
| Capture Source | Final Color (LDR) |
| FOV | 根据需求调整 |

![Scene Capture](screenshot/BP_DnSampleCap.png)

---

## 8. 设置 DMX Pixel Mapping

**Content Browser > 右键 > DMX > DMX Pixel Mapping**

命名：`DMXPM_PixelMap_CYC`

### 8.1 添加 Fixture Group

选择所有 LWCYC fixtures 加入 Pixel Mapping。

### 8.2 设置 Pixel Mapping

| 设置 | 值 |
|------|-----|
| Fixture Group | DMXLib_CYC |
| Input Texture | RT_DownSample_CYC |
| Distribution | 根据灯具排列设置 |
| Apply Blur | (可选) |

![Pixel Mapping](screenshot/dmx_pxielmapping.png)

---

## 9. Blueprint 整合

创建 **BP_PixelMappingManager_CYC**

### Event Graph

```
Event Tick
    |
    v
[Render Texture Sample and Send DMX]
    |
    +-- Delta Seconds: 0
    +-- Tick Component
    |
    v
[Get DMX Pixel Mapping Renderer Component]
    |
    +-- PixelMapping: DMXPM_PixelMap_CYC
    +-- Pixel Mapping: RT_DownSample_CYC
```

![Blueprint Manager](screenshot/BP_PxManager.png)

---

## 10. 场景设置

在 Level 中放置：

1. **BP_DownSampleSceneCapture_CYC** - 捕捉场景画面
2. **BP_PixelMappingManager_CYC** - 处理并发送 DMX
3. **DirectionalLight** - 场景主光源
4. **SEQ_ABC_Test** - (可选) Sequencer 动画

![Level Setup](screenshot/bp_pix.png)

---

## 11. 测试 - DMX Control Console

**Window > DMX Control Console**

可以手动测试每个灯具的 RGBW 输出：

| Fixture | Red | Green | Blue | White |
|---------|-----|-------|------|-------|
| LWCYC | 0-255 | 0-255 | 0-255 | 0-255 |

![DMX Console](screenshot/dmx_mini-console_fortesting.png)

---

## 资产清单

| 类型 | 名称 | 路径 |
|------|------|------|
| DMX Library | DMXLib_CYC | Content/DMX/ |
| Render Target | RT_DownSample_CYC | Content/DMX/ |
| Pixel Mapping | DMXPM_PixelMap_CYC | Content/DMX/ |
| Blueprint | BP_PixelMappingManager_CYC | Content/DMX/BP/ |
| Blueprint | BP_DownSampleSceneCapture_CYC | Content/DMX/BP/ |

---

## 流程总结

```
Scene Capture 2D
      |
      v
Render Target (128x128)
      |
      v
DMX Pixel Mapping
      |
      v
DMX Library (Fixture Patch)
      |
      v
Art-Net Output (UDP Broadcast)
      |
      v
Netron EN12 (Art-Net to DMX)
      |
      v
Obsidian DMX Splitter
      |
      v
LED CYC Fixtures (RGBW)
```

---

*Unreal Engine 5.5 | Last Updated: 2026-02-04*
