# Solve the issue of Ubuntu 25.04 (Wifi, No Bluetooth, No sound)
There are three major issues when you update to Ubuntu 25.04, especially when you use the laptopn with Intel Ultra seriers Chips.

The three major issues are
1. Wifi cannot be detected automatically
   * You have to connect wifi manually everything when you access to the OS
2. Bluetooth is not functional
   * System cannot find bluetooth hardware
3. No sound
   * Have sound for the first 5 seconds and then no sound
   * No sound for all different ways, including speaker and earphone port
  
Notice: these issue specifically existed in the Asus laptop (e.g. viviobook s14 oled) with Intel Ultra seriers Chips (lunar lake architecture).

## Solution: Wifi cannot be detected automatically
1. We need to update kernal to "6.15.0-061500-generic"6.15.0-061500-generic"
```
xin@xin-Q423SA:~$ uname -a
Linux xin-Q423SA 6.15.0-061500-generic #202505260036 SMP PREEMPT_DYNAMIC Mon May 26 01:07:21 UTC 2025 x86_64 x86_64 x86_64 GNU/Linux
```

2. How to update kernal
```
✅ Method 1: Use the Ubuntu Mainline Kernel Installer

sudo apt install mainline
sudo add-apt-repository ppa:cappelikan/ppa
sudo apt update
sudo apt install mainline
mainline --install-latest
reboot
```
## Solution: Bluetooth is not functional
1: Manually Install the Missing Intel Firmware
```
Fix: Manually Install the Missing Intel Firmware

We'll install the missing .sfi and .ddc files from the official Linux firmware repository.
1. Open Terminal and Download the Firmware Files

cd /lib/firmware/intel/
sudo wget https://git.kernel.org/pub/scm/linux/kernel/git/firmware/linux-firmware.git/plain/intel/ibt-0190-0291-pci.sfi
sudo wget https://git.kernel.org/pub/scm/linux/kernel/git/firmware/linux-firmware.git/plain/intel/ibt-0190-0291-pci.ddc

2. Update Initramfs and Reboot

sudo update-initramfs -u
sudo reboot
```

## Solution: No sound
1. Creat manually create the firmware driver
```
```
 
2. Create the temporary HDA verb script and run it 
```
Use a Temporary HDA Verb Script

If the above methods don't work, you can try a temporary workaround using hda-verb:
Ask Ubuntu

    Install alsa-tools:

  sudo apt install alsa-tools

    Create the Script:

  sudo nano /usr/local/bin/fix-audio.sh

Paste the following content:
Bugzilla

  #!/bin/bash
  hda-verb /dev/snd/hwC0D0 0x20 0x500 0x1b
  hda-verb /dev/snd/hwC0D0 0x20 0x477 0x4a4b
  hda-verb /dev/snd/hwC0D0 0x20 0x500 0xf
  hda-verb /dev/snd/hwC0D0 0x20 0x477 0x74

Save and exit the editor.
Ask Ubuntu+1Ubuntu Community Hub+1

    Make the Script Executable:

  sudo chmod +x /usr/local/bin/fix-audio.sh

    Run the Script:

  sudo /usr/local/bin/fix-audio.sh

Note that this fix is temporary and may need to be reapplied after rebooting or switching between operating systems.
```


