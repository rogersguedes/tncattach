TNC Attach
==========
Attach KISS TNC devices as network interfaces in Linux. This program allows you to attach TNCs or any KISS-compatible device as a network interface. This program does not need any kernel modules, and has no external dependencies outside the standard Linux and GNU C libraries.

## Installation

Currently it is recommended to compile and install __tncattach__ from source with the below commands. If that is not possible for you, precompiled __amd64__ and __armhf__ (Raspberry Pi) binaries exist in the releases section.

```sh
# If you don't already have a compiler installed
sudo apt install build-essential

# Clone repository from GitHub
git clone https://github.com/markqvist/tncattach.git

# Move into source directory
cd tncattach

# Make program
make

# Install to system
sudo make install
```

## Using tncattach

Using __tncattach__ is simple. Run the program from the command line, specifying which serial port the TNC is connected to, and the serial port baud-rate, and __tncattach__ takes care of the rest. In most cases, depending on what you intend to do, you probably want to use some of the options, though. See the examples section below for usage examples.

```sh
Usage: tncattach [OPTION...] port baudrate

Attach TNC devices as system network interfaces

  -d, --daemon               Run tncattach as a daemon
  -e, --ethernet             Create a full ethernet device
  -i, --ipv4=IP_ADDRESS      Configure an IPv4 address on interface
  -m, --mtu=MTU              Specify interface MTU
  -n, --noipv6               Filter IPv6 traffic from reaching TNC
      --noup                 Only create interface, don't bring it up
  -v, --verbose              Enable verbose output
  -?, --help                 Give this help list
      --usage                Give a short usage message
  -V, --version              Print program version
```

The program supports attaching TNCs as point-to-point tunnel devices, or generic ethernet devices. The ethernet mode is suitable for point-to-multipoint setups, and can be enabled with the corresponding command line switch. If you only need point-to-point links, it is advisable to just use the standard point-to-point mode, since it doesn't incur the ethernet header overhead on each packet.

Additionally, it is worth noting that __tncattach__ can filter out IPv6 packets from reaching the TNC. Most operating systems attempts to autoconfigure IPv6 when an interface is brought up, which results in a substantial amount of IPv6 traffic generated by router solicitations and similar, which is usually unwanted for packet radio links and similar.

If you intend to use __tncattach__ on a system with mDNS services enabled (avahi-daemon, for example), you may want to consider modifying your mDNS setup to exclude TNC interfaces, or turning it off entirely, since it will generate a lot of traffic that might be unwanted.

## Examples

Create an ethernet device with a USB-connected TNC, set the MTU, filter IPv6 traffic, and set an IPv4 address:

```sh
# Attach interface
sudo tncattach /dev/ttyUSB0 115200 --ethernet --mtu 576 --noipv6 --ipv4 10.92.0.10/24
```

You can interact with the interface like any other using the __ip__ or __ifconfig__ utilities:

```sh
# Check interface is running
ifconfig
tnc0: flags=579<UP,BROADCAST,RUNNING,ALLMULTI>  mtu 576
        inet 10.92.0.10  netmask 255.255.255.0  broadcast 10.92.0.255
        ether 02:56:ad:f2:40:33  txqueuelen 1000  (Ethernet)
        RX packets 0  bytes 0 (0.0 B)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 0  bytes 0 (0.0 B)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0
```

Create a point-to-point link:

```sh
# Attach interface
sudo tncattach /dev/ttyUSB0 115200 --mtu 400 --noipv6 --noup

# Configure IP addresses for point-to-point link
sudo ifconfig tnc0 10.93.0.1 pointopoint 10.93.0.2

# Check interface
ifconfig

tnc0: flags=4305<UP,POINTOPOINT,RUNNING,NOARP,MULTICAST>  mtu 400
        inet 10.93.0.1  netmask 255.255.255.255  destination 10.93.0.2
        unspec 00-00-00-00-00-00-00-00-00-00-00-00-00-00-00-00  txqueuelen 500  (UNSPEC)
        RX packets 0  bytes 0 (0.0 B)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 0  bytes 0 (0.0 B)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

```
