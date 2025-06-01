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
1. Find, download and put the correct firmware driver into your laptop
```
sof-lnl.ri
sof-lnl.tplg

1: sof-lnl.ri can be found in:
https://github.com/thesofproject/sof-bin/tree/main/v2.12.x/sof-ipc4-v2.12/lnl

2: sof-lnl.tplg can be found in (find the one that fit your sound card)
sudo wget https://github.com/thesofproject/sof-bin/raw/main/v2.12.x/sof-ipc4-tplg-v2.12.1/sof-lnl-cs42l43-l0-cs35l56-l23-2ch.tplg
```

2: copy file to the system level directory
```
1. sof-lnl.tplg
/lib/firmware/intel/sof-ipc4-tplg/

2. sof-lnl.ri (this the one the system level topology file need)
/lib/firmware/intel/sof-ipc4/lnl

3: update system lib
sudo update-initramfs -u
```

3. Make sure the correct "Firmware file" and "Topology file" be linked
```
xin@xin-Q423SA:~$ sudo dmesg | grep -i  'sof-audio-pci-intel-lnl'
[    3.522578] sof-audio-pci-intel-lnl 0000:00:1f.3: enabling device (0000 -> 0002)
[    3.522698] sof-audio-pci-intel-lnl 0000:00:1f.3: DSP detected with PCI class/subclass/prog-if 0x040100
[    3.924606] sof-audio-pci-intel-lnl 0000:00:1f.3: bound 0000:00:02.0 (ops intel_audio_component_bind_ops [xe])
[    3.931794] sof-audio-pci-intel-lnl 0000:00:1f.3: use msi interrupt mode
[    3.952402] sof-audio-pci-intel-lnl 0000:00:1f.3: hda codecs found, mask 5
[    3.952411] sof-audio-pci-intel-lnl 0000:00:1f.3: using HDA machine driver skl_hda_dsp_generic now
[    3.952413] sof-audio-pci-intel-lnl 0000:00:1f.3: NHLT device BT(0) detected, ssp_mask 0x4
[    3.952415] sof-audio-pci-intel-lnl 0000:00:1f.3: BT link detected in NHLT tables: 0x4
[    3.952416] sof-audio-pci-intel-lnl 0000:00:1f.3: DMICs detected in NHLT tables: 2
[    3.955935] sof-audio-pci-intel-lnl 0000:00:1f.3: Firmware paths/files for ipc type 1:
[    3.955944] sof-audio-pci-intel-lnl 0000:00:1f.3:  Firmware file:     intel/sof-ipc4/lnl/sof-lnl.ri
[    3.955946] sof-audio-pci-intel-lnl 0000:00:1f.3:  Firmware lib path: intel/sof-ipc4-lib/lnl
[    3.955948] sof-audio-pci-intel-lnl 0000:00:1f.3:  Topology file:     intel/sof-ipc4-tplg/sof-hda-generic-2ch.tplg
[    3.957558] sof-audio-pci-intel-lnl 0000:00:1f.3: Loaded firmware library: ADSPFW, version: 2.12.0.1
[    4.232304] sof-audio-pci-intel-lnl 0000:00:1f.3: Booted firmware version: 2.12.0.1
[    4.241812] sof-audio-pci-intel-lnl 0000:00:1f.3: Topology: ABI 3:29:1 Kernel ABI 3:23:1
[    4.243092] sof-audio-pci-intel-lnl 0000:00:1f.3: Loaded firmware library: ADSPFW, version: 2.12.0.1
```

4. Creat manually create the firmware driver
```
sudo nano /lib/firmware/alc294-sound-patch.fw
sudo nano /etc/modprobe.d/alsa-base.conf
reboot
```
  * alc294-sound-patch.fw
```
  [codec]
  0x10ec0294 0x1043194e 0

  [pincfg]
  0x19 0x03a11050
  0x1a 0x03a11c30
  0x21 0x03211420

  [verb]
  0x20 0x500 0x62
  0x20 0x400 0xa007
  0x20 0x500 0x10
  0x20 0x400 0x8420
  0x20 0x500 0x0f
  0x20 0x400 0x7774
```
  * alsa-base.conf
```
# autoloader aliases
install sound-slot-0 /sbin/modprobe snd-card-0
install sound-slot-1 /sbin/modprobe snd-card-1
install sound-slot-2 /sbin/modprobe snd-card-2
install sound-slot-3 /sbin/modprobe snd-card-3
install sound-slot-4 /sbin/modprobe snd-card-4
install sound-slot-5 /sbin/modprobe snd-card-5
install sound-slot-6 /sbin/modprobe snd-card-6
install sound-slot-7 /sbin/modprobe snd-card-7

# Cause optional modules to be loaded above generic modules
install snd /sbin/modprobe --ignore-install snd $CMDLINE_OPTS && { /sbin/modprobe --quiet --use-blacklist snd-ioctl32 ; /sbin/modprobe --quiet --use-blacklist snd-seq ; }
#
# Workaround at bug #499695 (reverted in Ubuntu see LP #319505)
install snd-pcm /sbin/modprobe --ignore-install snd-pcm $CMDLINE_OPTS && { /sbin/modprobe --quiet --use-blacklist snd-pcm-oss ; : ; }
install snd-mixer /sbin/modprobe --ignore-install snd-mixer $CMDLINE_OPTS && { /sbin/modprobe --quiet --use-blacklist snd-mixer-oss ; : ; }
install snd-seq /sbin/modprobe --ignore-install snd-seq $CMDLINE_OPTS && { /sbin/modprobe --quiet --use-blacklist snd-seq-midi ; /sbin/modprobe --quiet --use-blacklist snd-seq-oss ; : ; }
#
install snd-rawmidi /sbin/modprobe --ignore-install snd-rawmidi $CMDLINE_OPTS && { /sbin/modprobe --quiet --use-blacklist snd-seq-midi ; : ; }
# Cause optional modules to be loaded above sound card driver modules
install snd-emu10k1 /sbin/modprobe --ignore-install snd-emu10k1 $CMDLINE_OPTS && { /sbin/modprobe --quiet --use-blacklist snd-emu10k1-synth ; }
install snd-via82xx /sbin/modprobe --ignore-install snd-via82xx $CMDLINE_OPTS && { /sbin/modprobe --quiet --use-blacklist snd-seq ; }

# Load saa7134-alsa instead of saa7134 (which gets dragged in by it anyway)
install saa7134 /sbin/modprobe --ignore-install saa7134 $CMDLINE_OPTS && { /sbin/modprobe --quiet --use-blacklist saa7134-alsa ; : ; }
# Prevent abnormal drivers from grabbing index 0
options bt87x index=-2
options cx88_alsa index=-2
options saa7134-alsa index=-2
options snd-atiixp-modem index=-2
options snd-intel8x0m index=-2
options snd-via82xx-modem index=-2
options snd-usb-audio index=-2
options snd-usb-caiaq index=-2
options snd-usb-ua101 index=-2
options snd-usb-us122l index=-2
options snd-usb-usx2y index=-2
# Ubuntu #62691, enable MPU for snd-cmipci
options snd-cmipci mpu_port=0x330 fm_port=0x388
# Keep snd-pcsp from being loaded as first soundcard
options snd-pcsp index=-2
# Keep snd-usb-audio from beeing loaded as first soundcard
options snd-usb-audio index=-2
options snd-hda-intel patch=alc294-sound-patch.fw
```
 
5. Create the temporary HDA verb script and run it 
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

**Notice**: Actually once it works, you do not have run "fix-audio.sh" everytime.


