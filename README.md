# TJA110X Linux Phy Driver Release v0.5.1 - User Notes

## Changelog
v0.1: (19.01.2017)
- Initial release; support for TJA1100 and TJA1102

v0.2: (28.06.2017)
- Add support for TJA1100 PHY with phyID 0x0180DC60, as used on the SJA1105 Application Board
- Add a new sysfs node in the configuration folder, that allows manipulation of the SNR warning limit, called snr_wlimit_cfg

v0.3: (12.10.2017)
- Support for TJA1101
- Set state of phydev to PHY_CHANGELINK, and update the linkstate member if link goes up or down. This notifies a connected netdevice of a linkstate
- Add verbosity parameter to conditionally show debug output
- Add a no_poll module parameter, that disables polling if it is given a nonzero value
- The TJA1102_p1 driver is only loaded, if the macro CONFIG_TJA1102_FIX is defined
- If the macro NETDEV_NOTIFICATION_FIX is defined, a netdevice_notifier is registered, that starts/stops polling according to the netdev state.
 This fixes an issue, where mdio read errors would occur, while the netdev was down.
- Bug fixes

v0.4: (10.07.2018)
- Support for interrupts
- Added CONFIG_STANDALONE_PHY switch to enable PHYs not connected to a linux ethernet/switch interface

v0.5: (31.08.2018)
- Support for multiple Linux versions (see 'Supported Linux versions')
- Bug fixes

v0.5.1: (20.12.2018)
- Native support for PHY loopback mode for Linux v4.13.1 and above
- Correctly set the 'supported' property of phydev

---

