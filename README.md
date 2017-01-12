# Management Complex Firmware

The Management Complex (MC) is a key component of DPAA2 (Data Path Acceleration Architecture, second version).
MC runs an NXP-supplied firmware image that abstracts and simplifies the allocation and configuration of the other hardware elements using DPAA2 “objects”.
These objects provide familiar services such as the core of network interfaces, switching services, access to accelerators, etc.


## Boards personalities

### LS2080 personalities

LS2085A, LS2045A, LS2080A, LS2040A

### LS2088 personalities

LS2088A, LS2048A, LS2084A, LS2044A

### LS1088 personalities

LS1088A, LS1048A, LS1084A, LS1044A


## Glossary of terms

- MC: Management Complex Firmware, an itb format file.
- DPC: Data Path Control (DPC) file, a dtb format file.
- DPL: Data Path Layout (DPL) file, a dtb format file.


## Deploying MC, DPC and DPL binaries on boards

This section describes the steps to program the MC, DPC and DPL binaries to NOR or QSPI flash on LS2080ARDB/LS2088ARDB/LS1088ARDB devices.
All commands are executed from bank0 (LS2080ARDB/LS2088ARDB devices) or QSPI flash0 (LS1088ARDB device).

### LS2080ARDB/LS2088ARDB

The names of the images files used are for LS2088 personalities. For LS2080 personalities please use: mc_10.0.4_ls2080a.itb, dpc_ls2080a_rdb.dtb, dpl_ls2080a_rdb.dtb.

#### Deployment to bank4 via USB

The following U-Boot commands read images from USB device, erase old images and program new images to the NOR flash address of the alternate bank.

```
=> usb start
=> fatload usb 0:1 0x80000000 mc_10.0.4_ls2088a.itb && erase 0x584300000 +0x300000 && cp.b 0x80000000 0x584300000 0x300000
=> fatload usb 0:1 0x80000000 dpc_ls2088a_rdb.dtb && erase 0x584800000 +$filesize && cp.b 0x80000000 0x584800000 $filesize
=> fatload usb 0:1 0x80000000 dpl_ls2088a_rdb.dtb && erase 0x584700000 +$filesize && cp.b 0x80000000 0x584700000 $filesize
```

#### Deployment to bank4 via TFTP

This section assumes that existing MC and DPC images are already deployed into NOR flash, MC boots up successfully and networking is operational.
U-boot lists the available network interfaces using names like "DPMAC1@xgmii [PRIME], DPMAC2@xgmii, DPMAC3@xgmii, DPMAC4@xgmii, DPMAC5@xgmii".
Use the following commands to test U-boot networking (replace the name of the network interface as needed):

```
=> setenv ethact DPMAC5@xgmii
=> setenv ipaddr 192.168.1.100
=> setenv netmask 255.255.255.0
=> setenv serverip 192.168.1.254
=> ping $serverip
Using DPMAC5@xgmii device
host 192.168.1.254 is alive
```

The following U-Boot commands TFTP images, erase old images and program new images to the NOR flash addresses of the alternate bank.
The paths to the files are relative to the TFTP server files location.

```
=> tftp 0x80000000 mc_10.0.4_ls2088a.itb && erase 0x584300000 +0x300000 && cp.b 0x80000000 0x584300000 0x300000
=> tftp 0x80000000 dpc_ls2088a_rdb.dtb && erase 0x584800000 +$filesize && cp.b 0x80000000 0x584800000 $filesize
=> tftp 0x80000000 dpl_ls2088a_rdb.dtb && erase 0x584700000 +$filesize && cp.b 0x80000000 0x584700000 $filesize
```

#### Starting MC and loading DPL

Boot the board from alternate bank (bank4):

```
=> qixis_reset altbank
```

Start MC using the following command:

```
=> fsl_mc start mc 0x580300000 0x580800000
```

Alternatively U-boot can be configured to automatically start MC at boot time by setting the 'mcinitcmd' environment varable:

```
=> pri mcinitcmd
mcinitcmd=fsl_mc start mc 0x580300000 0x580800000
```

If the firmware has successfully started a message like the following will be displayed in U-boot console:

```
fsl-mc: Booting Management Complex ... SUCCESS
fsl-mc: Management Complex booted (version: 10.0.4, boot status: 0x1)
```

The DPL can be loaded after MC has successfully started:

```
=> fsl_mc apply dpl 0x580700000
```

