> **Firmware:** `SNMP_Modbus_Gateway` v1.0.2  
> **Platform:** Tibbo TPP2W(G2) in a TPB2L(G2) enclosure

<br><br>

---

## 1. Overview

The SNMP–Modbus Gateway lets **SNMP monitoring software read** values from
**Modbus equipment** over **Ethernet**. It is used when equipment such as meters, sensors, 
controllers, or other industrial devices communicates with Modbus, while the
customer's monitoring system uses SNMP.

The gateway regularly reads the Modbus values selected by the user and makes
them available as SNMP values.
A monitoring system can then display the values, record them, or use them to raise alarms.

Where specifically configured and authorized, the gateway can also accept an
**SNMP write** command and send the corresponding Modbus write to the field device.
This **remote-control function** should be enabled only for approved control points.

![Modbus field device to SNMP monitoring-system gateway flow](https://raw.githubusercontent.com/imcase/Im_snmp_modbus_gateway_assets/refs/heads/main/assets/snmp_modbus_gateway_flow.png)

The user configures which Modbus values to read, and how each value appears to SNMP software.
This means the same gateway can be used with different types of
Modbus equipment instead of being limited to a single sensor or meter.

<br><br>

---

## 2. Hardware

| Item | Detail |
| --- | --- |
| Controller | Tibbo TPP2W(G2) Tibbo Project PCB |
| Enclosure | TPB2L(G2), DIN-rail mounting kit, vibration-proof kit |
| Network | On-board 10/100 Base-T Ethernet with RJ45 |
| Field interface | Tibbit #05 RS-485 port for Modbus RTU |
| Field connector | Tibbit #21 (4-terminal block) or Tibbit #20 (9-terminal block) |
| Power | Tibbit #10 power supply module in socket 11 |
| Power adapter | APR-P0008 adapter |
| Storage | 1 MB AT45DB081E dataflash, LittleFS file system |
| Status LEDs | On-board red/green LEDs used for the setup-error indication |
| MD button | On-board setup (MD) button. Hold for 5 s and release to reset all settings to their defaults |
| RST button | On-board restart (RST) button |

<br><br>

---

## 3. Network configuration and services

| Item | Setting |
| --- | --- |
| Ethernet | static IP/netmask/gateway, or DHCP  |
| SNMP | SNMPv2c and SNMPv3 agent, port 161, configurable read/write communities and SNMPv3 security credentials |
| Time sync | SNTP against 216.239.35.0 (time.google.com), applied with the configured time zone |
| Device Console | HTTP, port 80, 3 concurrent connections |

<br>

> **NOTE:**
> - Out of the box, the device uses the **default** static IP address
> `192.168.1.40`. This placeholder makes the device reachable on first
> power-up; change it to match your network.
> - A **network connection** is required for communication between the gateway
> and its configured SNTP servers, as well as authorized SNMP managers.

<br><br>

---

## 4. Device Console

> The browser-based configuration interface is called the **Device Console**. It
> may also be referred to as the Web Console in AppBlocks documentation.

<br>

### 4.1 Database
For every value you want to monitor, configure the gateway with the relevant Modbus device and register.
The gateway reads that register at the **configured polling interval** and makes the latest value available to
your SNMP monitoring system.

| Table | Storage | Entries | User-defined | After reboot | Purpose |
| --- | --- | --- | --- | --- | --- |
| `TABLE_DEVICES_INPUT` | Flash | Max. 30 | Yes | Retained | Modbus-to-SNMP mappings |
| `TABLE_DEVICES_OUTPUT` | RAM | Max. 30 | No | Rebuilt | Latest polling results |
| `TABLE_SETTINGS` | RAM | 22 predefined | No | Recreated | Device Console settings |

<br>

> **NOTE:**
> - Use `TABLE_DEVICES_INPUT` to define which Modbus values the gateway monitors.
> - `TABLE_DEVICES_OUTPUT` shows the current polling results; it is a live view, not a saved history, and is rebuilt after restart.
> - `TABLE_SETTINGS` contains the settings used by the Device Console.
> - **After any change to `TABLE_DEVICES_INPUT`, or to `Settings` through the Device Console or SNMP, restart the device.**
> The change is retained immediately, but the polling engine does not apply it until the device restarts.

<br>

### Column reference

The following table explains the columns shown in the Device Console database
tables.

| Column | Used by table | Meaning |
| --- | --- | --- |
| `NAME` | Input, Output, Settings | A descriptive name for the mapping or setting. |
| `DEVICE_ID` | Input, Output | The Modbus address of the field device. |
| `REG_ADDR` | Input, Output | The Modbus register address to read or write. |
| `FUNC_CODE` | Input | The Modbus function code used to access the register. |
| `OID_COLUMN` | Input, Output, Settings | The identifier used to associate the entry with its SNMP object. |
| `HASH_ID` | Output | An internal unique identifier for the polling result. |
| `VALUE` | Output | The latest value read from the Modbus device. |
| `INFO` | Settings | Additional information describing the Device Console setting. |

<br>

> **NOTE:**
> - **Register addressing and value interpretation:**
> Enter the register number exactly as shown in the connected device's Modbus
> register map.
> - **Warning:** If the register address is incorrect, the gateway may not receive a result
> when polling the Modbus device.

<br>

### 4.2 Settings

Use the Settings area to review device information and configure polling,
network, and SNMP access. 

> **Reminder:** `RESTART` the device after changing any setting so the new value is applied.

<br>

#### 4.2.1 Info

The `Info` group provides **maintenance** and **capacity** information.

![Device Console Settings, Info group](https://raw.githubusercontent.com/imcase/Im_snmp_modbus_gateway_assets/refs/heads/main/assets/snmp_modbus_gateway_webconsole_settings_info.png)

<br>

#### 4.2.2 Config

Use the `Config` group to set the **Polling Interval** and **Debugprint**.

> **NOTE:** Adjust the `Polling Interval` to suit your application requirements.

![Device Console Settings, Config group](https://raw.githubusercontent.com/imcase/Im_snmp_modbus_gateway_assets/refs/heads/main/assets/snmp_modbus_gateway_webconsole_settings_config.png)

<br>

#### 4.2.3 Network

The Network group lets you enable DHCP or configure a static Ethernet IP
address, netmask, and gateway.

> **NOTE:** When DHCP is enabled, the static Ethernet address fields are unavailable.

![Device Console Settings, Network group](https://raw.githubusercontent.com/imcase/Im_snmp_modbus_gateway_assets/refs/heads/main/assets/snmp_modbus_gateway_webconsole_settings_network.png)

<br>

#### 4.2.4 SNMP

Use the `SNMP` group to configure how your monitoring system connects to the gateway.
You can set the SNMP port, read and write community strings, device description, system object ID, and SNMPv3 user and security settings.

![Device Console Settings, SNMP basic settings](https://raw.githubusercontent.com/imcase/Im_snmp_modbus_gateway_assets/refs/heads/main/assets/snmp_modbus_gateway_webconsole_settings_snmp_basic_settings.png)

> **NOTE:**
> - **Read:** Use SNMP monitoring software to read gateway values with `snmpget` or `snmpwalk`.
> - **Write:** Authorized SNMP clients can configure __ALL__ gateway SNMP objects with `snmpset`.
> - Before deploying the gateway, use credentials and security settings approved by your organization.

<br>

![Device Console Settings, SNMPv3 security settings](https://raw.githubusercontent.com/imcase/Im_snmp_modbus_gateway_assets/refs/heads/main/assets/snmp_modbus_gateway_webconsole_settings_snmp_security_settings.png)

<br>

#### 4.2.4.1 Scalar Instance Object-ID

Each mapped gateway value is available through a **full SNMP OID**.
Its composition is:

```text
<SysObjectID>.1.<OID_COLUMN>.0
```

> **NOTE:**
> - By default, `sysObjectID` is set to `1.3.6.1.4.1.20738`. It may be
>   extended with a child OID, for example `1.3.6.1.4.1.20738.45.46`.
> - The final `.0` is required when querying an individual value.
> - Each `OID_COLUMN` value must be **UNIQUE**.
> - To minimize mistakes, use the full OID returned by `snmpwalk`.

<br>

### 4.3 System

The `System` area provides two device-management actions: **Reboot Device**
and **Initialize Settings**.

#### 4.3.1 Reboot Device

Use **Reboot Device** to restart the device.

> Restart the device after changing `TABLE_DEVICES_INPUT` or Device Console
> Settings so the new configuration is applied.

#### 4.3.2 Initialize Settings

Use **Initialize Settings** to restore the device's configuration to its factory default settings.

> **MD-button:**
>
> You can also restore the default settings using the on-board **MD button**:
>
> - Press and hold the button for at least **5 seconds**, then release it.
> - Releasing the button _before_ 5 seconds have elapsed has _no effect_.

<br>

![Device Console, System area](https://raw.githubusercontent.com/imcase/Im_snmp_modbus_gateway_assets/refs/heads/main/assets/snmp_modbus_gateway_webconsole_settings_system.png)


<br><br>

---

## 5. Deployment Example

This example shows how to connect a Modbus device to the gateway, configure one
Modbus value for polling, and make that value available to an SNMP monitoring
system using `snmpwalk`.

It also includes a brief SNMP write example, showing how an authorized client
can use `snmpset` to write a Modbus value and update a gateway setting.

<br>

### 5.1 Connect the Modbus BP#02 to the Gateway

This example uses a Tibbo Modbus BP#02, an ambient-temperature and
relative-humidity sensor. The BP#02 is used only as an example: you can
connect another compatible Modbus device instead.

> **IMPORTANT:** Before replacing a Modbus sensor or changing its wiring,
> disconnect power from the gateway. Reconnect power only after the new sensor
> and all wiring have been connected correctly.

Connect the BP#02 to the TPP2W(G2) as shown below. Connect each wire to the
terminal with the matching label: `VCC` to `VCC`, `GND` to `GND`, `Tx` to
`Tx`, and `Rx` to `Rx`. When using a different Modbus device, follow its
documentation for the required power and communication connections.

![Connecting the BP#02 sensor to the TPP2W(G2)](https://raw.githubusercontent.com/imcase/Im_snmp_modbus_gateway_assets/refs/heads/main/assets/tpp2wg2_modbus.png)

Should you be interested, more Tibbo Modbus models are available [here](https://tibbo.com/store/rs485-modbus-sensors.html).

<br>

### 5.2 Connect the Network Cable and Power Adapter

Connect an Ethernet cable from the gateway's RJ45 port to your local network.
Then connect the supplied power adapter to the gateway. The gateway starts when
power is connected. Allow up to 20 seconds for the device to fully boot and
become ready for use.

![Connecting the Ethernet cable and power adapter to the TPP2W(G2)](https://raw.githubusercontent.com/imcase/Im_snmp_modbus_gateway_assets/refs/heads/main/assets/tpp2wg2_ethernet_power.png)

### 5.3 Connect to the Device Console

Using your preferred web browser, navigate to [http://192.168.1.40](http://192.168.1.40). The **Home** page should be displayed.

![Device Console, Home area](https://raw.githubusercontent.com/imcase/Im_snmp_modbus_gateway_assets/refs/heads/main/assets/snmp_modbus_gateway_webconsole_home.png)

<br>

### 5.4 Configure Settings

Open **Settings** in the Device Console. Review the network, SNMP, and polling
settings required for your installation. Restart the gateway after making any
changes so that the new settings are applied.

> **NOTE:** For an overview of the available settings, see [Section 4.2, Settings](#42-settings).

#### 5.4.1 Polling Interval

In the **Config** group, set the **Polling Interval** to the value that suits
your application. This setting determines how often the gateway reads values
from the connected Modbus device.

#### 5.4.2 Network

In the **Network** group, choose the network method required by your
installation:

- Enable **DHCP** if your network automatically assigns IP addresses.
- Leave **DHCP** disabled to use a static IP address, then enter the IP address,
  netmask, and gateway supplied by your network administrator.

> **NOTE:** This guide uses `192.168.1.40` throughout the example.


After changing the network settings and restarting the gateway, reconnect to
the Device Console using its new IP address.

#### 5.4.3 SNMP

In the **SNMP** group, configure the SNMP port and access credentials required
by your monitoring system. Configure either SNMPv2c community strings or SNMPv3
user and security settings, as required by your organization.

Enable write access only when it is required for approved control points. Use
credentials and security settings approved by your organization.

> **NOTE:** This guide uses the default SNMP settings throughout the example.

<br>

### 5.5 Configure Modbus Mapping in `TABLE_DEVICES_INPUT`

Open **Database** in the Device Console. In `TABLE_DEVICES_INPUT`, add a
mapping for each Modbus value that you want to make available through SNMP.

> **NOTE:**
> - Enter the `DEVICE_ID`, `REG_ADDR`, and `FUNC_CODE` exactly as specified in
>   the connected field device's Modbus register map or datasheet.
> - Each `OID_COLUMN` value must be **UNIQUE**. For details, see
>   [Section 4.2.4.1, Scalar Instance Object-ID](#4241-scalar-instance-object-id).
> - `OID_COLUMN = 207` is used later in this guide to demonstrate **Write Data
>   to Modbus**.

<br>

![Device Console Database, TABLE_DEVICES_INPUT](https://raw.githubusercontent.com/imcase/Im_snmp_modbus_gateway_assets/refs/heads/main/assets/snmp_modbus_gateway_webconsole_database_table_devics_input.png)



<br>

### 5.6 Restart Device

Restart the device so that the new configuration is applied.


<br>

### 5.7 Verify Modbus Polled Data via the Device Console



<br>

### 5.8 Verify Modbus Polled Data via SNMP



<br>

### 5.9 Write Data to Modbus



<br>

### 5.10 Change Settings via SNMP




<br><br>

--- 

## 6. Commissioning checklist

1. Record the field device's Modbus register map, communication parameters,
   data representation, and expected engineering values.
2. Install and verify the RS-485 hardware and field wiring.
3. Configure Ethernet addressing for the local network and limit management
   access to authorized hosts.
4. Create the Modbus-to-SNMP mappings and restart the gateway if required.
5. Prove the southbound path first: confirm that the gateway successfully
   polls the expected raw Modbus value.
6. From an SNMP manager, walk the private enterprise branch and use the exact
   returned scalar OID for an SNMP GET.
7. Change the physical input or simulated register, wait for a successful
   Modbus poll, and verify that the SNMP value changes accordingly.
8. Where a write mapping is deliberately enabled, test it in a safe bench
   setup and verify the resulting field-device register separately.

<br><br>

---

## 7. Troubleshooting

| Symptom | Checks |
| --- | --- |
| SNMP responds but value is stale or zero | Verify the Modbus poll result, unit ID, function code, register addressing, serial settings, scaling, and byte/word order. An SNMP response alone does not prove the field link works. |
| SNMP GET returns `No Such Object` or `No Such Instance` | Walk the configured enterprise branch and query the full scalar OID, including its `.0` instance suffix. |
| No Modbus response | Check RS-485 polarity, termination/biasing, serial parameters, selected Tibbit/socket, unit ID, and whether the target actually supports the requested function/register. |
| SNMP SET fails | Confirm that the object is configured read/write, the caller is authorized, the write mapping exists, and the field device accepts the function/value. |
| Value has the wrong magnitude | Check signedness, scale factor, register count, and byte/word order. |