## Table of Contents
1. [TJA110X loading](#TJA110X-loading)
2. [TJA110X configuration management](#TJA110X-configuration-management)
3. [TJA110X autonomous and managed mode](#TJA110X-autonomous-and-managed-mode)
4. [DTS Information](#DTS-Information)
5. [Compile time configuration](#Compile-time-configuration)
6. [Interrupts](#Interrupts)
7. [Supported Linux versions](#Supported-Linux-versions)
8. [Limitations and known issues](#Limitations-and-known-issues)

---

## TJA110X loading
- Load the kernel module: eg. "insmod tja110x.ko"
- Available module parameters are:
	- `managed_mode`: a nonzero value enables *managed mode* (see 3 for more information)
	- `no_poll`: (only if CONFIG_POLL is set): a nonzero value disables polling of the Interrupt Status Register
	- `verbosity`: set verbosity level of debug messages. Possible values are between 0 (none) and 5 (all)

## TJA110X configuration management
Configuration of the PHY is mainly done via sysFS nodes, that are located in: `/sys/bus/mdio_bus/devices/{devName}/configuration`. All of them can be read (R), some can additionally be written to (W) in order to apply some configuration.

- `cable_test`
	> R: execute a cable test and return the result

	> W: /

- `led_cfg`
	> R: return the current led mode, one of:
	> * `DISABLED`
	> * `LINKUP`
	> * `FRAMEREC`
	> * `SYMERR`
	> * `CRSSIG`}

	> W: set the led mode, with:
	> * 0=`DISABLED`
	> * 1=`LINKUP`
	> * 2=`FRAMEREC`
	> * 3=`SYMERR`
	> * 4=`CRSSIG`

- `link_status`
	> R: return the current link status, one of:
	> * `up`
	> * `down`

	> W: /

- `loopback_cfg`
	> R: return the current loopback mode, one of:
	> * `LOOPBACK_DISABLED`
	> * `INTERNAL_LOOPBACK`
	> * `EXTERNAL_LOOPBACK`
	> * `REMOTE_LOOPBACK`}

	> W: set the loopback mode, with:
	> * 0=`LOOPBACK_DISABLED`
	> * 1=`INTERNAL_LOOPBACK`
	> * 2=`EXTERNAL_LOOPBACK`
	> * 3=`REMOTE_LOOPBACK`

- `master_cfg`
	> R: return the current master/slave configuration, one of:
	> * `master`
	> * `slave`

	> W: set the master/slave configuration, with:
	> * 0=`slave`
	> * 1=`master`

- `power_cfg`
	> R: return the current power mode, one of:
	> * `POWER_MODE_NORMAL`
	> * `POWER_MODE_SLEEPREQUEST`
	> * `POWER_MODE_SLEEP`
	> * `POWER_MODE_SILENT`
	> * `POWER_MODE_STANDBY`
	> * `POWER_MODE_NOCHANGE`}

	> W: initiate a power mode transition, with:
	> * 0=`SUSPEND`
	> * 1=`RESUME`
	> * 2=`SLEEPREQUEST`
	> * 3=`WAKEUP`

- `test_mode`
	> R: return the current test mode, one of:
	> * `NO_TMODE`
	> * `TMODE1`
	> * `TMODE2`
	> * `TMODE3`
	> * `TMODE4`
	> * `TMODE5`
	> * `TMODE6`}

	> W: set the test mode, with:
	> * 0=`NO_TMODE`
	> * 1=`TMODE1`
	> * 2=`TMODE2`
	> * 3=`TMODE3`
	> * 4=`TMODE4`
	> * 5=`TMODE5`
	> * 6=`TMODE6`

- `wakeup_cfg`
	> R: get the current wakeup configuration, is equal to:
	> `fwdphyloc[STATUS], remwuphy[STATUS], locwuphy[STATUS], fwdphyrem[STATUS]`, where `STATUS` is either `on` or `off`

	> W: set the wakeup configuration: pass a hexadecimal number, the first four bits are interpreted as switches for the four configurable options
	> - TJA1102: all four options are configurable separately
	> - TJA1100: `FWDPHYLOC` MUST be off, `REMWUPHY` MUST be on, `LOCWUPHY` and `FWDPHYREM` are configurable  
	>	Thus possible configurations are:
	>	- BOTH enabled (then led MUST be off)
	>	- BOTH disabled (then led CAN be on)
	>	- all other configurations are invalid.
	>
	>	And therefore valid values to write to sysfs are:
	>	- `2` (LOCWUPHY and FWDPHYREM off)
	>	- `E` (LOCWUPHY and FWDPHYREM on)

- `snr_wlimit_cfg`
	> R: return the current snr warning limit, one of:
	> * `no fail limit`
	> * `CLASS_A`
	> * `CLASS_B`
	> * `CLASS_C`
	> * `CLASS_D`
	> * `CLASS_E`
	> * `CLASS_F`
	> * `CLASS_G`

	> W: set the loopback mode, with:
	> * 0=fail limit
	> * 1=CLASS_A
	> * 2=CLASS_B
	> * 3=CLASS_C
	> * 4=CLASS_D
	> * 5=CLASS_E
	> * 6=CLASS_F
	> * 7=CLASS_G

## TJA110X autonomous and managed mode
To choose between managed and autonomous mode, the kernel module parameter `managedMode` shall be used.
- if `managedMode` is given a nonzero value (eg. `insmod <MODULE_NAME> managedMode=1`), the Phy operates in *managed mode*
- if `managedMode` is set to zero (default value), the PHY operates in *autonomous mode*. In *autonomous mode*, the following features are not available:
	- cable test
	- loopback modes
	- test modes
	- power modes (ie. the sleep, wakeup, suspend and resume commands)

## DTS Information
- Autodetection of PHYs works, if the `ethernet` node does not have a `mdio` child-node with hardcoded `ethernet-phy` child-nodes. If present, only those hardcoded PHYs are available.
- The `reg` property of an `ethernet-phy` node determines the **phy address** used by the system.
- The `compatible` string of an `ethernet-phy` node determines the **phy id** (eg. the value of `/sys/bus/mdio_bus/devices/<DEVICE>/phy_id`).

## Compile time configuration
- `CONFIG_TJA1102_FIX`: If this macro is defined, the TJA1102_p1 driver is loaded. It needs to be ensured, that no other mdio device with id 0 is present
- `CONFIG_POLL`: See Section Interrupts for more information
- `CONFIG_STANDALONE_PHY`: (only meaningful if `CONFIG_POLL` is set)  
	**WARNING**: This is mainly intended for development/debugging purposes.
	If this macro is defined, some PHY functionality can be used without an attached netdevice.
	During probing, config_init is called manually to initialize the PHY and and a `netdevice_notifier` is registered, that starts/stops polling according to some netdev state.
	Using the fix requires the following macros to be set:
	- `MDIO_INTERFACE_NAME`: Name of the network interface that controls the mdio bus, to which the PHY(s) is/are connected to (for example "eth0")
	- `MII_BUS_NAME`: Name of the mdio bus, to which the PHY(s) is/are connected to

## Interrupts
The driver supports two methods for reading and reacting to interrupts:
- Polling (`CONFIG_POLL` is set):
  The driver regularly polls the PHY's Interrupt Status register (Period is determined by `POLL_PAUSE`), and handles any interrupts that may have occurred.

- Interrupts (`CONFIG_POLL` is not set):
  The PHY signals interrupts via the INT_N pin to a GPIO of the CPU. The driver expects a property called `irq-gpio` in the `ethernet-phy` node in the device tree,
  which determines the gpio pin to which INT_N is connected.
  The entry may look like
  > `irq-gpio = <&siul2 18 GPIO_ACTIVE_LOW>;`

  where `siul2` is the interrupt controller of the gpio and 18 is the number of the GPIO to be used. The `GPIO_ACTIVE_LOW` flag indicates, that the INT_N pin is active low.

## Supported Linux versions
The driver was tested with **Linux v4.1.26** and **Linux v4.14.34**.
However, it should work for every Linux version that is **v4.0** or newer.

## Limitations and known issues
- The TJA1100 disables the SMI Bus when entering sleep. Once entered, it can only be woken up by bus activity or via the WAKE pin

- MDIO Timeouts on interface down (only if `CONFIG_STANDALONE_PHY` is used, for example on shutdown)
	Error message: "fec XXXXXXX.ethernet eth0: MDIO read timeout" and "TJA1101 XXXXXXX.ethernet:06: read error: did_interrupt failed"
	Phys that are not attached to a netdevice will not be removed when the netdevice goes down, so polling will not be stopped.
	The ethernet controller deactivates the MDIO-Bus, which results in MDIO-timeouts for PHYs that are still running.
	Since v0.4, in case `CONFIG_STANDALONE_PHY` (previously called NETDEV_NOTIFICATION_FIX) is set, the driver listens for the NETDEV_GOING_DOWN event and manually stops NXP PHYs,
	however this fix requires knowledge of the MDIO-Bus name and the network interface name that controls it.

- If the **SJA1105** driver is used, a load order has to be respected:
	load **TJA110x** FIRST, THEN **SJA1105**. Reason: `phy_attach_direct()` (used in the **SJA1105** driver) checks if there is a driver loaded for the attaching PHY,
	if not it assumes the `genphy` driver and binds it. As a result, the **TJA110x** driver will not be bound once loaded.

- In case loopback mode is used, link status will not correctly be set to `NO_LINK` once loopback mode is disabled.
	Reason: the `LINK_STATUS_FAIL` flag in the Interrupt Status Register will not be set and thus no interrupt will be fired.

- Shared Interrupts:
	Between `33c133cc7598e60976a069344910d63e56cc4401` (linux v3.13-rc7) and `c3e70edd7c2eed6acd234627a6007627f5c76e8e` (linux v4.8-rc5)
	it is not possible to register multiple PHY interrupt handlers for a single IRQ in Linux, since the `IRQF_SHARED` flag is erroneously not set
	when calling `request_irq()` in `phy_start_interrupts()`.
	A kernel patch (i.e. commit `c3e70edd7c2eed6acd234627a6007627f5c76e8e` from Linux kernel mainline) needs to be applied in order to use interrupts
	with a kernel version that lies in between those commits.
	Alternatively, polling may be used.