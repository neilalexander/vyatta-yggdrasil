# yggdrasil for Ubiquiti EdgeOS / VyOS

### Introduction

The `vyatta-yggdrasil` package provides yggdrasil support on supported Ubiquiti EdgeMAX, VyOS and potentially other Vyatta-based routers.  It is integrated with the command line interface (CLI) allowing yggdrasil to be configured through the standard configuration system.

### Compatibility

|                       | Architecture | Compatible |                      Notes                                    |
|-----------------------|:------------:|:----------:|:-------------------------------------------------------------:|
|    EdgeRouter X (ERX) |    mipsel    |     Yes    |                                                               |

### Install / Upgrade

Either download or build a release and copy it to the router, then install/upgrade it:
```
sudo dpkg -i vyatta-yggdrasil-x.x.x-xxxxxx.deb
```
If you are upgrading from a previous release of `vyatta-yggdrasil`, then restart yggdrasil once the upgraded package is installed (replacing `tunX` with your chosen TUN adapter):
```
restart yggdrasil tunX
```

### Initial

Start by creating the default configuration on the interface (replacing `tunX` with your chosen TUN adapter):
```
configure
set interfaces yggdrasil tunX
set interfaces yggdrasil tunX description yggdrasil
commit
```
This automatically generates a new private key and then populates the IPv6 address, public key and private key into the config.

### Add Peers

Add additional peers (replacing `tunX` with your chosen TUN adapter):
```
configure
set interfaces yggdrasil tunX peers hostname.com:12345
set interfaces yggdrasil tunX peers a.b.c.d:12345
set interfaces yggdrasil tunX peers [a:b:c::d]:12345
commit
restart yggdrasil tun0
```

### Set multicast

Enable or disable multicast (replacing `tunX` with your chosen TUN adapter):
```
configure
set interfaces yggdrasil tunX multicast true
set interfaces yggdrasil tunX multicast false
commit
restart yggdrasil tunX
```

### Set MTU

Set the maximum MTU of the Yggdrasil interface, from 1280-65535 (replacing `tunX` with your chosen TUN adapter):
```
configure
set interfaces yggdrasil tunX mtu 1500
commit
restart yggdrasil tunX
```

### Configuration

Other changes should be made to `/config/yggdrasil.tunX.conf` by hand. To make effective, restart yggdrasil (replacing `tunX` with your chosen TUN adapter):
```
restart yggdrasil tunX
```

### Masquerade

If you want to allow other IPv6 hosts on your network to communicate through yggdrasil, you can configure an IPv6 masquerade rule. All traffic sent from other hosts on the network through the yggdrasil interface will be NAT'd.

For example:
```
configure
set interfaces yggdrasil tun0 masquerade from xxxx:xxxx:xxxx::/48
commit
```
If you have multiple IPv6 subnets, then they can be configured individually by setting multiple `masquerade from` source ranges. Both private/ULA and public IPv6 subnets are acceptable.

IPv6 masquerade is not supported on VyOS 1.1.x due to missing support in the kernel.

#### Crash Detection

To make sure that the process is restarted if it crashes, schedule the `vyatta-check-yggdrasil` script to run at a regular interval:
```
configure
set system task-scheduler task check-yggdrasil executable path /opt/vyatta/sbin/vyatta-check-yggdrasil
set system task-scheduler task check-yggdrasil interval 1m
commit
```
