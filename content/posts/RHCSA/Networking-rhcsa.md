---
title: "RHCSA - Manage Basic Networking"
date: 2023-10-16T21:07:45+03:00
draft: false
hideToc: false
enableToc: true
enableTocContent: false
description: "Learn the basic linux networking commands every sysadmin needs to know"
author: Wissam
authorEmoji: 💻
tags: 
- RHCSA
- Linux
- Networking
- Red Hat
categories:
- Linux
- Networking
series:
- RHCSA Notes
---


## Hostnames
- A hostname is a unique alphanumeric label that is assigned to a node to identify it on the network.
- <abbr title="Fully Qualified Domain Name">**FQDN**</abbr>: the name of the host and the <abbr title="Domain Name System">DNS</abbr> domain in which the host resides (e.g. `server1.example.com`)
- Hosts can contact one another based on hostnames by setting up hostname resolution using DNS.

### show hostname
- There are mutiple ways to view the hostname:

```bash-session
$ hostname
TechyNotes
$ uname -n
TechyNotes
$ cat /etc/hostname
TechyNotes
$ nmcli general hostname
TechyNotes
$ hostnamectl --static
TechyNotes
```

{{< collapse "**Curious about the `--static` option?**" >}} 
`hostnamectl` distinguishes three different hostnames: the high-level `pretty` hostname which might include special characters (e.g. `John's   Laptop`), the `static` hostname which is the user-configured hostname (e.g. `Johns-laptop`), and the `transient` hostname which is a fallback value received from network configuration (e.g. `node12345678`). If a static hostname is set to a valid value, then the transient hostname is not used.
{{< /collapse >}}

- `hostnamectl status` or just `hostnamectl` can be used to view the hostname and other information as well

```bash-session
$ hostnamectl 
 Static hostname: TechyNotes
       Icon name: computer-laptop
         Chassis: laptop 💻
      Machine ID: b257f3c8f00b427ca8584380494190a1
         Boot ID: 6738ac4961d34e36b290dabf76ba2545
Operating System: Fedora Linux 37 (Xfce)          
     CPE OS Name: cpe:/o:fedoraproject:fedora:37
          Kernel: Linux 6.1.13-200.fc37.x86_64
    Architecture: x86-64
 Hardware Vendor: TOSHIBA
  Hardware Model: Satellite L655
Firmware Version: 2.80
```

### Configure the hostname

- one way to change the hostname is by changing `/etc/hostname` file in your favorite editor and then restart `systemd-hostnamed` service

```shell
sudo systemctl restart systemd-hostnamed.service
```

- or using `hostnamectl` command:

```shell
sudo hostnamectl set-hostname TechyNotes
```

- Apart from DNS, you can configure hostname resolution in the `/etc/hosts` file. For each host a single line should be present with the following information:

```console
IP_address canonical_hostname [aliases...]
```

- for example:

```console
192.168.1.10    server1.example.com       server1
192.168.1.13    server2.example.com       server2 mycoolserver
```

- You can then ping `server1` by name:

```shell
ping server1
```

## `ip` Command
- `ip` command syntax:

```shell
ip [ OPTIONS ] OBJECT { COMMAND | help }
```

- `OBJECT` is the object type that you want to manage. The most frequently used objects are:
  - `link` (`l`) - Display and modify network interfaces.
  - `address` (`a`) - Display and modify IP Addresses.
  - `route` (`r`) - Display and alter the routing table.


-The configurations set with the ip command are not persistent. After a system restart, all changes are lost.
- `ifconfig` was used in earlier Linux versions. Don't use `ifconfig` because many new features have been added to Linux networking, use `ip` command instead
- `ip address show` or just `ip address` gives the current network settings. Highlighted lines below show the mac address, ip address and default gateway on my wifi interface `wlp2s0`.

```bash-session {hl_lines=["11-12"]}
$ ip address show 
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host 
       valid_lft forever preferred_lft forever
2: enp3s0: <NO-CARRIER,BROADCAST,MULTICAST,UP> mtu 1500 qdisc fq_codel state DOWN group default qlen 1000
    link/ether c8:0a:a9:75:08:4b brd ff:ff:ff:ff:ff:ff
3: wlp2s0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc mq state UP group default qlen 1000
    link/ether 20:7c:8f:02:45:37 brd ff:ff:ff:ff:ff:ff
    inet 192.168.43.10/24 brd 192.168.43.255 scope global dynamic noprefixroute wlp2s0
       valid_lft 3519sec preferred_lft 3519sec
    inet6 fe80::994d:d24:621b:d44b/64 scope link noprefixroute 
       valid_lft forever preferred_lft forever
```

- `ip` command is smart, you just have to give it enough letters to for the options you want, for example all the following commands give the same result

```bash-session
$ ip address show
$ ip address
$ ip addr
$ ip add
$ ip a s
$ ip a
```

- To show current link statistics:

```shell
ip -s link show
```

- When configuring network interfaces, you must execute the commands as `root` or user with `sudo` privileges.

