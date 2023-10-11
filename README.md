# Installing and setting up xrdp on Ubuntu 22.04.3 LTS

## Prerequistes

1. A working knowledge of Linux and how to install it on real hardware or in a Virtual Environment.
2. A running [Ubuntu Gnome Desktop](https://ubuntu.com/download/desktop) installed on hardware or a virtual machine, (mine is running on a [Proxmox Virtual Machine](https://www.proxmox.com/en/downloads)).

## Setting up Gnome with xrdp

Download and install [Gnome Desktop]((https://ubuntu.com/download/desktop))

The next parts of the installation will need to be done on the Ubuntu Desktop, if it's a virtual install then via the virtual console, if it's on real hardware then with a keyboard, mouse and monitor connected. Depending on how you installed Ubuntu, you'll have to add yourself to the sudo group, to check that you're in the sudo group type the following at the prompt.

```bash
id
```

Instructructions on adding yourself to the group are [here](https://askubuntu.com/questions/124166/how-do-i-add-myself-into-the-sudoers-group#124200)

### Run through the Ubuntu installer and update, then install xrdp

```bash
apt update && apt upgrade
apt install xrdp
```

#### Add the xrdp user to the ssl-cert group

```bash
sudo usermod -a -G ssl-cert xrdp
```

#### Setup Ubuntu Gnome Looks and Feel

```bash
nano .xsessionrc
```

#### Add the following text

```bash
export GNOME_SHELL_SESSION_MODE=ubuntu
export XDG_CURRENT_DESKTOP=ubuntu:GNOME
export XDG_CONFIG_DIRS=/etc/xdg/xdg-ubuntu:/etc/xdg
```

#### Restart xrdp

```bash
sudo systemctl restart xrdp
```

### Allow Colord without the admin prompt at login

``` bash
sudo nano /etc/polkit-1/localauthority/50-local.d/40-allow-colord.pkla
```

#### Paste the following

``` bash
[Allow Colord]
Identity=unix-user:*
Action=org.freedesktop.color-manager.*
ResultAny=no
ResultInactive=no
ResultActive=yes
```

### Allow printer access in Gnome

```bash
sudo nano /etc/polkit-1/localauthority/50-local.d/50-printer-open-access.pkla
```

#### Add the following

```bash
[Printer Administration]
Identity=unix-group:lpadmin
Action=org.opensuse.cupspkhelper.mechanism.*
ResultAny=auth_admin_keep
ResultInactive=no
ResultActive=no
```

### Allow adminstrative functions under Gnome

```bash
sudo nano /etc/polkit-1/localauthority/50-local.d/47-user-admin.pkla
```

#### Paste in the text below

```bash
[user admin]
Identity=unix-user:*
Action=org.gnome.controlcenter.user-accounts.administration
ResultAny=auth_admin_keep
ResultInactive=no
ResultActive=no
```

### Enable Sound Redirection

#### Download the build tools

```bash script
sudo apt install build-essential dpkg-dev libpulse-dev git autoconf libtool
```

#### Download the github repositery

```bash
git clone https://github.com/neutrinolabs/pulseaudio-module-xrdp.git
```

#### Change directory into the repository

```bash
cd pulseaudio-module-xrdp
```

#### Run the setup script

```bash
scripts/install_pulseaudio_sources_apt_wrapper.sh
```

Go for a coffee or something because depending on your hardware, it may take a long time. If your running it on a cheap Chinese PC under a [proxmox hypervisor](https://www.proxmox.com/en/proxmox-ve) then....Coffee!

#### Run Configure script

```bash
./bootstrap && ./configure PULSE_DIR=~/pulseaudio.src
```

#### Run the make script

```bash
make
```

#### Run the make install script

```bash
sudo make install
```

#### Check to see if the modules have been installed

```bash
ls $(pkg-config --variable=modlibexecdir libpulse) | grep xrdp
```

#### You should get the following output

```bash
module-xrdp-sink.la
module-xrdp-sink.so
module-xrdp-source.la
module-xrdp-source.so
```

#### Reboot and Test

```bash
sudo reboot
```

#### Cleaning up afterwards, removing dependecies and files

```bash
cd ~
sudo apt-get purge debootstrap schroot && apt-get autoremove
rm -rif pulseaudio-module-xrdp
```

All going well, your sound setting in [Ubuntu Gnome](https://ubuntu.com/download/desktop) should look like this, feel free to play a wave or mp3 file to test.

![Gnome Sound settings](/assets/Sound.png)

## References

- [How to install XRDP on Ubuntu 22.04](https://youtu.be/U2xHSRuwJ-A?si=1Xs84HscmLwlZAtj) - Easy Hacks
- [neutrinalobs pulseaudio module xrdp](https://github.com/neutrinolabs/pulseaudio-module-xrdp) - github page
- Griffon's [IT Library](https://c-nergy.be/blog/?cat=8)
- Gnome [Policy Kit](https://manpages.ubuntu.com/manpages/focal/man8/pklocalauthority.8.html)
