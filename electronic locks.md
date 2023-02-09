# Electronic Locks


## Caveat Lector

This isn't really my field of expertise; with PhySec, I'm more knowlegeable about mechanical locksmithing, and with computers, I'm more familiar with more conventional IT security vs. this sorta OT type stuff. So I may be a little high-level and lacking detail, but this little writeup should point you in the right direction for knowing how to google things related to this topic. 

Also, for home lab purposes, this stuff get's really expensive to play with, so it's not like mechanical lockpicking or computer hacking. The door hardware and bypass tools get ridiculously expensive compared to like mechanical lockpicking or even like conventional software exploitation, so be warned. 

## What we're not talking about...

Bluntly, I'm not going to be covering what I'd call "freestanding consumer devices". This would be like some one off padlock that has electronics integrated into it. The reason I'm not, is because they're usually pretty one-off and product specific as far as vulns go, and when it comes down to it, most of these consumer devices tend to skimp on real physical security engineering, and so can typically be bypassed or brute forced fairly easily. 

To that end, if you need to break into some specific consumer electrified padlock or something, it would be better for you to just google the specific product you need to get into, since someone probably has some youtube video or something demonstrating some simple bypass technique for that particular product. 

Instead, what we're going to be covering is really gonna be devices that are tied into physical access control systems (PACS), as these are probably more in line with what you expected to be covered anyway, and are much more interesting to cover from a high level like this post is going to aim to do.

## Parts of a PACS

So you have different components to a physical access control system:

* The Door controller

This is sorta the heart of any PACS. More or less, these are embedded linux devices that actually process most of the logic for the PACS at the door itself. 

* Locking hardware. 

We'll go into this in more detail below, but more or less, this is the electrified hardware that actually serves to secure the door. 

* Credentialed access devices 

This would be like your card readers, etc, that take credentialed input and pass it through the door controller for access. They usually pass data to the door controller using the Wiegand protocol. More below. 

* Egress devices 

These are devices that can open a door without credentials; if voltage is placed on their lines, the door opens. 

* Sensors 

Your most common is gonna be the door contact switch, which we'll cover more below. 

* (Optional) The central Database

In enterprise installs, you'll typically have a host somewhere on the network that is tied to the door controllers and updates them on creds and can even be used to remote open doors, etc. 

## Door Controllers

Like I said above, these are basically little embedded Linux devices that actually serve as the boots on the ground brains of the operation. In effect, these are like an arduino or similar device where you'll have a system on a chip with a bunch of pinout locations to wire in the other devices in the PACS. 

This stuff gets product specifc, so always RTFM for the specific door controller you're interested in, but in general, you're gonna have inputs and outputs: 

You'll generally have: 

