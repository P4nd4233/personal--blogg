---
layout: default
---

# [PART 2] Renovating an old network for nearly free

* * *

## Updating the firmware of your WiFi Router for better security and management


In <a href="tp_link-1.html" target="_blank">PART 1</a> I have tested and explained what dangers and inconveniences poses, old embedded hardware. And how a badly organized network is a big problem.
Here I am going to show you, how you can modernize your out-of-date router. We will stick to the agenda bellow:

> ### AGENDA
> <a href="tp_link-2.html#1">1) Updating the firmware of the router, from the vendors website (if the product is not out-of-date)</a> <br>
> <a href="tp_link-2.html#2"> 2) Installing custom open source firmware </a><br>
> <a href="tp_link-2.html#3">- Exposing the UART port </a><br>
> <a href="tp_link-2.html#4">- OpenWRT Installation and initial configuration </a><br>


> 3) Video on Best Practices in general (Shown on OpenWRT)
>  - <a href="tp_link-2.html#5">Guest only WiFi network </a> <br>
>  - VLANs <br>
>  - Strong Password Policy <br>
>  - WPA Enterprise <br>

<div id="1"></div>

## 1) Updating the firmware of the router, from the vendors website (if the product is not out-of-date)

You can just google the make and model followed by firmware update, and download the image, provided on the manufacturer's website .

<a href="pics/networking/tplink/google.png" target="_blank"><img src="pics/networking/tplink/google.png"></a>

As in my case i will download the firmware for wr740n ver 2.

<a href="pics/networking/tplink/stock-update-1.png" target="_blank"><img src="pics/networking/tplink/stock-update-1.png"></a>
<a href="pics/networking/tplink/stock-update-2.png" target="_blank"><img src="pics/networking/tplink/stock-update-2.png"></a>

After you download you extract the zip, and you can, go to the web management console of the router, then to the firmware upgrade page and proceed.

<a href="pics/networking/tplink/stock-update-3.png" target="_blank"><img src="pics/networking/tplink/stock-update-3.png"></a>

In our case that is not an option since the latest firmware was released back in 2010-09-10 ...

<div id="2"></div>

## 2) Installing custom open source firmware

<div id="3"></div>

### Exposing the UART port - Optional BUT Recommended

> I recommend watching this <a href="https://www.youtube.com/watch?v=sTHckUyxwp8" target="_blank"> UART Protocol Explanation Video</a> .<br>Here I will explain it plain and simple.

<a href="https://en.wikipedia.org/wiki/Universal_asynchronous_receiver-transmitter" target="_blank">UART</a> is basically fancy for Serial Port. The UART port in practice is used for debugging the software that is running on the embedded system. During the engineering process, one of the ways programmers are testing their firmware that is going to be put on the final product is via UART. Most devices have UART or a <a href="https://en.wikipedia.org/wiki/JTAG" target="_blank">JTAG connector</a> (which is different, we will "cover" only UART here).

#### What it gives to us?
So if we can use that type of interface to interact with the device on a level that is not possible other way. It gives us access that only the Manufacturer/Developer was intended to have. And in our case we want exactly that.
There are many videos online covering exactly what we can do, and how we do it.

Because you don't want to brick your hardware, since you are installing something that is custom, always a good practice is having a backup plan if something goes wrong. 
If the wifi and lan does not respond and something goes wrong during the upgrade process, and for further use of the router, it is good to have alternative interface option. We will use the router's built in UART serial console <a href="https://en.wikipedia.org/wiki/Universal_asynchronous_receiver-transmitter" target="_blank">[1]</a>. By doing this you can get direct access to the router's bootloader and flash firmware this way. More on that can be found on "Debricking" on the OpenWRT site.<a href="https://openwrt.org/toh/tp-link/tl-wr740n" target="blank">[2]</a> In ver 2 of this particular model, it is a little bit quirky, but still works, since most devices have through holes on the PCB, which are very distinguishable, our does not. Although with some googling we can find that there are probe points on the board, which are the RX and TX pins use for the UART communications.

