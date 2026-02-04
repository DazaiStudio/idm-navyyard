# Unreal Engine 5.5 - DMX Image Based Lighting Guide

Using DMX Pixel Mapping to convert scene capture to DMX signal output for LED CYC fixtures.

**Demo:**

<img src="ue_demo.gif" width="800">

---

## Overview

- [Part 1: Physical Setup](#part-1-physical-setup) - Hardware connection and network configuration
- [Part 2: Unreal Engine Setup](#part-2-unreal-engine-setup) - DMX plugin configuration and pixel mapping

---

# Part 1: Physical Setup

## Equipment List

| Device | Purpose |
|--------|---------|
| Art-Net to DMX Converter | Art-Net to DMX512 + 12-port splitter |
| LED CYC | RGBW fixture |

## Connection Diagram

```
PC (Ethernet) ──► Obsidian Netron EN12 (Art-Net to DMX) ──► LED CYC
```

<img src="screenshot/physical/connection.JPG" width="600">

## Network Settings

**Windows Path:** Settings > Network & Internet > Ethernet

<table>
<tr>
<td>

| Setting | Value |
|---------|-------|
| IP assignment | Manual |
| IPv4 address | 10.108.12.51 |
| IPv4 mask | 255.255.255.0 |

</td>
<td><img src="screenshot/physical/pc-network.png" width="500"></td>
</tr>
</table>

> **Why 10.108.12.x?** The Netron EN12 default IP is 10.108.12.43. PC must be on the same subnet (10.108.12.x) to communicate via Art-Net. Choose any available IP in range 10.108.12.1-254 (except .43).

## Netron EN12 Settings

<table>
<tr>
<td>

| Setting | Value |
|---------|-------|
| IP Address | 10.108.12.43 |
| Status | No Cue (Standby) |

</td>
<td><img src="screenshot/physical/dmxbox_setting.JPG" width="500"></td>
</tr>
</table>

---

# Part 2: Unreal Engine Setup

## 1. Enable DMX Plugins

**Edit > Plugins** search "DMX", enable all DMX-related plugins.

<img src="screenshot/dmx-plugin.png" width="700">

---

## 2. Project Settings - DMX

**Edit > Project Settings > Plugins > DMX**

<table>
<tr>
<td>

| Setting | Value |
|---------|-------|
| Protocol Type | Art Net |
| Communication Type | Broadcast |
| Network Interface Card IP | 10.108.12.51 |
| Input into Engine | ✓ |
| Local Universe Start | 1 |
| Amount of Universes | 10 |
| Protocol Universe Remap | ✓ |
| Send DMX by default | ✓ |
| Receive DMX by default | ✓ |

</td>
<td><img src="screenshot/dmx_output_setting-2.png" width="500"></td>
</tr>
</table>

---

## 3. DMX Library

Manages fixtures, patches, and output ports.

<img src="screenshot/dmx_pxiel_essential - dmxlib.png" width="700">

Name: `DMXLib_CYC`

### 3.1 Configure Output Ports

In the **Library Settings** tab:

<img src="screenshot/dmx_output_setting-1.png" width="700">

### 3.2 Fixture Type

In **Fixture Types** tab:

| Ch | Name | Attribute |
|----|------|-----------|
| 1 | `<Function>` | Red |
| 2 | `<Function>` | Green |
| 3 | `<Function>` | Blue |
| 4 | `<Function>` | White |

<img src="screenshot/fix_type.png" width="700">

### 3.3 Fixture Patch

Assigns DMX addresses to each fixture. Each fixture uses 4 channels (RGBW).

<img src="screenshot/fix-patch.png" width="700">

---

## 4. Render Target

Captures downsampled scene image for pixel mapping.

<img src="screenshot/dmx_pxiel_essential - rt.png" width="700">

Name: `RT_DownSample_CYC`

<table>
<tr>
<td>

| Setting | Value |
|---------|-------|
| Size X | 128 |
| Size Y | 128 |
| Render Target Format | RTF RGBA16f |
| Clear Color | 0, 0, 0, 1 |
| Address X | Clamp |
| Address Y | Clamp |

</td>
<td><img src="screenshot/dmx_pxiel_rendertarget.png" width="700"></td>
</tr>
</table>

---

## 5. Scene Capture 2D

Captures scene and outputs to Render Target.

<img src="screenshot/BP_DnSampleCap.png" width="700">

---

## 6. DMX Pixel Mapping

Maps Render Target pixels to DMX fixtures.

<img src="screenshot/dmx_pxiel_essential - dmxpixmapping.png" width="700">

Name: `DMXPM_PixelMap_CYC`

### 6.1 Add Fixture Group

Select all LWCYC fixtures to add to Pixel Mapping.

### 6.2 Pixel Mapping Settings

| Setting | Value |
|---------|-------|
| Fixture Group | DMXLib_CYC |
| Input Texture | RT_DownSample_CYC |
| Distribution | Configure based on fixture layout |
| Apply Blur | (Optional) |

<img src="screenshot/dmx_pxielmapping.png" width="700">

---

## 7. Blueprint We Use

Sends pixel mapping data to DMX output every frame.

<img src="screenshot/bp_pix.png" width="700">

---

## 8. Testing - DMX Control Console

**Window > DMX Control Console**

<img src="screenshot/dmx_pxiel_essential - dmxconsole.png" width="700">

Manually test RGBW output for each fixture:

| Fixture | Red | Green | Blue | White |
|---------|-----|-------|------|-------|
| LWCYC | 0-255 | 0-255 | 0-255 | 0-255 |

<img src="screenshot/dmx_mini-console_fortesting.png" width="700">

---

## Asset List

| Type | Name | Path |
|------|------|------|
| DMX Library | DMXLib_CYC | Content/DMX/ |
| Render Target | RT_DownSample_CYC | Content/DMX/ |
| Pixel Mapping | DMXPM_PixelMap_CYC | Content/DMX/ |
| Blueprint | BP_PixelMappingManager_CYC | Content/DMX/BP/ |
| Blueprint | BP_DownSampleSceneCapture_CYC | Content/DMX/BP/ |

---

## Workflow Summary

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
Obsidian Netron EN12 (Art-Net to DMX)
      |
      v
LED CYC Fixtures (RGBW)
```

---

*Unreal Engine 5.5 | Last Updated: 2026-02-04*
