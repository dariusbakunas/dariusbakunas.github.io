---
layout: post
comments: true
title:  "Arduino with XBee and ROS"
date:   2014-10-25 16:39:00
tags: [diy, arduino]
category: diy
permalink: arduino-with-xbee-and-ros
description: Small DIY project to show how to use ROS and XBee modules to control arduino device
---

Not so long ago, I was trying to figure out how to communicate with my Arduino Bot using XBee Series 2 radios (API mode) and Robot Operating System (ROS). It took me a while to make the XBee module work in API mode and I'm still learning the ins and outs of the ROS. So this is probably not the most optimal solution, but I bet it is a good starting point.

Here is a picture of my bot:
![Arduino Bot][0]

This guy is controlled by simply connecting an XBee explorer dongle to Ubuntu, running ROS, and then just using the keyboard to make it move.

Requirements
------------

These are just to show you what I used for my project. It could be much simpler, like turning a LED on or off remotely.

### Hardware

* 1x <a href="https://www.sparkfun.com/products/12866" target="_blank">Magician Chassis</a>
* 1x <a href="http://arduino.cc/en/Main/arduinoBoardUno" target="_blank">Arduino Uno</a> "brain" for the bot.
* 1x <a href="https://www.sparkfun.com/products/9815" target="_blank">Motor Driver Shield</a> to control two DC motors.
* 2x <a href="https://www.sparkfun.com/products/10414" target="_blank">XBee Series 2</a> used for wireless communication.
* 1x <a href="https://www.sparkfun.com/products/11697" target="_blank">XBee Explorer Dongle</a> to connect XBee radio to your computer.
* 1x <a href="https://www.sparkfun.com/products/12847" target="_blank">XBee Shield</a> for connecting XBee to arduino board.
* 1x <a href="http://www.amazon.com/PowerGen®-5200mAh-External-Battery-Capacity/dp/B005VBNYDS" target="_blank">PowerGen® 5200mAh battery pack</a> to power arduino bot.
* 1x Computer (PC, Mac) to run <a href="http://www.ros.org" target="_blank">ROS</a>.
* (Optional) <a href="https://www.sparkfun.com/products/10897" target="_blank">Jumper wires</a> very helpful in projects like this.
* You might need these accessories:
    - <a href="https://www.sparkfun.com/products/10007" target="_blank">Arduino Stackable Header Kit</a>
    - <a href="https://www.sparkfun.com/products/8084" target="_blank">Screw Terminals 3.5mm Pitch (2-Pin)</a>

### Software

##### Install Ubuntu (<a href="http://www.ubuntu.com/download/desktop" target="_blank">download</a>)

I used a Ubuntu 14.04 64bit virtual machine, running in VirtualBox. Ubuntu currently is the only Linux distribution officially supported by ROS.

##### Install ROS

I recommend going to www.ros.org. It has really good installation <a href="http://www.ros.org/install/" target="_blank">instructions</a>. Also, make sure to go through <a href="http://wiki.ros.org/ROS/Tutorials" target="_blank">beginner tutorials</a> to better understand how ROS works.

##### X-CTU Software (<a href="http://www.digi.com/products/wireless-wired-embedded-solutions/zigbee-rf-modules/xctu" target="_blank">download</a>)

You will need this to configure XBee radios.

##### Arduino IDE (<a href="http://arduino.cc/en/main/software" target="_blank">download</a>)

