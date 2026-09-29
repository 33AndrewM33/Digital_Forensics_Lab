# Digital Forensics Labs

Hands-on labs from a digital forensics course, done mostly in Kali Linux, plus a few other virtual machines, all running in Oracle VirtualBox. Each section has screenshots from the lab with a short note on what I was doing and what I learned. Everything was done on files and VMs I set up myself in an isolated lab.

## Skills Covered

- Setting up an isolated forensics lab in a virtual machine
- Acquiring and preserving disk images
- Reading partition tables and file systems
- Finding deleted files in disk images
- Extracting artifacts from Windows Registry hives
- Inspecting raw data with a hex editor
- Hiding data inside an image (steganography) and recovering it
- Cracking a password-protected zip file with John the Ripper
- Capturing and reading network traffic with Wireshark
- Basic OSINT: checking where a username appears online

## Tools Used

| Tool | What I used it for |
|---|---|
| Kali Linux | Main OS for the labs |
| Oracle VirtualBox | Running Kali in a virtual machine |
| Guymager | Creating forensic disk images |
| The Sleuth Kit (`mmls`, `fsstat`, `fls`) | Command-line analysis of disk images |
| Autopsy | Browser-based forensic examination |
| RegRipper | Pulling artifacts from Windows Registry hives |
| hexedit | Byte-level inspection of a drive |
| 7-Zip and Windows Command Prompt | Hiding a zip file inside an image and extracting it |
| zip2john and John the Ripper | Recovering a zip file's password |
| Wireshark | Capturing and reading network traffic |
| Metasploitable | Intentionally vulnerable VM used as a capture target |
| Sherlock | Checking where a username shows up online |
| CSI Linux | Forensics-focused Linux distribution in VirtualBox |

---

## Setting Up the Environment

Got comfortable with the Kali terminal and looked at the virtual machine's network and disks.

![ifconfig output](screenshots/setup-ifconfig.png)
*`ifconfig` shows the VM's network adapter (`eth0`) with its IP and hardware (MAC) address, plus the loopback interface.*

![Listing the home directory](screenshots/setup-ls-home.png)
*`ls` lists what's in my home directory.*

![Navigating and creating a directory](screenshots/setup-cd-mkdir.png)
*Moving into a folder with `cd`, creating a directory with `mkdir`, and confirming with `ls`. `lsw` isn't a real command, so Kali offered to install a package instead.*

![fdisk partition listing](screenshots/setup-fdisk.png)
*`sudo fdisk -l` lists the disks and partitions. Here it's the VM's 80 GiB virtual disk with one Linux partition.*

**What I learned:** The command line is the foundation for every tool that follows, and you should know your environment's network and disk layout before touching any evidence.

---

## Acquiring a Forensic Image with Guymager

![Creating an images directory](screenshots/guymager-mkdir-images.png)
*Created an `/images` directory to store the acquired image.*

![Guymager acquisition dialog](screenshots/guymager-acquire.png)
*Setting up the acquisition of `/dev/sdb` in Guymager: Expert Witness Format, case and evidence numbers, examiner name, destination folder, and SHA-256 hash calculation.*

**What I learned:** A good acquisition documents who collected the evidence and when, and the hash lets you prove later that the image hasn't changed.

---

## Analyzing a Disk Image with The Sleuth Kit and Autopsy

I analyzed the same disk image two ways: first with command-line tools, then with Autopsy.

![mmls partition layout](screenshots/sleuthkit-mmls.png)
*`mmls` shows the image's partition layout. There's one FAT partition starting at sector 2048.*

![fsstat file system details](screenshots/sleuthkit-fsstat.png)
*`fsstat` identified the file system as FAT12 and showed its layout. The "Cannot determine file system type" error at the top went away once I passed the partition's starting sector with `-o 2048`.*

![fls listing deleted files](screenshots/sleuthkit-fls.png)
*`fls` lists the files in the image. An asterisk marks deleted files, and the `-rdp` options show only the deleted ones.*

![Autopsy new case](screenshots/autopsy-new-case.png)
*Creating a new case in Autopsy with a case name and investigator.*

![Autopsy add host](screenshots/autopsy-add-host.png)
*Autopsy created a host for the case, and the next step is importing the image.*

![Autopsy add image](screenshots/autopsy-add-image.png)
*Pointing Autopsy at the image file and importing it with a symlink so the original isn't copied or moved.*

![Autopsy image details](screenshots/autopsy-image-details.png)
*Autopsy detected the partition and file system type (FAT12) from the image.*

![Autopsy host manager](screenshots/autopsy-host-manager.png)
*The image is loaded, and I selected the volume to analyze.*

![Autopsy file analysis](screenshots/autopsy-file-analysis.png)
*File analysis view. Deleted files show up in red with their timestamps, and I exported one of the JPEG files.*

![Exported file in Downloads](screenshots/autopsy-exported-file.png)
*The exported file showing up in my Downloads folder. My first `cd` failed because Linux paths are case-sensitive.*

**What I learned:** Deleted files often still exist in the file system's records, and the command-line tools and Autopsy show the same evidence in different ways.

---

## Windows Registry Analysis with RegRipper

I ran RegRipper on three Registry hives (SAM, SYSTEM, and SECURITY) and read the reports.

![RegRipper on the SAM hive](screenshots/regripper-sam.png)
*Setting up RegRipper with the SAM hive as input and a text file for the report.*

![SAM report](screenshots/regripper-sam-report.png)
*The SAM report lists user accounts. Built-in accounts like Administrator and Guest show as disabled with zero logins.*

