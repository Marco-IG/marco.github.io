# FreeIPA server on Linux
### Introduction
What you're expecting is some kind of step-by-step tutorial for an OpenLDAP server but I won't do that. The main reason we are here is to learn what is an FreeIPA server and how to create our own lab to work on.

The things we are going to do here:
- A virtual router
- A DHCP server
- A FreeIPA server
- 2 clients

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


Next, we need to create an iptables rule that allows packets from the servers (we will create them after this) to reach the external network and receive responses back through the router. In other words, we want the router to perform NAT so that internal devices can access the outside network using the router’s external IP address. To do this, we add the following rule and we keep it permanent (remember to install **iptables-persistent** to use netfilter-persistent):
            
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

So by now we are going to create the DHCP server. Just remember, the adapter has to be on internal mode.

The network configuration is going to be this one:
      
      --> Subnet: 10.0.0.0/24
  
      --> Address: 10.0.0.2
  
      --> Gateway: 10.0.0.1
  
      --> Name servers: 8.8.8.8, 8.8.4.4


![](../images/7.png)


Next, we will install the ISC DHCP server software (developed by the open-source organization ISC) so that we can begin configuring our DHCP server.

      $ sudo apt update
      $ sudo apt install isc-dhcp-server -y

If you want to look at the status of the service you could use this command:

      $ sudo systemctl status isc-dhcp-server


After installing the software, we need to configure the server so that it can start assigning dynamic IP addresses to devices on the network. To do this, we will edit the dhcpd.conf file located in the /etc/dhcp directory, where all the main settings for the DHCP server are defined, including the IP address range, subnet mask, gateway, and DNS servers. This configuration file is essential because it tells the server how to manage and distribute network settings to clients automatically.

At the end of the configuration file, we add the following lines:
      
      # Subnet configuration for the internal network (10.0.0.0/24)
      subnet 10.0.0.0 netmask 255.255.255.0 {
              range 10.0.0.10 10.0.0.50;
              option routers 10.0.0.1;
              option domain-name-servers 8.8.8.8, 8.8.4.4;
              default-lease-time 600;
              max-lease-time 7200;
      }

![](../images/8.png)



This block defines the DHCP settings for your internal network:
1. **subnet 10.0.0.0 netmask 255.255.255.0** specifies the network where the DHCP server will assign IPs.
2. **range 10.0.0.10 10.0.0.50** defines the pool of addresses that clients can receive dynamically.
3. **option routers 10.0.0.1** sets the default gateway for the clients (usually the router).
4. **option domain-name-servers 8.8.8.8, 8.8.4.4** tells clients which DNS servers to use.
5. **default-lease-time 600 and max-lease-time 7200** define the lease duration in seconds (how long a client can keep the assigned IP).

In short, this configuration ensures that any device connecting to your internal LAN automatically receives an IP address, gateway, and DNS settings, making network management much simpler.

After saving the configuration we need to edit the iscp-dhcp-server file located in the /etc/default directory. This file is used to set default parameters for the DHCP server service. It specifies which network interfaces the DHCP server should listen on, and can include other startup options. Editing this file ensures that when the DHCP service starts, it knows exactly which network interface(s) to manage, so it only assigns IP addresses to clients on your internal LAN and avoids affecting other networks. Essentially, it acts as a bridge between your system’s configuration and the running DHCP service.

From there, in the line INTERFACESv4="", we will specify the network interface on which the DHCP server should “listen” and offer IP addresses to clients that send requests for an IP. In my case, the machine’s interface is enp0s3.

![](../images/9.png)


So now our DHCP server is fully configured. We could assign static IPs if needed, but at the moment it’s not necessary since we don’t have any clients or servers that require a fixed IP address.
Before we go to the DNS server I'm going to explain to you how you can fix an IP address to any client but I won't go too deep.


First of all, we need the MAC address for it. If you still don’t know what a MAC address is, it stands for Media Access Control address, a unique identifier assigned to the network interface of every device. Think of it like a device’s fingerprint on the network. The DHCP server uses this MAC address to recognize the device and, if configured, assign it a specific IP address every time it connects. To get it we need the commando **$ ip a**. After this, We will look for the line link/ether, which contains the MAC address of the machine. We will need this address later to manually assign a static IP to this client from the DHCP server. The client’s MAC address is 08:00:27:6b:50:8b.


