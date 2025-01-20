# Network Topologies

When setting up a network, it's important for any administrator to select a network that is in line with their goals. Do you want centralized control that enables you to manage all of your machines from a single hub or do you prefer enabling every machine to talk to every other machine? How flexible and fault tolerant do you need the environment to be?

When trying to make these judgement calls, it can be helpful to understand some of the options you might deploy.

## Mesh Network:

In a mesh network, every machine can route to every other machine in the network. Assuming you had four machines: A, B, C, and D, each machine would have a direct connection to each other machine.

- A can talk to B, C, and D
- B can talk to A, C, and D
- C can talk to A, B, and D
- D can talk to A, B, and C

This network topology offers a reliable and robust method for packets to travel across the network. Even if you lost machines A and B, C and D would not be impacted by this loss because no machine acts as a [single point of failure](https://en.wikipedia.org/wiki/Single_point_of_failure) in the network.

Although a Mesh network is the most resilient, it does come with potential drawbacks. 

- The implementation and maintenance of a mesh network is more complicated and time consuming.

- This implementation assumes that all devices are on the same network. Devices behind NATs and Firewalls will add complexity to your network.

- The number of connections can quickly become overwhelming because each new node adds a connection for each existing node, and each previous node needs to be updated with the new device.
    - A 4 node network has 6 connections to manage
    - A 5 node network has 10 connections to manage
    - A 6 node network has 15 connections to manage


## Hub and Spoke Network:

In a Hub and Spoke Network, there is a central hub that acts as the link that binds the "spoke" machines together and performs all routing functions. Assuming you had four machines: A, B, C, and D, where machine A is your hub:

- A can talk to B, C, and D
- B can talk to A and the others through A
- C can talk to A and the others through A
- D can talk to A and the others through A

This method is far easier to set up and manage because once you have set up one spoke to talk the hub, each added spoke just needs a connection to the hub. Furthermore, A hub and spoke topology doesn't care about network segregation. As long as the Hub has a public IP address that is accessible to all of the spokes, then each node can communicate by using the [hole punching](https://en.wikipedia.org/wiki/Hole_punching_(networking)) properties of the hub.

The Hub and Spoke network has one major issue, and that is that the hub acts as a single point of failure. If the machine hosting "node A" goes down, none of the machines can communicate.

## Hybrid Topology

In a Hybrid topology, you combine the simplicity of the Hub and Spoke with the resiliency of the mesh. This is accomplished by creating multiple hub nodes in the network and configuring the spokes to communicate with both hubs. Assuming you have 4 machines: A, B, C, and D, where machines A and B are both hubs:

- A can talk to B, C, and D
- B can talk to A, C, and D
- C can talk to A, B and the others through A and B
- D can talk to A, B and the others through A and B

This topology strikes a solid balance between reliability and simplicity. It also serves as a potential method for creating a site-to-site network where all the spoke computers can talk to all the spoke computers on another network because both hubs traditionally sit on an easily reachable network like the internet.

The downsides for this Topology are inherited from "Mesh" and "Hub and Spoke".

- It's more complicated to set up than Hub and Spoke.
- It's less resilient than Mesh.

But it is a solid balance between the two.

# Reference

For a visual presentation of these ideas, please see the video [Using WireGuard for Hub and Spoke Site-to-site VPN](https://www.youtube.com/watch?v=_CbFG_4BInk). The lesson can be found at timestamp: (1:24 - 4:09)
