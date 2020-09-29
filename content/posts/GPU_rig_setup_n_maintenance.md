---
title: "GPU_rig_setup_n_maintenance"
date: 2020-09-25T08:38:16+01:00
draft: false
toc: false
images:
tags:
  - hashcat
  - gpu
  - cracking
  - hash
---
A few days ago, a poor GPU rig, aka Ring0 (long story, don't ask), was full of sticky dust and (probably) dry thermal paste, so I decided that something had to be done; at the end of the day, Ring0 never did anything wrong to deserve that. Keeping in mind my decision, I started to tear down Ring0 aiming for a deep cleaning; fortunately [Dennis](https://www.linkedin.com/in/dennis-varischetti) was there to help making the whole process way quicker.  
  
Once Ring0 was nice and clean, wearing its new jumper (more details later), I realised that I had not upgraded it for a while and many new versions of Ubuntu Server have been released (was running on [Ubuntu Server 16.04](https://releases.ubuntu.com/16.04/). Knowing that, I decided that my Monday was not painful enough, so I brutally formatted the ssd, downloaded and installed [Ubuntu Server 20.04.1](https://ubuntu.com/download/server).  
  
I just want to specify that this rig was born as an ETH miner, not as a password cracking rig; for this reason, the CPU is not powerful enough to generate wordlists on the fly or just to mutate them using some rules. The CPU is a [i7-4790](https://ark.intel.com/content/www/us/en/ark/products/80806/intel-core-i7-4790-processor-8m-cache-up-to-4-00-ghz.html), which is one of the best CPUs for the  FCLGA1150, but still not enough; just to give you a (very) rough example, it can generate enough words to fill up little less than 4x 1070 cracking in `-m16800`. Fortunately, it is ok to run wordlists as they are on disk. As soon as I will become a real adult with a real job I might think about a [Threadripper](https://www.amd.com/en/products/cpu/amd-ryzen-threadripper-3990x) or a double CPU, we'll see.   
  
Oh right, almost forgot to say: cracking hashes without permission is illegal. Don't do illegal stuff, that's bad; and go to sleep early, it is good for your health.
___
## System Installation
I will skip the Ubuntu Server installation process, you can find the instructions pretty much [everywhere](https://ubuntu.com/tutorials/install-ubuntu-server#1-overview). Just one note: the GPU ring has NO SCREEN/DISPLAY attached, everything is done via ssh using the `root` user.

Vital thing first:
```bash
sed -i 's/#force_color_prompt=yes/force_color_prompt=yes/g' ~/.bashrc
source ~/.bashrc
```

Updating and Upgrading is always good EXCEPT on the Kali VM for the OSCP exam, but that's another [story](https://lapolis.blue/posts/2020/09/oscp_survival_101/).
```bash
apt-get update
apt-get upgrade
```
Since the last time I've done this process, lots of different flavours of the same Nvidia drivers have become available. Since I feel brave today I will go with the latest one: the [Nvidia 450](https://www.nvidia.com/Download/driverResults.aspx/160555/en-us). 
```bash
apt-get install xserver-xorg-video-nvidia-450-server \
				nvidia-headless-450-server \
				nvidia-utils-450-server \
				nvidia-settings \
				nvidia-cuda-toolkit
```
Once all of the Nvidia drivers are installed and since I did not find any other way to control the GPU fan speed without having a display manager running, I install [LightDM](https://wiki.ubuntu.com/LightDM). Please, if you know how it should be done correctly let me know.
```bash
apt-get install lightdm
```
The last thing before glory is to tell the Nvidia X driver how to behave. The config file can be generated using [nvidia-xconfig](http://manpages.ubuntu.com/manpages/precise/en/man1/alt-nvidia-current-updates-xconfig.1.html#description) and not painfully write it by hand.
```bash
nvidia-xconfig --allow-empty-initial-configuration \
				--enable-all-gpus \
				--cool-bits=4 # if you want to OC put 24
```
Maybe `reboot` at this point. And just in case, why not checking if all the GPUs are detected at this point?  

{{< image src="/img/gpu/nvidiasmi.png" alt="Ring0 jumper-naked" position="center" style="border-radius: 8px;" >}}  

Since I have different GPUs with different temps, I cannot do a nice and clean loop to start all the GPU's fans at the same speed, so I've done this simple bash script. Of course, the 2 GPUs at 80% are the [MSI](https://www.msi.com/Graphics-card/geforce-gtx-1070-gaming-x-8g) u.U .
```bash
#!/bin/bash

export DISPLAY=:0 
export XAUTHORITY=/var/run/lightdm/root/:0

sudo nvidia-settings -c :0 -a [gpu:0]/GPUFanControlState=1
sudo nvidia-settings -c :0 -a [fan:0]/GPUTargetFanSpeed=80

sudo nvidia-settings -c :0 -a [gpu:1]/GPUFanControlState=1
sudo nvidia-settings -c :0 -a [fan:1]/GPUTargetFanSpeed=65

sudo nvidia-settings -c :0 -a [gpu:2]/GPUFanControlState=1
sudo nvidia-settings -c :0 -a [fan:2]/GPUTargetFanSpeed=65

sudo nvidia-settings -c :0 -a [gpu:3]/GPUFanControlState=1
sudo nvidia-settings -c :0 -a [fan:3]/GPUTargetFanSpeed=80
```
___
## Hashcat Installation
Well, we all knew that [Hashcat](https://hashcat.net/hashcat/) was on today's menu. This is a fairly quick process.
```bash
git clone https://github.com/hashcat/hashcat.git
cd hashcat
make
```
And finally!
```bash
./hashcat --benchmark -w4 -O
```
*Results at the bottom.*  
___
## Physical cleaning
All the cleaning was done using [Isopropyl Alcohol (99%)](https://www.amazon.co.uk/gp/product/B079YVPZDF/ref=ppx_yo_dt_b_search_asin_title?ie=UTF8&psc=1) and random Microfibre Cleaning Cloths; where the cloths were not available, we used a soft toothbrush to gently (not really :P) remove all the dust trying to stay with Ring0 forever. The thermal paste we used is a classic [ARCTIC MX-4](https://www.amazon.co.uk/gp/product/B07LF66ZSV/ref=ppx_yo_dt_b_search_asin_title?ie=UTF8&psc=1).
The whole thing was a mess, poor Ring0.  

{{< image src="/img/gpu/mobo.gif" alt="Mobo Dusty-Clean" position="center" style="border-radius: 8px;" >}}  

The GPUs collected a lot of dust, but it is always nice to see the final results.  
After a couple of hours fighting against the effect of the Isopropyl Alcohol inhalation, everything started to look right again.  

{{< image src="/img/gpu/msi.gif" alt="MSI Dusty-Clean" position="center" style="border-radius: 8px;" >}}  
{{< image src="/img/gpu/msi_evga.gif" alt="MSI_n_EVGA Dusty-Clean" position="center" style="border-radius: 8px;" >}}  

Since Ring0 is not running 24/7, the lovely owner of [Lunatic_pin](https://www.lunaticpin.com/), [Lisa](https://www.linkedin.com/in/lisa-polesello-4b900a180), created a nice total black on-size jumper. YOu can also find her on [Instagram](https://www.instagram.com/lunatic_pin/)!  

{{< image src="/img/gpu/ring0.gif" alt="Ring0 jumper-naked" position="center" style="border-radius: 8px;" >}}  

Useless to say, now Ring0 is happy, clean and finally protected. If anyone was wondering why 2 GPUs are missing, well, I wanted to play GTA V in 4k.
___
___
```
Hashmode: 0 - MD5

Speed.#1.........: 20816.9 MH/s (47.95ms) @ Accel:64 Loops:1024 Thr:1024 Vec:8
Speed.#2.........: 21196.2 MH/s (47.12ms) @ Accel:64 Loops:1024 Thr:1024 Vec:8
Speed.#3.........: 19932.4 MH/s (49.87ms) @ Accel:64 Loops:1024 Thr:1024 Vec:8
Speed.#4.........: 20119.0 MH/s (49.44ms) @ Accel:64 Loops:1024 Thr:1024 Vec:8
Speed.#*.........: 82064.4 MH/s

Hashmode: 100 - SHA1

Speed.#1.........:  6953.4 MH/s (144.29ms) @ Accel:64 Loops:1024 Thr:1024 Vec:1
Speed.#2.........:  7091.4 MH/s (141.47ms) @ Accel:64 Loops:1024 Thr:1024 Vec:1
Speed.#3.........:  6755.1 MH/s (148.62ms) @ Accel:64 Loops:1024 Thr:1024 Vec:1
Speed.#4.........:  6841.9 MH/s (146.60ms) @ Accel:64 Loops:1024 Thr:1024 Vec:1
Speed.#*.........: 27641.9 MH/s

Hashmode: 1400 - SHA2-256

Speed.#1.........:  2531.5 MH/s (397.27ms) @ Accel:64 Loops:1024 Thr:1024 Vec:1
Speed.#2.........:  2582.3 MH/s (389.46ms) @ Accel:64 Loops:1024 Thr:1024 Vec:1
Speed.#3.........:  2497.3 MH/s (402.74ms) @ Accel:64 Loops:1024 Thr:1024 Vec:1
Speed.#4.........:  2519.8 MH/s (399.06ms) @ Accel:64 Loops:1024 Thr:1024 Vec:1
Speed.#*.........: 10130.8 MH/s

Hashmode: 1700 - SHA2-512

Speed.#1.........:   858.2 MH/s (292.82ms) @ Accel:64 Loops:256 Thr:1024 Vec:1
Speed.#2.........:   875.6 MH/s (287.06ms) @ Accel:64 Loops:256 Thr:1024 Vec:1
Speed.#3.........:   847.9 MH/s (296.26ms) @ Accel:64 Loops:256 Thr:1024 Vec:1
Speed.#4.........:   854.0 MH/s (294.33ms) @ Accel:64 Loops:256 Thr:1024 Vec:1
Speed.#*.........:  3435.6 MH/s

Hashmode: 22000 - WPA-PBKDF2-PMKID+EAPOL (Iterations: 4095)

Speed.#1.........:   374.3 kH/s (327.24ms) @ Accel:64 Loops:512 Thr:1024 Vec:1
Speed.#2.........:   380.3 kH/s (322.02ms) @ Accel:64 Loops:512 Thr:1024 Vec:1
Speed.#3.........:   354.1 kH/s (345.86ms) @ Accel:64 Loops:512 Thr:1024 Vec:1
Speed.#4.........:   356.5 kH/s (343.62ms) @ Accel:64 Loops:512 Thr:1024 Vec:1
Speed.#*.........:  1465.2 kH/s

Hashmode: 1000 - NTLM

Speed.#1.........: 37338.5 MH/s (26.63ms) @ Accel:64 Loops:1024 Thr:1024 Vec:8
Speed.#2.........: 37776.7 MH/s (26.19ms) @ Accel:64 Loops:1024 Thr:1024 Vec:8
Speed.#3.........: 34536.9 MH/s (28.30ms) @ Accel:64 Loops:1024 Thr:1024 Vec:8
Speed.#4.........: 34923.0 MH/s (28.15ms) @ Accel:64 Loops:1024 Thr:1024 Vec:8
Speed.#*.........:   144.6 GH/s

Hashmode: 3000 - LM

Speed.#1.........: 16593.0 MH/s (60.02ms) @ Accel:1024 Loops:1024 Thr:64 Vec:1
Speed.#2.........: 16864.5 MH/s (58.99ms) @ Accel:1024 Loops:1024 Thr:64 Vec:1
Speed.#3.........: 16692.8 MH/s (59.55ms) @ Accel:1024 Loops:1024 Thr:64 Vec:1
Speed.#4.........: 16810.2 MH/s (59.07ms) @ Accel:1024 Loops:1024 Thr:64 Vec:1
Speed.#*.........: 66960.6 MH/s

Hashmode: 5500 - NetNTLMv1 / NetNTLMv1+ESS

Speed.#1.........: 19982.4 MH/s (49.91ms) @ Accel:64 Loops:1024 Thr:1024 Vec:2
Speed.#2.........: 20447.9 MH/s (48.92ms) @ Accel:64 Loops:1024 Thr:1024 Vec:2
Speed.#3.........: 19200.5 MH/s (51.67ms) @ Accel:64 Loops:1024 Thr:1024 Vec:2
Speed.#4.........: 19352.6 MH/s (51.14ms) @ Accel:64 Loops:1024 Thr:1024 Vec:2
Speed.#*.........: 78983.4 MH/s

Hashmode: 5600 - NetNTLMv2

Speed.#1.........:  1433.6 MH/s (350.64ms) @ Accel:64 Loops:512 Thr:1024 Vec:1
Speed.#2.........:  1452.8 MH/s (346.00ms) @ Accel:64 Loops:512 Thr:1024 Vec:1
Speed.#3.........:  1399.4 MH/s (359.19ms) @ Accel:64 Loops:512 Thr:1024 Vec:1
Speed.#4.........:  1403.2 MH/s (358.24ms) @ Accel:64 Loops:512 Thr:1024 Vec:1
Speed.#*.........:  5689.1 MH/s

Hashmode: 1500 - descrypt, DES (Unix), Traditional DES

Speed.#1.........:   694.8 MH/s (361.55ms) @ Accel:256 Loops:1024 Thr:64 Vec:1
Speed.#2.........:   707.6 MH/s (355.05ms) @ Accel:256 Loops:1024 Thr:64 Vec:1
Speed.#3.........:   699.2 MH/s (359.26ms) @ Accel:256 Loops:1024 Thr:64 Vec:1
Speed.#4.........:   700.7 MH/s (358.48ms) @ Accel:256 Loops:1024 Thr:64 Vec:1
Speed.#*.........:  2802.3 MH/s

Hashmode: 500 - md5crypt, MD5 (Unix), Cisco-IOS $1$ (MD5) (Iterations: 1000)

Speed.#1.........:  8001.8 kH/s (118.10ms) @ Accel:64 Loops:1000 Thr:1024 Vec:1
Speed.#2.........:  8153.1 kH/s (115.78ms) @ Accel:64 Loops:1000 Thr:1024 Vec:1
Speed.#3.........:  7725.8 kH/s (122.62ms) @ Accel:64 Loops:1000 Thr:1024 Vec:1
Speed.#4.........:  7808.7 kH/s (121.27ms) @ Accel:64 Loops:1000 Thr:1024 Vec:1
Speed.#*.........: 31689.4 kH/s

Hashmode: 3200 - bcrypt $2*$, Blowfish (Unix) (Iterations: 32)

Speed.#1.........:    17098 H/s (163.72ms) @ Accel:32 Loops:16 Thr:12 Vec:1
Speed.#2.........:    17368 H/s (161.13ms) @ Accel:32 Loops:16 Thr:12 Vec:1
Speed.#3.........:    17717 H/s (158.00ms) @ Accel:32 Loops:16 Thr:12 Vec:1
Speed.#4.........:    17774 H/s (157.65ms) @ Accel:32 Loops:16 Thr:12 Vec:1
Speed.#*.........:    69957 H/s

Hashmode: 1800 - sha512crypt $6$, SHA512 (Unix) (Iterations: 5000)

Speed.#1.........:   123.7 kH/s (393.19ms) @ Accel:16 Loops:1024 Thr:1024 Vec:1
Speed.#2.........:   125.7 kH/s (386.89ms) @ Accel:16 Loops:1024 Thr:1024 Vec:1
Speed.#3.........:   118.6 kH/s (410.28ms) @ Accel:16 Loops:1024 Thr:1024 Vec:1
Speed.#4.........:   119.4 kH/s (407.55ms) @ Accel:16 Loops:1024 Thr:1024 Vec:1
Speed.#*.........:   487.3 kH/s

Hashmode: 7500 - Kerberos 5, etype 23, AS-REQ Pre-Auth

Speed.#1.........:   235.9 MH/s (266.13ms) @ Accel:512 Loops:128 Thr:64 Vec:1
Speed.#2.........:   242.0 MH/s (259.54ms) @ Accel:512 Loops:128 Thr:64 Vec:1
Speed.#3.........:   246.8 MH/s (254.35ms) @ Accel:512 Loops:128 Thr:64 Vec:1
Speed.#4.........:   245.1 MH/s (256.14ms) @ Accel:512 Loops:128 Thr:64 Vec:1
Speed.#*.........:   969.8 MH/s

Hashmode: 13100 - Kerberos 5, etype 23, TGS-REP

Speed.#1.........:   235.4 MH/s (266.75ms) @ Accel:512 Loops:128 Thr:64 Vec:1
Speed.#2.........:   241.7 MH/s (259.74ms) @ Accel:512 Loops:128 Thr:64 Vec:1
Speed.#3.........:   245.2 MH/s (255.98ms) @ Accel:512 Loops:128 Thr:64 Vec:1
Speed.#4.........:   247.1 MH/s (254.09ms) @ Accel:512 Loops:128 Thr:64 Vec:1
Speed.#*.........:   969.4 MH/s

Hashmode: 15300 - DPAPI masterkey file v1 (Iterations: 23999)

Speed.#1.........:    63700 H/s (327.20ms) @ Accel:64 Loops:512 Thr:1024 Vec:1
Speed.#2.........:    64290 H/s (323.94ms) @ Accel:64 Loops:512 Thr:1024 Vec:1
Speed.#3.........:    60097 H/s (346.28ms) @ Accel:64 Loops:512 Thr:1024 Vec:1
Speed.#4.........:    60523 H/s (344.09ms) @ Accel:64 Loops:512 Thr:1024 Vec:1
Speed.#*.........:   248.6 kH/s

Hashmode: 15900 - DPAPI masterkey file v2 (Iterations: 12899)

Speed.#1.........:    27856 H/s (349.57ms) @ Accel:8 Loops:1024 Thr:1024 Vec:1
Speed.#2.........:    28341 H/s (343.56ms) @ Accel:8 Loops:1024 Thr:1024 Vec:1
Speed.#3.........:    27382 H/s (355.39ms) @ Accel:8 Loops:1024 Thr:1024 Vec:1
Speed.#4.........:    27491 H/s (354.03ms) @ Accel:8 Loops:1024 Thr:1024 Vec:1
Speed.#*.........:   111.1 kH/s

Hashmode: 7100 - macOS v10.8+ (PBKDF2-SHA512) (Iterations: 1023)

Speed.#1.........:   345.9 kH/s (311.52ms) @ Accel:64 Loops:127 Thr:1024 Vec:1
Speed.#2.........:   351.2 kH/s (306.82ms) @ Accel:64 Loops:127 Thr:1024 Vec:1
Speed.#3.........:   319.8 kH/s (378.75ms) @ Accel:8 Loops:1023 Thr:1024 Vec:1
Speed.#4.........:   339.8 kH/s (356.22ms) @ Accel:8 Loops:1023 Thr:1024 Vec:1
Speed.#*.........:  1356.7 kH/s

Hashmode: 11600 - 7-Zip (Iterations: 16384)

Speed.#1.........:   284.2 kH/s (378.10ms) @ Accel:32 Loops:4096 Thr:1024 Vec:1
Speed.#2.........:   291.2 kH/s (371.52ms) @ Accel:32 Loops:4096 Thr:1024 Vec:1
Speed.#3.........:   277.2 kH/s (390.50ms) @ Accel:32 Loops:4096 Thr:1024 Vec:1
Speed.#4.........:   278.2 kH/s (387.37ms) @ Accel:32 Loops:4096 Thr:1024 Vec:1
Speed.#*.........:  1130.9 kH/s

Hashmode: 12500 - RAR3-hp (Iterations: 262144)

Speed.#1.........:    38832 H/s (395.21ms) @ Accel:128 Loops:16384 Thr:128 Vec:1
Speed.#2.........:    38872 H/s (394.57ms) @ Accel:128 Loops:16384 Thr:128 Vec:1
Speed.#3.........:    38727 H/s (396.13ms) @ Accel:128 Loops:16384 Thr:128 Vec:1
Speed.#4.........:    38817 H/s (395.36ms) @ Accel:128 Loops:16384 Thr:128 Vec:1
Speed.#*.........:   155.2 kH/s

Hashmode: 13000 - RAR5 (Iterations: 32799)

Speed.#1.........:    30698 H/s (249.43ms) @ Accel:16 Loops:1024 Thr:1024 Vec:1
Speed.#2.........:    31269 H/s (244.84ms) @ Accel:16 Loops:1024 Thr:1024 Vec:1
Speed.#3.........:    30535 H/s (500.91ms) @ Accel:32 Loops:1024 Thr:1024 Vec:1
Speed.#4.........:    30485 H/s (501.72ms) @ Accel:32 Loops:1024 Thr:1024 Vec:1
Speed.#*.........:   123.0 kH/s

Hashmode: 6211 - TrueCrypt RIPEMD160 + XTS 512 bit (Iterations: 1999)

Speed.#1.........:   225.6 kH/s (255.81ms) @ Accel:8 Loops:1024 Thr:1024 Vec:1
Speed.#2.........:   229.8 kH/s (251.06ms) @ Accel:8 Loops:1024 Thr:1024 Vec:1
Speed.#3.........:   220.1 kH/s (262.32ms) @ Accel:8 Loops:1024 Thr:1024 Vec:1
Speed.#4.........:   223.3 kH/s (258.53ms) @ Accel:8 Loops:1024 Thr:1024 Vec:1
Speed.#*.........:   898.8 kH/s

Hashmode: 13400 - KeePass 1 (AES/Twofish) and KeePass 2 (AES) (Iterations: 24569)

Speed.#1.........:    25895 H/s (790.25ms) @ Accel:32 Loops:1024 Thr:1024 Vec:1
Speed.#2.........:    26263 H/s (779.14ms) @ Accel:32 Loops:1024 Thr:1024 Vec:1
Speed.#3.........:    26586 H/s (769.22ms) @ Accel:64 Loops:512 Thr:1024 Vec:1
Speed.#4.........:    26744 H/s (764.51ms) @ Accel:64 Loops:512 Thr:1024 Vec:1
Speed.#*.........:   105.5 kH/s

Hashmode: 6800 - LastPass + LastPass sniffed (Iterations: 499)

Speed.#1.........:  1925.8 kH/s (242.26ms) @ Accel:32 Loops:499 Thr:1024 Vec:1
Speed.#2.........:  1962.1 kH/s (237.62ms) @ Accel:32 Loops:499 Thr:1024 Vec:1
Speed.#3.........:  1898.3 kH/s (493.12ms) @ Accel:64 Loops:499 Thr:1024 Vec:1
Speed.#4.........:  1909.5 kH/s (489.92ms) @ Accel:64 Loops:499 Thr:1024 Vec:1
Speed.#*.........:  7695.7 kH/s

Hashmode: 11300 - Bitcoin/Litecoin wallet.dat (Iterations: 200459)

Speed.#1.........:     3855 H/s (325.09ms) @ Accel:16 Loops:1024 Thr:1024 Vec:1
Speed.#2.........:     3912 H/s (319.03ms) @ Accel:16 Loops:1024 Thr:1024 Vec:1
Speed.#3.........:     3770 H/s (332.43ms) @ Accel:16 Loops:1024 Thr:1024 Vec:1
Speed.#4.........:     3813 H/s (328.73ms) @ Accel:16 Loops:1024 Thr:1024 Vec:1
Speed.#*.........:    15351 H/s
```