# Fedora on the Metabox Flo
I decided to capture my thoughts and optimisations that I have for running Fedora on a Metabox Flo laptop. 

### My use case
I bough this laptop because it is light, very light 1kg. Also, you can spec it out to be a neat developer laptop. 

Metabox has a long history of selling gaming laptops in Australia. Assembly, QA and shipping took about 7 business days.

The **Metabox Flo L140c** is based on the **Clevo L140cu** chassis. This is the same chassis that System76 uses for their 2020 **Lemur Pro** laptop. So you would expect Linux support to be good.

**Specs**
- Intel i5 10th generation
- 40GB of RAM
- 1TB NVME
- Screen size: 14 inch, res 1920x1080
- Weight as configured: 1074 grams (yep that is all)
- Size: it is physically about the same size as my 2017 13inch Macbook Pro

## What do I like
- Fedora 32 on it just works 
- Fedora 32 on it works very well with some minor optimisations
- Touchpad surprised me - it is very good. Better than what I used on a Lenovo T540.
- Construction seems good
- Battery life is excellent (see customisations below)
- Pricewise the combination of portability, high specs and batter life is very hard to beat

## What could be better
- Good thing I did not get it for the speakers. They are very tinny. 
- Web Cam is not great 720P only again it does the job
- Page Up/Down buttons is very close to left and right arrow keys. I will probably remap them to left and right to avoid accidentally paging up/down

# Setup
## Enter BIOS
To enter the BIOS hit F2

## Set AHCI
In the bios set storage to AHCI rather than RST. Fedora will not see NVME drive otherwise.

## Battery Usage Laptop Optimisation - All day usage!
There are some simple battery optemisations. 
- Install TLP, you can also use powertop.
- I use this Opera Web Browser with Battery Saver Mode. You can do some optimisation in other browsers as well, but it is hard to beat setting an option when unplugging.

## Fix battery drain on suspend
I did encounter one issue and that was what I would consider severe battery drain while the latop was suspended. It turns out it went into suspend state ***s2idle*** and not into S3 which is a deeper suspend state. 

The fix was easy.

Verify suspend state. 
```bash
sudo journalctl | grep "PM: suspend" | tail
May 26 15:01:05 localhost-live kernel: PM: suspend entry (deep)
May 26 15:04:49 localhost-live kernel: PM: suspend devices took 1.061 seconds
May 26 15:04:49 localhost-live kernel: PM: suspend exit
```

or try verify setting as such: Not what you want
```bash
cat /sys/power/mem_sleep
[s2idle] deep
```

If it says ***entry (deep)*** then you are good. If not apply the next steps.

**The Fix***
Add mem_sleep_default=deep to GRUB_CMDLINE_LINUX in /etc/default/grub:
* you may also want to fix bluetooth issues with mouse going to sleep *if you have it: btusb.enable_autosuspend=0
```bash
GRUB_CMDLINE_LINUX="resume=/dev/mapper/frink-swap rd.lvm.lv=frink/root rd.lvm.lv=frink/swap crashkernel=128M mem_sleep_default=deep rhgb quiet btusb.enable_autosuspend=0"
```


Regenerate grub.cfg file (the following command is for UEFI installations):
Fedora:
```bash
grub2-mkconfig -o /boot/efi/EFI/fedora/grub.cfg
```
Arch
```bash
sudo grub-mkconfig -o /boot/grub/grub.cfg
```

Make sure that the grub path in the last command is correct. 

Reboot and test

# Face ID login - not working on Fedora
You can install Howdy depends on how secure you think it is. I have not been able to get it to work in Fedora 32 yet.

https://github.com/boltgolt/howdy
