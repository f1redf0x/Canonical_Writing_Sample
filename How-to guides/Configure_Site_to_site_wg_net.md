# Setting up A Hub and Spoke Site-to-Site VPN with WireGuard

In the following How-to, you're going to set up the following:

1. A Hub device that routes traffic to two spoke nodes and sits on a public network (192.168.100.30).

2. A Spoke Node, hereafter Node1, device that sits on it's own private network (10.1.0.0/24) and public network (192.168.100.31)

3. A second Spoke Node, hereafter Node2, device that sits on two of it's own private networks (10.2.0.0/24 and 10.3.0.0/24) and public network (192.168.100.32)

4. A VPN network (169.254.0.0/16). This address was chosen because this IP space is dedicated for [Link-Local](https://en.wikipedia.org/wiki/Link-local_address) activities like VPNs and won't suffer any conflicts.

5. Static routing between nodes attached to the hub for site to site communication.

6. Three "Client Machines".
    1. Client1 which sits on the 10.1.0.0 network and has an ip address of: 10.1.0.11.
    2. Client2 which sits on the 10.2.0.0 network and has an ip address of: 10.2.0.11.
    3. Client3 which sits on the 10.3.0.0 network and has an ip address of: 10.3.0.11

By the end, devices sitting in the first spoke's private network should communicate with devices on the second spoke's private networks.

# What you need to begin

* A Hypervisor like VMWare, Xen, or VirtualBox.

* 6 Ubuntu 22.04 LTS virtual machines - each configured with the public addressing schemes held above.

* A "Nat Network" Network adapter on the 192.168.100.0/24 subnet that connects all 3 VMs.

* An "Internal Network" Network adapter for private network 10.1.0.0/24 for Spoke Node 1 and client1.

* An "Internal Network" Network adapter for Private network 10.2.0.0/24 for spoke Node 2 and client2.

* An "Internal Network" Network adapter for Private network 10.3.0.0/24 for spoke Node 2 and client3.

* The root account on each machine. The assumption for all of these commands is that you will be using root (run `sudo su`). Commands run from an average user account will likely cause unintended permissions errors. 

For reference on Network Adapters, the following video is instuctive for understanding [VirtualBox network adapters](https://www.youtube.com/watch?v=Fhdxk4bmJCs&t=418s).

# Creating Wireguard Keys

On all 3 devices, the administrator will need to install WireGuard, configure public-private key pairs, and set up WireGuard's configuration file. The following commands will be executed on all 3 nodes as the root user:

First update your Ubuntu 22.04 LTS 22.04 LTS instances to make sure that their internal package list is up-to-date.
```
apt update
```

Second, install the WireGuard package using the apt package manager.
```
apt install wireguard
```

Third, go to the WireGuard configuration directory and set restrictive permissions. This will ensure that only the root user can access WireGuard's keys
```
cd /etc/wireguard
```
```
umask 007
```
Fourth, generate the key-pairs on all three hosts. The following command will give the administrator 2 files, one called privatekey and the other called publickey:
```
wg genkey | tee privatekey | wg pubkey > publickey
```
To print out the keys for copying and pasting, use the command:
```
grep '' *
```

<div class="p-notification--information">
  <div class="p-notification__content">
    <h5 class="p-notification__title">Notes</h5>
    <p class="p-notification__message">WireGuard does not provide a method for you to transfer keys from host to host and leaves that to your discretion. One quick method for transferring keys is to copy-paste using the last provided command. If the user is pasting between virtual machines, ensure that the hypervisor has enabled "bi-directional" clipboard access. If the user isaccessing or configuring virtual machines with ssh, the method for copying will depend on the user's terminal emulator. This is often accomplished with ctrl + shift + c and ctrl + shift + p. </p>
  </div>
</div>

At this point, all 3 virtual machines have WireGuard installed and their keys are ready for configuration.

# Setting up the Point to Point Link Between Hub and Node1

To Begin, we are going to configure the Hub machine's wireguard instance to point at node1. From the hub, run the following commands:

Navigate to the WireGuard Configuration Directory:
``` 
cd /etc/wireguard
```
Use a text editor to create the WireGuard interface config file. In this case we are using vi:
```
vi wg0.conf
```
The configuration file should resemble the following:

```
[Interface]
PrivateKey = <insert_private_key_here>
ListenPort = 60031

#node1
[Peer]
PublicKey = <insert_node1_public_key_here>
Endpoint = 192.168.10.31:60031
AllowedIPs = 0.0.0.0/0
```

<div class="p-notification--information">
  <div class="p-notification__content">
    <h5 class="p-notification__title">Notes</h5>
    <p class="p-notification__message">WireGuard uses the AllowedIPs field in the configuration file to set up routing tables whenever the admin launches the config using the wq-quick utility. If a user is setting up static routes, it's important that they use the ip range of the private networks that the admin wants to expose to the vpn, otherwise WireGuard will be unaware of these routes. If the Administrator wants to set up dynamic routing, then the 0.0.0.0/0 subnet is signal to WireGuard that all addresses are welcome to talk to the adapter, but it's up to the admin or a routing daemon to handle configuration. In this "How-To", the user will be setting up manual routes. </p>
  </div>
</div>

Now exit the configuration by pressing the escape key, followed by ":wq", then hit enter. This is how to save a document in vi. Now go to node1.

On node1, go to the configuaration directory:
```
cd /etc/wireguard
```
Use a text editor to create the WireGuard interface config file.
```
vi wg0.conf
```
The configuration file for node1 will read as follows:
```
[Interface]
PrivateKey = <insert_node1_private_key_here>
ListenPort = 60031

#hub
[Peer]
PublicKey = <insert_hub_public_key_here>
Endpoint = 192.168.10.30:60031
AllowedIPs = 0.0.0.0/0

```
Now that both Hub and Node1 are configured, it's time to configure the adapters on both. WireGuard provides a script called wg-quick that can quickly setup WireGuard interfaces. Although this is the easiest method, this "How-To Guide" will be using the manual method, as the user will gain more understanding on how wireguard interfaces work.

On both the Hub device and Node1, run the following commands:

First, use the ip command to setup the the interface wg0 on the machine.
```
ip link add dev wg0 type wireguard
```
Check to make sure that the interface is properly set up, but down:
```
ip address show dev wg0
```
Now the administrator is going to focus on the Hub specifically. Configure the wg0 interface on the hub to have ip address 169.254.0.1 and tell it to expect a peer of 169.254.0.2:
```
ip a add 169.254.0.1 peer 169.254.0.2 dev wg0
```
Confirm that the interface is configured correctly with:
```
ip -br a s dev wg0
```
On Node1, repeat the above configuration commands, but in reverse:
```
ip a add 169.254.0.2 peer 169.254.0.1 dev wg0
```
And confirm that the interface is configured correctly:
```
ip -br a s dev wg0
```
Now that the interfaces have been configured on Ubuntu 22.04 LTS, it's time to tell Wireguard to use them. On both Hub and Node1, run the following commands:
This first command tells WireGuard to use the wg0.conf file for the newly configured wg0 adapter:
```
wg setconf wg0 wg0.conf
```
The next command tells linux to bring the adapter on line.
```
ip link set wg0 up
```

# Testing the vpn link

The Hub machine and Node1 are now connected via WireGuard. To confirm the connection, ping Node1 from the Hub:
```
ping 169.254.0.2
```
A successful ping indicates that the tunnel is up and the nodes are connected. Another way to check the tunnel is to use the wg command, which shows basic configuration information including networking info.
```
wg
```
At this point, the user has successfully created a WireGuard tunnel between the Hub and Node1. It's time to work on the connection between the Hub and Node2.

# Setting up the Point to Point Link Between Hub and Node2

To save some typing on the hub, copy the wg0.conf file to wg1.conf by first going to the configuration directory on the hub:
```
cd /etc/wireguard
```
And copy the config file with cp:
```
cp wg0.conf wg1.conf
```
There are 4 changes that need to be made:
```
...snip...
ListenPort = 60032
...snip...
PublicKey = <insert_node2_public_key_here>
Endpoint = 192.168.10.32:60032
AllowedIPs = 0.0.0.0/0
...snip...

```
A few observations:

* In this case, ...snip... is an indication to skip over the lines that are not explicitly written out. 

* The ListenPort needs to be changed to avoid a sharing conflict. 

* The Public Key needs to be replaced with Node2's Public Key.

* The Endpoint address needs to point to Node2's IP and the new port.

To configure Node2, go to Node2's terminal interface and type the following:
```
cd /etc/wireguard
```
```
vi wg0.conf
```
The wg0.conf file should read as follows:
```
[Interface]
ListenPort = 60032
PrivateKey = <insert_node2_private_key_here>

#hub
[Peer]
PublicKey = <insert_hub_public_key_here>
Endpoint = 192.168.10.30:60032
AllowedIPs = 0.0.0.0/0
```

Now that the configuration files are setup, it's time to set up the adapters, starting with the hub. On the hub only, run:
```
ip link add dev wg1 type wireguard
```
```
ip a add 169.254.0.1 peer 169.254.0.3 dev wg1
```
```
wg setconf wg1 wg1.conf
```
```
ip link set wg1 up
```
On Node2, run the following commands to set up the wg0 interface:
```
ip link add dev wg0 type wireguard
```
```
ip a add 169.254.0.3 peer 169.254.0.1 dev wg0
```
```
wg setconf wg0 wg0.conf
```
```
ip link set wg0 up
```
Then ping the Hub from Node2 to confirm both links are working:
```
ping 169.254.0.1
```
A successful ping indicates that the tunnel between Node2 and the Hub is up and ready to go. 

# Setting up IPv4 Static Routes
At this point, the administrator has set up a transitive link from Node1 to the Hub, and from the Hub to Node2. To confirm this, run `wg` on the hub to see if it has 2 active peers.

Each Node can talk to the Hub and the Hub can talk to each node separately, but the nodes can't talk to eachother. To enable inter-node communication, 2 things must happen:

1. Each device needs to have packet-forwarding enabled. Packet forwarding tells each device to transmit packets intended for another device instead of dropping them. This requires a configuration change.

2. Each device needs the other nodes' routes in their routing tables. This ensures that each device knows which interface to send packets so that they reach their destination.

## Setting up packet forwarding
On all three devices run the following:
```
vi /etc/sysctl.conf
```
In the sysctl.conf file. Uncomment the line that says:
```
#net.ipv4.ip_forward=1
```
It should look like this, note the missing "pound sign" or "hashtag":
```
net.ipv4.ip_forward=1
```
Since the hub acts as the main interface for the Spoke Nodes, the routing table needs to be set up on the Hub:
```
ip r add 10.1.0.0/24 dev wg0
```
```
ip r add 10.2.0.0/24 dev wg1
```
```
ip r add 10.3.0.0/24 dev wg1
```
Next, let's tell Node1 how to find Node2's private networks. From Node1, run:
```
ip r add 10.2.0.0/24 dev wg0
```
```
ip r add 10.3.0.0/24 dev wg0
```
Finally, let's tell Node2 how to find Node1's private networks. From Node2, run:
```
ip r add 10.1.0.0/24 dev wg0
```

To confirm that each subnet is associated with the correct network adapter, run the following:
```
ip r get 10.1
```
This command checks the routing table for networks beginning with 10.1. This should be pointed to the wg0 device. Confirm that the networks are pointing to the right adapter. If the private network is pointed to an ethernet adapter, it's likely that you are on the node with that private network. For example, Node1 will have 10.1.0.0 on it's ethernet interface since it's the host.

At this point, test your client device connections. Despite being on seperate subnets, client1 can ping client2:
```
ping 10.2.0.11
```
and client3
```
ping 10.3.0.11
```

# Next Steps

This project is now complete, but there are a number of improvements that can be made:

1. Although the client machines can traverse the hub and talk to other client machines, they cannot access the internet. This requires "Masquerading" to be enabled on the Hub and Spokes using iptables or nftables.

2. Static Routing is pretty fragile, and will require manual maintenance. It's recommended to use a dynamic routing protocol like OSPF to onboard new devices.

3. Wireguard should be configured to act as a systemd daemon to enable easier management and logging.
