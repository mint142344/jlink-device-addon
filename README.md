# JLink Device Addon

> ONLY FOR **PYTHON36 X64**

Open cmsis pack and add device info to JLink, used to support JLink flasher

## Usage

- Install python36 x64

- Execute `./setup.sh` install dependences

- Execute `jlink-device-addon -x <JLinkDevices.xml path> <your cmsis pack path>` 

- And then the device info list will be add to your `JLinkDevices.xml`

## Notice

Recommended for using it in `Embedded IDE`

## Initial JLinkDevices.xml

3.14+ 能利用 'Deferred Evaluation' 特性跳过 NameError 报错

First, Create `~/.config/SEGGER/JLinkDevices/JLinkDevices.xml` with content:

```xml
<Database>
</Database>
```
