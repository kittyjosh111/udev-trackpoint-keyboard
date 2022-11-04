# Controlling Trackpoint Sensitivity on a Lenovo ThinkPad Compact USB Keyboard with TrackPoint under Linux with udev
These instructions were used to make a udev rule to configure the sensitivity of an external ThinkPad Compact USB Keyboard using udev rules under Fedora 36. Although I could use xinput to manually set speed and sensitivity for the trackpoint, having to deal with changing id numbers or needing to reapply the script after every reboot/suspend was a hassle. I came across this article (https://askubuntu.com/questions/37824/what-is-the-best-way-to-configure-a-thinkpads-trackpoint), which worked to change my Thinkpad's built-in trackpoint sensitivity, but wanted to do the same for the trackpoint on the USB keyboard. Keep in mind I do not have the bluetooth Trackpoint Keyboard II, so I do not think the following instructions will work verbatim.


# udevadm monitor

The askubuntu answer searched for the term "Trackpoint" in the path /sys/devices/platform/i8042, which does not contain the external keyboard's trackpoint. Thus we must use udevadm monitor to search for it.
1) Unplug USB Trackpoint Keyboard
2) In terminal, run `udevadm monitor --kernel --property --subsystem-match=usb` (for bluetooth keyboards, this is probably the step that has to be different)
3) You will get an output similar to this:
`KERNEL[118637.208967] add      /devices/pci0000:00/0000:00:07.2/0000:50:00.0/0000:51:02.0/0000:54:00.0/usb7/7-2 (usb)
ACTION=add
DEVPATH=/devices/pci0000:00/0000:00:07.2/0000:50:00.0/0000:51:02.0/0000:54:00.0/usb7/7-2
SUBSYSTEM=usb
DEVNAME=/dev/bus/usb/007/010
DEVTYPE=usb_device
PRODUCT=17ef/6047/330
TYPE=0/0/0
BUSNUM=007
DEVNUM=010
SEQNUM=9530
MAJOR=189
MINOR=777`
4) Get the value for DEVPATH. udevadm may output multiple devices, choose the least specific path. For me, this was `/devices/pci0000:00/0000:00:07.2/0000:50:00.0/0000:51:02.0/0000:54:00.0/usb7/7-2`
5) Run the command `find /sys/DEVPATH -name name | xargs grep -Fl TrackPoint | sed 's/\/input\/input[0-9]*\/name$//'` Change DEVPATH to the path you found in step 4. My command was thus `find /sys/devices/pci0000:00/0000:00:07.2/0000:50:00.0/0000:51:02.0/0000:54:00.0/usb7/7-2 -name name | xargs grep -Fl TrackPoint | sed 's/\/input\/input[0-9]*\/name$//'` Make sure /sys/ preceeds the path from step 4.
6) The command from step 5 will output two lines. For me they were `/sys/devices/pci0000:00/0000:00:07.2/0000:50:00.0/0000:51:02.0/0000:54:00.0/usb7/7-2/7-2:1.1/0003:17EF:6047.0013
/sys/devices/pci0000:00/0000:00:07.2/0000:50:00.0/0000:51:02.0/0000:54:00.0/usb7/7-2/7-2:1.0/0003:17EF:6047.0012` Keep these lines for the next few steps.

# Manually testing sensitivities

Here, we will now try testing changing the sensitivity of the trackpoint. Remember that we found two lines at the end of the previous set of instructions.
1)  In a terminal, run `echo 50 | sudo tee PATH/sensitivity`, where PATH is one of the lines that were the output in the last section. For me, I first tried `echo 50 | sudo tee /sys/devices/pci0000:00/0000:00:07.2/0000:50:00.0/0000:51:02.0/0000:54:00.0/usb7/7-2/7-2:1.1/0003:17EF:6047.0013/sensitivity`. If the command returns the number 50 without errors, then you have found the correct path to work with. You also should start seeing the trackpoint sensitivity change.
2) If you chose the wrong path in step 1, the output will look like `tee: '/sys/devices/pci0000:00/0000:00:07.2/0000:50:00.0/0000:51:02.0/0000:54:00.0/usb7/7-2/7-2:1.1/0003:17EF:6047.0012/sensitivity': No such file or directory`
3) Change the number 50 in the command of step 1 to any value between 1-255 to adjust sensitivity to your liking. Remember the value you like the most for the next steps.

# Writing a udev rule
Now that we have the value for sensitivity that we want, we can write a udev rule to make changes persistent. Since I am bad at using a trackpoint, my sensitivity was set to a low number. You may find an example udev rule in this repository.
1) In a terminal, navigate to `cd /etc/udev/rules.d/`
2) Using your preferred text editor, create a udev rule. This can be anything, but conventionally starts with a number, then a  name, then .rules. A simple file name can be `10-trackpoint.rules`
3) Inside, paste the following:  `SUBSYSTEM=="serio", DRIVERS=="psmouse", DEVPATH=="", ATTR{sensitivity}=""` Insert the path you found when manually testing the sensitivites into DEVPATH. Change the sensitivity value to the sensitivity value you prefer.
4) Save the file. To apply changes, you can either reboot or run `sudo udevadm control --reload-rules && sudo udevadm trigger `

Hopefully these steps work for you. Thanks to the threads found on https://askubuntu.com/questions/37824/what-is-the-best-way-to-configure-a-thinkpads-trackpoint for including instructions on how to make udev rules for the internal Thinkpad trackpoints.
