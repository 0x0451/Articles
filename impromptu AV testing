# An impromptu test of a few AV products

Tested: defender, mcaffee, AVG, kaspersky, bitdefender and malewarebytes.

Payload was an x64 meterpreter reverse https shell with a DLL proxy loader that does process injection into explorer.exe using direct syscalls. The malicious DLL is called by an EXE that is 100% clean. The whole package is disguised as a game thats packaged in a zip file, and downloaded from my attack box into the test VM. target environment is x64 windows 10.

test consisted of running the exe, seeing if I got a shell, and then trying various "aggressive" actions, including:

- meterpreter getsystem
- meterpreter hashdump
- taking a screenshot
- streaming the victim machine's screen
- running a keylogger and retrieving user's keystrokes
- running metasploit's local exploit suggester
- attempting to execute the fodhelper UAC bypass module with a non-evasive stock x64 meterpreter payload.

## Defender:

Shell loaded. Incorrectly detects the clean EXE that loads the malicious DLL if I do any aggressive act; auto deletes it. Fails to detect the DLL. Does not detect what process the shell injects into, so even after it makes a detection, I still have an active shell in explorer.exe's memory. Detects the shell only if used to load fodhelper module payload, which is not evasive.

Does not detect it upon download.

## AVG:

Shell loads. When it does so, the AV alerts that something loaded into explorer.exe's memory space, but does not kill the shell and otherwise it performs the same as Defender and triggers for about the same things.

Does not detect it upon download.

## McAffee:

Performs similar to defender; detects and kills the shell in explorer.exe when I try to run the local exploit suggester. Does not detect the EXE or DLL.

Does not detect it upon download.

note: while it was updating, it turned off all monitoring I was able to gain admin on my test environment with fodhelper. If I'd been a bit more aggressive, I probably could have uninstalled it before it came back online.

## Bitdefender:

Performed similar to defender. Tripped when local exploit suggester ran and killed the shell. flagged the EXE. missed the dll.

does not detect upon download.

## Kaspersky:

Shell is able to load, but is killed within about 30 seconds with any activity (even simple commands like ls or getuid get it caught), aggressive or not. Incorrectly flags the clean EXE as malicious. Does not detect the malicious DLL.

Permitted the archive I packaged the malicious program in to download, but upon extraction, deletes every EXE and DLL in it, clean or not.

## Malwarebytes:

Did not detect anything. Was able load my shell, do all aggressive acts and then gain admin using the fodhelper module (which means it can't even detect an unencoded, unobsfucated, plain-jane out-of-the-box meterpreter shell) and then I used these privs to uninstall it remotely.

## verdict:

of everthing I tested, nothing except for kaspersky and malwarebytes stood out; everything else performed about the same as stock windows defender. Kaspersky would have stopped me from being able to gain a foothold at all, and is best in show out of this group. Malwarebytes was so bad that it's literally worse than defender and should not be used at all.

## note:

a lot of these only detected malicious behaviour when I used the fodhelper metasploit module and loaded a stock shell as the payload. it's likely that these were all flagging on the stock shell, since meterpreter is so well known; had I not been doing this just for fun and put more effort into it, it's very likely that if I'd built another evasive payload and used a manual exploit to take advantage of the fodhelper vuln instead of the metasploit module, I would have been able to get admin on all of the "performed like defender" products above and then gone on to disable or uninstall these products. Only kaspersky (out of the ones i tested) would have been able to stop this.

