---
layout: post
title:  "Resurrecting a Brother HL-2250DN after a failed firmware update"
date:   2022-07-07 20:40:00 +0200
categories: howtos
---

Yesterday, the status monitor software of my Brother HL-2250DN informed me about a new firmware update. I decided to install the update, but at 90%, the update process interrupted. The update tool complained that it cannot connect to the printer anymore.

After a few minutes, I switched the printer off and on again, but apart from short flashing of the LEDs it was dead. No fan blowing as usual, no network connection. When I connected the printer using USB, it was recognized as "BrotherHL2-Maintenance", which seems to be an interface to the printer's bootloader for authorized service partners which can be used to restore the firmware.

So I called a service partner, but the result was disappointing: Firmware updates are not covered by Brother's warranty.

# Firmware Restore Tool / Driver
In order to use the BrotherHL2-Maintenance interface to restore the firmware, you need a driver for the interface. These drivers are available to authorized service partners only. After a bit of research, I found out that the driver ZIP archive is called `BHL2-Maintenance.zip`. The file can be found using Google, for example here. You also need the firmware restore tool which is called `FILEDG32.exe`.

The device driver requires a 32-bit Windows XP or older. As I'm using a 64-bit Mac, I had to set up a virtual machine (VMware Fusion) running Windows XP.

After installing the device driver, the FILEDG32.exe tool can be used to upload firmware to the printer by simply dragging the firmware file onto the `Brother HL2 Maintenance Printer` icon.

# Finding the Firmware
The hardest part was to find the appropriate firmware for the HL-2250DN. At the Brother website, you can only download a firmware update tool which does not contain the actual firmware data but downloads it from the web. Of course, the tool does not recognize the BrotherHL2-Maintenance and fails (no printer found).

The Mac OS X version of the firmware update tool is a Java application which can be analyzed quite easily after decompressing the JAR file. Analysis of the application shows that it gets the link to the appropriate firmware from a web service located at `firmverup.brother.co.jp`.

You can simply forge a request for the HL-2250DN printer to retrieve the link. Just create a file, e.g. request.xml, containing the following request:

    <REQUESTINFO>
        <FIRMUPDATETOOLINFO>
            <FIRMCATEGORY>MAIN</FIRMCATEGORY>
            <OS>MAC</OS>
            <INSPECTMODE>1</INSPECTMODE>
        </FIRMUPDATETOOLINFO>

        <FIRMUPDATEINFO>
            <MODELINFO>
                <SELIALNO></SELIALNO>
                <NAME>HL-2250DN series</NAME>
                <SPEC></SPEC>
                <DRIVER></DRIVER>
                <FIRMINFO>
                    <FIRM>
                        <ID>MAIN</ID>
                        <VERSION>1.15</VERSION>
                    </FIRM>
                    <FIRM>
                        <ID>BRNET</ID>
                        <VERSION>1.10</VERSION>
                    </FIRM>
                </FIRMINFO>
            </MODELINFO>
            <DRIVERCNT>1</DRIVERCNT>
            <LOGNO>2</LOGNO>
            <ERRBIT></ERRBIT>
            <NEEDRESPONSE>1</NEEDRESPONSE>
        </FIRMUPDATEINFO>
    </REQUESTINFO>

Then, post it to the web service, e.g. using curl:

    $ curl -X POST -d @request.xml https://firmverup.brother.co.jp/kne_bh7_update_nt_ssl/ifax2.asmx/fileUpdate -H "Content-Type:text/xml" --sslv3

You will get a response containing the firmware download link:

    <?xml version="1.0" encoding="UTF-8" ?><RESPONSEINFO><FIRMUPDATEINFO><VERSIONCHECK>0</VERSIONCHECK><FIRMID>MAIN</FIRMID><LATESTVERSION>1.17</LATESTVERSION><PATH>http://update-akamai.brother.co.jp/CS/LZ3514_J.blf</PATH><DLTIME>180000</DLTIME></FIRMUPDATEINFO></RESPONSEINFO>

Now, just download the firmware (.blf file) from that location.

# Restoring the firmware

Connect the printer to a Windows XP machine and install the device driver (see above). Start `FILEDG32.exe` and drag the firmware file (e.g. `LZ3514_J.blf`) to the `Brother HL2 Maintenance Printer` icon.

![filedg32.exe utility used to flash Brother printer firmware](/assets/2022/07/filedg32.png)

The printer's LEDs will start to flash during the process. The process is finished when all LEDs are on. Power-cycle the printer and it should come back to life!

# Other printers

This procedure should work for other Brother printers as well, as long as it is being recognized as `BrotherHL2-Maintenance` USB device in the device manager. You will have to find the appropriate firmware for your printer by using the `firmverup.brother.co.jp` web service. In the XML request file, replace the `MODEL` and `SPEC` fields and the `FIRMINFO` entries. Note that the `SPEC` field is empty for the HL-2250DN, but may contain a value for other printers.

You will have to know the correct `MODEL` and `SPEC` values and also the `FIRMINFO` entries. This can be quite difficult. I found out the values for the HL-2250DN by querying a functional printer of the same model using SNMP:

    $ snmpwalk -c public <IP-ADDRESS> iso.3.6.1.4.1.2435.2.4.3.99.3.1.6.1.2 

    SNMPv2-SMI::enterprises.2435.2.4.3.99.3.1.6.1.2.1 = STRING: "MODEL=\"HL-2250DN series\"
    "
    SNMPv2-SMI::enterprises.2435.2.4.3.99.3.1.6.1.2.2 = STRING: "SERIAL=\"...\"
    "
    SNMPv2-SMI::enterprises.2435.2.4.3.99.3.1.6.1.2.3 = STRING: "SPEC=\"\"
    "
    SNMPv2-SMI::enterprises.2435.2.4.3.99.3.1.6.1.2.4 = STRING: "FIRMID=\"MAIN\"
    "
    SNMPv2-SMI::enterprises.2435.2.4.3.99.3.1.6.1.2.5 = STRING: "FIRMVER=\"1.15\"
    "
    SNMPv2-SMI::enterprises.2435.2.4.3.99.3.1.6.1.2.6 = STRING: "FIRMID=\"BRNET\"
    "
    SNMPv2-SMI::enterprises.2435.2.4.3.99.3.1.6.1.2.7 = STRING: "FIRMVER=\"1.10\"
    "
