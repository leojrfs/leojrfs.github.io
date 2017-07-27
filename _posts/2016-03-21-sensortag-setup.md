---
layout: post
title: Setting up TI SensorTag on OSX and Linux
---

Setting up the [TI SensorTag](www.ti.com/sensortag) on OSX to work with [Contiki](http://www.contiki-os.org/) turned out to be tricky. After many hours of searching and hacking around, i found the solution:

### OSX:

- Get DSLite to flash the sensortag by Downloading and installing the [latest Energia for OSX](http://energia.nu/downloads/downloadv4.php?file=energia-1.6.10E18-macosx-signed.zip)

- DSLite resides in

  ```shell
  bash# /Applications/Energia.app/Contents/Resources/Java/tools/common/DSLite/DebugServer/bin/DSLite
  ```

- Flash with:

  ```shell
bash# /Applications/Energia.app/Contents/Java/hardware/tools/DSLite/DebugServer/bin/DSLite load -c /Applications/Energia.app/Contents/Java/hardware/tools/DSLite/CC2650F128_TIXDS110_Connection.ccxml -f FileToFlash.elf
  ```


I recommend creating an alias in .bash_profile like so:

```shell
bash# echo 'alias sensortag-flash="/Applications/Energia.app/Contents/Java/hardware/tools/DSLite/DebugServer/bin/DSLite \
	load -c /Applications/Energia.app/Contents/Java/hardware/tools/DSLite/CC2650F128_TIXDS110_Connection.ccxml -f "' >> ~/.bash_profile
```

Then you can flash by:

```shell
bash# sensortag-flash filetoflash.elf
```

**Compiler**

With homebrew:

```shell
bash# brew tap osx-cross/arm
```

**Serial:**

To get a Serial output look at /dev/tty.usbmodem0000000x devices provided by the Debugger DevPack:

- for picocom use ```--imap lfcrlf``` to get the output aligned


### Linux:

To setup in linux or the Instant Contiki VM [follow this post](http://piratefache.ch/getting-started-with-ti-6lowpan-sensortag/)

**In that post there is also a solution for the Debugger DevPack firmware upgrade issue.**
