+++
title = "Pi-hole : Take back control of your network"

[taxonomies]
tags = ["Home Automation", "DNS", "Network"]
+++

## Get started

This is a step-by-step guide to install Pi-hole on a Raspberry Pi. I will only use IPv4 as IPv6 doesn't provide significant benefits for local network. Please consider reading pi-hole's official documentation : <https://pi-hole.net/>

1. Install Raspberry Pi OS (formerly Raspbian) via <https://www.raspberrypi.org/software/>
2. Boot on this OS
3. Run `sudo raspi-config` and disable WiFi + Bluetooth (security concerns)
4. Enable SSH daemon when starting.

<!-- more -->

## Initial configuration

### Configure hostname

1. Change hosts file : `sudo vi /etc/hosts` and replace raspberrypi by the relevant name (ex: Network-Manager)
2. Change hostname file : `sudo vi /etc/hostname` and replace raspberrypi by the relevant name (ex: Network-Manager)
3. Change hostname : `sudo hostname relevant-hostname` (ex: `sudo hostname Network-Manager`)

### Configure users (Optional)

As `pi` user is really common, one may want to switch to another username (small obfuscation).

1. Create a new user foo : `sudo adduser foo`
2. Add foo to sudoers `sudo usermod -aG sudo foo`
3. Give group of pi user
   - Retrieve groups : `sudo cat /etc/group | grep pi`
   - Add groups : `sudo usermod -G adm,dialout,cdrom,sudo,audio,video,plugdev,games,users,input,netdev,i2c,gpio,lpadmin foo`
4. Remove spi group : `sudo groupdel spi`
5. Remove pi user : `sudo userdel pi`

### Configure netplan

_Trivia : We use Netplan to define a static IP address via configuration file. This part is optional if you want to define it with another method._

