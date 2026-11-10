# Appendix

Detail for a reviewer of the DFIR Madness Case 001 report. This is not part of the main report. All times in the dual-clock timeline are as recorded. Host clocks read about 1 hour ahead of the packet capture clock.

## A. Full evidence inventory

Seven archives were downloaded and verified by MD5, as listed in section 2 of the main report. Four archives were not downloaded: DC01-pagefile.zip, DC01-ProtectedFiles.zip, Desktop-SDN1RPT-pagefile.zip, and DESKTOP-SDN1RPT-Protected Files.zip.

Extracted files and hashes:

- case001.pcap, 197,583,252 bytes, SHA256 09abf49efea1852e047987d92907704d47f36d75f6c8056e2cafa6cc027791cb
- autorunsc-citadel-dc01.csv, SHA256 2a2ea03644569ad8cbb183189938ae5a1551af82cf5fe4e6b49b81bb04b78482
- autoruns-desktop-sdn1rpt.csv, SHA256 04c7de1fa5924ac927add5e6719e781ca76496b3e9004cc33ae38975e4aa0a98
- citadeldc01.mem, 2,147,483,648 bytes, SHA256 8079a7459b1739caf7d4fbf6dde5eb0ae7a9d24dbde657debf4d5202c8dc6b62
- DESKTOP-SDN1RPT.mem, 2,147,483,648 bytes, SHA256 ac6ee0e8c273103b94d776f54192d2c6acf8303aabbd6a199254030c69f667c3

DC01 disk image: 20200918_0347_CDrive.E01, case 20200918-001, evidence CITADEL-DC01, source MD5 58c73a9c4b223382eb7b89a5008eada3. Desktop disk image: 20200918_0417_DESKTOP-SDN1RPT.E01, case 20200918-001, evidence DESKTOP-SDN1RPT, source MD5 bcd3aef20406df00585341f0c743a1ce.

Other file hashes used in the report: Szechuan Sauce.txt SHA256 5eac4f8d67ab0cf84e13d10827c7bd30974e530cabca90eba031b65b03dea017. Recovered original SECRET_beth.txt SHA256 5820a51016c70dd1126547783b0ab574e2f0c6b72df677a48c31d8fbed96e88d. My Social Security Number.txt, 22 bytes, MD5 ae1857b694f862c5c7e76a9079995c91, SHA256 9b425e57b3117c17c16f47b7fef241972a318b14741e24f7b415efd4eb0aa10f. The number in that file is not reproduced here.

## B. Tools

Volatility 3, tshark 4.2.2, Wireshark, the Sleuth Kit, ewf-tools, python-evtx, regipy, impacket secretsdump 0.13.1, and a small Python NTLM cracker using Cryptodome MD4 with a targeted 769 candidate wordlist. No malware was executed.

## C. Full NTLM hash list

NTLM hashes recovered from ntds.dit on DC01. A targeted 769 candidate crack recovered none of the passwords.

- Administrator: 10e63d3f2c9924bae49241cff847e405
- jerrysmith: bc51f858ccacc9db408c0ba511d5d639
- summersmith: 26b2cc706093c4fa46e0519ec5feaeaf
- ricksanchez: 746447f27820a9d863eea94d176cc135
- mortysmith: dc8b282b8f4e1dd3c5f95fd491ff6d8d
- bethsmith: b9cc9177094af2e17b413a0cbf63fac2
- birdman: 944055b77ebe7d6fd80f24b5fce634fb
- krbtgt: 25c9610b742a5bca9aa3801c08b8ca4e

## D. Detailed dual-clock timeline

Host clocks read about 1 hour ahead of the packet capture clock. This table gives both, so a reviewer can check the shift. Host time is the time as recorded on the machine. Capture time is host time minus 1 hour.

| Host time UTC | Capture time UTC | Event |
| --- | --- | --- |
| 22:58 Sep 18 | 21:58 Sep 18 | Packet capture starts |
| 02:22:38 Sep 19 | 01:22:38 Sep 19 | DC01 boots |
| 02:24:07 Sep 19 | 01:24:07 Sep 19 | Desktop boots |
| 03:19:28 Sep 19 | 02:19:28 Sep 19 | RDP brute force starts |
| 03:24:06 Sep 19 | 02:24:06 Sep 19 | coreupdater.exe downloaded to DC01 |
| 03:24:12 Sep 19 | 02:24:12 Sep 19 | coreupdater.exe moved to System32 on DC01 |
| 03:27:49 Sep 19 | 02:27:49 Sep 19 | coreupdater service key last written, DC01 |
| 03:29:40 Sep 19 | 02:29:40 Sep 19 | spoolsv.exe on DC01 starts with injected code |
| 03:32:39 Sep 19 | 02:32:39 Sep 19 | Secret.zip created on DC01 |
| 03:34:18 Sep 19 | 02:34:18 Sep 19 | Secret.zip deleted |
| 03:34:27 Sep 19 | 02:34:27 Sep 19 | Original SECRET_beth.txt deleted to Recycle Bin |
| 03:34:56 Sep 19 | 02:34:56 Sep 19 | New Text Document.txt created, later renamed Beth_Secret.txt |
| 03:35:06 Sep 19 | 02:35:06 Sep 19 | File renamed to Beth_Secret.txt |
| 03:35:15 Sep 19 | 02:35:15 Sep 19 | Fake content written to Beth_Secret.txt |
| 03:35:55 Sep 19 | 02:35:55 Sep 19 | RDP from DC01 to desktop starts |
| 03:38:20 Sep 19 | 02:38:20 Sep 19 | BASIC_INFO_CHANGE for Beth_Secret.txt |
| 03:39:57 Sep 19 | 02:39:57 Sep 19 | coreupdater[1].exe downloaded on desktop |
| 03:40:00 Sep 19 | 02:40:00 Sep 19 | coreupdater.exe moved to System32 on desktop |
| 03:42:42 Sep 19 | 02:42:42 Sep 19 | Desktop coreupdater service key last written |
| 03:45:34 Sep 19 | 02:45:34 Sep 19 | My Social Security Number.txt opened on desktop |
| 03:45:39 Sep 19 | 02:45:39 Sep 19 | Plans.txt opened on desktop |
| 03:45:54 Sep 19 | 02:45:54 Sep 19 | Portal_gun.png opened on desktop |
| 03:46:15 Sep 19 | 02:46:15 Sep 19 | loot.zip created on desktop |
| 03:46:18 Sep 19 | 02:46:18 Sep 19 | Zip renamed to loot.zip |
| 03:47:09 Sep 19 | 02:47:09 Sep 19 | loot.zip deleted |
| 04:39:59 Sep 19 | 03:39:59 Sep 19 | DC01 memory captured |
| 05:10:39 Sep 19 | 04:10:39 Sep 19 | Desktop memory captured |
| 06:38:51 Sep 19 | 05:38:51 Sep 19 | Last command and control contact |
| 06:38:57 Sep 19 | 05:38:57 Sep 19 | Packet capture ends |

Beth_Secret.txt MFT times, as stored: $STANDARD_INFORMATION Created 2020-09-18 23:33:54 UTC, Modified 2020-09-18 23:35:35 UTC, Accessed 2020-09-18 23:33:54 UTC. $FILE_NAME Created, Modified, and Accessed 2020-09-19 03:34:56.970 UTC. The $SI times are whole seconds, and the $SI creation time is 4 hours 1 minute 2 seconds before the $FN creation time as stored.