If you want to be able to program Arduino, this is must-have (yes, you can do it the HARD way, but I won't recommend it).

Assembly Notes
--------------

You can find Magician Chasis assembly instructions <a href="http://dlnmh9ip6v2uc.cloudfront.net/datasheets/Robotics/MagicianChassisInst.pdf" target="_blank">here</a>. If you don't have this assembly, you can just look at these instructions to get the idea of how it comes together.

Also, under the **References** section, I've posted a couple of links that should help you with XBee and Ardumoto shields.

Make sure that you connect your XBee router module to the Arduino. Use an XBee coordinator with the dongle to communicate with ROS (I will explain how to configure coordinator and router modules below).

Configuring XBee
----------------

First of all, write down the 64-bit serial number that is located on the bottom side of each module (it is unique and permanently assigned). We will use it for addressing (eg. 0013A200-XXXXXXXX). The first 4 bytes of this number are called **DH (Destination Address High)** and the last 4 bytes - **DL (Destination Address Low)** in XCTU application.

##### Prepare coordinator radio

ZigBee networks always have a single-coordinator radio. It is responsible for forming the network, handing out addresses and managing other functions to keep the network alive and healthy.

1. Pick one of the XBee modules (you can optionally mark it with a marker) and connect it to your computer using an XBee explorer dongle.

2. Launch X-CTU utility.

3. Click the "Add a radio module" button on the top left and select your radio (in my case, it was usbserial-AH0014VV) and click "Finish."

	![Adding XBee module][1]

	You should see your XBee module listed on the left and its information loaded on the right (if information is not visible, just left click on your module):

	![Showing XBee information][2]

	If you look at the picture above, you can see that the current "Function set" is **ZigBee Coordinator API**. If you see Router AT or similar, click on the "Update firmware" button, select **ZigBee Coordinator API** in the list and click "Finish."
4. (Optional) I'd recommend clicking the "Load default firmware settings" button before performing the configuration.

5. For this radio, we need to configure three fields:

	**ID (PAN ID)** - Used to uniquely identify your ZigBee network. Set it to a random hex number (eg. 1ABC)

	**DH (Destination Address High)** - The first 4 bytes of the XBee address (serial number, for example: 0013A200)

	**DL (Destination Address Low)** - The second part of this address

	These are destination addresses, which means that you need to use a serial number for the XBee Router module, not the one we are configuring at the moment.

    When you set these addresses, make sure to click the "Write radio settings" button.

    At this point, your coordinator module is configured.

#### Prepare router radio

Connect the second XBee module (using XBee explorer dongle). Add it to the XCTU module list and this time, pick **ZigBee Router API** for the "Function set" (use the "Update firmware" button). For this radio, you need to set the same network **PAN ID** as you set before. Put in **DH** and **DL** addresses of the coordinator module. Don't forget to click the "Write radio settings" button.

Also, before you disconnect your XBee module, go to the addressing section in XCTU and write down **MY (16-bit Network Address)** for the module. You will need it when setting up the ROS node.

Both XBee modules should now be ready. You can set them aside for now.

Programming Arduino
-------------------
If you never launched Arduino IDE before, launch it now so that it creates an Arduino folder in your user's Documents folder. Now, go to this new folder and clone the ArduinoBot repository:

{% highlight bash %}
$ cd ~/Documents/Arduino/
$ git clone https://github.com/dariusbakunas/ArduinoBot.git
{% endhighlight %}

You will also need the XBee library for your Arduino board:

{% highlight bash %}
$ cd ~/Documents/Arduino/libraries/
$ git clone https://code.google.com/p/xbee-arduino/
{% endhighlight %}

Now, launch Arduino IDE (relaunch if it was already open). Open the "File" menu. Go to the "Sketchbook" submenu and open the newly available RedWheeler sketch:

![Opening Arduino sketch][3]

At the top of the sketch, you will notice these lines:

{% highlight C %}
//ardumoto shield
int PWM_A = 3;
int PWM_B = 11;
int DIR_A = 12;
int DIR_B = 13;
{% endhighlight %}

These are standard pin settings for the Arduino DC motor shield. You should see the markings with the numbers on the shield itself (double check just in case). Now, click the "Verify" button in Arduino IDE and make sure the sketch compiles without any problems:

![Successfull sketch compilation][4]

Now, connect your Arduino board using a USB port.

If you have an XBee shield already attached to the Arduino board, you need to make sure that the UART switch is set to the DLINE position (it is located on the XBee shield). Otherwise, you won't be able to upload the sketch.

Go back to the Arduino IDE and select your Arduino board under the tools menu (*in my case, it was tty.usbmodem1411, it could be different*):
![Selecting arduino board][5]

And then finally, click the "Upload" button. If all goes well, your Arduino is ready and you can disconnect it for now.

Installing ROS packages
-----------------------

At this point, you need to log in to your Ubuntu system that has ROS installed. If you haven't done so already, create the ROS Workspace:

{% highlight bash %}
$ mkdir -p ~/catkin_ws/src
$ cd ~/catkin_ws/src
$ catkin_init_workspace
{% endhighlight %}

Build it:

{% highlight bash %}
$ cd ~/catkin_ws/
$ catkin_make
{% endhighlight %}

Activate it:

{% highlight bash %}
$ source ~/catkin_ws/devel/setup.bash
{% endhighlight %}

I recommend putting the last line in your .bash_profile if you don't want to manually switch to this workspace every time you login. First, you need to install the Python XBee library:

{% highlight bash %}
$ pip install xbee --user
{% endhighlight %}

Now, it is time to download the ROS package that I've created for this project:

{% highlight bash %}
$ cd ~/catkin_ws/src
$ git clone https://github.com/dariusbakunas/arduino_bot_ros.git
{% endhighlight %}

One thing you should change before going any further is your XBee router module address:

{% highlight bash %}
$ cd ~/catkin_ws/src/arduino_bot_ros
$ nano src/scripts/cmd_vel_listener.py
{% endhighlight %}

Find definitions for XBEE_ADDR_LONG and XBEE_ADDR_SHORT. You need to set these to your XBee router module addresses. XBEE_ADDR_LONG will be **DH** and **DL** address portions bundled together and XBEE_ADDR_SHORT is **MY** the address you wrote down earlier while using XCTU.

Build it:

{% highlight bash %}
$ cd ~/catkin_ws/
$ catkin_make
{% endhighlight %}

I'm using VirtualBox, so your steps might be little different. But the idea is the same - you need to make sure that every time you connect the XBee dongle, it gets connected to your virtual machine, not the host computer:

1. Connect the XBee explorer dongle.
2. Open the VirtualBox Settings window (for the Ubuntu VM) and then go to the Ports tab and click on "USB."
3. Click the "Add new USB filter" button. From the pop-up list, select the "FTDI FT232R USB UART" or similar and click "OK":

    ![Redirecting usb][6]

Now, every time you connect your XBee dongle, it will get connected inside your virtual machine. Go ahead and reconnect your XBee dongle.

If you run dmesg on your Ubuntu machine, you should see a similar output:

{% highlight bash %}
$ dmesg | tail
[ 6191.654557] usbserial: USB Serial support registered for generic
[ 6191.658496] usbcore: registered new interface driver ftdi_sio
[ 6191.658827] usbserial: USB Serial support registered for FTDI USB Serial Device
[ 6191.659117] ftdi_sio 1-2:1.0: FTDI USB Serial Device converter detected
[ 6191.659178] usb 1-2: Detected FT232RL
[ 6191.659182] usb 1-2: Number of endpoints 2
[ 6191.659185] usb 1-2: Endpoint 1 MaxPacketSize 64
[ 6191.659187] usb 1-2: Endpoint 2 MaxPacketSize 64
[ 6191.659190] usb 1-2: Setting MaxPacketSize 64
[ 6191.667270] usb 1-2: FTDI USB Serial Device converter now attached to ttyUSB0
{% endhighlight %}

**ttyUSB0** is your device:

{% highlight bash %}
$ ls -al /dev/ttyUSB0
$ crw-rw---- 1 root dialout 188, 0 Oct 25 12:42 /dev/ttyUSB0
{% endhighlight %}

As you can see, the only group able to write to this device is *dialout*. We need to add a current user to this group:

{% highlight bash %}
$ sudo adduser $USER dialout
{% endhighlight %}

Make sure to re-login to get those write permissions.

If your device is named exactly as above, you don't need to change anything else. But if it is different, you need to modify one Python script, located in arduino_bot_ros package:

{% highlight bash %}
$ roscd arduino_bot_ros
$ nano src/scripts/cmd_vel_listener.py
{% endhighlight %}

Then, find the line *DEVICE = '/dev/ttyUSB0'* and change it, so that it points to your device. Press CTRL+X and when it asks you to save, type 'y' + ENTER.

Make sure to set the UART switch on the XBEE shield back to the UART position (you only use a DLINE when programming your Arduino).

At this point, your ROS node is ready and your Arduino is configured to receive wireless commands. Time to start the ROS:

{% highlight bash %}
$ roscore
<...>
auto-starting new master
process[master]: started with pid [5588]
ROS_MASTER_URI=http://linuxvm:11311/
{% endhighlight %}

{% highlight bash %}
setting /run_id to e3127026-5c68-11e4-9c28-080027f27617
process[rosout-1]: started with pid [5601]
started core service [/rosout]
{% endhighlight %}

You should see a similar output, which means roscore started successfully. Now, you need to open another terminal window and start arduino_bot_ros node:

{% highlight bash %}
$ rosrun arduino_bot_ros cmd_vel_listener.py
{% endhighlight %}

It should not give you any error messages. Now, the only thing left to do is to set up the ROS teleop_twist_keyboard package:

{% highlight bash %}
$ sudo apt-get install ros-indigo-teleop-twist-keyboard
$ rosrun teleop_twist_keyboard teleop_twist_keyboard.py
{% endhighlight %}

You should get a similar output:

{% highlight bash %}
Reading from the keyboard and Publishing to Twist!
---------------------------
Moving around:
u    i    o
j    k    l
m    ,    .
{% endhighlight %}

The keys i,j,k,l,m are your control commands. Press those and your bot should start moving. To better understand what is going on behind the scenes, you could install the rqt package for ROS:

{% highlight bash %}
$ sudo apt-get install ros-indigo-rqt ros-indigo-rqt-common-plugins
$ rqt_graph
{% endhighlight %}

![rqt_graph][7]

As you can see, there are two nodes - '/teleop_twist_keyboard' and '/cmd_vel_listener_XXX_XXXX'. The first one is the one that interacts with your keyboard. The second one sends commands over the ZigBee network. Both communicate over /cmd_vel topic, which means instead of teleop_twist_keyboard, you could use anything else, as long as it can publish to the same topic.

So there it is, a complete Arduino - XBee - ROS example.

References
----------
* <a href="https://learn.sparkfun.com/tutorials/ardumoto-shield-hookup-guide" target="_blank">Ardumoto Shield Hookup Guide</a>
* <a href="https://learn.sparkfun.com/tutorials/xbee-shield-hookup-guide" target="_blank">XBee Shield Hookup Guide</a>
* <a href="https://learn.sparkfun.com/tutorials/exploring-xbees-and-xctu" target="_blank">Great XCTU tutorial</a>

[0]: /assets/bot.jpg "Arduino Bot"
[1]: /assets/xbee01.png "XCTU: Adding XBee module"
[2]: /assets/xbee02.png "XCTU: Showing XBee info"
[3]: /assets/arduino01.png "Arduino: Opening Sketch"
[4]: /assets/arduino02.png "Arduino: Verify sketch"
[5]: /assets/arduino03.png "Arduino: Selecting arduino"
[6]: /assets/vbox01.png "VBox: Redirecting USB"
[7]: /assets/ros01.png "ROS: rqt_graph"