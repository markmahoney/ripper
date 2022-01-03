# Ripper: a small collection of console CD game backup tools

I'm starting a small collection of command line tools for backing up CD-based console games on a Mac. You may find these handy? IDK.

## `mkbincue`

A thin wrapper script around `cdrdao` that creates a `bin`/`cue` fileset from a CD. Though less convenient than the `iso` format, `bin`/`cue` sets are necessary for ripping games with multiple tracks, including Redbook audio.

### Usage

Requires `cdrdao` and `toc2cue`:
```sh
brew install cdrdao toc2cue
```

Usage:
```sh
./mkbincue <CD device file> <output filename, no extension>
```

Example:
```sh
./mkbincue /dev/disk3 "/Volumes/Games/ROMs/Sega CD/Sonic CD"
```

You can find your drive's device file in the `/dev` directory by right-clicking the drive in `Disk Utility.app`, selecting "Get Info", and looking for "BSD device node", which should be something like "disk3", in which case the device file location would be `/dev/disk3`.

I've tested this script with TurboGrafx and Sega/Mega CD games, but it should also work with PSX games, and I guess maybe Neo Geo CD games?

### Caveats

**tl;dr**

This script currently assumes you only have one optical drive connected, so if that's not true, things may not work correctly. I want to fix this without making people copy very long strings, but I currently don't know how, so I just let `cdrdao` pick the first optical drive it finds and hope for the best.

**tl;reading anyway**

`cdrdao` on a Mac has a really weird way of specifying which CD device to use if you don't let it use the first drive it finds. If you run `cdrdao scanbus`, you'll see something like this:

```sh
$ diskutil unmountDisk /dev/disk2 && cdrdao scanbus 
Unmount of all volumes on disk2 was successful
Cdrdao version 1.2.4 - (C) Andreas Mueller <andreas@daneb.de>
IOService:/AppleACPIPlatformExpert/PCI0@0/AppleACPIPCI/XHC1@14/XHC1@14000000/HS01@14100000/MacBook Air SuperDrive@14100000/IOUSBHostInterface@0/IOUSBMassStorageInterfaceNub/IOUSBMassStorageDriverNub/com_apple_driver_AppleUSBODD/IOSCSILogicalUnitNub@0/com_apple_driver_AppleUSBODDType05/IODVDServices : HL-DT-ST, DVDRW  GX50N, RR06
```

This shows an `IOService` entry representing my single external SuperDrive DVD-RW. The extremely long `IOService:/.../IODVDServices` string is the value that `cdrdao --device` wants us to pass in to specify which drive to use, and it's really hard to map that string to a device file under the `/dev` directory, which is what we need to pass to `diskutil` for unmounting the volume to allow reading. `cdrdao` will happily let you pass a `/dev` file on Linux, but not on a Max.

It *might* possible to programmatically associate the `IOService` string (which is basically a path into Mac's IO registry) with a device file using `ioreg` and a lot of scripting magic. If you run `ioreg -c IOMedia -rl`, then one of the nodes it renders will be your optical drive, and in a dictionary under that node you'll see an entry like `"BSD Name" = "disk2"`. But I can't figure out how to turn that knowledge into the ``IOService:/...` path reliably.
