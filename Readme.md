#  Linux-router

 Share your Linux's Internet access to other devices.
 This is a fork of [create_ap](https://github.com/oblique/create_ap).
 
##  Features

- Share Internet via NAT
- DHCP server
- DNS server 
- DNS query log
- Transparent proxy (redsocks)
- Set DNS proxy

Creating Wifi hotspot:

- Channel selecting
- Choose encryptions: WPA, WPA2, WPA/WPA2, No encryption
- Hide  SSID
- IEEE 802.11n & 802.11ac support
- Create AP on the same interface you are getting Internet (require same channel)


Useful in these situations:
```
Internet ----(eth0/wlan0)- Linux-(wlanX)(AP)
                                        |
                                        |----client
                                        |
                                        |----client
```

```
                                    Internet
Wifi AP(no DHCP)                        |
    |                                   |
    |----(wlan1)-Linux-(eth0/wlan0)------
    |           (DHCP)
    |
    |----client
    |
    |----client
```



```
                                    Internet
 Switch                                 |
    |                                   |
    |---(eth1)-Linux-(eth0/wlan0)--------
    |
    |----client
    |
    |----client
```

```
Internet ----(eth0/wlan0)-Linux-(eth1)--------Another PC
```


```
Internet ----(eth0/wlan0)-Linux-(virtual interface)-----VM guests/container guests
```
 
## Usage

### Share Internet to an interface

```
# lnxrouter -i eth1
```

### Create Wifi hotspot

```
# lnxrouter --ap wlan0 MyAccessPoint --password MyPassPhrase
```

### Make a LAN without Internet

```
# lnxrouter -i eth1 -n
```
```
# lnxrouter --ap wlan0 MyAccessPoint --password MyPassPhrase -n
```

### Transparent proxy with tor

```
# lnxrouter -i eth1 --tp 9040 --dns-proxy 9053
```

In `torrc`

```
TransPort 0.0.0.0:9040 
DNSPort 0.0.0.0:9053
```
### Using with LXC
Create a bridge
```
# brctl addbr lxcbr1
```
In LXC container's `config`
```
lxc.network.type = veth
lxc.network.flags = up
lxc.network.link = lxcbr1
lxc.network.hwaddr = xx:xx:xx:xx:xx:xx
```
```
# lnxrouter -i lxcbr1
```
### Using with VirtualBox
On VirtualBox's global settings, create a host-only network `vboxnet1` with DHCP disabled.
```
# lnxrouter -i vboxnet1
```
### CLI usage and other features

```
Usage: lnxrouter [options] 

Options:
  -h, --help              Show this help
  --version               Print version number

  -i <interface>          Interface to share Internet to. An NATed subnet will be made on it.
                          To create Wifi hotspot use --ap instead
  -n                      Disable Internet sharing
  --tp <port>             Transparent proxy (redsocks), redirect tcp and udp traffic to port.
                          Usually use with --dns-proxy

  -g <gateway>            Set Gateway IPv4 address, netmask is /24 (default: 192.168.18.1)
  --dns-proxy <port>      Redirect 53 port to DNS proxy port. dnsmasq DNS is disabled
  --no-serve-dns          dnsmasq DNS disabled
  --no-dnsmasq            Disable dnsmasq server completely (dhcp and dns)
  --log-dns               Show dnsmasq DNS server query log
  --dhcp-dns <IP1[,IP2]>|no
                          Set DNS offered by DHCP, or no DNS offered (default: gateway as DNS)
  -d                      DNS server will take into account /etc/hosts
  -e <hosts_file>         DNS server will take into account additional hosts file

  --mac <MAC>             Set MAC address

 Wifi hotspot options:
  --ap <wlan card interface> <access point name>
                          Create Wifi access point using wlan card, and set SSID
  --password <passphrase> Wifi password

  --hidden                Make the Access Point hidden (do not broadcast the SSID)
  --no-virt               Do not create virtual interface. 
                          Using this you can't use same wlan card as Internet and AP
  -c <channel>            Channel number (default: 1)
  --country <code>        Set two-letter country code for regularity (example: US)
  --freq-band <GHz>       Set frequency band. Valid inputs: 2.4, 5 (default: 2.4)
  --driver                Choose your WiFi adapter driver (default: nl80211)
  -w <WPA version>        Use 1 for WPA, use 2 for WPA2, use 1+2 for both (default: 1+2)
  --psk                   Use 64 hex digits pre-shared-key instead of passphrase
  --mac-filter            Enable Wifi hotspot MAC address filtering
  --mac-filter-accept     Location of Wifi hotspot MAC address filter list (defaults to /etc/hostapd/hostapd.accept)
  --hostapd-debug <level> With level between 1 and 2, passes arguments -d or -dd to hostapd for debugging.
  --isolate-clients       Disable communication between clients
  --ieee80211n            Enable IEEE 802.11n (HT)
  --ieee80211ac           Enable IEEE 802.11ac (VHT)
  --ht_capab <HT>         HT capabilities (default: [HT40+])
  --vht_capab <VHT>       VHT capabilities
  --no-haveged            Do not run 'haveged' automatically when needed

 Instance managing:
  --daemon                Run lnxrouter in the background
  --stop <id>             Send stop command to an already running lnxrouter. For an <id>
                          you can put the PID of lnxrouter or interface. You can
                          get them with --list-running
  --list-running          Show the lnxrouter processes that are already running
  --list-clients <id>     List the clients connected to lnxrouter instance associated with <id>.
                          For an <id> you can put the PID of lnxrouter or interface.
                          If virtual WiFi interface was created, then use that one.
                          You can get them with --list-running

 Tools: 
  --fix-unmanaged         If NetworkManager shows your interface as unmanaged after you
                          close lnxrouter, then use this option to switch your interface
                          back to managed
```


### Ddependencies
- bash
- procps or procps-ng
- iproute2
- dnsmasq
- iptables

Wifi hotspot:

- hostapd
- iw
- iwconfig (you only need this if 'iw' can not recognize your adapter)
- haveged (optional)

## TODO

- Ban private network access
- IPv6 support 
- Random MAC, IP, SSID, password

## Thanks

Thank the upstream project [create_ap](https://github.com/oblique/create_ap). It brings many Wifi features
