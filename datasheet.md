# SNMP–Modbus Gateway

**Firmware:** `SNMP_Modbus_Gateway` v1.0.2
**Platform:** Tibbo TPP2W(G2) in a TPB2L(G2) enclosure

> ## Configuration storage and restart behavior
>
> Only the user-configured polling definitions in `TABLE_DEVICES_INPUT` are
> stored in flash and survive a restart. `TABLE_DEVICES_OUTPUT` and
> `TABLE_SETTINGS` are RAM tables: their contents do not persist across a
> restart.
>
> **After any change to `TABLE_DEVICES_INPUT`, or to Settings through the Web
> Console or SNMP, restart the device.** Until it restarts, the change is
> stored but not applied. After restart, verify that the persistent input
> mappings have loaded, the runtime tables have been rebuilt, and the expected
> polling result and SNMP object are present.

---

## 1. Overview

The SNMP–Modbus Gateway lets **SNMP monitoring software read** values from
**Modbus equipment** over **Ethernet**. It is used when equipment such as meters, sensors,
controllers, or other industrial devices communicates with Modbus, while the
customer's monitoring system uses SNMP.

The gateway regularly reads the Modbus values selected by the user and makes
them available as SNMP values. A monitoring system can then display the values,
record them, or use them to raise alarms.

Where specifically configured and authorized, the gateway can also accept an
**SNMP write** command and send the corresponding Modbus write to the field device.
This **remote-control function** should be enabled only for approved control points.

