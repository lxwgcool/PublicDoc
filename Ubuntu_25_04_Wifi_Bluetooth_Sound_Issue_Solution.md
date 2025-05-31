# Solve the issue of Ubuntu 25.04
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
We need to update kernal to "6.15.0-061500-generic"6.15.0-061500-generic
```
xin@xin-Q423SA:~$ uname -a
Linux xin-Q423SA 6.15.0-061500-generic #202505260036 SMP PREEMPT_DYNAMIC Mon May 26 01:07:21 UTC 2025 x86_64 x86_64 x86_64 GNU/Linux
```
How to update kernal
```

```
