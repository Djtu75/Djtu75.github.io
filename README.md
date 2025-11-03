# Djtu75.github.io

# Arch Linux Installation Guide (VMware Workstation)

## 1. Prep

### Resources

-   **Official Guide:**
    <https://wiki.archlinux.org/title/Installation_guide>
-   **Download Mirrors:**
    <https://archlinux.org/download/#download-mirrors>

Choose a mirror, I chose Berkley:\
\> <https://mirrors.ocf.berkeley.edu/archlinux/iso/2025.10.01/>

------------------------------------------------------------------------

## 2. VM Setup

1.  Create a new VM in **VMware Workstation**.

2.  Choose your **Arch ISO** for the VM.

3.  Choose **Linux 5.x and later kernel (64-bit)**.

4.  Allocate:

    -   **8 GB** disk space (at least)
    -   **1 GB** memory

5.  Before starting, edit the VM configuration file (`VM_NAME.vmx`) and
    add:

    ``` bash
    firmware="efi"
    ```

6.  Boot the VM and select **Arch Linux install medium**.

------------------------------------------------------------------------

## 3. Initial Config

``` bash
loadkeys en #setup keyboard
cat /sys/firmware/efi/fw_platform_size
```

> It should return **64**. If not, check the `.vmx` file or
> restart.

Check internet:

``` bash
ping ping.archlinux.org
```

> If it fails, double check your network connection or restart.

------------------------------------------------------------------------

## 4. Disk Partitioning

> Run this command before starting the next section.

``` bash
`fdisk /dev/sda`
```

> Then input these commands in this exact order.


| Command | Description                                                                                                          |
| ------- | -------------------------------------------------------------------------------------------------------------------- |
| `g`     | Create new GPT partition table                                                                                       |
| `n`     | Create EFI boot partition                                                                                            |
| `ENTER` | Default partition number                                                                                             |
| `ENTER` | Default first sector                                                                                                 |
| `+512M` | Set partition size                                                                                                   |
| `t`     | Select the partition you just made's type                                                                            |
| `1`     | Set type to EFI System                                                                                               |
| `n`     | Create root partition                                                                                                |
| `ENTER` | Default partition number                                                                                             |
| `ENTER` | Default sector                                                                                                       |
| `ENTER` | Use remaining disk space                                                                                             |
| `p`     | Should show `/dev/sda1` and `/dev/sda2` with the correct types. If you failed, just run `q` and restart this section |
| `w`     | Write changes                                                                                                        |


------------------------------------------------------------------------

## 5. Format and Mount Partitions

``` bash
mkfs.ext4 /dev/sda2        # Root filesystem
mkfs.fat -F 32 /dev/sda1   # EFI boot partition

mount /dev/sda2 /mnt
mount --mkdir /dev/sda1 /mnt/boot
```

------------------------------------------------------------------------

## 6. Base System Installation

``` bash
pacstrap -K /mnt base linux linux-firmware efibootmgr grub vi vim networkmanager zsh openssh man sudo
```

> These are the initial packages. Don’t add too many, if you over-install it will get pissed at you, and you’ll have to restart from the beginning.
> This also covers the install another shell and install ssh requirements.

------------------------------------------------------------------------

## 7. Filesystem Table

``` bash
genfstab -U /mnt >> /mnt/etc/fstab
```

> This sets it up so you don't have to manually mount on boot every time.

------------------------------------------------------------------------

## 8. Enter your Mount

``` bash
arch-chroot /mnt
```

------------------------------------------------------------------------

## 9. Time and Localization

``` bash
ln -sf /usr/share/zoneinfo/America/Chicago /etc/localtime
hwclock --systohc
```

> Sets up the clock and timezone.

### Locale Setup

``` bash
vim /etc/locale.gen
```

-   Uncomment:

        en_US.UTF-8 UTF-8

``` bash
echo "LANG=en_US.UTF-8" > /etc/locale.conf
echo "KEYMAP=en" > /etc/vconsole.conf
```

``` bash
    locale-gen
```

------------------------------------------------------------------------

## 10. Hostname and Hosts Configuration

``` bash
echo "your_hostname" > /etc/hostname
vim /etc/hosts 
```

Add the following:

    127.0.0.1   localhost
    ::1         localhost
    127.0.1.1   your_hostname

> TBH idk if this second part is necessary, but I saw it in a network setup guide and I don't want to risk breaking stuff.

------------------------------------------------------------------------

## 11. Users and Passwords

``` bash
passwd
useradd -mG wheel -s /bin/bash thechadministrator
passwd thechadministrator
useradd -mG wheel -s /bin/bash codi
passwd codi
visudo
```

-   Uncomment:

        %wheel ALL=(ALL:ALL) ALL

------------------------------------------------------------------------

## 12. Bootloader Setup

``` bash
grub-install --target=x86_64-efi --efi-directory=/boot --bootloader-id=GRUB
grub-mkconfig -o /boot/grub/grub.cfg
```

------------------------------------------------------------------------

## 13. Network Setup

``` bash
systemctl enable NetworkManager
```

------------------------------------------------------------------------

## 14. Test if Install was Successful

``` bash
exit
umount -R /mnt
reboot
```

> At this point you should have a functioning terminal.
> Log in as one of the users or 'root'.
> Run ls or ls /usr (if you're in root) to check if everything looks okay.

------------------------------------------------------------------------

## 15. Double-Check Timesync

``` bash
timedatectl set-ntp true
```

------------------------------------------------------------------------

## 16. Desktop Environment (LXQT)

``` bash
sudo pacman -S lxqt breeze-icons sddm   #accept the defaults by just pressing enter
sudo systemctl enable sddm
reboot
```

------------------------------------------------------------------------

## 17. Customize Terminal

Open a terminal and kill two birds with one stone in `.bashrc`:

``` bash
vim ~/.bashrc
```

Add:

``` bash
alias usealias='echo "alias used!"'
```

Change `PS1` line to:

``` bash
PS1='\[\033[0;32m\]\u@\h\[\033[0m\]:\[\033[0;34m\]\w\[\033[0m\]\$ '
```

You will need to switch users and come back for the changes to take effect.

------------------------------------------------------------------------

##  Completion

**Congratulations!**\
You have successfully installed **Arch Linux**!