Alternatively DPL loading command can be appended to 'mcinitcmd' environment variable:

```
=> pri mcinitcmd
mcinitcmd=fsl_mc start mc 0x580300000 0x580800000;fsl_mc apply dpl 0x580700000
```

Successful loading of DPL is indicated in U-boot console with a message like the following:

```
fsl-mc: Deploying data path layout ... SUCCESS
```


### LS1088ARDB

Before powering on the board make sure SW2 bits 6:8 on the board are off-off-off to boot from QSPI flash0.

#### Deployment to QSPI flash1 via USB

The following U-Boot commands read images from USB device, erase old images and program new images to the QSPI flash1 address.
"i2c mw 66 50 20;sf probe 0:0" is used to move flash1 on QSPI controller CS0.

```
=> i2c mw 66 50 20;sf probe 0:0
=> usb start
=> fatload usb 0:1 0x80000000 mc_10.0.4_ls1088a.itb && sf erase 0x000300000 0x300000 && sf write 0x80000000 0x000300000 0x300000
=> fatload usb 0:1 0x80000000 dpc_ls1088a_rdb.dtb && sf erase 0x000800000 0x100000 && sf write 0x80000000 0x000800000 $filesize
=> fatload usb 0:1 0x80000000 dpl_ls1088a_rdb.dtb && sf erase 0x000700000 0x100000 && sf write 0x80000000 0x000700000 $filesize
```

#### Deployment to QSPI flash1 via TFTP

This section assumes that existing MC and DPC images are already deployed into QSPI flash, MC boots up successfully and networking is operational.
U-boot lists the available network interfaces using names like "DPMAC1@xgmii [PRIME], DPMAC2@xgmii".
Use the following commands to test U-boot networking (replace the name of the network interface as needed):

```
=> setenv ethact DPMAC2@xgmii
=> setenv ipaddr 192.168.1.100
=> setenv netmask 255.255.255.0
=> setenv serverip 192.168.1.254
=> ping $serverip
Using DPMAC2@xgmii device
host 192.168.1.254 is alive
```

The following U-Boot commands TFTP images, erase old images and program new images to the QSPI flash1 address.
"i2c mw 66 50 20;sf probe 0:0;" is used to move flash1 on QSPI controller CS0.
The paths to the files are relative to the TFTP server files location.

```
=> i2c mw 66 50 20;sf probe 0:0
=> tftp 0x80000000 mc_10.0.4_ls1088a.itb && sf erase 0x000300000 0x300000 && sf write 0x80000000 0x000300000 0x300000
=> tftp 0x80000000 dpc_ls1088a_rdb.dtb && sf erase 0x000800000 0x100000 && sf write 0x80000000 0x000800000 $filesize
=> tftp 0x80000000 dpl_ls1088a_rdb.dtb && sf erase 0x000700000 0x100000 && sf write 0x80000000 0x000700000 $filesize
```

#### Starting MC and loading the DPL

After writing images to QSPI flash1, power down the board, set SW2 bits 6:8 to off-off-on to boot from flash1, then power on the board.
Switching from flash0 to flash1 can also be done while the board is booting from flash0 using the following commands:

```
=> i2c mw 66 50 20;sf probe 0:0
=> i2c mw 66 10 10;i2c mw 66 10 21
```

Start MC using the following command:

```
=> sf probe 0:0;sf read 0x81000000 0x300000 0x100000;sf read 0x82000000 0x800000 0x100000;fsl_mc start mc 0x81000000 0x82000000
```

Alternatively U-boot can be configured to automatically start MC at boot time by setting the 'mcinitcmd' environment variable:

```
=> pri mcinitcmd
mcinitcmd=sf probe 0:0;sf read 0x81000000 0x300000 0x100000;sf read 0x82000000 0x800000 0x100000;fsl_mc start mc 0x81000000 0x82000000
```

The DPL can be loaded after MC has successfully started:

```
=> sf probe 0:0;sf read 0x83000000 0x700000 0x100000;fsl_mc apply dpl 0x83000000
```

Alternatively DPL loading command can be appended to 'mcinitcmd' environment variable:

```
=> pri mcinitcmd
mcinitcmd=sf probe 0:0;sf read 0x81000000 0x300000 0x100000;sf read 0x82000000 0x800000 0x100000;sf read 0x83000000 0x700000 0x100000;fsl_mc start mc 0x81000000 0x82000000;fsl_mc apply dpl 0x83000000
```

