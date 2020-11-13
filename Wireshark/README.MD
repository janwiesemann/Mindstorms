# Wireshark USB Capture

is a free network package capture software. I can also be used to capture traffic from USB. This is usefull if you want to sniff the communication between the new 51515 Hub and a computer.

This will work an all systems. It is harder if you are using Mac or Linux. I'm using a Mac running a Virtual Windows machine.

1. Download Wireshark from [here](https://www.wireshark.org)
2. If you are running a Windows device make sure to install USBPcap as well. This is a simple checkbox in the Installer. If you are running macOS or Linux you can follow [this guide](https://wiki.wireshark.org/CaptureSetup/USB). On macOS you first have to disable [SIP](https://en.wikipedia.org/wiki/System_Integrity_Protection) this is why I'm using a VM.
3. Open Wireshark and select an appropriate interface. On Windows you have the Option to select a specific device via the small cog icon.
4. Capture everything you want.