```bash-session
$ ip link set dev enp3s0 up
RTNETLINK answers: Operation not permitted
```

- To bring `enp3s0` device up *TEMPORARILY*:

```shell
sudo ip link set dev enp3s0 up
```

- To bring `enp3s0` device down *TEMPORARILY*:

```shell
sudo ip link set dev enp3s0 down
```

- To show the default gateway (in my case, it's `192.168.43.1`): 
```bash-session {hl_lines=2}
$ ip route show
default via 192.168.43.1 dev wlp2s0 proto dhcp src 192.168.43.10 metric 600 
192.168.43.0/24 dev wlp2s0 proto kernel scope link src 192.168.43.10 metric 600 
```

- you can multiple IP addresses on a device. To add an ip address to a device:

```shell
ip addr add 10.0.0.10/24 dev <devicename>
```
{{<notice warning>}}
you won't see the second IP address in the output of `ifconfig` command. it's deprecated and you shouldn't use it.
{{</notice>}}

## Network Devices and Connections
- A connection is the configuration that is used on a network adapter.
- You can have multiple connections or connection profiles with different configuration settings. for example you can have one connection profile for your home network and another for corporate network, but only one of them can be actived at a time.
- Previously, Connection configuration files (or connection profiles) used to be stored in `/etc/sysconfig/network-scripts` directory with filenames `ifcfg-` followed by the name of network card like `ifcfg-enp3s0`, `ifcfg-wlp2s0`, and `ifcfg-em1`. Those files are called ifcfg-files.
- the `ifcfg` format is deprecated.
- In RHEL 9, NetworkManager now uses the key file format to store new connection profiles. Note that the `ifcfg` format is still supported.
- NetworkManager now stores new network profiles in keyfile format in the `/etc/NetworkManager/system-connections/` directory.

## Configuring the Network with `nmcli`
- Everything you do with the ip command, though, is nonpersistent. If you want to make your configuration persistent, use `nmtui` or `nmcli`.
- The basic format of a nmcli command is as follows:

```shell
nmcli [OPTIONS] OBJECT { COMMAND | help }
```

- OBJECT can be one of the following options.
  - **General:** retrieves NetworkManager's status and global configuration.
  - **Networking:** commands to query a network connection's status and enable or disable connections.
  - **Radio:** show radio switches status, or enable and disable the switches
  - **Monitor** monitor NetworkManager activity and observe network connections' status changes.
  - **Connection:** bring network interfaces up and down, to add new connections, and to delete existing connections.
  - **Device:** modify parameters associated with a device (e.g., the interface name) or to connect a device using an existing connection.
  - **Secret** registers nmcli as a NetworkManager secret agent listening for secret messages. This is very rarely required  because nmcli does this automatically when connecting to networks.

- `nmcli` is smart, you just have to give it enough letters to for the options you want
- Each object has a set of different commands. you can always use the `TAB` key to see available commands for a specific object. You can use `help` command at any point too. `nmcli` man page has some great examples too. see `man nmcli-examples`   

```bash-session
$ nmcli help
$ nmcli device help
$ nmcli connection show help
```

**Let's see some examples:**

- show all connections (active and inactive) :
  
```bash-session
$ nmcli connection show
$ nmcli con sh
$ nmcli c
```

- Show information about a specific connection:

```shell
nmcli connection show <connection_name>
```

- To find out about these settings and informationin the output of the previous command, use `man 5 nm-settings`
- To show a list of all devices:

```bash-session
$ nmcli dev status
$ nmcli dev     (status is the default command)
$ nmcli d
```

- Show settings for a specific deviec:

```shell
nmcli dev show <devicename>
```

- Users are supposed to have permissions be able to connect their local system to a network. but users are supposed to be able to connect
their local system to a network.

```bash-session
$ nmcli general permissions
$ nmcli gen perm
$ nmcli g p
```


- add an ethernet connection with a name `dhcp-connection` on device `enp3s0` that uses DHCP:

```shell
nmcli con add con-name dhcp-connection type ethernet ifname enp3s0 ipv4.method auto.
```

- add an ethernet connection with a name `static-connecton` on device `enp3s0` with a static IP `192.168.1.10` and subnetmask `255.255.255.0` and default gateway `192.168.1.1`

```shell
nmcli con add con-name static-connecton ifname enp3s0 type ethernet ip4 192.168.1.10/24 gw4 192.168.1.1 ipv4.method manual
```

{{<notice note>}}
Notice the difference between `ip4` command and `ipv4` when changing connection parameters like `ipv4.method`
{{</notice>}}

- To activate a connection:

```shell
nmcli con up <connection_name>
```

- To deactivate a connection:

```shell
nmcli con down <connection_name>
```

- Make sure that the `static-connecton` connection does not connect automatically by using

```shell
nmcli connection modify static-connecton connection.autoconnect no.
```

- change IP address of the conection  `static-connecton` to `192.168.11`

```shell
nmcli con mod static-connecton ipv4.addresses 192.168.1.11/24
```

- Add a DNS server to the `static-connecton` connection

```shell
nmcli con mod static-connecton ipv4.dns 192.168.1.9
```

- Add a second DNS server to the `static-connecton` connection (Notice the plus sign):

```shell
nmcli con mod static-connecton +ipv4.dns 8.8.8.8
```

- Add a second IP address `192.168.0.11` to the `static-connecton` connection (notice the plus sign):

```shell
nmcli con mod static-connecton +ipv4.addresses 192.168.0.11/24
```

- You can configure a network interface using the NetworkManager's tool, `nmtui`.  it's simple text user interface (TUI) for NetworkManager. to install it:

```shell
sudo dnf install NetworkManager-tui
```

## The `ss` Command

- The `ss` (socket statistics) tool is a simpler and faster version of the now obsolete `netstat` command.
- Common options used with `ss` command:

| **Option**        | **meaning**                                               |
| :---------------- | :-------------------------------------------------------- |
| `-n, --numeric`   | Do not try to resolve service names                       |
| `-a, --all`       | Display both listening and non-listening sockets.         |
| `-l, --listening` | Display only listening sockets                            |
| `-t, --tcp`       | Display TCP sockets.                                      |
| `-u, --udp`       | Display UDP sockets.                                      |
| `-x, --unix`      | Display Unix domain sockets (alias for `-f unix`).        |
| `-p, --processes` | Show process using socket.                                |
| `-4, --ipv4`      | Display only IP version 4 sockets (alias for `-f inet`).  |
| `-6, --ipv6`      | Display only IP version 6 sockets (alias for `-f inet6`). |

- When no option is used `ss` displays a list of open non-listening sockets (e.g. TCP/UNIX/UDP) that have established connection.

```bash-session
$ ss 
Netid     State     Recv-Q     Send-Q                                   Local Address:Port             Peer Address:Port       Process     
u_str     ESTAB     0          0                                   /run/user/1000/bus 324959                      * 326728                 
u_str     ESTAB     0          0                                 @/tmp/.ICE-unix/1288 109874                      * 109176                 
u_str     ESTAB     0          0                                                    * 22433                       * 23286                  
u_str     ESTAB     0          0                                                    * 22467                       * 25950                  
u_str     ESTAB     0          0                                                    * 25802                       * 25346                  
tcp       ESTAB     0          0                                            127.0.0.1:49388               127.0.0.1:xtel                   
tcp       ESTAB     0          0                                            127.0.0.1:xtel                127.0.0.1:49388                  
```

The columns show the following details:

| Header               | Meaning                                                      |
| :------------------- | ------------------------------------------------------------ |
| `Netid`              | Type of socket. Common types are *TCP*, *UDP*, *u_str* (Unix stream), and *u_seq* (Unix sequence). |
| `State`              | State of the socket. Most commonly *ESTAB* (established), *UNCONN* (unconnected), *LISTEN* (listening). |
| `Recv-Q`             | Number of received packets in the queue                      |
| `Send-Q`             | Number of sent packets in the queue.                         |
| `Local address:port` | Address of local machine and port                            |
| `Peer address:port`  | Address of remote machine and port.                          |

- To show a list of listening TCP ports:

```bash-session
$ ss -lt
State           Recv-Q          Send-Q                   Local Address:Port                      Peer Address:Port         Process         
LISTEN          0               4096                           0.0.0.0:hostmon                        0.0.0.0:*                            
LISTEN          0               4096                     127.0.0.53%lo:domain                         0.0.0.0:*                            
LISTEN          0               4096                         127.0.0.1:xtel                           0.0.0.0:*                            
LISTEN          0               128                          127.0.0.1:ipp                            0.0.0.0:*                            
LISTEN          0               4096                        127.0.0.54:domain                         0.0.0.0:*                            
LISTEN          0               4096                              [::]:hostmon                           [::]:*                            
LISTEN          0               128                              [::1]:ipp                               [::]:*                            
```

- Ports listening on the IPv4 loopback address `127.0.0.1` or the IPv6 loopback address `::1` are locally accessible only and cannot be reached from external machines.
- Some ports are listening on `*`, which stands for all IPv4 addresses, or on `:::*`, which represents all ports on all IPv6 addresses.
- To see a list of  UDP and TCP ports that are listening

```shell
ss -tul
```

## Other

- Both <abbr title="Transmission Control Protocol">TCP</abbr> and <abbr title="User Datagram Protocol">UDP</abbr> use port numbers to identify sending and receiving application on a host. you can see a big list of well-known port numbers in `/etc/services` files.
- Protocol field inside IPv4 header defines the protocol used in the data portion of the IP datagram. <abbr title="Internet Assigned Numbers Authority">**IANA**</abbr> maintains a list of IP protocol numbers. For example TCP protocol number is 6, UDP is 17, OSPF is 89 etc. you can see those numbers in `/etc/protocols`. The file is documented in `man 5 protocols`


 