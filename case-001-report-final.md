# DFIR Madness Case 001, The Stolen Szechuan Sauce, Case Report

November 9, 2026. I worked this practice case from the memory dumps, packet capture, autoruns, and both disk images. It is not a real incident. Two Windows machines, 10.42.85.0/24, domain C137.local, Colorado, September 2020. A Szechuan sauce recipe turned up on the dark web. The question was how it got there.

I got one thing wrong in my first report. I said nothing was stolen from the desktop. That was wrong. Files were stolen from both machines, and this version corrects that. ## What happened

Yes, there was a breach. On both machines.

An attacker on the internet brute forced Remote Desktop on the domain controller, CITADEL-DC01, got in, and downloaded a small malware file. That file opened a command and control channel. The attacker then moved to the desktop, DESKTOP-SDN1RPT, over Remote Desktop and put the same malware there as a Windows service.

On the domain controller, the whole Secret folder was zipped as Secret.zip, sauce recipe included, and the zip was deleted 99 seconds later. On the desktop, three of mortysmith's files were zipped as loot.zip, and that zip was deleted 54 seconds later. Neither zip was ever recovered, so what was in each zip is inferred from which files were opened and read when each zip was made.

The attacker also deleted the real Beth secret, put a fake one in its place, and backdated the fake file. Treat both machines as fully compromised. The attacker stopped typing at about 02:57 UTC, but command and control traffic kept going until 05:38:51 UTC.

## How I checked it

The evidence comes from the DFIR Madness case page for Case 001, by James Smith. I downloaded seven archives and checked each MD5 against the case page. All seven matched. I worked on an isolated Linux virtual machine, on copies of the evidence, using Volatility, Wireshark and tshark, the Sleuth Kit, python-evtx, regipy, a custom USN journal parser, and impacket secretsdump. All times here are UTC, lined up to the packet capture clock. The host clocks read about 1 hour ahead, so I shifted host times back 1 hour.

The Security logs on both machines end right after boot, before the attack. On DC01, Security.evtx has 6,160 records, ends at 01:23:23 UTC on September 19, with 0 Type 10 RDP logons. On the desktop, Security.evtx has 1,937 records, ends at 01:24:08 UTC, also with 0 Type 10 logons. No 7045 service install event for coreupdater on either host, and no 1102 log clear event. I rechecked this on November 7. So the brute force is proven by the packet capture only, not by the logs, and the service install times come from the registry.

## How they got in

Port 3389 was open to the internet on the domain controller. That is the whole story, really. An internet facing Remote Desktop port on a domain controller, with a password that could be brute forced.

From 194.61.24.102, the attacker sent 29,318 SYN packets to DC01 port 3389, starting at 02:19:28 UTC. A successful RDP session followed, running to 02:52:47 UTC. I cannot prove from a log event which account was used, because the logs stop before the attack. What points to Administrator is that the shortcut files for the Secret files the attacker opened sit in the Administrator profile on DC01. Supporting evidence, not proof. 
## The malware: coreupdater.exe

At 02:24:06 UTC the attacker downloaded coreupdater.exe to DC01 over HTTP from 194.61.24.102, saved as coreupdater[1].exe. Six seconds later it was moved to C:\Windows\System32\coreupdater.exe. The desktop got the same treatment: downloaded at 02:39:58 UTC, moved to System32 at 02:40:00 UTC. Both copies are byte identical, 7,168 bytes. Hashes are in the IOC table below.

It is a small 64 bit Windows executable with two imports, VirtualAlloc and ExitProcess, plus an encoded payload section. That matches a small Metasploit reverse HTTPS stager, a standard payload any attacker can generate in minutes. It is not custom malware.

On both hosts, coreupdater is registered as a Windows service, set to start automatically as LocalSystem, with DelayedAutostart, and no signer. The DC01 service key was last written at 02:27:49 UTC, the desktop one at 02:42:42 UTC. The malware calls home to 203.78.103.109 on port 443, first from DC01 at 02:25:18 UTC, and from the desktop at 02:40:49 UTC. The last traffic was at 05:38:51 UTC, after both memory captures. On both hosts, Meterpreter code was injected into spoolsv.exe, the Windows print spooler: PID 3724 on DC01, listening on 0.0.0.0:62475, and PID 2188 on the desktop, listening on 0.0.0.0:49669.

## What was taken

### Secret.zip on DC01

Yes, the sauce was stolen. The zip was not recovered, so its contents are inferred. Szechuan Sauce.txt is 478 bytes, MD5 09c5c56e458cf7f47e626817d21ea330. The Secret folder on DC01 held Szechuan Sauce.txt, SECRET_beth.txt, NoJerry.txt, and PortalGunPlans.txt. At 02:32:39 UTC the attacker zipped the whole folder as C:\FileShare\Secret.zip, and it was permanently deleted at 02:34:18 UTC, 99 seconds later. 

### loot.zip on the desktop

This is the part my first report missed. That report searched the USN journal only for secret, szechuan, beth, and FileShare. Loot.zip uses none of those words. A narrow search, a wrong answer.

The attacker opened three files: My Social Security Number.txt at 02:45:34 UTC, Plans.txt at 02:45:39 UTC, and Portal_gun.png at 02:45:54 UTC. At 02:46:15 UTC a zip was created in C:\Users\mortysmith\Documents, and the accessed time of all three files updated to that same instant. That is how I know all three were read when the zip was made. At 02:46:18 UTC the zip was renamed to loot.zip, size 131,041 bytes. At 02:47:09 UTC it was permanently deleted, 54 seconds later. The zip was not recovered, so its contents are inferred. The SSN number is not reproduced here; the file is identified by its hash in the appendix.

### Beth's secret, and the time stomped file

