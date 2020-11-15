# ASUS FX504 Hackintosh Opencore

## Opencore notes

Configurations here are considered experimental. Use at your own risk.

Tested version: 0.6.3 [OpenCorePkg](https://github.com/acidanthera/OpenCorePkg/releases/tag/0.6.3)

## Patch DSDT

### Sleep and Wake

1. Applied static patching of USB \_PRW 0x6D (instant wake) for Skylake
2. Patch \_PRW method for XDCI device as follows:

   ```bash
   Device (XDCI)
   {
       ...
       Method (_PRW, 0, NotSerialized)  // _PRW: Power Resources for Wake
       {
           Return (Package (0x02)
           {
               0x6D,
               Zero
           })
       }
       ...
   }

   ```

3. Remove \_PRW method from CNVW device

### Blacklight Control

Brightness adjustment keys working by modifying:

```bash
Scope (_SB.PCI0.LPCB.EC0) {
     ...
     Method (_Q11, 0, NotSerialized)  // _Qxx: EC Query
     {
         Notify (PS2K, 0x0405) // Brightness down
     }
     Method (_Q12, 0, NotSerialized)  // _Qxx: EC Query
     {
         Notify (PS2K, 0x0406) // Brightness up
     }
     ...
}
```

### I2C ELAN1200 Precision TouchPad

1. DSDT patch: [Windows] Windows 10 Patch
2. Patch TPD0 inside as follows:

   ```bash
   Device (TPD0)
   {
       ...

       Name (SBFG, ResourceTemplate ()
       {
           GpioInt (Level, ActiveLow, ExclusiveAndWake, PullDefault, 0x0000,
               "\\_SB.PCI0.GPI0", 0x00, ResourceConsumer, ,
               )
               {   // Pin list
                   0x0000
               }
       })

       ...

       Method (_CRS, 0, NotSerialized)  // _CRS: Current Resource Settings
       {
           Return (ConcatenateResTemplate (SBFB, SBFG)) // ** MODIFIED **
       }
   ```

## Configurations

### ACPI

#### **Add**

- DSDT.aml (Patch one)
- SSDT-AWAC.aml
- SSDT-EC-USBX-LAPTOP.aml (Fixes both the embedded controller and USB power)
- SSDT-PLUG.aml (Allows for native CPU power management )
- SSDT-PMC.aml (For shutdown and restart)
- SSDT-PNLF-CFL.aml (Disable GPU)

#### **Patch**

- Change \_OSI to XOSI
- Rename HDAS to HDEF (Audio)

#### **Quirks**

All no

### Booter

#### **MmioWhitelist**

Empty

#### **Quirks**

- AvoidRuntimeDefrag: YES
- DevirtualiseMmio: YES
- EnableWriteUnprotector: NO
- ProtectUefiServices: NO
- RebuildAppleMemoryMap: YES
- SetupVirtualMap: YES
- SyncRuntimePermissions: YES

### DeviceProperties

#### **Add**

- PciRoot(0x0)/Pci(0x2,0x0)
  | Keys\* | Value | Type |
  |--------------------------|:--------:|-----:|
  | AAPL,ig-platform-id | 00009B3E | DATA |
  | framebuffer-patch-enable | 01000000 | DATA |
  | framebuffer-stolenmem | 00003001 | DATA |

#### **Delete**

Empty

### Kernel

#### **Add**

- Lilu.kext
- VirtualSMC.kext
- WhataverGreen.kext
- AppleALC.kext
- RealtekRTL8111.kext
- USBUnjectAll.kext
- VoodooPS2Controller.kext (disable VoodooInput: panic)
- SMCBatteryManager.kext
- SMCProcessor.kext
- VoodooI2C.kext
- VoodooI2CHID.kext
- itlwm.kext (wifi working with Heliport, [OpenIntelWireless/HeliPort](https://github.com/OpenIntelWireless/HeliPort))
- CodecCommander.kext (for fix headphone audio, [Audio Distortion when using Headphones on Laptops](https://www.elitemacx86.com/threads/fix-audio-distortion-when-using-headphones-on-laptops.185/))
- CPUFriend.kext
- CPUFriendDataProvider.kext
- NoTouchID.kext
- SATA-300-series-unsupported.kext
- SMCLightSensor.kext (really needed)
- SMCSuperIO.kext
- XHCI-unsupported.kext

#### **Emulate**

- CpuidMask: Leave this blank
- CpuidData: Leave this blank

#### **Force**

Empty

#### **Block**

Empty

#### **Patch**

Empty

#### **Quirks**

- AppleCpuPmCfgLock: NO
- AppleXcpmCfgLock: YES
- CustomSMBIOSGuid: NO
- DisableIoMapper: YES
- DisableLinkeditJettison: YES
- DisableRtcChecksum: NO
- ExtendBTFeatureFlags NO
- LapicKernelPanic: NO
- LegacyCommpage: NO
- PanicNoKextDump: YES
- PowerTimeoutKernelPanic: YES
- XhciPortLimit: YES

#### **Scheme**

- FuzzyMatch: True
- KernelArch: x86_64
- KernelCache: Auto

### Misc

#### **Boot**

Settings for boot screen (Leave everything as default).

#### **Debug**

- AppleDebug: YES
- ApplePanic: YES
- DisableWatchDog: YES
- DisplayLevel: 2147483650
- SerialInit: NO
- SysReport: NO
- Target: 67

#### **Security**

- AllowNvramReset: YES
- AllowSetDefault: YES
- ApECID: 0
- AuthRestart: NO
- BootProtect: Bootstrap
- DmgLoading: Signed
- ExposeSensitiveData: 6
- Vault: Optional
- ScanPolicy: 0
- SecureBootModel: Default

### NVRAM

#### **Add**

7C436110-AB2A-4BBB-A880-FE41995C9F82

- boot-args: -v debug=0x100 keepsyms=1 alcid=3 -wegnoegpu
- csr-active-config: 00000000
- run-efi-updater: No
- prev-lang:kbd: en-US:0

#### **Delete**

- WhiteFlash: YES

### PlatformInfo

SMBIOS: MacBookPro15,2

### UEFI

ConnectDrivers: YES

#### **Drivers**

- HfsPlus.efi
- OpenRuntime.efi

#### **APFS**

Settings related to the APFS driver, leave everything here as default.

#### **Quirks**

- DeduplicateBootOrder: YES
- RequestBootVarRouting: YES
- UnblockFsConnect: NO

## For more information about config, follow this [ OpenCore Install Guide - coffee lake](https://dortania.github.io/OpenCore-Install-Guide/config.plist/coffee-lake.html#uefi)
