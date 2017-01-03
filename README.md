# redirector
Manage propietary iKVM functionality from command line.

This is a fork of Anton Kachalov (mouse@) redirector project wich was initially created for internal needs in Yandex LLC: different and cheap servers without proper BMC management tools often with flacky UI only.

In case of our hardware there was a need to have easy way to get a iKVM console (do not differ with SOL) and most of the time process looks as a quest: you need to open BMC UI, confirm all security warnings, sometimes even downgrade a browser, then click n-times to get a right menu entry, then download java applet and click again to confirm OS security enforcements and at last all java questions. Sounds easy, huh?

But with redirector you need only working java and some common unix tools like GNU sed, jq and GNU getopt (in case of MacOS). Then just run one CLI command and script will do all magic for you:

 * login into BMC UI
 * find the right menu entry and fill all options
 * download the applet
 * invoke java with needed parameters
 * do a garbage collection after work

script itlsef able to detect variety of BMC hardware so you just need to provide a correct BMC ip and login credentials.

## System requirements
* GNU sed
* GNU getopt
* jq
* ipmitool
* curl

## Use cases

### iKVM

Example session run:
```
$ ./redirector -U Administrator -P <BMC password> -H <some BMC ip> kvm

Java HotSpot(TM) 64-Bit Server VM warning: ignoring option PermSize=32M; support was removed in 8.0

Java HotSpot(TM) 64-Bit Server VM warning: ignoring option MaxPermSize=32M; support was removed in 8.0

objc[49880]: Class JavaLaunchHelper is implemented in both /Library/Internet Plug-Ins/JavaAppletPlugin.plugin/Contents/Home/bin/java and /Library/Internet Plug-Ins/JavaAppletPlugin.plugin/Contents/Home/lib/jli/./libjli.dylib. One of the two will be used. Which one is undefined.
```

### BMC info aka 'mc info'

As you see, the applet fully functional and usable outside of BMC UI.

Another example with ipmitool-like 'mc info' for Supermicro HW:
```
$ ./redirector -U Administrator -P <BMC password> -H <some BMC ip> mc info
BMC Firmware Revision     : 2.29
BMC Firmware Tag          : BL_SUPERMICRO_X7SB3_2015-11-13_B
BMC Build Time            : 11/13/2015
BMC Kernel Version        : 2.6.28.9
Platform BIOS Version     : 2.0
Platform BIOS Build Time  : 12/17/2015
Platform CPLD Version     : 01.a1.00
Platform EC Version       : 00.00
Manufacturer ID           : 10876
Manufacturer Name         : Supermicro
Product ID                : 2175 (0x087f)
Product Name              : Unknown (0x87f)
```

### Firmware update

And the killer feature - ability to do BMC/BIOS upgrades on Supermicro:
```
$ ./redirector -U Administrator -P XXXXXX -H XXXXXX lan set biosup ~/Downloads/SM_BIOS/X10DRFG5.C17/UEFI/X10DRFG5.C17

Preparing BMC to flash [ygtaffhrjwdxpksb]...done

Locking firmware for update...done

Uploading image...done

Going to replace -> Old="BIOS Date: <some outdated BIOS>"  New="BIOS Date: 12/17/2015 Rev 2.0"...

Setting flash options...done

Flashing...

Completed 100%

Flash complete. Finalizing things...

Unlocking UI...done

Rebooting the system...

Please wait up to 5min to complete boot cycle
```
But script itself is not limited to Supermicro, it supports iLO (both 3 and 4) and iDRAC at some level (kvm/BMC reboot/mc info).