1. Install [netplan](https://netplan.io/) > 0.95 (see appendix if required version is not available)
2. Define a config (see [Netplan config](./netplan.yaml))
3. Run `netplan try` and then `netplan apply` to define your IP
4. Check your IP (via `ip a` or `ifconfig`)

Refer to netplan's doc if there is any problem for these steps.

## Pi-hole

### Installation

Install pi-hole : `curl -sSL https://install.pi-hole.net | sudo bash` (see <https://docs.pi-hole.net/main/basic-install/>)

### Settings (in admin panel)

Go to <http://192.168.1.2/admin/> (replace IP with the one from netplan)

#### DNS

1. Check this [Encrypted DNS Resolvers list](https://privacyguides.org/providers/dns/) and choose the one you like (mine is Quad9 filtered & DNSSEC).
2. Select a generic DNS server or insert an IP of a custom DNS
3. Enable `Never forward non-FQDNs`, `Never forward reverse lookups for private IP ranges` and `use DNSSEC`

#### DHCP

1. Enable DHCP Server (and disable your router's DHCP server)
2. Set a range of IP and define your gateway
3. Define a local domain name (ex `foo.lan`)
4. Enable or disable rapid commit
5. Disable IPv6 support (YAGNI and complex management involved)
6. Define static DHCP leases for your servers

NB : rapid commit should only be enabled if either the server is the only server for the subnet, or multiple servers are present and they each commit a binding for all clients.

#### Web interface

- Enable the theme you like (mine is `Pi-hole midnight theme (dark)` in boxed layout)
- Set styling options

#### Blacklist

You may want to block some URLs for personal/political reason. No offense, these are just examples :

- `(^|.)fr$` : Block some french domains
- `(^|.)gov.uk$` : Block UK's government domain

#### Whitelist

With aggressive adlists, some website we still want to use could be blocked. You may need these URL as a whitelist :

- link.epicgames.com (EpicGames)
- accounts.nvgs.nvidia.cn (Nvidia Geforce Experience)
- p.shure.com (Shure application)
- ichnaea.netflix.com (may sometimes break Netflix if blocked)
- cdn.samsungcloudsolution.com (updates for Samsung TV)
- p.typekit.net (Adobe Fonts)
- (\.|^)newrelic\.com$ (Allow NewRelic / current job)
- bam.nr-data.net (Allow NewRelic data / current job)

#### Blocklist (named Adlists in UI)

This is the downside of pi-hole, you have to find by yourself your needs. Best advice I can give you is to start with some of these blocklists and then tune them up.

- Block list of doom : <https://dbl.oisd.nl> (<https://beaconsandwich.co.uk/2020/05/03/shut-your-pi-hole/>)
- StevenBlack unified host complete (adwares, malwares, fakenews, gambling, porn) : <https://raw.githubusercontent.com/StevenBlack/hosts/master/alternates/fakenews-gambling-porn/hosts>
- Malwares : <https://mirror1.malwaredomains.com/files/justdomains>
- Cameleon (ads) : <http://sysctl.org/cameleon/hosts>
- Disconnect me :
  - Ads : <https://s3.amazonaws.com/lists.disconnect.me/simple_ad.txt>
  - Tracking : <https://s3.amazonaws.com/lists.disconnect.me/simple_tracking.txt>
- Deathbybandaid :
  - FR list : <https://raw.githubusercontent.com/deathbybandaid/piholeparser/master/Subscribable-Lists/CountryCodesLists/France.txt>
  - EU list : <https://raw.githubusercontent.com/deathbybandaid/piholeparser/master/Subscribable-Lists/CountryCodesLists/EuropeanUnion.txt>
  - EasyList : <https://raw.githubusercontent.com/deathbybandaid/piholeparser/master/Subscribable-Lists/ParsedBlacklists/EasyList.txt>
  - EasyList FR : <https://raw.githubusercontent.com/deathbybandaid/piholeparser/master/Subscribable-Lists/ParsedBlacklists/EasyList-Liste-FR.txt>
  - EU Cookies : <https://raw.githubusercontent.com/deathbybandaid/piholeparser/master/Subscribable-Lists/ParsedBlacklists/Block-EU-Cookie-Shit-List.txt>
- 0Zinc :
  - EasyList : <https://raw.githubusercontent.com/0Zinc/easylists-for-pihole/master/easylist.txt>
  - EasyPrivacy : <https://raw.githubusercontent.com/0Zinc/easylists-for-pihole/master/easyprivacy.txt>
- Perflyst :
  - SmartTV filter : <https://raw.githubusercontent.com/Perflyst/PiHoleBlocklist/master/SmartTV.txt>
- Spotify :
  - x0uid/SpotifyAdBlock : <https://raw.githubusercontent.com/x0uid/SpotifyAdBlock/master/SpotifyBlocklist.txt>

You may also want to check this : <https://firebog.net/>

## Maintain

Use `screen -S update` and later `screen -r update` (detach with `ctrl + a` then `d`).

Why : provide a robust terminal. Raspberry Pi may freeze during a big update, leading to a timeout of the ssh connection (and often a failure during the update).

### Update Raspberry Pi

Since it's a debian based system :

- `sudo apt update`
- `sudo apt upgrade`

### Update Pi-hole

- `pihole -up`

Check the admin panel if a new version is available.

### Change RATE_LIMIT

Trivia : There is a rate limiter in FTLDNS (part of Pihole responsible of handling DNS requests). Sometimes default value is too small for common usage (entertainment, work, etc), we may want to give more request (per day/hour/minute) to users.

Create or edit a configuration file for FTLDNS `/etc/pihole/pihole-FTL.conf`.
Insert a line with `RATE_LIMIT=X/Y` and adjust the value as below :

- X equals to a number of accepted requests (default is 1000)
- Y equals to a number of seconds (default is 60).

The default value is 1000 requests per minute (60 seconds). For 5 requests per hour, set the rate limit to 5/3600 (ie. 5 requests per 3600 seconds = 1 hour).

Set a rate limit to 3000/60 if there are warnings in Pihole diagnosis "Client A.B.C.D has been rate-limited (current config allows up to 1000 queries in 60 seconds)"

---

## Appendix : Install Netplan.io from Bullseye package

_Trivia : last release of Netplan.io on Debian Buster (0.95-2) has a bug and you may still be running Debian Buster. You need a newer version of that package from Debian Bullseye._

You may check current version here : <https://packages.debian.org/search?lang=en&suite=default&arch=any&searchon=names&keywords=netplan.io>

### Add Bullseye packages to sources.list

1. Edit `/etc/apt/sources.list` and add `deb http://raspbian.raspberrypi.org/raspbian/ bullseye main contrib non-free rpi` for Bullseye (Debian 11).
2. Create a file `80default-distrib` in `/etc/apt/apt.conf.d`
3. Edit the new file and add `APT::Default-Release "buster";` to ensure you are still using Buster as default.

_NB : With this configuration, Bullseye package will be install only if we strictly ask to do so._

### Install a Bullseye package

1. Update repositories : `sudo apt update`
2. Install netplan.io (Bullseye package) : `sudo apt -t bullseye install netplan.io`
