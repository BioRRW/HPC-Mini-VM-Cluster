# Getting Ethernet and WiFi to work

While ethernet was straight forward, WiFi ended up being a trickier problem than expected due to the hardware I had. 


Running `dmesg | grep iwlwifi`, I determined that I had the Intel AX2ll which did not work well with the Rocky kernel. The ultimate solution to this was to install a newer kernel using ElRepo.


# Steps to connect via ethernet:

`nmcli device status` indicated ethernet was managed but disconnected.

Then I manually recreated the connection for the Ethernet interface:
`sudo nmcli connection add type ethernot ifname eno0 con-name eno0-conn autoconnect yes`
- This defined the connection, gave the connection a name and set it to autoconnect.

Lasstly, I brought up the connection manually just to be sure and confirmed it was connected by checking the status and the ip:
`sudo nmcli connection up eno0-conn`
`nmcli device status`
`ip a`

# Steps to connect to WiFi:

Here, I did go in circles a bit trying to force the connection to be up and forcing it to be managed. However, it was clear that the AX211 card did not work out of the box on Rocky 9.6 kernel due to older firmware.

This was discovered when `nmcli` indicated that `WIFI-HW` showed `missing`.

However, before getting to this point I first modified the `/etc/NetworkManagerNetworkManager.conf`:
added: 
```
[main]
plugins=keyfile,ifcfg-rh

[ifupdown]
managed=true
```

This was when I got stuck and discovered the `WIFI-HW` showed `missing`:
`nmcli general status`

Thus, I checked the firmware version and though Googling found that this AX211 needed alternatice firmware:
`dmesg | grep iwlwifi`

I downloaded the ELRepo firmware and kernel modules:
`sudo dnf install https://www.elrepo.org/elrepo-release-9.el9.elrepo.noarch.rpm`

Latest kernel-firmware (containing the newer iwlwifi firmware) was installed:
`sudo dnf --enablerepo=elrepo-kernel install kernel-firmware`

Install Wi-Fi support for NetworkManager:
`sudo dnf install NetworkManager-wifi -y`

Restarted NetworkManager:
`sudo systemctl restart NetworkManager`

Ensurde Wi-Fi radio is turned on:
`sudo nmcli radio wifi on`

Scaedn available Wi-Fi networks (this is where it finally showed it worked!):
`nmcli device wifi list`

Connected to my Wi-Fi:
`nmcli device wifi connect "SSID" password "Password"`

Made it auto-connect at boot:
`nmcli connection modify "SSID" connection.autoconnect yes`