The real secret was "Earth beth is the real beth." The attacker deleted the original SECRET_beth.txt to the Recycle Bin at 02:34:27 UTC, created a new file at 02:34:56 UTC, renamed it to Beth_Secret.txt at 02:35:06 UTC, and wrote "Space Beth is the real Beth" into it at 02:35:15 UTC. The original was recovered from the Recycle Bin, 28 bytes, MD5 424b69db6ec1a337c8d32579658c784e. The fake is 27 bytes.

Beth_Secret.txt is the time stomped file. Its $STANDARD_INFORMATION timestamps say it was created hours before the attack, while its $FILE_NAME timestamps say it was created during the attack, at 02:34:56 UTC. A second proof came from the November 7 recheck: a BASIC_INFO_CHANGE entry for Beth_Secret.txt in the USN journal at 02:38:20 UTC.

## Timeline

All times UTC, September 18, 2020, except the last row, September 19.

| Time UTC | Event |
| --- | --- |
| 02:19:28 | RDP brute force starts from 194.61.24.102 to DC01 port 3389 |
| 02:24:06 | coreupdater.exe downloaded to DC01 over HTTP |
| 02:32:39 | Secret folder zipped as Secret.zip on DC01 |
| 02:34:18 | Secret.zip permanently deleted |
| 02:38:20 | BASIC_INFO_CHANGE for Beth_Secret.txt, the timestomp |
| 02:46:15 | loot.zip created on the desktop, from mortysmith's three files |
| 02:47:09 | loot.zip permanently deleted |
| 05:38:51 | Last command and control contact, DC01 to 203.78.103.109:443 |

## Answers to the case questions

1. **Operating systems.** CITADEL-DC01: Windows Server 2012 R2. DESKTOP-SDN1RPT: Windows 10.
2. **Local time of the server.** Colorado is UTC minus 6 hours.
3. **Was there a breach?** Yes, on both machines.
4. **Initial entry vector.** RDP brute force from 194.61.24.102 to DC01 port 3389, from 02:19:28 UTC.
5. **Was malware used, and what was it?** Yes. coreupdater.exe, 7,168 bytes, a Metasploit reverse HTTPS stager.
6. **Malware details.** Meterpreter in spoolsv.exe, PID 3724 on DC01, PID 2188 on the desktop. Delivered by 194.61.24.102 over HTTP, calling home to 203.78.103.109:443. Lives at C:\Windows\System32\coreupdater.exe on both hosts, moved from an Internet Explorer download. Standard payload, automatic service on both hosts.
7. **Malicious IPs.** 194.61.24.102, the attacker. 203.78.103.109, the command and control server on port 443.
8. **Did the attacker access other systems?** Yes. DC01 to the desktop over RDP at 02:35:55 UTC, and files stolen from both machines.
9. **Network layout.** Two hosts on 10.42.85.0/24, domain C137.local. CITADEL-DC01, 10.42.85.10, the domain controller. DESKTOP-SDN1RPT, 10.42.85.115, a user desktop. Port 3389 was open to the internet on DC01.
10. **Architecture changes to make immediately.** Block RDP from the internet, use a VPN with multi factor authentication, rebuild both machines, and reset every domain password.
11. **Was the Szechuan sauce stolen, and when?** Yes, about 02:32 to 02:34 UTC on September 18, 2020, when Secret.zip was created and deleted.
12. **Were other sensitive files stolen or accessed, and when?** Yes. On DC01, SECRET_beth.txt, NoJerry.txt, and PortalGunPlans.txt went with the sauce in Secret.zip at 02:32:39 UTC. On the desktop, three mortysmith files were zipped as loot.zip at 02:46:15 UTC.
13. **Last known contact with the adversary.** Command and control traffic to 203.78.103.109:443 was last seen at 05:38:51 UTC. Interactive RDP activity had ended earlier, at about 02:57 UTC.

**Bonus questions.** The controls that would have helped most are the CIS Controls for administrative privileges, secure configuration, account monitoring, and boundary defense, with RDP blocked from the internet. The logs stop before the attack, so they do not show who was logged on during it. The NTLM hashes were recovered from ntds.dit, and a targeted 769 candidate crack recovered none of the passwords. The time stomped file is Beth_Secret.txt.

## Indicators of compromise

| Indicator | Detail |
| --- | --- |
| 194.61.24.102 | Attacker IP, RDP brute force, malware download over HTTP port 80, RDP sessions |
| 203.78.103.109:443 | Command and control server, both hosts, last seen 05:38:51 UTC |
| coreupdater.exe | MD5 EED41B4500E473F97C50C7385EF5E374, SHA256 10F3B92002BB98467334161CF85D0B1730851F9256F83C27DB125E9A0C1CFDA6, 7,168 bytes |
| C:\Windows\System32\coreupdater.exe | Malware location on both hosts |
| coreupdater service | Automatic Windows service, LocalSystem, DelayedAutostart, no signer |
| Beth_Secret.txt | Fake replacement, 27 bytes, "Space Beth is the real Beth", time stomped |

## What to fix

1. Take both machines offline and rebuild them from known good media. Do not just remove the malware.
2. Reset every password in the domain, including service accounts and KRBTGT twice.
3. Block RDP from the internet. Use a VPN with multi factor authentication instead, and turn on RDP logging and account lockout.
4. Block 194.61.24.102 and 203.78.103.109 at the firewall, and search logs for any other contact with them.
5. Remove the coreupdater service and C:\Windows\System32\coreupdater.exe, and search every other machine for the same hashes.
6. Segment the network so a desktop cannot be reached from a server, and watch for injection into spoolsv.exe and new unsigned services in System32.

Detail for a reviewer is in appendix.md.
