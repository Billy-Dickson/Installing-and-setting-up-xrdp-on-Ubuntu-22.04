# Installing and setting up xrdp on Ubuntu 22.04.3 LTS

## Prerequistes

1. A running [Ubuntu Gnome Desktop](https://ubuntu.com/download/desktop) installed on hardware or a virtual machine, (mine is running on a [Proxmox Virtual Machine](https://www.proxmox.com/en/downloads)).

## Setting up Gnome with xrdp

Download and install [Gnome Desktop]((https://ubuntu.com/download/desktop))

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

### (Optional) - Setting up a Cloudflare Tunnel for remote access

#### Add cloudflare gpg key

``` bash
sudo mkdir -p --mode=0755 /usr/share/keyrings
curl -fsSL https://pkg.cloudflare.com/cloudflare-main.gpg | sudo tee /usr/share/keyrings/cloudflare-main.gpg >/dev/null
```

#### Add this repo to your apt repositories

```bash
echo 'deb [signed-by=/usr/share/keyrings/cloudflare-main.gpg] https://pkg.cloudflare.com/cloudflared bookworm main' | sudo tee /etc/apt/sources.list.d/cloudflared.list
```

## Creating a Cloudflare Tunnel with cloudflared

The next steps are to authenticate cloudflared with the Cloudflare account, and then use it to create the Cloudflare Tunnel.  

### Setting up the Cloudflare Access Application

The first step is to setup an application in Cloudflare Access. This ensures that when the tunnel is created Cloudflare knows what access control measures to apply to incoming connections.

This can be done through the following steps:

1. Go to the [Cloudflare for Teams Dashboard](https://dash.teams.cloudflare.com/)
2. Open the Zero Trust menu and select Access then Applications
3. Click the Add an application button
4. Select self-hosted
5. Enter an application name, session duration and application domain and click the Next button  
6. Setup the application policies to suit your requirements and click the Next button  
7. Click the Add application button

Now that the application is configured, it’s time to install cloudflared as the bastion host on a designated server.

#### Install cloudflared

```bash
sudo apt update && sudo apt install cloudflared
```

#### Authenticating cloudflared

```bash
cloudflared tunnel login
```

#### Create a tunnel and give it a name

```bash
cloudflared tunnel create Ubuntu-Desktop
```

Running this command will

1. Create a tunnel by establishing a persistent relationship between the name you provide and a UUID for your tunnel.
2. At this point, no connection is active within the tunnel yet.
3. Generate a tunnel credentials file in the default cloudflared directory.
4. Create a subdomain of my registered domain.

From the output of the command, take note of the tunnel’s UUID and the path to your tunnel’s credentials file.

Confirm that the tunnel has been successfully created by running

```bash
cloudflared tunnel list
```

![Tunnel List](/assets/cloudflared_tunnel_list.png)

### Run cloudflared as a service

#### Install the cloudflared service

```bash
cloudflared service install
```

#### Start the service

```bash
systemctl start cloudflared
```

#### (Optional) View the status of the service

```bash
systemctl status cloudflared
```

### Next step

You can now route traffic through your tunnel. If you add IP routes or otherwise change the configuration, restart the service to load the new configuration.

```bash
systemctl restart cloudflared
```

## References

- [How to install XRDP on Ubuntu 22.04](https://youtu.be/U2xHSRuwJ-A?si=1Xs84HscmLwlZAtj) - Easy Hacks
- [neutrinalobs pulseaudio module xrdp](https://github.com/neutrinolabs/pulseaudio-module-xrdp) - github page
- Griffon's [IT Library](https://c-nergy.be/blog/?cat=8)
- Gnome [Policy Kit](https://manpages.ubuntu.com/manpages/focal/man8/pklocalauthority.8.html)