After this, we will configure a new entry in the dhcpd.conf file located in /etc/dhcp. The entry is as follows:
      
      host client {
        hardware ethernet 08:00:27:6b:50:8b;
        fixed-address 10.0.0.55;
        option routers 10.0.0.1;
        option subnet-mask 255.255.255.0;
      }

This line in the dhcpd.conf file creates a static IP reservation. The host client block tells the DHCP server that the device named "client" should receive a special configuration. The server uses the device’s MAC address (hardware ethernet) to uniquely identify it. When the client connects, the server always assigns it the same predefined IP address (fixed-address 10.0.0.55) instead of an address from the dynamic range.

Lo último que deberíamos hacer es resetear el servicio y la máquina, y tras esto ya tendríamos el cliente con su nueva IP. Para resetear el servicio:
      
      $ sudo systemctl restart isc-dhpc-service


I'm not going to show anything related to the fixed IP because it's not necessary for the FreeIPA, but you got to know that even if you configure the IP of the client manually or you put it on automatic, you should fix the address in the DHCP server, so it won't produce erros or problems if any other device recive the same IP. Just as advice.


### FreeIPA Server
We will now begin the deployment of our FreeIPA server. We have selected CentOS for this environment, as FreeIPA offers better native compatibility and a more robust integration path compared to Ubuntu Server (you can obtain the CentOS 10 ISO from the official repository here: https://www.centos.org/download/).
Before proceeding with the configuration, let’s review the network topology. We will be provisioning three virtual machines within our private LAN:
1. IPA Master Server
2. IPA Replica Node
3. Client Node"

<br/>

**Understanding IdM and FreeIPA**

You can think of FreeIPA as the Linux equivalent of Active Directory. It is an open-source Identity Management (IdM) solution that centralizes authentication, authorization, and policy enforcement.

<br/>

**Why use a Centralized Domain?**

In a standard environment, every server acts as an island—passwords are stored locally, and each machine must be managed individually. This creates massive administrative overhead and security risks. IdM solves this by creating a unified Linux domain where all information is stored in one place and applied uniformly to every machine.

<br/>

**How it Works?**

The architecture is built on three core pillars:
1. **LDAP:** The directory service that stores all user, group, and policy data.
2. **Kerberos:** The engine for Single Sign-On (SSO). Once a user authenticates, they can access any service in the domain without re-entering their credentials.
3. **Integrated Services:** The server also manages DNS for service discovery and a Certificate Authority (CA) for secure communication.

<br/>

**The Role of Clients**

IdM Clients are machines enrolled in the domain to receive these identities and policies. While the domain is Linux-native (meaning it does not support Windows clients directly), it can be integrated with existing Active Directory environments to share identities across your entire enterprise.


So now we know what is FreeIPA and its relation with IdM and RedHat.

For now, we will focus on the installation. Ensure that you configure the network settings during the initial setup. While you can choose your own partitioning and system settings, make sure your network adapter is set to Internal Network (or Host-Only) to isolate the traffic within your lab.

Once the OS is installed and you have logged in, our first task on the Master Server is to update the hostname. For this guide, I will use ipa1.lab.local.

I am using .local because of specific restrictions regarding Top-Level Domains (TLDs). Using reserved names like localhost or public domains like example.com can cause conflicts. For more details, see (https://datatracker.ietf.org/doc/html/rfc2606).+


Why do we have to change the hostname?

Well, the reason is clear: In a domain, your machine needs a full identity.

To understand this, think of the hostname as the "first name" of your computer (example: ipa1) and the domain as its "last name" or "neighborhood" (example: lab.local). When you put them together, you get the FQDN (Fully Qualified Domain Name): ipa1.lab.local.


In FreeIPA, this relationship is vital because:

- **It’s like an ID Card:** FreeIPA uses a system called Kerberos to handle logins. Kerberos is very strict; it doesn't recognize machines by their IP addresses, only by their full names (FQDN). If the name is wrong, you can't log in.

- **DNS is the Foundation:** FreeIPA acts as a phonebook for your network. For other machines to find the server and trust it, the server must have a fixed, clear name that doesn't change.

- **Security Certificates:** The server creates security certificates (like digital signatures) to protect your data. These signatures are tied specifically to the hostname. If the name is incorrect, the security system will block the connection.


In short, changing the hostname ensures that your server has a unique and permanent identity so the entire network knows exactly who it is talking to.