* Egress device inputs (usually like 2 pins; usually from an egress device, but some devices like keypads may wire into this too, since cheap ones can handle creds internally)
* Wiegand input (usually 4-6 pins; this is from credential readers)
* Alarm line (for like door contact switches, keypad tamper output, etc)
* Fire control input (basically, in enterprise setups where there's also a fire panel, you can rig input to open all doors in a fire situation)
* RS-232 I/O (i.e. serial port. You use this to daisy chain controllers in some setups)
* (Maybe) RJ-45 (for networking the controller back to the central DB)
* Door outputs (can get pretty specific, esp. with products like maglock that need specific voltages for maintaining the electromagnetic lock)

The specific pin outs are product specific, so if you're like interested in installing door controllers, you need to play RTFM with the specific product you plan to install. Again, this is sorta a high-level overview. Not too much of that is, imo, really pertinent to the offensive side of this topic, though. 

Still, in most cases, these are the sorts of I/O pinouts you'll find on a door controller. 

The door controller itself has internal memory to store credentials, but this is not readily updated without connection to a DB. I believe they have limited memory as well, but I cannot remember. 


## The Controller in Situ

More or less, the door controller will live in a little panel box that, if you're familiar with industrial or enterprise faciliies, you've probably seen -- little nondescript grey boxes that hang out on the wall. 

Inside, you'll see the door controller itself (just a single PCB computer -- looks like a motherboard or something), the power supply (lots of these sorta look like old laptop harddrives imo), and the backup power supply (usually looks like golf cart batteries). 


## How Door Controllers are usually setup

Door controllers can be setup several different ways: 

### The solitary door controller. 

Bascially what it says on the tin. You have one or a small number of doors and a small number of credentials for access. You basically have a controller just because it's necessary. You'll see this sometimes in like small business contexts. 

### Master/Slave Door Controllers. 

Basically, in a setup like this, you wire multiple door controllers together with RS-232 (serial port) as "slave" units that actually operate the readers and door hardware, but they're all rigged into a "master" controller that has all of the stored credentials on it. This has major fault tolerance problems, since the devices are wired in series, so if one device fails, all devices downstream of it will likewise fail. 

### Networked

On the tin: you rig the controller via RJ-45 into an ethernet network directly and it gets creds from the central DB that's on the network. If all of your controllers are rigged like this, there's more fault tolerance, since each controller would be getting creds from the DB directly. 

Note well that you can have hybrid systems as well. E.g. you could have a master/slave system where the masters are rigged into the DB over the network, etc. 


## Wiegand -- a word you should know= 

Wiegand refers to 2 things. 

1. Wiegand wires, which are little strips of specially made wire that when passed through a magnetic field, generate a detectable electrical pulse. If you embed a bunch of these into a plastic card, you get the original access control card, which used this effect to encode a 26 bit number in a sort of "barcode" of these wires that was embedded into the plastic.
2.  The 26 bit format that was establised with the original Wiegand wire cards. This format became so widespread due to the ubiquity of the original wiegand wire access card that it is effectively the standard format that credentials are passed from the reader to the door controller with. Regardless of what format (e.g. RFID, magstripe, etc) is used between the access token and the reader, the format used between the reader and the door is almost always this same old unencrypted 32-bit Wiegand format.


## Attacking Wiegand

Because the Wiegand protocol is unencrypted, we can use Wiegand Intercept Devices to intercept the Wiegand credential and replay it. There have been several devices of this sort made availible online, but the one most people talk about these days is the ESPKey, which is a small interceptor that has it's own wifi interface that can be used to perform replay attacks or dump creds from a mobile app. 

Devices like this will usually be used by disassembling the credentialed access device in the field in order to expose the wiegand protocol wires that run to the door controller, and then punching those lines into the appropriate punch-down locations on the device's board. You'd then reinstall the credentialed access device and wait for someone to use their credentials on the reader. Since it's sitting in-line between the reader and the door controller, the ESPKey is able to pick up the 26-bit credential, which can then be used to either clone cards or directly to perform replay attacks using the ESPKey itself. 

Obviously, though, this is a pretty "loud" attack, since you have to disassemble a card reader right there in the field, which is obviously not something someone should be doing in most settings...

## Credentialed access devices

Basically, these are things that take a credential and pass it over the Wiegand format to the door controller. 

Common reader technologies include: 

### Wiegand cards -- 

the original key card technology. Deprecated. You may still see these in ancient installs, but it's not a common technology anymore.

### Magstripe cards -- 

These encode credentials on a magnetic stripe. Again, deprecated, but a little more common than old school wiegand cards. Magnetic developer fluid can be poured on the magstripe in order to visually decode these, and they're very simple to find cloner tools and blanks for online. 

### RFID cards -- 

These encode data on an RFID chip and are the most common type of access token in the modern world. RFID can use 2 frequencies bands which are not compatible, with high frequency being significantly more secure than low frequency. 

Again, like it was said above: the format between the card and the reader is arbitrary and media specific, but the actual hard credential being sent to the door controller is almost always 26-bit Wiegand format.  

## RFID

RFID is one of the more common types of credentialed access devices that you'll see in the modern world. 

RFID in general works like this:

Inside the card, there is a chip and a coiled antenna. When the card gets close to a reader, radio waves emitted by the reader generate current on the antenna and power the chip. From here, communication can occur. 

There are two main types of RFID: 

### Low frequency RFID -- 

125-135 KHz. These can be identified in the field by shining a light through a card and inspecting the antenna; LF antennas are tightly coiled and will appear to look like a solid ring inside the card. These are a very simple standard and are one-way communicators. The typical process for how a LF credential will be read is:

1. the reader will power and query the card
2. the card will broadcast the RFID credential back to the reader
3. the reader translates this into 26 bit wiegand and ships it to the door controller for authentication. 

Because of it being simple and one-way, it's fairly simple to sniff, clone or emulate, as the credentials cannot be encrypted. Additionally, it is the longer range format, and so with long range antennas, is much easier to covertly sniff. 

### High Frequency RFID a.k.a. Near Field Communication (NFC) --

13.56 MHz. Like LF, these can be identified by shining a light through the card; in this case, an NFC antenna will have distinct gaps between it's coils and look sorta like a spiral. NFC can use 2 way communication and is able to use encrypted credentials, and is thus much more difficult to attack, with some of the more well designed standards still being publically secure. 

Additionally, there are many brands and product lines of RFID credential technologies. HiD is probably the most common brand in North America, but even these guys have several different formats. E.g. Prox (and it's numbered iterations), iClass (and it's iterations like iClass SE, etc) MiFare (again, it has iterations), as well as several other acquired product lines I cannot recall off the top of my head. Each of these have different encoding standards and in the NFC world, they employ different security features to prevent card cloning and emulation. And again, not all of them have been publically cracked yet and are still secure. 

## Attacking RFID

Before we get into attack methods, we need to talk about RFID tools, as you'll obviously need some sort of device to actually be able to read, write and spoof credentials with. 

Some devices worth mentioning (in order of price):

### Proxmark 3 Easy

The proxmark tools are an open source hardware tool for interacting with RFID. The "easy" variant is a cheap chinese made variant that can be had for around $80-$90 USD. These appear to based on the older Proxmark3 RDV3. These chinese clones are appearently quite capable of anything that a genuine proxmark is able to do, but with significant hardware problems, especially concerning the antennas, and component quality in general. Most people seem to advise not using this as a field device on engagements, but rather as a "desk" device for hobby use. 

### FlipperZero 

The FlipperZero is a fairly new tool that's cropped up recently that's generated quite a bit of hype due to it's broad range of capabilities. Unlike the other devices on this list, this is NOT really a dedicated RFID tool, but is moreso a general wireless multitool, often billed as a "swiss army knife" or "jack of all trades" sort of device. While it does have LF RFID and NFC capabilities, it is significantly less capable than a Proxmark in that it recognizes far fewer card formats and does not appear to be able to do more than simply read NFC cards (and thus likely lacks the ability to clone or emulate encrypted NFC cards). However, unlike the Proxmarks, the FlipperZero has the ability to work with various other common non-RFID wireless technologies, such as infrared and sub-gigahertz radio (i.e. stuff like garage door openers, etc). This tradeoff means it may be a product you find useful, but again: it isn't a dedicated RFID tool. Retails for $169. 

### Proxmark 3 RDV4 

This is the name-brand read-deal that is indeed designed to be a professional tool for pentesters and other people who intend to take it into the field on engagements. Made with high quality hardware, this device is capable of cloning over 80% of known card formats, and can perform brute-force attacks, sniff traffic between cards and readers, emulate cards, clone cards, etc etc. Retails for $360. This is the tool you want if you want to seriously attack RFID. 


Attacks generally consist of:

### Sniffing/Cloning/emulation

Basically, you use your wireless device to scan an existing card and then either clone that card to a blank card, or use the device to replay the device's credentials. This is trivial with LF RFID.

### Brute Forcing

You use the device to brute force the card reader in the field. 

### Reader attacks 

Basically, you query an HF reader in order to get information needed to attempt to crack encrypted cards. In some cases, you may need to intercept the full handshake between a known-good card and the reader. 


Sorry if the explanations are a bit bad; again, RFID is absolutely not my field of expertise. I myself need to go out and get some RFID tools and hardware to actually start labbing with this stuff. Hopefully I've at least pointed you in the right direction to go do research in, though. 



## Egress Devices

While I'm calling these "egress" (i.e. exit) devices, bear in mind that I'm simply talking about ALL devices that do not communicate wiegand credentials to the door controller, and instead provide simple low voltage current over a line that opens the door when powered. MOST of these are true egress devivces, but they can certainly be used on the secure side of the door for ingress as well. 

### Simple Buttons 

What it says on the tin. A button that triggers the door controller to open. Bear in mind that these can be more secure, though; e.g. they can be placed behind a mechanically locked cover, or placed behind a security/reception desk so that authorized personnel have to "buzz" you in/out. Often in this latter setup, you'll see them used in tandem with an intercom system. 

### Keyed Switches

Basically, this is a mechanical lock that has electrical contacts on its tail piece. When the key is inserted, the tailpiece rotates like in a conventional lock, and this rotation causes an electrical switch to close, placing voltage on the egress line. As part of a PACS, these are very common in psychiatric hospitals and other minimum security locations where controlled egress is necessary, however keyed switches are also extremely common in industrial settings as control switches on electrical equipment (although this is not an access control application). They're vulnerable to all of the mechanical attacks one can employ against a lock, e.g. picking, bumping, etc. 

### Electronic Keypads

While usually used for ingress, many of these are actually wired like egress devices. In this case, they'll have internal memory to store their PIN numbers, and power the egress line when their internal logic controllers detect one of these internally stored PINs being correctly entered. Many of these can be disassembled in the field, exposing the underlying wiring, which can then be short circuited in order to send the egress signal. Be warned though: many of the ones that can be field disassembled also tend to have tamper sensors that are wired into the door controller's alarm port, and will trigger an alarm condition if properly wired. Do your due dilligence and RTFM if you identify that one of these is going to be encountered on your next engagement -- find the schematics for that device and try to figure out where the contacts are inside the panel, and if there are any viable means to bypass the tamper sensor. There are several videos on lockpickinglawyer's youtube showing a few examples of this sort of procedure to give a general idea of what this process looks like. 

### PIR REX sensors. 

PIR stands for Passive InfraRed, and REX is Request for EXit. PIR REX sensors are little IR motion sensors that are often mounted just inside of a secure area and are designed to detect when people are attempting to exit. When the device detects a change in the temperature gradient, ostensibly from someone walking up, they power the egress line and open the door. You'll often see these in hospitals or financial institutions, almost always in tandem with a set of double doors that are secured by a mag-lock. These can be easily bypassed if there is a gap between the doors; simply hold a can of PC air duster upside down and spray it through the door. The cryogenic propellant will cause enough of a temperature change to activate the device and open the door. If the PIR REX is mounted a bit away from the door, balloons can be slipped through the gap, inflated, and then let go. They may trigger the PIR as they fly past it. Many other whimsical substances have been demonstrated by pentesters to show the possibilities of this attack, with both vape clouds and whisky being spat through the door gap being demonstrated to be effective at triggering the device. 


## Door Contact Sensors

Before we get into locking hardware itself, I wanted to make sure I covered these, as they can be vitally important. 

Door contact sensors are devices that are designed to detect if a door is open or not. How they work is fairly simple: 

Either on or in the door frame, there is a little device that contains a spring loaded switch. Correspondingly, on the door itself, there is a small magnet that lines up with the spring loaded switch when the door is closed. When the door is open, the springs in the switch on the frame hold the switch closed so that current passes through to the door controller. When the door is closed, the magnet on the door overcomes the springs in the switch on the frame, causing the switch to open and not supply current. 

In normal operation, this would mean that the contact sensor will send voltage to the door controller any time the door is opened. How this is usually used is that the controller looks for whether the contact sensor reports both a credential read/egress request that corresponds with the door opening. If the door controller detects an opening, but does not detect a credential or egress request, this will trigger an alarm condition, as such a situation would usually indicate forced entry. 

Because of this, any time you intend to open a door that's rigged into a PACS using a mechanical bypass technique, you'll need to determine whether or not there is a door contact sensor and mitigate it if possible; if the door opens inwards, it is not possible to bypass the sensor. 



Detecting these is pretty simple. If the door has an externally mounted sensor, you should simply be able to see the device. If the switch is embedded in the frame, you can use a tool called a magnetic pole detector. These are cheap little things that are usually under $10, and are basically a small plastic handle that has a gimbal on the end that contains a small magnet. When you pass the tool near a magnet, the little magnet in the gimbal on the tool will point towards the other magnet. More or less, you'll take this tool and pass it along the gap between the door and door frame. When it indicates that you've found a magnet, that's likely the location of the contact sensor magnet. 

Once you've found the magnet, you'll want to tape a neodymium wafer magnet to the appropriate location to hold open the switch. If this is an external switch, simple tape it to the device on the frame. If it is an embedded switch, tape it to the inside of the door frame

Again, bear strongly in mind that *you can only bypass door contact sensors if the door opens outwards towards you*. If the door opens inwards, you will be physically unable to place your bypass magnet properly. 


## Locking hardware

Finally, we get to discussing the threat's originally topic: electronic *locks* themselves. 

There are several types of electronic locks that are in common use, but they tend to operate on similar principles.  

### First: Fail-safe vs. Fail-secure

There are two modes of operation that we can consider for electronic locking hardware:

1. Fail-safe: this means that the device defaults to the *unlocked* state when power is cut. 
2. Fail-secure: obviously, the opposite -- the device defaults to the *locked* state when power is cut. 

While it may be tempting to think that if you were to somehow cause a power outage, that you could then open any fail-safe door, be advised that in many commercial and institutional settings, all of the components of the PACS (locks included) will likely be tied into battery backed emergency power. 


### A note on attacking electronic locks 

While I would think the audience wants some magical high-tech solution to electronic locks, we must realise immediately that attacking the electronic components of an electrified lock is usually impractical. In most cases we will attack either 

1. The mechanical components of the lock with a traditional mechanical attack such as latch slipping, bumping, picking, etc. 
2. The credentialed access or egree devices 

This is because in most cases, the electronics are buried in the wall or are otherwise inaccessable; if you are able to disassemble an electrified lock sufficiently to manipulate the electronics, you've likely long gotten the door to a state where it can be opened. 

Because many of these locks tend to be installed as retrofits into existing mechanical systems, they often bring their own fun vulnerabilities, often caused by careless installation. 

### Magnetic attacks 

To somewhat contradict myself, let's talk about magnetic attacks. 

Many electrified locking components operate by having a powered solenoid that extends and retracts in order to bind mechanical components and cause the device to enter or exit the locked state. If a locking device suffers from improper security design, these solenoids (or other internal mechanical components) may be able to be moved from the outside with a strong rare-earth magnet. This becomes increasingly likely for products designed in the past, especially prior to the 1980s, since rare-earth magnets were not in common use until after that time period. 

In any case, one would use a large rare-earth magnet to find the solenoid and then manipulate it such that the solenoid is moved into the unlocked position. 

I can't really be more clear on this, as it is a rather product specific flaw and would need to be researched on a per product basis. Either way, it is certainly a possibility, especially on older electronic safes, and some door hardware. 



### Electrified strike plates 

Electrified strikeplates are strikeplates (the little metal hole the bolt of a door goes into) that have a hinged section that opens when in the unlocked state. This allows the bolt of the door to freely exit the strike, whether the mechanical lock on the door itself is locked or not. 

These are frequently installed as retrofits into existing mechanical systems, with the original mechanical lock serving as a keyed override.

Note that electrified strike plates can often be configured to be either fail-safe *or* fail-secure. 

These are a pretty common site around offices and other commercial and institutional facilities. 

#### Common Vulnerabilities 

* The lock on the door itself is likely 100% mechanical, and thus vulnerable to traditional mechanical attacks. 
* For cost savings, install contractors may cut costs by only stocking one size of strikeplate; often the largest. If an oversized strikeplate like this is installed, it leads to tolerance issues that leave the door particularly vulnerable to traditional mechanical attacks that target the latch, e.g. slipping, loiding, sawing, etc. 



### Electrified Locks 

Unlike electrified strike plates, in this case, it is the lock itself that has electrical components. Here, you'll often find that there's a solenoid in the mechanism that secures the lock. These usually also have keyed overrides, and like electrified strikeplates, can be configured for either fail-safe or fail-secure operation. 

You'll find these as both: 

1. electrified mortise locks -- these are big square units that slot into a hole on the side of the door and often have a threaded cylinder lock and seperate knob. 
2. electrified cylindrical locks -- these are smaller units that look just like a doorknob, often with a KiK cylinder. They slot through a round hold drilled through the door where the knob should be. 

Both of these form factors operate similarly, with electrified solenoids obstructing some part of the locking mechanism. 

These are a pretty common site around offices and other commercial and institutional facilities. 


#### Common Vulnerabilities 

* As these are just mechanical locks with electronics integrated into them, they're vulnerable to all traditional mechanical attacks that would otherwise apply to whatever hardware setup you run across. 
* Since these are often *not* installed as simple retrofits, and the door is often intended to be operated 100% of the time through the PACS, some very dumb things can occur at install time. I've personally observed electrified cylindrical locks which have been installed *with no pins in the mechanical cylinder*. This means that the mechanical override on these doors could be opened by simply inserting a turning tool and actuating the lock. 

### Mag-Locks 

Unlike the above two units, mag-locks are not simply mechanical locking components that have had some small electronics wired into them, but instead are very specifically electrical in nature. They consist of a large electro-magnet and a steel plate, with one attached to the door and the other to the frame. When powered and in the locked state, the force of attraction between the electro-magnet and the steel plate keeps the door secure. 

Since these require power to operate, these are only capable of fail-safe operation, however, they are also generally on battery backup and will likely remain secure even during a general power outage in any decent facility. 

These are occasionally seen in secure business facilities, but most often, I've tended to see them in institutional settings such hospitals, especially near the maternity and psychiatric wards. 

#### Common Vulnerabilities 

* As these are in no way related to traditional mechanical locks, they are *not* vulnerable to any traditional mechanical attacks that I'm aware of. 
* These are often paired with a PIR REX sensor and are vulnerable to attacks against those systems, such as using compressed air to trigger the REX, etc. 
* With sufficient force, the magnet can be separated from the steel plate, but I do not think this is practical for a human, as the magnets are secure with a ridiculous amount of force.
* As stated, these devices are fail-safe only and can be vulnerable to power outages if their backup power supply has not been properly configured. 

In all honesty, it is likely best to focus on attacking the credentialed access devices or egress devices tied to the mag-lock rather than the lock itself. 


## Conclusion 

Hopefully this has been a helpful introduction to electronic locks. As stated from the outset, this isn't my field of expertise, so I hope it wasn't too vague or useless. This should give you a good ground for knowing the next things you need to google in order to plan an attack against electronic locking systems. 
