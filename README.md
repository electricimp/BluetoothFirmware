# Bluetooth Firmware 1.0.0 #

The Bluetooth Firmware library contains the firmware strings required to [initialize an imp API **bluetooth** object](https://developer.electricimp.com/api/hardware/bluetooth/open). The library contains the recommended bluetooth firmware for use with the [Cypress 43438 found on the imp004m](https://developer.electricimp.com/hardware/resources/reference-designs/imp004mbreakout#bluetooth-le), and the Cypress 43455 found on the imp006 module and on the WiFi/Bluetooth mikroBUS Click board [for use with the impC001](https://developer.electricimp.com/hardware/resources/reference-designs/impc001breakout#mikrobus). The Cypress 43438 firmware is approximately 16KB in size. The Cypress 43455 firmware is approximately 60KB in size. The firmware will only consume memory if referenced in your code.

**To include this library in your project, add** `#require "bt_firmware.lib.nut:1.0.0"` **at the top of your code**

## Library Usage ##

The library consists of an enum, *BT_FIRMWARE*, with the following elements, all of which are strings.

| Element | Value |
| --- | --- |
| *VERSION* | Electric Imp Bluetooth Firmware library release version |
| *CYW_43438_VERSION* | Broadcom 43438 firmware version string,<br />eg. `"BCM4343A1_001_002_009_0018_0028_Generic_UART_37_4MHz_wlbga_ref_hcd"` |
| *CYW_43455_VERSION* | Broadcom 43455 firmware version string,<br />eg. `"BCM4345C0 Murata Type-1MW UART 37.4 MHz BT 4.2-0144"` |
| *CYW_43438* | Bluetooth Firmware for the Cypress 43438 found on the imp004m |
| *CYW_43455* | Bluetooth Firmware for the Cypress 43455 found on the imp006 and WiFi/Bluetooth Click |

### imp004m Example ###

```squirrel
bt <- hardware.bluetooth.open(bt_uart, BT_FIRMWARE.CYW_43438);
```

### imp006 Example ###

```squirrel
bt <- hardware.bluetooth.open(bt_uart, BT_FIRMWARE.CYW_43455);
```

**Note** This library is not required if your imp006 is running impOS 41.28 or above (see [**How To Use Bluetooth With The imp006**](https://developer.electricimp.com/resources/bluetooth_imp006)).

## Full Examples ##

This library can be used on either the agent or the device. These examples will walk through basic usage for both.

### Library On Device ###

When the library is used on the device, please be sure your application has sufficient memory to store the firmware.

#### Device Code ####

```squirrel
#require "bt_firmware.lib.nut:1.0.0"

// Set up BT on imp004m Breakout Board
bt <- null;
bt_uart <- hardware.uartFGJH;
bt_lpo_in <- hardware.pinE;
bt_reg_on <- hardware.pinJ;

// Boot up BT
bt_lpo_in.configure(DIGITAL_OUT, 0);
bt_reg_on.configure(DIGITAL_OUT, 1);

// Pause to ensure pins are in the newly
// configured state before proceeding...
imp.sleep(0.05);

try {
    // Instantiate BT with the imp004m firmware
    bt = hardware.bluetooth.open(bt_uart, BT_FIRMWARE.CYW_43438);
    server.log("BLE initialized");
} catch (err) {
    server.log("BLE failed: " + err);
}
```

### Library On Agent ###

When the library is used on the agent, the firmware will need to be sent to the device, so that the device will be able to boot Bluetooth. The device can persist the received firmware in its SPI flash storage.

#### Agent Code ####

```squirrel
#require "bt_firmware.lib.nut:1.0.0"

device.on("get.firmware", function(ignore) {
    // Send imp004m BT firmware to the device
    server.log("Sending device bluetooth firmware version " + BT_FIRMWARE.CYW_43438_VERSION);
    device.send("set.firmware", BT_FIRMWARE.CYW_43438);
})

server.log("AGENT RUNNING...");
```

#### Device Code ####

```squirrel
#require "SPIFlashFileSystem.device.lib.nut:2.0.0"

const BT_FIRMWARE_FILE_NAME = "btFirmware";

// Store Bluetooth firmware in SPI Flash
function storeBTFirmware(firmware) {
    server.log("Storing bluetooth firmware to SPI flash");

    // We only want one version of firmware stored at a time, so
    // erase if we already have stored firmware
    if (haveStoredFirmware) sffs.eraseFile(BT_FIRMWARE_FILE_NAME);

    local file = sffs.open(BT_FIRMWARE_FILE_NAME, "w");
    file.write(firmware);
    file.close();

    // Make sure we update stored firmware flag
    haveStoredFirmware <- sffs.fileExists(BT_FIRMWARE_FILE_NAME);
}

// Retrieve Bluetooth firmware from SPI Flash
function getBTFirmware() {
    server.log("Retrieving stored bluetooth firmware");
    if (!haveStoredFirmware) return null;

    local file = sffs.open(BT_FIRMWARE_FILE_NAME, "r");
    local firmware = file.read();
    file.close();
    return firmware;
}

// Use stored firmware to boot bluetooth
function bootBT() {
    server.log("Booting Bluetooth with stored firmware");
    bt_lpo_in.configure(DIGITAL_OUT, 0);
    bt_reg_on.configure(DIGITAL_OUT, 1);

    // Pause to ensure pins are in the newly
    // configured state before proceeding
    imp.sleep(0.05);

    try {
        // Instantiate BT
        bt = hardware.bluetooth.open(bt_uart, getBTFirmware());
        server.log("BLE initialized");
    } catch (err) {
        server.log("BLE failed: " + err);
    }
}

// Configure global imp004m bluetooth hardware variables
bt <- null;
bt_uart <- hardware.uartFGJH;
bt_lpo_in <- hardware.pinE;
bt_reg_on <- hardware.pinJ;

// Configure SPI flash storage
sffs <- SPIFlashFileSystem(0x000000, 0x020000);
sffs.init();
haveStoredFirmware <- sffs.fileExists(BT_FIRMWARE_FILE_NAME);

// Create listener for firmware message from agent
agent.on("set.firmware", function(firmware) {
    server.log("Received bluetooth firmware from agent");
    storeBTFirmware(firmware);
    bootBT();
});

server.log("DEVICE RUNNING...");
imp.enableblinkup(true);
server.log(imp.getsoftwareversion());

if (haveStoredFirmware) {
    // Boot bluetooth with stored firmware
    bootBT();
} else {
    // Get bluetooth firmware from agent
    server.log("No stored Bluetooth firmware. Requesting firmware from agent...");
    agent.send("get.firmware", null);
}
```

## License ##

This library is licensed under the [MIT License](./LICENSE).<br />Firmware copyright Cypress Semiconductor Corp.
