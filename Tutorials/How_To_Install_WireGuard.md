# Installing Wireguard on an Ubuntu 22.04 LTS Device

WireGuard is a fast, lightweight, and easy to set up VPN that uses [UDP](https://en.wikipedia.org/wiki/User_Datagram_Protocol) for transporting packets quickly and [asymmetric cryptography](https://en.wikipedia.org/wiki/Public-key_cryptography) for keeping those packets safe. To install it follow these steps:

# For traditional Wireguard installation (.debs)

## Update your system

From your Ubuntu 22.04 LTS machine, update your packages with:

```
sudo apt update
```

## Install the WireGuard Package:

```
sudo apt install wireguard
```

# For Snap WireGuard Installation

Installing WireGuard with snap is as simple as running the command:

```
sudo snap install wireguard
```

snap will automatically update the core package, followed by the wireguard package. 