![Modbus field device to SNMP monitoring-system gateway flow](https://raw.githubusercontent.com/imcase/Im_snmp_modbus_gateway_assets/refs/heads/main/assets/snmp_modbus_gateway_flow.png)

The user configures which Modbus values to read, and how each value appears to
SNMP software. This means the same gateway can be used with different types of
Modbus equipment instead of being limited to a single sensor or meter.

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

## 3. Network configuration and services

| Item | Setting |
| --- | --- |
| Ethernet | static IP/netmask/gateway, or DHCP  |
| SNMP | SNMPv2c and SNMPv3 agent, port 161, configurable read/write communities and SNMPv3 security credentials |
| Time sync | SNTP against 216.239.35.0 (time.google.com), applied with the configured time zone |
| Web console | HTTP, port 80, 3 concurrent connections |

> **Default IP address.** Out of the box the device uses the static address
> `192.168.1.40`. This is a placeholder so the device can be reached on first
> power-up; you will most likely want to change it. On the Settings page,
> Ethernet tab, either enable **DHCP** so the device obtains an address from your
> network, or enter a static IP, netmask and gateway of your own choosing.
> Restart the device for the new address to take effect.

The **network** must allow communication between the gateway and its configured
SNTP servers, as well as between the gateway and authorized SNMP managers.

![Ethernet settings](https://raw.githubusercontent.com/imcase/Im_snmp_modbus_gateway_assets/refs/heads/main/assets/snmp_modbus_gateway_ethernet_config.png)

## 4. Web Console

### Database
For every value you want to monitor, configure the gateway with the relevant
Modbus device and register. The gateway reads that register at the configured
polling interval (5 seconds by default) and makes the latest value available to
your SNMP monitoring system.

| Table | Storage | Purpose | Restart behavior |
| --- | --- | --- | --- |
| `TABLE_DEVICES_INPUT` | Flash | User-defined Modbus polling and SNMP mappings | Retained across restart |
| `TABLE_DEVICES_OUTPUT` | RAM | Current output of polling based on `TABLE_DEVICES_INPUT` | Cleared and rebuilt as polling resumes |
| `TABLE_SETTINGS` | RAM | Predefined Web Console settings used by the application | Recreated from the application at startup |

> Use `TABLE_DEVICES_INPUT` to define which Modbus values the gateway monitors.
> `TABLE_DEVICES_OUTPUT` shows the current polling results; it is a live view,
> not a saved history, and is rebuilt after restart. `TABLE_SETTINGS` contains
> the settings used by the Web Console.

> **After any change to `TABLE_DEVICES_INPUT`, or to `Settings` through the Web
> Console or SNMP, restart the device.** The change is retained immediately, but
> the polling engine does not apply it until the device restarts.



**CONTINE FROM HERE!**



| Mapping property | Meaning |
| --- | --- |
| Name | Human-readable name for the mapping |
| Modbus unit ID | Address of the target Modbus server/device (1–255) |
| Function code | Modbus operation used for the mapping; confirm the device supports it |
| Register address | Protocol address expected by the configured Modbus client (0–65535) |
| SNMP object column | Object identifier column used below the configured private enterprise branch |
| Value type | SNMP representation of the mapped value, such as integer or string |
| Access | Read-only for telemetry, or read/write only for approved control points |

### Register addressing and value interpretation

Do not assume a datasheet reference number is the address used on the wire.
For example, documentation may call a holding register `40001`, while the
Modbus protocol address is `0`. Confirm the address format expected by the
gateway and the connected device.

For multi-register, signed, floating-point, or scaled measurements, define the
byte order, word order, signedness, and scale in the mapping record. A value
that is numerically plausible but decoded in the wrong order is not valid.

### Write path

For a configured read/write object, an SNMP SET is validated and matched to a
write mapping. The gateway sends the configured Modbus write function and
reports success, failure, or a reboot-required state to the management path.
The application includes retry handling for Modbus writes; communication
failures must still be investigated at the RS-485/device layer.

Remote writes can affect physical equipment. Protect them with least-privilege
SNMP credentials, restricted network access, and a documented rollback plan.

## 5. SNMP object addressing

The project uses a private enterprise branch rooted at:

```text
1.3.6.1.4.1.20738
```

For scalar runtime objects, the effective object form is:

```text
<sysObjectID>.1.<column>.0
```

The final `.0` is the scalar instance identifier and is required for an SNMP
GET of a scalar object. The application configuration may display a shorter
base/column value; use the complete registered scalar OID returned by an SNMP
walk or supplied MIB when querying the device.

Example bench query (replace the address, community, and OID with the deployed
configuration):

```bash
snmpget -v2c -c <bench-community> -On <gateway-ip> \
  1.3.6.1.4.1.20738.1.<column>.0
```

## 6. RS-485 installation

Power down equipment before connecting the field bus. Wire the RS-485 pair to
the matching terminals for the fitted Tibbit/socket configuration.

```text
Modbus field device                 TPP2W(G2) with Tibbit #05

RS-485 A / D+     ----------------  matching A / D+ terminal
RS-485 B / D-     ----------------  matching B / D- terminal
Signal reference  ----------------  reference terminal, if required
```

- Vendors do not label A/B consistently. Verify the electrical D+/D- mapping
  against both devices before applying power.
- Use suitable twisted-pair cable. Terminate only the physical ends of a bus
  when required by the cable topology and equipment documentation.
- Apply biasing in one appropriate location. Follow the equipment guidance for
  shielding, isolation, and signal reference; do not defeat isolation by
  blindly joining grounds.
- Configure matching baud rate, data bits, parity, stop bits, and unit ID on
  both the gateway and field device.

Example bench parameters only:

```text
Baud rate: 9600
Data bits: 8
Parity:    None
Stop bits: 1
Unit ID:   1
```

## 7. Commissioning checklist

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

## 8. Troubleshooting

| Symptom | Checks |
| --- | --- |
| SNMP responds but value is stale or zero | Verify the Modbus poll result, unit ID, function code, register addressing, serial settings, scaling, and byte/word order. An SNMP response alone does not prove the field link works. |
| SNMP GET returns `No Such Object` or `No Such Instance` | Walk the configured enterprise branch and query the full scalar OID, including its `.0` instance suffix. |
| No Modbus response | Check RS-485 polarity, termination/biasing, serial parameters, selected Tibbit/socket, unit ID, and whether the target actually supports the requested function/register. |
| SNMP SET fails | Confirm that the object is configured read/write, the caller is authorized, the write mapping exists, and the field device accepts the function/value. |
| Value has the wrong magnitude | Check signedness, scale factor, register count, and byte/word order. |

## 9. Deployment record

Keep this record with each installed gateway:

| Item | Installed value |
| --- | --- |
| Gateway serial / location | |
| Firmware version | |
| Ethernet address / VLAN | |
| SNMP version and credential reference | |
| `sysObjectID` / enterprise OID branch | |
| Tibbit #05 socket and terminal assignment | |
| RS-485 serial settings | |
| Modbus device/unit IDs and register map revision | |
| OID mapping and scale/byte-order definition | |
| Commissioning date and test result | |
