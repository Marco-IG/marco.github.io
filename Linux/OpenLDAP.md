# OpenLDAP server on Linux
### Introduction
What you're expecting is some kind of step-by-step tutorial for an OpenLDAP server but I won't do that. The main reason we are here is to learn what is an OpenLDAP server and how to create our own lab to work on.

The things we are going to do here:
- A virtual router
- A DHCP server
- A DNS server
- An OpenLDAP server

Everything We are going to do will be hosted on Ubuntu server 24.04. You can do this on another distro if you want but I did this before on this distro so I'm comfortable with it.
Requirements:
- A machine to deploy every server
- 1 Client

If you want to create the VMs on VirtualBox, you can go for it (I'm going to show how to do it on this program). You can use Proxmox or VMware if It fits you the most, but take in account that proxmox will need and entire pc with decent specifications.

<br/>

### Virtual Router
First of all I'm going to explain why we are doing this. Well, we could just use our own router but to be more flexible and just to make it a bit harder we are going to create our own router to route traffic. Why? Because we can control everything: firewall, traffic, routing tables...
The other thing I would like to show is how NAT works, because by the time we create the rest of the servers inside our LAN we would like to make them go outside of the LAN and connect with Internet, so we will need iptables (maybe we will learn nftables, a modern version of iptables).

After the explanation, before installing the distro we have to give our router two network adapters:
1. Bridged
2. Internal red (the name is up to you, I'm going to leave it on default)

![](../images/1.png)
![](../images/2.png)

After that we got to install everything. I'm not going to show everything about the installation process so I'm going to show you the configuration of the network and leave the rest up to you.

My configuration is this one (remember to change every adapter to manual, because DHCP is on by default):

For enp0s3 (the bridged adapter):
      
      --> Subnet: 192.168.0.0/21
  
      --> Address: 192.168.1.100
  
      --> Gateway: 192.168.1.1
  
      --> Name servers: 8.8.8.8, 8.8.4.4


![](../images/3.png)
<br/>

For enp0s8 (the internal adapter):
      
      --> Subnet: 10.0.0.0/24
  
      --> Address: 10.0.0.1
  
      --> Gateway: 192.168.1.100
  
      --> Name servers: 8.8.8.8, 8.8.4.4


<br/>

![](../images/4.png)
<br/>
<br/>
<br/>



After the installation we could use netplan, but... What is it?
Well, Netplan is a network configuration tool used in modern Ubuntu systems.
It uses YAML files in **/etc/netplan/** to set up network interfaces, including IP addresses, DHCP, gateways, and DNS.


It works with NetworkManager or systemd-networkd to apply the network settings.

Right now we don't want to touch anything related to netplan but I'm going to show you how it looks like right now:
<br/><br/>
![](../images/5.png)

<br/>
<br/>

After that we should install SSH just to facilitate the use of the terminal. If you don't know already, SSH (Secure Shell) is a protocol used to securely connect to remote computers over a network.
It allows you to log in, execute commands, and transfer files safely using encryption. It is commonly used by system administrators and developers to manage servers and virtual machines. The syntax is pretty simple:
      
      $ ssh -p [port of the server] [user]@[server_address]


<br/>
<br/>

The server address should be the IP and the user should be the user we could use in the server. The port, well, it's going to be 22, the SSH default port.


Installing SSH is easy because you just need **$ sudo apt install ssh -y**.

The next think we got to do is enable IP forwarding permanently. Here is a brief explanation about this:

  --> IP forwarding is a setting in the Linux kernel that allows the router to send network packets from one network interface to another. Without it, the router will receive packets from the client but won’t forward them to the Internet.

  --> The FORWARD chain in iptables/nftables handles packets that are being routed through the system, so enabling IP forwarding allows these packets to actually pass through the router.
 
 To make this change permanent (so it stays active after reboot), we edit the sysctl.conf file (/etc/sysctl.conf). This file contains kernel settings that Linux applies at startup. So we just open the file and then, look for the line containing net.ipv4.ip_forward and set it to 1:
 <br/><br/>
 ![](../images/6.png)


After this, we simply apply the updated system parameters without needing to reboot the router. We do this by running:

      $ sudo sysctl -p


Next, we need to create an iptables rule that allows packets from the servers (we will create them after this) to reach the external network and receive responses back through the router. In other words, we want the router to perform NAT so that internal devices can access the outside network using the router’s external IP address. To do this, we add the following rule and we keep it permanent:
            
      $ sudo iptables -t nat -A POSTROUTING -o enp0s3 -j SNAT --to-source 192.168.1.100
      $ sudo netfilter-persistent save

And now we have our own router and LAN, making everything posible thanks to the magic of the NAT technique.

The next thing we should do is make our own DHCP to give IP addresses to our clients. In the next part I will explain how to create it and make it work with a cliente (I won't explain the client part because it's easy and there are tons of tutorials out there to know how to setup a Linux cliente).

<br/>

### DHCP Server
In this part I'm going to create a DHCP server but what exactly is DHCP?

DHCP (Dynamic Host Configuration Protocol) is the service responsible for automatically giving devices on a network the information they need to connect and communicate. Instead of configuring every computer manually, a DHCP server assigns important settings such as the IP address, subnet mask, gateway, and DNS servers as soon as a device joins the network.

When a client connects, it sends out a broadcast request asking for network configuration. The DHCP server listens for that request and responds with a lease — a temporary set of network parameters the client can use. This makes network management simpler, prevents IP conflicts, and allows devices to join the network instantly without any manual setup.

In short, DHCP is what allows your devices to “plug in and work” by automatically configuring their network settings.
