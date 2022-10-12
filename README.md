# epicon
### What is serial communication software epicon?

- epicon is a Linux serial communication software.

- In practice IOT House uses the thermo-hygrometer AM2320 sensor and AI/DIO for control with TOCOS TWE-Lite via serial communication from the USB connection ToCoStick of Raspberry Pi & IOT-House_old_pc(i386 PC).
- I think that network devices such as Switch and Router that can be configured with serial ports and console PCs can be used regardless of manufacturer or model.
- When automating the settings, Cisco Switch and Router copy and paste the text data created in advance and fill in the config.
  At this time, it is important to send delay of characters and line breaks to prevent the config data from being missed.
- epicon supports the transmission delay of important characters and line breaks as this serial console, and you can copy and paste the config with confidence.
- In addition, file transfer software such as simple telnet and zmodem, shell, macro, start of external software, etc. CUI, but it is multifunctional and compact.

- Installation
Download
https://osdn.net/projects/pepolinux/releases/p3211
```
# tar xvfz epicon-XX.XX.tar.gz
# cd epicon
# ./configure
# make
# make install
uninstall
# make uninstall
```
- How to use
  - Startup, no options (com1:/dev/ttys0 port, 9600bps, 8bit non-parity)
```
# epicon

** Welcome to epicon Version-5.2 Copyright Isamu Yamauchi compiled:Feb 24 2021 **
      exec shell         ~!
      send binary files  ~f
      send break         ~b
      call rz,sz,sx,rx   ~rz,~sz,~sx,~rx
      call kermit        ~sk,~rk
      external command   ~C
      change speed       ~c
      exit               ~.
      Connected /dev/ttyS0
```
  - How to exit
  <kbd>Enter</kbd><kbd>~</kbd><kbd>.</kbd>
  - Communication state->to shell->command input->Return to the original communication state with exit
  <kbd>Enter</kbd><kbd>~</kbd><kbd>!</kbd>
```
  epicon wait
# ls
AUTHORS      README	     configure.ac   install-sh
COPYING      aclocal.m4      depcomp	    missing
ChangeLog    autom4te.cache  epicon.c	    patch-gkermit1.0+counter-CentOS4.2
INSTALL      config.h	     epicon.h	    patch-gkermit1.0+counter1.2.1
Makefile.am  config.h.in     epicon.nr	    sample.scr
Makefile.in  config.status   epicon_main.c  stamp-h1
NEWS	     configure	     epicon_uty.c
# exit
epicon run
```
  - Switch settings and options available (/dev/ttyUSB0,19200bps, character delay:30ms, CR delay:50ms) In my old experience, setting this delay will not cause the config to be missed.
```
# epicon -d 30 -D 50 -s 19200 -l /dev/ttyUSB0
```
  - Switch settings and options available (telnet, character delay:20ms, CR delay:50ms)
```
# epicon -d 20 -D 50 -n 192.168.0.1:23

** Welcome to epicon Version-5.2 Copyright Isamu Yamauchi compiled:Feb 24 2021 **
      exec shell         ~!
      send binary files  ~f
      send break         ~b
      call rz,sz,sx,rx   ~rz,~sz,~sx,~rx
      call kermit        ~sk,~rk
      external command   ~C
      change speed       ~c
      exit               ~.

Telnet Server 1.10  All rights reserved.


login   :
```
- "-c" option: Specify external script
    - https://mono-wireless.com mono-wireless :Tocos
    - Thermo-hygrometer AM2321 sensor, analog input, digital input/output signal from a remote location with wireless modules TWELITE and MONOSTICK Programs for remote monitoring and control
  It is used in <a href="/https://github.com/kujiranodanna/IOT-House/blob/master/raspberrypi/usr/local/bin/pepotocosctl"> pepotocsctl </a>.
  <img width = "640" alt = "twelite_dip_monostick" src = "https://kujiranodanna.github.io/images/twelite_dip_monostick_en.png">
  
 - The following creates a script that reads the temperature and humidity from the AM2321 sensor and starts epicon.
```
# /usr/local/bin/epicon -s 115200 -ql /dev/ttyUSBTWE-Lite -c $CMD
# cat $CMD
#!/bin/bash
RETRY=5
I2CRD="-1"
while [ ${RETRY} -ne 0 ];do
  retry_time=`echo -en $RANDOM |cut -c 1-2`
  echo -en ":7888AA015C0000X\r\n"
  msleep 50
  read -s -t 1 I2CRD || I2CRD="-1"
  echo -en ":7888AA015C03020004X\r\n"
  msleep 50
  read -s -t 1 I2CRD || I2CRD="-1"
  msleep
  echo -en ":7888AA025C0006X\r\n"
  msleep 50
  read -s -n 28 -t 1 I2CRD || I2CRD="-1"
  TMP=`echo -en ${I2CRD} | wc -c`
  [ ${TMP} -eq 28 ] && break
  RETRY=$((${RETRY} - 1))
  [ ${RETRY} -eq 0 ] && break
  RETRY=$((${RETRY} - 1))
  msleep $retry_time
  I2CRD="-1"
done
  echo -en ${I2CRD} >/dev/stderr
```
- Manual excerpt
```
# man epicon

epicon(1)                       epicon Manuals                       

NAME
       epicon  is  Easy Personal Interface Console terminal software.  First I
       am sorry. Because my English linguistic power is very shabby, this sen‐
       tence  is being translated by the machine.  Because of that, read it in
       the interpretation which it is tolerant of though it thinks that it  is
       a little funny translation.

SYNOPSIS
       usage:
       epicon [-options [argument] [-options [argument]]
              [-b ] <--escape cannot be used
              [-c external_command]
              [-d send_charcacter_delay(ms)]
              [-D send_CR_delay(ms)]
              [-e escape_char]
              [-f send_file]
              [-F send_file_effective_delay]
              [-m ] <--input echo mode
              [-M ] <--line mode
              [-l com_port]
              [-L output_log_file]
              [-n ip_address[:port]]
              [-p [server_port]]
              [-q ] <--quiet mode
              [-s speed]
              [-v ] <--show version
              [-x bit_length (5|6|7) parity(o|e|n) stop_bit (1|2)]
              [-z ] <--auto rz prohibition

        defaults:
            speed:  9600b/s (Higest of 460800)
```
- Please check the source code below
https://github.com/kujiranodanna/epicon
