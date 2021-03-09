# Useful commands:

### General(Basic) commands to help:

*Ip addresses and routes*

`ip link`: list and modify interfaces on host

`ip addr`: view ip addresses assigned to interfaces

`ip addr add`: assign ip to an interface

*Example* : `ip addr add 192.168.1.10/24 dev eth0`

*Note* Using `ip addr` is not persistent across reboots. If you want to persist the change, modify the `/etc/network-interfaces`

`ip route`: view routes in the routing table

`ip route add`: add routing entries into the routing table

*Example*: `ip route add 192.168.1.0/24 via 192.168.2.1`

`cat /proc/sys/net/ipv4/ip_forward`: Checks for forwarding on the interfaces

`ip netns`: list namespaces

`ip netns add`: add a network namespace

*Example*: `ip netns add red`

`ip netns exec red ip link`: Looks at the links within the `red` network namespace

`ip link add veth-red type peer name veth-blue`: Connect two namespaces

`ip link set veth-red netns red`: Set veth-red to the red ns

`ip -n red addr add 192.168.15.1 dev veth-red`: add an ip address to the veth-red interface within the red ns

`ip -n red link set veth-red up`: bring up the veth-red interface