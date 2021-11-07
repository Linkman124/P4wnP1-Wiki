# General questions

### Does this run on a Pi Zero (non WiFi)?
#### Yes
Most of the payloads do, but not the HID backdoor (if you'd enable USB networking for this payload, it would be possible to connect to the backdoor shell, but 127.0.0.1 attacks are out of scope for this project).
An example for a more sophisticated payload running without WiFi is the "HID frontdoor".

[![video link](https://www.youtube.com/vi/MI8DFlKLHBk/0jpg)](https://www.youtube.com/watch?v=MI8DFlKLHBk)

### How to connect P4wnP1 to Internet, once installed ?
#### Method 1
Here's a little tutorial video, showing the steps on Windows 10. Requirement: `network-only.txt`payload has to be running.

[![video link](https://www.youtube.com/vi/QEWaIoal5qU/0jpg)](https://www.youtube.com/watch?v=QEWaIoal5qU)


#### Method 2
Edit the variables that start with "WIFI_CLIENT" to match the credentials with an existing accesspoint with ethernet connectivity and set WIFI_CLIENT to true.
P4wnP1 will connect to this accesspoint instead of starting one.

### Does this run on other ARM devices?
#### No
P4wnP1 uses several features specific to raspbian. Some include:
- the RNDIS ratepatch is specific to the raspbian RNDIS kernel module
- the LED interaction is based on raspbian FS interface
- USB OTG detection is based on raspbian DEBUG FS integration
- the USB gadget stack is only tested against the UDC of the raspberry
If you have the know-how, feel free to port to other devices.

### Should I change passwords?
Sure... and don't forget the WiFi password ;-)

### The setup isn't easy, will there be a prebuilt image?
#### No
`INSTALL.md` will be reworked to give more guidance on how to get the Pi Zero ready.

### What's the purpose of "John the Ripper (JtR)" and "Responder.py"
The package came from an idea in early development stage. There's an attack mentioned in the README, which allows you to steal NetNTLMv2 hashes from locked Windows boxes (Rob ‘MUBIX’ Fuller: “Snagging creds from locked machines”). This was in fact the first payload that was implemented on P4wnP1 (Responder.py is there for the same reason, as it is needed to carry out the attack). The plan was to develop a "LOCKPICKER" for Windows hosts with weak credentials, which would do the following:
1. Steal the NetNTLMv2 hash with the 'Mubix' approach
2. Fetch the hash from Responder.db and hand it over to John the Ripper
3. Try to crack the hash until either P4wnP1 is shutdown or the John the Ripper reports a SUCCESS.
4. On SUCCESS, use the HID keyboard to type out the plaintext password and unlock the machine.

So if you ask, "Where is the lock picker?" ... It has never been finished, for multiple reasons.
- The chance for the "Snagging creds attack" to succeed is very low, since Microsoft patched the issue.
- The third party application I found vulnerable to this was patched, too (see `README.md` for reference).
- Implementing the lock picker isn't a real challenge and there's a whole lot of other work to do on P4wnP1. So feel free to contribute.

Anyway, both tools could come in handy on various payloads (even ones I couldn't even imagine today). The JtR, which could be fetched from the raspbian repo with `apt-get install john`, isn't really feature rich. It only supports a basic set of hash types. The package included in P4wnP1 is the JtR Jumbo version, which can handle a ton of hash types. It was compiled on the Pi itself. The hash rate for the mentioned NetNTLMv2 hashes on the Pi Zero W is about 100,000 hashes per second. This means if you have a dictionary with 100,000 passwords and your victim chooses one of them, cracking would take a second, max.
The Responder.py version is a slightly modified one, allowing it to respond to probe requests of Microsoft Windows to check its "online status" in such a manner, that the system believes it has Internet access. This could come in handy in several network attacks.

# Backdoor
***
### I am getting the "No screen to be attached" error on the backdoor payload, what can I do?

Many people seem to have this problem, try:

* Check to make sure you are using a cable that has data lines. If you just use a charging cable it won't work.
* Try waiting longer before you SSH, to allow the pi to start up all the required processes. For most, it seems to work after a couple of minutes from when the pi is plugged into the target. And don't SSH as soon as you connect to the wifi, give it some time.
* Make sure you are running the latest supported and tested version of raspbian by running `uname -a`

If these things still bring no result, you can try:

* Running `sudo python /home/pi/P4wnP1/hidtools/backdoor/P4wnP1.py` to see if it starts at all.
* Checking in the DuckyScripts/ folder to see if there are files in it. If not, you need to clone recursively.
* Reinstall everything from scratch, new raspbian image and everything.

### Is it possible to change the network settings?
#### Yes
In setup.cfg, you will find a header called "Wifi options", all of the wireless settings can be modified here

### If I close the server is there a way to reopen it?
#### Yes
Rather than closing the server, simply disconnect it with `ctrl+a` followed by `d` to detach the screen.  Run `screen -d -r` to re attach it to the terminal.

### Will the backdoor payload work on a locked machine?
#### No
The backdoor payload will only work on an unlocked (logged in) box. However, you can use the Windows 10 lockpicker payload to solve the locked problem.

### What's the difference between this and an ordinary BadUSB?
That is a really good question. Karsten Nohl was the first using the term "BadUSB". What he described (or showed) was that ordinary commercial USB devices could be reprogrammed to make them act differently. Thus it was possible to mod a normal USB flash drive (with PHISON controller) into an USB keyboard, running keyboard attacks. But the concept wasn't all about keyboards. In fact one could reprogram a vulnerable USB device to be anything, as long as it is defined in the USB specs (and one is able to reverse and modify a firmware based on an Intel 8051 derivate).
So the more precise question should be: "What's the difference between this and an ordinary RubberDucky attack?"

### What's the difference between this and an ordinary RubberDucky attack?
The backdoor payload does the same as a RubberDucky attack. The difference is that you can launch keyboard attacks from a WiFi based custom SSH shell on demand. The target sees only two HID devices, no other USB hardware.
From the moment you use the `FireStage1` command things change a bit:
As stated there are two HID devices. The first one is a HID keyboard (used to carry out the keyboard attacks). The second HID device is a GENERIC HID device. The `FireStage1` command uses the keyboard device, to type out code building up a sort of protocol stack. This protocol stack is used to communicate with P4wnP1 via the second HID device.

To be more precise:
The shell that spawns using the `shell` command doesn't use a network connection, a serial port or any other communication device. It is based on a pure Human Interface Device. Now try to explain to your firewall to block this communication channel (this isn't socket based). Or try to explain to your endpoint protection to block USB HID devices and say goodbye to all kinds of controllers using this standard. I guess the difference becomes clear!
So to carry out pure keyboard attacks, this kind of "covert channel" isn't needed, you can run them as soon as you SSH into P4wnP1s backdoor interface. But once the covert channel is up after issuing `FireStage1` there should be no need to run further keyboard attacks, as long as you know what you're doing. The input to the shell (and other spawned processes) is tunnelled through the HID channel.

### Where is the code for the client side payload?
[Here](https://github.com/mame82/P4wnP1_HID_backdoor_client.md)
