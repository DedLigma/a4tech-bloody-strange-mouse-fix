## a4tech bloody strange behavior mouse fix for linux

## About
This fix is designed to eliminate the strange behavior of the mouse when it behaves like a gamepad (the mouse has dead zones like a gamepad stick).

## Why does the mouse behave strangely
This happens because the event files of the mouse are split. To fix this problem, you need to merge them using evsieve. However, depending on the linux kernel type, the event file numbers may change after each system boot. This fix detects which event files your mouse uses and merges them together.

## Requirements
Install [evsieve](https://github.com/KarsMulder/evsieve)

Check if udevadm works correctly with 
```
udevadm --version
```

## How to use the fix
First, determine which event files your mouse is using with `sudo evtest`.

Usually, the difference in event file numbers is equal 2. If this is not the your case, change line 4 in the mouseFix file
```
eventNumber=$(($eventNumber+2))
```
to
```
eventNumber=$(($eventNumber+<your-difference>)) 
```
Then find which file creates these event files. You can do this either by looking at the links in the file properties of `/dev/input/by-id`, or by using udevadm
```
udevadm info /dev/input/event<your-number>
```
you need lines like this
```
S: input/by-path/pci-0000:00:14.0-usb-0:3:1.1-event-mouse
S: input/by-id/usb-COMPANY_USB_Device-if01-event-mouse
```
Insert the full path `/by-id/` into the mouseEventPath in the mouseFix bash script, adding `/dev/`
```
mouseEventPath="/dev/input/by-id/usb-COMPANY_USB_Device-if01-event-mouse"
```
Copy `<your-event-mouse>` from the line `S: input/by-path/<your-event-mouse>` and paste it into mouseFix.service in the `After=` parameter, adding `.device` at the end
```
After=pci-0000:00:14.0-usb-0:3:1.1-event-mouse.device
```
After that, move the mouseFix bash script to a convenient location and write the FULL path to it in mouseFix.service `ExecStart=`
```
ExecStart=/<path-to-mouseFix>/mouseFix
```
Move mouseFix.service to `/etc/systemd/`system using
```
sudo mv mouseFix.service /etc/systemd/system
```
Enable and start mouseFix.service
```
systemctl enable mouseFix
systemctl start mouseFix
```
Enjoy!


