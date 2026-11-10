# Digital Forensics Investigation

## What this is

This is my investigation of DFIR Madness Case 001, The Stolen Szechuan Sauce, a free practice case by James Smith. It is not a real incident. There is no real victim, and the files, users, and network are fictional case data.

A Szechuan sauce recipe was found on the dark web, and I investigated two Windows machines to find out how it got there: a domain controller, CITADEL-DC01, and a desktop, DESKTOP-SDN1RPT, on 10.42.85.0/24, domain C137.local, Colorado, September 2020. I analyzed the memory dumps, packet capture, autoruns, and both disk images, on an isolated Linux virtual machine. No malware was executed.

## Summary

Yes, there was a breach, on both machines. An attacker brute forced Remote Desktop on the domain controller, logged in, and downloaded a small malware file, coreupdater.exe. That file opened a command and control channel, and the attacker then moved to the desktop over Remote Desktop and installed the same malware there.

The attacker stole files from both machines. On the domain controller, the whole Secret folder was zipped as Secret.zip, including the sauce recipe, and the zip was deleted 99 seconds later. On the desktop, three of mortysmith's files were zipped as loot.zip, and that zip was deleted 54 seconds later. Neither zip was recovered, so what was in each zip is inferred from the files opened and read when the zip was made. The attacker also deleted the real Beth secret, replaced it with a fake one saying "Space Beth is the real Beth", and backdated the fake file. The real secret, recovered from the Recycle Bin, was "Earth beth is the real beth."

## Key findings

- Entry: RDP brute force from 194.61.24.102 to DC01 port 3389, from 02:19:28 UTC on September 18, 2020, 29,318 SYN packets.
- Malware: coreupdater.exe, 7,168 bytes, a standard Metasploit reverse HTTPS stager, the same file on both hosts. Meterpreter was injected into spoolsv.exe, PID 3724 on DC01 and PID 2188 on the desktop.
- Command and control: 203.78.103.109 on port 443. Last traffic at 05:38:51 UTC, after the attacker stopped typing at about 02:57 UTC. Persistence was an automatic Windows service named coreupdater on both hosts.
- Secret.zip: created on DC01 at 02:32:39 UTC, deleted at 02:34:18 UTC. It held Szechuan Sauce.txt, SECRET_beth.txt, NoJerry.txt, and PortalGunPlans.txt.
- loot.zip: created on the desktop at 02:46:15 UTC, deleted at 02:47:09 UTC, 131,041 bytes. It held My Social Security Number.txt, Plans.txt, and Portal_gun.png, inferred from the files read at zip time.
- Beth_Secret.txt was time stomped. Its $STANDARD_INFORMATION timestamps are about 4 hours earlier than its $FILE_NAME timestamps, and a BASIC_INFO_CHANGE entry at 02:38:20 UTC confirms it.
- The event logs stop before the attack. Security.evtx on DC01 ends at 01:23:23 UTC on September 19, with 0 Type 10 RDP logons, and the desktop log ends at 01:24:08 UTC, also with 0 Type 10 logons. The brute force is proven by the packet capture only.

## Indicators of compromise

| Indicator | Detail |
| --- | --- |
| 194.61.24.102 | Attacker IP, RDP brute force and malware download |
| 203.78.103.109:443 | Command and control server |
| coreupdater.exe | MD5 EED41B4500E473F97C50C7385EF5E374, SHA256 10F3B92002BB98467334161CF85D0B1730851F9256F83C27DB125E9A0C1CFDA6 |
| coreupdater service | Automatic Windows service, LocalSystem, no signer |

## What I used

Volatility 3, Wireshark and tshark, the Sleuth Kit, python-evtx, regipy, a custom USN journal parser, and impacket secretsdump.

## Files in this repo

- case-001-report-final.md: the full case report, with numbered answers to the case questions.
- appendix.md: evidence inventory, tool versions, the full NTLM hash list, and a dual-clock timeline.
- No evidence files are included. The disk images, memory dumps, and packet capture are large, and the malware file is a live Metasploit payload.

## Limitations

- Neither zip was recovered, so the contents of Secret.zip and loot.zip are inferred from the files opened and read when each zip was made.
- The event logs end right after boot, before the attack, so there are no logon or service install events from the attack window. The Protected Files copies were not downloaded.
- The domain NTLM hashes were recovered, but a targeted 769 candidate crack recovered none of the passwords.

## What I got wrong the first time

My first report said no files were stolen from the desktop. That was wrong: I had searched the USN journal only for secret, szechuan, beth, and FileShare, and loot.zip uses none of those words.
