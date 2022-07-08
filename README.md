# Arch Linux Installation

## DISCLAIMER
This is not a guide.
The following are simply my step to install Arch Linux
replicating the [official documentation](https://wiki.archlinux.org/title/Installation_guide).

## Pre-installation
Set up your keyboard layout if you need. For example, in Italian:
```
   # loadkeys it 
```
For a list of all acceptable keymaps:
```
   # localectl list-keymaps
```


### Verify Boot mode
Check if you have EFI BIOS.
```
   # ls /sys/firmware/efi/efivars
```
When you run this command you should see a list of files.
If you have EFI-enabled BIOS follow the next steps, **skip** steps when you see **[EFI]**.

### Internet connection
You can connect to WiFi interactively using this helpful utility called **iwctl**.
```
    iwctl
```
Next, you can list all your wireless interfaces/devices connected using the command:
```
    device list
```
You need to select the preferred one.
Once you select the wireless interface, scan for available network using the command below:
```
    station wlan0 scan
```
While it scans for the network, you don’t get to see the network names yet.
So, to see the connections available, you can type in:
```
    station wlan0 get-networks
```
Among the listed networks, you can connect to your target Wi-Fi using the command:
```
    station wlan0 connect "Name of Network/WiFi"
```
If it is protected by a password, you will be asked for it, enter the credentials and you should be connected to it.
Exit the network setup prompt using `Ctrl + D`.
Now, we’re connected to the network, but to make sure, you can check if you could use the internet by using the ping command:
```
   # ping 8.8.8.8
```
If you can't still can't connect, check [this](https://lists.01.org/hyperkitty/list/iwd@lists.01.org/thread/L5OXQ52CQCO2WAYUKOSZZHH6QR4UMQKB/).

### Update the system clock
```
   # timedatectl set-ntp true
```

### Partition disk
Your primary disk will be known from now on as `sda`.
You can check if this is your primary disk:
```
   # lsblk
```
Feel free to adapt the rest of the guide to `sdb` or any other if you
want to install Arch on a secondary hard drive.

This is an example partition layout on a ~60GB hard disk with only Arch Linux installed.
Check [here](https://wiki.archlinux.org/title/Partitioning#Example_layouts) for more examples.

* `/dev/sda1` boot partition (1G). **[EFI]**
* `/dev/sda2` swap partition (4G).
* `/dev/sda3` root partition (~55G).

You're going to start by removing all the previous partitions and creating
the new ones.
```
   # fdisk /dev/sda
```
This interactive CLI program allows you to enter commands for managing your HD.
I'm going to show you only the commands you need to enter but you can see more
[here](https://wiki.archlinux.org/title/fdisk#Create_a_partition_table_and_partitions).

#### Clear partitions table
Do this untill you have no remaining partitions.
```
   Command: d
```
#### EFI partition (boot) **[EFI]**
```
   Command: n
   ENTER
   ENTER
   +1G
   ENTER
```

#### SWAP partition
```
   Command: n
   ENTER
   ENTER
   +4G
```

#### Root partition (/)
```
   Command: n
   ENTER
   ENTER
   ENTER
```

#### Partition type
Use `Command: t` to select the partitions' types fitting your needs.
You can use `Command: L` to see all the supported partition type and aliases.

#### Save changes and exit
```
   Command: w
```

### Format partitions
First line **[EFI]**
```
   # mkfs.fat -F32 /dev/sda1
   # mkswap /dev/sda2
   # mkfs.ext4 /dev/sda3
```

### Mount partitions
```
   # mount /dev/sda3 /mnt
   # mkdir /mnt/{boot,home}
   # mount /dev/sda1 /mnt/boot
   # swapon /dev/sda2
```

If you run the `lsblk` command you should see something like this: **[EFI]**
```
   NAME   MAJ:MIN RM   SIZE RO TYPE MOUNTPOINT
   sda      8:0    0 232.9G  0 disk
   ├─sda1   8:1    0     1G  0 part /mnt/boot
   ├─sda2   8:2    0     4G  0 part [SWAP]
   └─sda3   8:5    0    50G  0 part /mnt
```
Or this:
```
   NAME   MAJ:MIN RM   SIZE RO TYPE MOUNTPOINT
   sda      8:0    0 232.9G  0 disk
   ├─sda1   8:2    0     4G  0 part [SWAP]
   └─sda2   8:5    0    50G  0 part /mnt
```

### Add best Arch mirrors
To install arch you have to download packages. It's a good idea to download them from the best connection mirror.
Make a backup of mirror list (just in case):
```
    cp /etc/pacman.d/mirrorlist /etc/pacman.d/mirrorlist.bak
```
```
   # pacman -Syy
   # pacman -S reflector
   # reflector --latest 5 --sort rate --save /etc/pacman.d/mirrorlist
```

## Installation

### Install Arch packages
```
   # pacstrap /mnt base base-devel openssh linux linux-firmware 
```

### Install utilities
```
   # pacstrap /mnt neovim lf networkmanager dhcpcd git
```

### Install custom firmwares
Install any other firmware you need.
For me, on a Surface Pro 3, i needed the installation of `linux-firmware-marvell`.
Check [this](https://archlinux.org/packages/) to search firmwares.
```
   # pacstrap /mnt linux-firmware-marvell
```

### Generate fstab file
```
   # genfstab -U /mnt >> /mnt/etc/fstab
```

## Add basic configuration

### Enter the new system
```
   # arch-chroot /mnt
```

### Configure timezone
For this example I'll use "Europe/Madrid", but adapt it to your zone. and aliases.
```
   # ln -sf /usr/share/zoneinfo/Europe/Rome /etc/localtime
   # hwclock —-systohc
```

### Language-related settings
```
   # nvim /etc/locale.gen
```
Now you have to uncomment the language of your choice, for example
`en_US.UTF-8 UTF-8`.
```
   # locale-gen
   # nvim /etc/locale.conf
```
Add this content to the file:
```
   LANG=en_US.UTF-8
   LANGUAGE=en_US
```
```
   # nvim /etc/vconsole.conf
```
Add the keymap to the file, eg:
```
   KEYMAP=us
```
###Network configuration

Create a `/etc/hostname` file and add the hostname entry to this file.
Hostname is basically the name of your computer on the network.
```
    echo computerName > /etc/hostname
```
The next part is to create the hosts file:
```
    touch /etc/hosts
```
And edit this `/etc/hosts` file with your editor to add the following lines:
```
    127.0.0.1	localhost
    ::1		    localhost
    127.0.1.1   computerName	
```

### Enable SSH, NetworkManager and DHCP
These services will be started automatically when the system boots up.
```
   # pacman -S dhcpcd networkmanager network-manager-applet
   # systemctl enable sshd
   # systemctl enable dhcpcd
   # systemctl enable NetworkManager
```

### Update root password
```
   # passwd
```

### Install bootloader
If your system is on **[EFI]**:
```
   # pacman -S grub efibootmgr
   # mkdir /boot/efi
   # mount /dev/sda1 /boot/efi
   # grub-install --target=x86_64-efi --bootloader-id=GRUB --efi-directory=/boot/efi
   # grub-mkconfig -o /boot/grub/grub.cfg
```

Else:
```
   # pacman -S grub
   # grub-install /dev/sda
   # grub-mkconfig -o /boot/grub/grub.cfg
```

### Install other useful packages
```
   # pacman -S iw wpa_supplicant dialog intel-ucode reflector lshw unzip htop
   # pacman -S wget pulseaudio pavucontrol xdg-user-dirs
```

### Final steps
```
   # exit
   # umount -R /mnt
   # swapoff /dev/sda2
   # reboot
```

Remove your installation medium.
And see [General Recommendations](https://wiki.archlinux.org/title/General_recommendations) before continue.

## Post-install configuration
Now your computer has restarted and in the login window on the tty1 console you
can log in with the root user and the password chosen in the previous step.

### Add your user
Assuming your chosen user is "me":
```
   # useradd -m -g users -G wheel,storage,power,audio me 
   # passwd me 
```

### Grant root access to our user
```
   # EDITOR=nvim visudo
```
If you prefer not to be prompted for a password every time you run a command
with "sudo" privileges you need to uncomment this line:
```
   %wheel ALL=(ALL) NOPASSWD: ALL
```
Or if you prefer the standard behavior of most Linux distros you need to
uncomment this line:
```
   %wheel ALL=(ALL) ALL
```

### Login into newly created user
```
   # su - me 
   $ xdg-user-dirs-update
```

### Install AUR package manager
In this guide we'll install [yay](https://github.com/Jguer/yay) as the
AUR package manager. More about [AUR](https://aur.archlinux.org/).

TL;DR AUR is a Community-driven package repository.
```
   $ mkdir Sources
   $ cd Sources
   $ git clone https://aur.archlinux.org/yay.git
   $ cd yay
   $ makepkg -si
```

### PulseAudio applet
If you want to manage your computer's volume from a small icon in the systray:
```
   $ yay -S pa-applet-git
```

### Manage Bluetooth
```
   $ sudo pacman -S bluez bluez-utils blueman
   $ sudo systemctl enable bluetooth
```

### Improve laptop battery consumption
```
   $ sudo pacman -S tlp tlp-rdw powertop acpi
   $ sudo systemctl enable tlp
   $ sudo systemctl mask systemd-rfkill.service
   $ sudo systemctl mask systemd-rfkill.socket
```

### Enable SSD TRIM
If you have a SSD.
```
   $ sudo systemctl enable fstrim.timer
```

## Desktop Environment
[More info here](https://wiki.archlinux.org/title/desktop_environment).

### Install graphical environment 
```
   $ sudo pacman -S xorg-server xorg-apps xorg-xinit
```
[More info and troubleshooting](https://wiki.archlinux.org/title/Xorg).

### Install window manager
```
   $ sudo pacman -S i3-gaps i3blocks i3lock i3status numlockx
```

### Install graphical compositor
```
   $ sudo pacman -S picom
```
The default configuration is available in `/etc/xdg/picom.conf`.
For modifications, it can be copied to `~/.config/picom/picom.conf` or `~/.config/picom.conf`.
To use another custom configuration file with picom, use the following command:
```
   $ picom --config path/to/picom.conf
```

### Install notification daemon
```
   $ sudo yay -S wired
   $ systemctl enable wired
```
See the [git repo](https://github.com/Toqozz/wired-notify) for more info and configs.

### Install display manager
The display manager allows us to log in to the system graphically and also
to automate the startup of some services.
[LightDM](https://wiki.archlinux.org/index.php/LightDM) is one of the most
lightweight display managers.
```
   $ sudo pacman -S lightdm lightdm-gtk-greeter --needed
   $ sudo systemctl enable lightdm
```

### Install backlight controls
```
   $ sudo yay -S clight 
```
See the [git repo](https://github.com/FedeDP/Clight) for usage.



### Install some basic fonts
```
   $ sudo pacman -S noto-fonts ttf-ubuntu-font-family ttf-dejavu ttf-freefont
   $ sudo pacman -S ttf-liberation ttf-droid ttf-roboto terminus-font
```

### Install some useful tools
```
   $ sudo pacman -S rxvt-unicode ranger rofi dmenu lazygit --needed
   $ sudo yay -S wordgrinder 
```

### Install some GUI programs
Install GUI programs as you need.
```
   $ sudo pacman -S firefox vlc --needed
```

### Apply previous settings
```
   $ sudo reboot
```

