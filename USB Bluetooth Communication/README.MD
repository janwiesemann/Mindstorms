
# COM Messages

The Hub uses a Serial COM. This applies in USB and Bluetooth. I haven’t tested it on bluetooth but I think the packages are the same.

1. Every 0.5 seconds the hub sends the current sensor data to your device running the app.
2. The messages seam to be encoded in JSON and split over multiple packages. The are not always send in the same parts. Sometimes the `“p”` until `}` is send in the same package.
   1. `{ “m":`
   2. `0`
   3. `,"p":[[75, [`
   4. `0, -2, 43, 0]], [75, [0, 0, -10, 0]], [0, []], [62, [null]], [75, [0, 2, -170, 0]], [75, [0, -2, -106, 0]], [-8, -717, 720], [-2, 0, -1], [-5, 0, 44], "", 3946]}`
3. As soon as you send a program to the hub it sends the following data
   1. `{“i":"pG25","m":"start_write_program","p":{"meta":{"created":1604829015218,"modified":1604829636901,"project_id":"ecVunojg7XjW","name":"UHJvamVrdCAx","type":"scratch"},"size":627,"slotid":0}}`
   2. `{"i":"uZCn","m":"write_package","p":{"data":"ZnJvbSBydW50aW1lIGltcG9ydCBWaXJ0dWFsTWFjaGluZQoKYXN5bmMgZGVmIHN0YWNrXzEodm0sIHN0YWNrKToKICAgIHdoaWxlIFRydWU6CiAgICAgICAgKGFjY2VsZXJhdGlvbiwgZGVjZWxlcmF0aW9uKSA9IHZtLnN0b3JlLm1vdG9yX2FjY2VsZXJhdGlvbigiQSIpCiAgICAgICAgdm0uc3RvcmUubW90b3JfbGFzdF9zdGF0dXMoIkEiLCBhd2FpdCB2bS5zeXN0ZW0ubW90b3JzLm9uX3BvcnQoIkEiKS5ydW5fZm9yX2RlZ3JlZXNfYXN5bmMoMzYwLCB2bS5zdG9yZS5tb3Rvcl9zcGVlZCgiQSIpLCBzdGFsbD12bS5zdG9yZS5tb3Rvcl9zdGFsbCgiQSIpLCBzdG9wPXZtLnN0b3JlLm1vdG9yX3N0b3AoIkEiKSwgYWNjZWxlcmF0aW9uPWFjY2VsZXJhdGlvbiwgZGVjZWxlcmF0aW9uPWRlY2VsZXJhdGlvbikpCiAgICAgICAgdm0uc3RvcmUubW90b3Jfc3BlZWQoIkEiLCA3NSkKICAgICAgICB5aWVsZAoKZGVmIHNldHVwKHJwYywgc3lzdGVtLCBzdG9wKToKICAgIHZtID0gVmlydHVhbE1hY2hpbmUocnA=","transferid":"31645"}}`
   3. `{"i":"Vrmw","m":"write_package","p":{"data":"Yywgc3lzdGVtLCBzdG9wLCAiQTZBekxGcEdNUHFKUGxQWlViRUsiKQoKICAgIHZtLnJlZ2lzdGVyX29uX3N0YXJ0KCJDSnh1V2I5NVIzY0tmU3NCN21DYiIsIHN0YWNrXzEpCgogICAgcmV0dXJuIHZtCg==","transferid":"31645"}}`
   4. `{"i":"4Dyz","m":"program_execute","p":{"slotid":0}}`
4. If you click on termite this will be send`{“i":"qq1O","m":"program_terminate","p":{}}`

# Lets explore the messages from 3. (data polling)

`"p"` Is an array with different child objects. since 4 messages started with `[75,` I think, these are my 4 motors. This means every object is identified using a number. This is the case for the first 6 Elements. These are the I/O ports where you can connect stuff.

0 => No connection

75 => 51515 Motor

62 => Ultrasonic distance sensor (distance in cm, `null` if out of range > 2m)

61 => Color sensor

the last 5 Elements are special. The do not have an ID associated with them.

1. Acceleration
2. Gyro rate
3. Tilt angle
4. empty string don't know, what this does
5. It's a number. It changes but I don't know when. Maybe a status or some flags.

# Lets explore the messages from 4. (sending of an application)

If you are experienced with computers you notice something special about the “data” part. These strange string are encoded using Base64. You can encode them using some [online tools](https://www.base64decode.org). If you then combine all These results you will receive following python code:

`from runtime import VirtualMachine`

`async def stack_1(vm, stack):`

`while True:`

`(acceleration, deceleration) = vm.store.motor_acceleration("A")`

`vm.store.motor_last_status("A", await vm.system.motors.on_port("A").run_for_degrees_async(360, vm.store.motor_speed("A"), stall=vm.store.motor_stall("A"), stop=vm.store.motor_stop("A"), acceleration=acceleration, deceleration=deceleration))`

`vm.store.motor_speed("A", 75)`

`yield`

`def setup(rpc, system, stop):`

`vm = VirtualMachine(rp`

`c, system, stop, "A6AzLFpGMPqJPlPZUbEK")`

`vm.register_on_start("CJxuWb95R3cKfSsB7mCb", stack_1)`

`return vm`

I did not wrote this. I was Using the Scratch Programming interface since I’m more of a C programming guy. This is the Scratch code:

&#x200B;

# Conclusion

51515 uses a simple communication interface. This means you could also write a plugin for Atom, VS Code or another great editor to finally have a nice interface for writing your applications.

# Set up your own Exploring

1. Download and install [Wireshark](https://www.wireshark.org)I recommend using a Window machine since it is the easiest. Make sure to also install USBPCap. This is a option in the Wireshark installer.On macOS you have to disable [SIP](https://en.wikipedia.org/wiki/System_Integrity_Protection)On Linux and Mac you can follow this [guide](https://wiki.wireshark.org/CaptureSetup/USB).
2. Start sniffing.Open Wireshark and select the right interface.On windows you have the option to Select a specific device on a USB-Node. This can be done over the small Cog-Icon.
3. Capture everything you want and explore it.

Filter for filtering the application data Messages from the app to the hub: `usb.src==host&&frame.len!=27`

Since I was using a Windows 10 VM on my mac I had to select this device.