![RegRipper on the SYSTEM hive](screenshots/regripper-system.png)
*Running the same process on the SYSTEM hive.*

![SYSTEM report](screenshots/regripper-system-report.png)
*The SYSTEM report includes system settings, like the app compatibility cache and which files are excluded from backups.*

![RegRipper on the SECURITY hive](screenshots/regripper-security.png)
*Finishing with the SECURITY hive.*

![SECURITY report](screenshots/regripper-security-report.png)
*The SECURITY report includes the audit policy, which shows auditing was not enabled on this system.*

**What I learned:** The Registry holds a lot of useful evidence about users and system settings, and RegRipper turns each hive into a readable report.

---

## Inspecting a Drive with a Hex Editor

![Installing hexedit](screenshots/hexedit-install.png)
*Installing `hexedit`.*

![Opening the drive in hexedit](screenshots/hexedit-open.png)
*Checking the disk layout, then opening the test drive's partition (`/dev/sdb1`) in the hex editor.*

![Boot sector in hex](screenshots/hexedit-boot-sector.png)
*The first sector of the FAT12 partition. The ASCII column shows `mkfs.fat` and a "This is not a bootable disk" message, and `55 AA` at the end of the sector is the boot signature.*

![Searching for JPG](screenshots/hexedit-search-jpg.png)
*Searching the drive for the ASCII string "JPG" to find the image files.*

![Directory entries in hex](screenshots/hexedit-directory-entries.png)
*The directory entries where the JPG files are listed. A deleted entry has its first character replaced with `E5`, which is why `_ECTURE2.JPG` looked different in Autopsy.*

**What I learned:** Everything the forensic tools showed me is stored as plain bytes, and reading them by hand explains how file systems track and delete files.

---

## Exploring CSI Linux

![CSI Linux desktop](screenshots/csi-linux-desktop.png)
*CSI Linux running in VirtualBox. The taskbar along the bottom has investigation tools built in, and there's a "Start a Case" shortcut on the desktop.*

![VirtualBox settings for the CSI Linux VM](screenshots/csi-linux-vm-settings.png)
*The VM's settings in VirtualBox: a 50 GB virtual disk and two network adapters, one NAT and one internal network.*

**What I learned:** Some Linux distributions ship with forensics and investigation tools already installed, which saves setup time compared to building a workstation from scratch.

---

## Hiding a Text File in an Image

I hid a small text file inside a JPEG using a Windows 10 VM, then pulled it back out.

![Files in the Downloads folder](screenshots/stego-downloads-files.png)
*Starting files: an image (`forensics.jpg`) and a short text file (`sample.txt`).*

![Contents of the text file](screenshots/stego-sample-text.png)
*The text I wanted to hide.*

![Zip file created](screenshots/stego-zip-created.png)
*Compressed the text file into `sample.zip`.*

![copy /b command](screenshots/stego-copy-b.png)
*Used `copy /b forensics.jpg+sample.zip forensics_II.jpg` to glue the zip onto the end of the image. (`ls` doesn't exist in Windows, so I used `dir`.)*

![Opening the new image in 7-Zip](screenshots/stego-7zip-open.png)
*7-Zip opens the new image as an archive. Its size is 13,188 bytes, which is the original image (13,030) plus the zip (158).*

![Hidden text recovered](screenshots/stego-hidden-text.png)
*The hidden `sample.txt` was inside the archive and opened normally.*

**What I learned:** A file can carry hidden data without looking any different when viewed as an image. Extra bytes after the end of a file, or a file size that doesn't add up, are clues an investigator can look for.

---

## Cracking a Zip File Password

![zip2john and John the Ripper](screenshots/zipcrack-zip2john-john.png)
*`zip2john` pulled the password hash out of a zip file I made and saved it to `hash.txt`. Then `john` ran it against its default wordlist and found the password.*

![Contents of the cracked zip](screenshots/zipcrack-cracked-contents.png)
*With the password, I opened the archive and read the file inside.*

**What I learned:** Short, common passwords fall quickly to a wordlist attack, so password length and randomness matter.

---

## Capturing a Login with Wireshark

I captured traffic between my Kali VM and a Metasploitable VM, which is an intentionally vulnerable machine used for practice.

![Metasploitable network settings](screenshots/wireshark-metasploitable-ifconfig.png)
*Checked the target VM's IP address with `ifconfig`. (My first attempt had a typo, so bash said the command wasn't found.)*

![Wireshark TCP stream showing the login prompt](screenshots/wireshark-login.png)
*Wireshark's "Follow TCP Stream" rebuilds a whole conversation. This is a Telnet session. Each character of the username shows up on its own line because Telnet sends keystrokes one at a time.*

![Wireshark TCP stream showing the password](screenshots/wireshark-password.png)
*The password is visible as plain text, followed by the successful login.*

**What I learned:** Telnet doesn't encrypt anything, so anyone capturing the traffic can read the credentials. That's a good reason to use encrypted protocols like SSH.

---

## Username Lookups with Sherlock

![Sherlock output](screenshots/sherlock-output.png)
*Sherlock checks a username against hundreds of websites and lists the ones where a profile page appears to exist.*

**What I learned:** A single username can connect someone's accounts across many sites, which is useful for investigators. The results still need to be verified by hand, since a page loading doesn't always mean the account is real.

---

## About Me

I'm a Computer Science student with a concentration in cybersecurity. I'm also working on a Cybersecurity Certificate and studying for CompTIA Security+. This repository is part of my public portfolio as I build toward a career in cybersecurity.

Feel free to [connect with me on LinkedIn](https://www.linkedin.com/in/andrew-meder33/) or check out my other repositories.
