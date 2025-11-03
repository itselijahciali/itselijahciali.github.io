<img width="1852" height="700" alt="image" src="https://github.com/user-attachments/assets/6a7c594d-e036-4c2a-a193-db777be04519" /># How to install Arch, by the way

# Installation Guide

## Acquiring Images

Since I use a laptop with Apple Silicon, all virtual machines are required to share the `AArch64-ARMv8` (no relation) architecture. There are some applications that allow for running `x86-64` images, but this is classified as emulation instead of virtualization and isn’t practical to run a full guest OS under.

Unfortunately, Arch Linux does not supply any official ARM-based images so I’m on my own here. Trying to avoid a full LFS build, I found a link suggested in the Harvey page for archlinuxarm.com. After figuring out which download was intended for my platform (`ARMv8`), I acquired the image `ArchLinuxARM-aarch64-latest.tar.gz`. Their web server doesn’t seem to be built for downloads as I had to manually complete the download by renaming the `.crdownload` file, but I quickly found that even the provided image was a dead end. Instead of a bootable `.iso` image, I was provided with an **archived filesystem**. I quickly realized by looking at the footer of the page that I was assumed to **provide my own kernel** for the system and that the Arch Linux ARM project was intended to be a group of packages built for easier installation of Arch on ARM.

Missed chance to call it Arch on AArch, by the way. Looking for another option I found [archboot.com](http://archboot.com) which supplied a full ISO image for Arch.

## VM Creation

VMWare doesn’t have a native “Arch Linux” option but rather a generic “Other Linux 6.x kernel 64-bit Arm” option. After selecting that I configured the VM to have a 20GB hard drive, and 8GB of RAM. I then supplied the tactically acquired ISO image to the VM to boot.

## Booting Arch

Archboot has many more GRUB options for IPXE boot and other methods. I choose the standard boot option and it takes me to an interactive shell. From here, I should be able to continue an install like usual. Just to be sure though, I ran a `ping` command to check if the VM and install were configured for internet access.

![Screenshot 2025-10-23 at 2.04.51 PM.png](https://github.com/itselijahciali/itselijahciali.github.io/blob/main/Screenshot_2025-10-23_at_2.04.51_PM.png?raw=true)

## Setting up the preinstallation environment

Some basic steps are configured for me:

- **Set keymap:** US default is fine
- **Check boot mode:** GRUB confirms UEFI boot mode
- **Internet setup:** Ping confirms good connection
- **Time and date setup:** `timedatectl` returns the correct time, as the time is automatically synced when a connection is made

## Partitioning disk

VMWare’s 20GB drive I made is recognized by the `fdisk -l` command:

![Screenshot 2025-10-23 at 2.11.50 PM.png](https://github.com/itselijahciali/itselijahciali.github.io/blob/main/Screenshot_2025-10-23_at_2.11.50_PM.png?raw=true)

Arch and most Linux distributions expect several partitions, being **root**, **swap**, and **efi**. I will make the last two first and then use the remaining size to make the root partition.

Running `fdisk /dev/nvme0n1` puts me in an interactive partitioning tool.

I press the **g** key to create a GPT table, **n** to create the first partition, and give it a size of **+512M**. I then hit **t** and **1** to give it the label “EFI system”.

I create another **4G** partition and give it the **19** type to indicate “Linux swap”.

Finally, I use the remaining space to create the Linux filesystem and press **w** to write this partition table.

## Formatting Partitions

Using `fdisk -l` as a reference for device names, I can run `mkfs.ext4 /dev/nvme0n1p3` to format the Linux filesystem. I also run `mkswap /dev/nvme0n1p2` and `mkfs.fat -F 32 /dev/nvme0n1p1` to correctly format the other 2 partitions.

## Mounting

![Screenshot 2025-10-28 at 2.36.11 PM.png](https://github.com/itselijahciali/itselijahciali.github.io/blob/main/Screenshot_2025-10-28_at_2.36.11_PM.png?raw=true)

I can now mount the 2 filesystems and enable the swap partition.

## Installing Packages

After checking `/etc/pacman.d/mirrorlist` to ensure mirrors are present, I can bootstrap the base packages into the system.

Running `pacstrap -K /mnt base networkmanager vi linux linux-firmware`  gives me all the nessecary packages. This installs the **base Arch Linux System** as well as the **Linux kernel** and firmware. The firmware is necessary even though we’re in a VM since VMWare emulates the hardware Linux would need firmware for. I also install `vi` so I can edit configuration files later. (I will later regret not choosing **vim** or **nano**)

Running this gave me a network error and `ping [google.com](http://google.com)` resulted in failure. After a reboot and remount, the package install worked find, but this is not a good omen for **archboot**.

## System Configuration

First, I create an `fstab` file. Since everything is already mounted we can generate this using `genfstab` pointed at the `/mnt` directory. My final command is `genfstab -U /mnt >> /mnt/etc/fstab`

Next, I can **chroot** into the new system by running `arch-chroot /mnt`

I can set the time zone and clock by running `ln -sf /usr/share/zoneinfo/US/Central /etc/localtime` and `hwclock -—systohc`

I used `vi` to uncomment the `en_US.UTF-8` locale in `/etc/locale.gen` and then ran `localegen` as well as creating the file `/etc/locale.conf` which I could then edit in the `LANG=en_US.UTF-8` variable.

I then create a `/etc/hostname` file and give the system the hostname `archie`. I’m sure this will be a problem later because of how original my name is but we’ll cross that bridge later.

Finally, I run `passwd` to create the root password. The password is totally not *password*.

## Installing GRUB

You know GRUB, I know GRUB, we’re using GRUB. Assuming it’s not cheating I’m going to go exit my chroot for a second and install the `grub` and `efibootmgr` packages into my system.

The installation guide presents a command that does not work on my system, but a small modification does fix it. Running `grub-install -—target=arm64-efi -—bootloader-id=grub -—efi-directory=/boot` and the subsequent configuration file results in no errors.

# I GAVE UP – Or, how I ended up using a ThinkPad

So you just read that last sentence, right? The one that had the words “no errors”– turns out, the only thing worse that getting an error in Linux is not getting one but still having problems. It turns out Grub wasn’t able to find Linux at all. After 2 days of troubleshooting I finally figured out that pacstrap didn’t really install Linux. It was able to make the initramfs and everything else, but never actually made the vmlinux image. AKA, the kernel itself. Turns out this is not a trivial fix. Out of time, I turned to what everybody turns to in their darkest hour.

![IMG_1681.HEIC](https://github.com/itselijahciali/itselijahciali.github.io/blob/main/IMG_1681.HEIC?raw=true)

I’ve never been let down by a ThinkPad, and this time would be no different. After burning an ISO onto a disk, it was clear my only way to move forward was to install Arch the way it’s developers intended– on a ThinkPad from 2012 with a broken screen. The high school laptop saves the day again!

# One last time– let’s get Arch Installed

A bit of bare metal housekeeping is in order to get the installation environment ready. Namely, we need to actually get connected to the internet, which was done for us with the VM. Since I don’t have Ethernet ready to go, I’ll get connected over wireless. Thankfully, this is much easier than it used to be with iwctl. This is trivial and just requires identifying the wireless adapter, scanning for networks and connecting. As a bonus, iwd automatically figures out DHCP for us for free. **We’re connected!**

## Disk Partitioning

I’ve already gone over fdisk but since this is a much larger disk size it’s worth it to show my partition map:

| Device | Type | Size |
| --- | --- | --- |
| /dev/sda1 | EFI System (formatted FAT32) | 512M |
| /dev/sda2 | Linux Swap | 4G |
| /dev/sda3 | Linux Filesystem (formatted ext4) | ~460G |

I also mounted these under /mnt and /mnt/boot in this step.

## Installing Packages

Since this is a bare metal install it’s worth ensuring I get all of this right before committing to the installed environment. I don’t need everything, but networking is important. Thankfully, this laptop uses an Intel wireless chip and is included in the linux-firmware package.

The packages I am installing are **linux**, **linux-firmware**, **base**, **intel-ucode**, **grub**, **efibootmgr**, **networkmanager**, **iw**, **iwd** and **nano**. (I learned my lesson– **vi** is not a good option for me)

## Back to Where We Started

Now that packages are installed, I can finally enter a chroot environment. After generating a fstab, I can chroot into the system. I can run the same commands as before to get the correct system clock, locale, and hostname. I also set the root password, it’s **secret** though!

## Something New and Something Borrowed

I didn’t have to set up the network previously as it was a bridged loopback from the VM. Thankfully iwd does a lot of the heavy lifting here and I’m able to proceed the same way I did in the preinstallation environment. It already had the password which made me wonder if I will have to reconnect when no longer in a chroot environment.

## You Again

Finally, back to grub. This is where I failed last time, so I’m hoping I have less issues this time. The commend is slightly different, being `grub-install --target=x86_64-efi --efi-directory=/boot --bootloader-id=GRUB` since we’re on an amd64 system but it’s otherwise the same. Running `grub-mkconfig -o /boot/grub/grub.cfg` actually recognized Arch, so we’re in a good spot now.

## We’re All Alone

Once I exited the chroot environment and rebooted, I was on my own. Just a simple tty terminal is what I’m left with.

![Screenshot 2025-11-02 at 10.00.00 PM.png](https://github.com/itselijahciali/itselijahciali.github.io/blob/main/Screenshot_2025-11-02_at_10.00.00_PM.png?raw=true)

> *Is this hell?*

After reconfiguring the network (and configuring iwd to setup DHCP) and setting up Cloudflare DNS, I’m connected to the internet. I can now install packages!

## Probably a Good Idea

The assignment doesn’t call for it, but I should definitely not be interacting as root. I install the sudo package, create an elijah user, and add me to the wheel. I then modify the sudoers file to allow anybody within the wheel group to use sudo.

Now that it’s slightly more safe to use the system on the internet, I’ll go ahead and install some other packages to make the terminal nice to use.

First, I install `zsh` and `zsh-completions`. I can change the default to `zsh` by running `chsh -s /usr/bin/zsh`  and I made it look nice by installing git and getting the oh-my-zsh script.

```bash
sh -c "$(curl -fsSL https://raw.githubusercontent.com/ohmyzsh/ohmyzsh/master/tools/install.sh)"
```

Once that was installed though, I was able to switch to a nice theme by editing ~/.zshrc. I chose the **eastwood** theme because it plays nice with the bare console environment while I set up the rest of the system.

## The Rest of The System

First, I install the openssh package and enable the sshd service. This allows me to connect over the network. This is pretty helpful considering the broken screen we have.

## Now You See Me

Finally, I want to install a desktop environment. I’m installing KDE Plasma because it’s very customizable and much cooler than the standard `gnome`

- Following an installation guide on GitHub, I begin by installing the `xorg` display server and `xf86-video-intel` graphics (this isn’t reccomended for newer Intel iGPUs but mine is quite old so it’s worth trying to install)
- Next, I install and enable the `sddm` display manager
- Finally, I install KDE’s desktop environment `plasma`, as well as applications `konsole`, `dolphin`, `ark`, `kwrite`, `spectacle`, and `krunner`

Once everything is installed SDDM should land me in a graphical login on the next reboot.

## Final Stretch
![IMG_1681.heic](https://github.com/itselijahciali/itselijahciali.github.io/blob/main/IMG_1681.heic?raw=true)
