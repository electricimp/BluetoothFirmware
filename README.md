# Bluetooth Firmware #

The Bluetooth Firmware library contains the firmware strings needed to [initialize an imp bluetooth hardware object](https://developer.electricimp.com/api/hardware/bluetooth/open). The library contains the latest recommended firmware for use on the imp004m and imp006 modules. The imp004m firmware is approximately 16KB in size. The imp006 firmware is approximately 60KB in size. The firmware will only consume memory if referenced in your code. 

**To include this library in your project, add** `#require "bt_firmware.lib.nut:1.0.0"` **at the top of your code**

## Library Usage ##

The library consists of an enum `BT_FIRMWARE` with the following elements, all strings.

| Element | Value |
| --- | --- |
| *VERSION* | Bluetooth Firmware library release version |
| *IMP_004M* | Firmware for the imp004m |
| *IMP_006* | Firmware for the imp006 |

### imp004m Example ###

```
bt <- hardware.bluetooth.open(bt_uart, BT_FIRMWARE.IMP_004M);
```

### imp006 Example ###

```
bt <- hardware.bluetooth.open(bt_uart, BT_FIRMWARE.IMP_006);
```

## Full Examples ##

This library can be used on either the agent or the device. If used on the agent the firmware will need to be sent to the device to storage in SPI Flash or use. If used on the device please be sure your application has enough memory to store the firmware.

### Device ###

```

```

### Agent ###

```

```

## License ##

This library is licensed under the [MIT License](./LICENSE).