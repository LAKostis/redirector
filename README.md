# redirector
Proprietary BMC functions via unified CLI.

This is a fork of Anton Kachalov (mouse@) [redirector
project](https://github.com/ya-mouse/redirector) wich was initially created for
internal needs in Yandex LLC: different and cheap servers without proper BMC
management tools often with flacky UI only.

In case of our hardware there was a need to have easy way to get a iKVM console
(do not differ with SOL) and most of the time process looks as a quest:

* you need to open BMC UI, confirm all security warnings, (might even downgrade
  a browser)
* click n-times to get a right menu entry
* then download java applet
* click again n-times to confirm OS security enforcements
* click n-times to override all java security enforcements

Sounds easy, huh?

Or let's say you need to change some options by ticking/unticking them in UI
and you need this for 1000+ servers? And there's no way to do that via
ipmitool?

But with redirector you need only working java and some common unix tools like
GNU sed, jq and GNU getopt (in case of MacOS). Then just run one CLI command
and script will do all magic for you:

 * login into BMC UI
 * find the right menu entry and fill all options
 * download the applet
 * invoke java with needed parameters
 * do a garbage collection after work

Script able to detect variety of BMC hardware so you just need to
provide a correct BMC ip and login credentials.

## System requirements
* Bash shell (due bashisms inside)
* GNU sed
* GNU getopt
* [jq](https://stedolan.github.io/jq) (optional)
* ipmitool
* curl
* java runtime (optional)

## Supported OSes
* MacOS (with homebrew installed)
* Linux/\*NIX

## BMC supported
* ATEN (Supermicro)
* iDRAC (DELL)
* iLO 3/4 (HP), iLO 2 unsupported
* HP (non-iLO variants)
* AMI (HP Cloud Compute servers)
* HUAWEI

Most supported BMC variants are ATEN and iDRAC/iLO, AMI (more or less) and the
rest is provided as-is, so I can't guarantee that it even work. It doesn't mean
there're abadoned, they just need testers/maintainers.

## Use cases

### iKVM

Example session run:
```
$ ./redirector -U ADMIN -P <BMC password> -H <some BMC ip> kvm

Java HotSpot(TM) 64-Bit Server VM warning: ignoring option PermSize=32M; support was removed in 8.0

Java HotSpot(TM) 64-Bit Server VM warning: ignoring option MaxPermSize=32M; support was removed in 8.0

objc[49880]: Class JavaLaunchHelper is implemented in both /Library/Internet Plug-Ins/JavaAppletPlugin.plugin/Contents/Home/bin/java and /Library/Internet Plug-Ins/JavaAppletPlugin.plugin/Contents/Home/lib/jli/./libjli.dylib. One of the two will be used. Which one is undefined.
```

### BMC info aka 'mc info'

As you see, the applet fully functional and usable outside of BMC UI.

Another example with ipmitool-like 'mc info' for Supermicro HW:
```
$ ./redirector -U ADMIN -P <BMC password> -H <some BMC ip> mc info
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
$ ./redirector -U ADMIN -P XXXXXX -H XXXXXX lan set biosup ~/Downloads/SM_BIOS/X10DRFG5.C17/UEFI/X10DRFG5.C17

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