<a href="pics/networking/tplink/tp-link-uart-1.jpg" target="_blank"><img src="pics/networking/tplink/tp-link-uart-1.jpg"></a> <br>
And if we test with the multimeter, indeed the labeling is correct. <strong>TP4</strong> is <strong>TX</strong> and <strong>TP5</strong> is <strong>RX</strong>.

<a href="pics/networking/tplink/tx-1.png" target="_blank"><img src="pics/networking/tplink/tx-1.png"></a> <br>

So I have soldered 3 wires to these pins, 1 to TX, 1 to RX and 1 to a ground pad. Run the wires out of the case, by breaking the plastic on the bottom, and soldering header pins to the end of the wires, so we can connect them to <a href="https://www.aliexpress.com/wholesale?catId=0&initiative_id=SB_20200831113029&SearchText=usb+uart+adapter" target="_blank">UART-to-USB interface</a>, and get a serial console. <br> 
> Tip 1: Use different-colored wires for the different pads, if you don't have like me, make sure to label them correctly <br>
> Tip 2: because the connection between the header and the wire is not as structurally strong, i have used super glue and sprinkled some baking soda on top, to make the bonding harden more, and stick the 2 plastic pieces together.

<a href="pics/networking/tplink/uart-2.jpg" target="_blank"><img src="pics/networking/tplink/uart-2.jpg"></a> <br>
<a href="pics/networking/tplink/uart-3.jpg" target="_blank"><img src="pics/networking/tplink/uart-3.jpg"></a> <br>

If we connect to the adapter, we can see output inside the terminal, by running `screen /dev/ttyUSB0 115200` on linux, or with <a href="https://www.putty.org/" target="_blank">Putty</a> on Windows, by specifying the correct COM port and baud rate of 115200.


<a href="pics/networking/tplink/uart-4.jpg" target="_blank"><img src="pics/networking/tplink/uart-4.jpg"></a> <br>

<div id="4"></div>

### Installing OpenWRT
There are several open source projects online, which give us the option the put second life to our obsolete router hardware.
Some of the most famous are <a href="https://dd-wrt.com/" target="_blank">DD WRT</a> and <a href="https://openwrt.org/" target="_blank">Open WRT</a> .
We will not go on deep dive which one is better, etc ... You can do your research online which one to use, here we will explain how to flash Open WRT on WR 740 N.

If we google `"wr740n openwrt"`, we get the official project web page, from where we can see instructions and download the custom image.
<br><a href="https://openwrt.org/toh/tp-link/tl-wr740n" target="_blank">`https://openwrt.org/toh/tp-link/tl-wr740n`</a><br> Then we download the image for our hardware version, and repeat the flashing process from <a href="tp_link-2.html#1">(1)</a>.

Since the article will become too long, and I do know that is boring, and you want to get straight to the point, I am going to create a quick video tutorial that will show you how to install and configure for initial use OpenWRT on WR 740n.


<div style="padding:56.25% 0 0 0;position:relative;"><iframe src="https://player.vimeo.com/video/453635394?color=ff9933&byline=0&portrait=0" style="position:absolute;top:0;left:0;width:100%;height:100%;" frameborder="0" allow="autoplay; fullscreen" allowfullscreen></iframe></div><script src="https://player.vimeo.com/api/player.js"></script>

<div id="5"></div>

### Video on setting up Guest WiFi Network


<div style="padding:56.25% 0 0 0;position:relative;"><iframe src="https://player.vimeo.com/video/453652697?color=ff9933&byline=0&portrait=0" style="position:absolute;top:0;left:0;width:100%;height:100%;" frameborder="0" allow="autoplay; fullscreen" allowfullscreen></iframe></div><script src="https://player.vimeo.com/api/player.js"></script>


## Conclusion

With Custom Router Firmware you get the best out of the hardware, you unlock new features, you get better manageability, even network level automation, creating backups of the network configuration, features you can get from the most modern equipment only. Of course this will not be a placebo to everything, but it is a good beginning, and also it is <strong> completely free of charge </strong>.