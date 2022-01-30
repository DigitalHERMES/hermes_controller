# hermes_controller

## Some references of HL2 protocol and implementations

There is HL2 protocol wiki (On the protocol wiki page, the two links to
openhpsdr documents in the first paragraph are both essential as the wiki
page primarily covers HL2-specific changes):
https://github.com/softerhardware/Hermes-Lite2/wiki/Protocol

The HPSDR USB-port protocol documentation is here:
https://github.com/TAPR/OpenHPSDR-SVN/tree/master/Documentation

To understand get/set parameters:ss
https://github.com/softerhardware/Hermes-Lite2/wiki/Address-Management-Python

C hl2setup: https://github.com/softerhardware/Hermes-Lite2/tree/master/software/hl2setup

Python hermeslite.py (THE reference):
https://github.com/softerhardware/Hermes-Lite2/tree/master/software/hermeslite

C hl2_tcp: https://github.com/hotpaw2/hl2_tcp

Go OpenWebSDR connector: https://github.com/jancona/hpsdrconnector

C Pihpsdr: https://github.com/g0orx/pihpsdr

C Linhpsdr: https://github.com/g0orx/linhpsdr

Go library: https://github.com/jancona/hpsdr

Python Quisk:
    http://james.ahlstrom.name/quisk/

Steve's recommendation for a simple python code (rx4000.py):
https://github.com/softerhardware/Hermes-Lite2/blob/master/software/ft8/rx4000.py


## Some comments from RDP by Steve


- Notes on _getdata() in rx4000.py:

  - Samples are divided by 8388607.0 (hex 0x7fffff) to convert the 24 bit 
    integer sample values to floats with range -1..1, 

  - 'I' samples are multiplied by  100 to provide gain

  - 'Q' samples are multiplied by -100 to provide gain and to work around a
    long standing bug in gateware and the applications that's too old to fix

- Notes on use of port 1024 vs port 1025

  - Gateware responds to commands to port 1025 as well as 1024 so that two
    computers and/or applications can interact with a HL2 at the same time

  - This allows hermeslite.py to connect on port 1025 at the same time as
    some other SDR application is connected on port 1024

- Notes on the sequence 0xef, 0xfe, 0x05 in ethernet frames

  - The 0xef, 0xfe, 0x05 sequence indicates a special command sent to port 1025

  - The definitive source code for what is recognized by the HL2 is in the
    dsopenhpsdr1.v file around lines 183 to 195:
    https://github.com/softerhardware/Hermes-Lite2/blob/master/gateware/rtl/dsopenhpsdr1.v

- My interpretation of (and comments on) the sequence 0xef, 0xfe, 0x05 

  - It's a mechanism to send a command to the SDR on the alternate port 1025
    regardless of whether or not the start command has been sent 

  - Python function 'command()' in hermeslite.py shows the message sent is:
    bytes([0xef,0xfe,0x05,0x7f,addr<<1])+cmd+bytes([0x0]*51)
    where command can be either a 32 bit number or a four byte sequence

  - 'addr' corresponds to C0 and 'cmd' corresponds to C1 C2 C3 C4 of the
    main openhpsdr protocol

- Notes on the 10 RX 'variant' image

  - The December 2020 build of the 10 RX gateware variant does not support port
    1025 or the 0xef, 0xfe, 0x05 sequence because of the need to conserve gates,   
    see line 100 of:
    https://github.com/softerhardware/Hermes-Lite2/blob/master/gateware/variants/hl2b5up_cicrx/hermeslite.v

  - If you need to use this 10 RX gateware or older versions and you don't 
    need to support two computers accessing HL2 at one time, then you should 
    be able to modify hermeslite.py to use port 1024 instead of port 1025,
    or just work with the standard 4 RX which supports port 1025 and the 
    command sequence

  - The July 2021 build of the 10 RX gateware variant does support port 1025
    and the 0xef, 0xfe, 0x05 sequence, there was enough space to enable it
