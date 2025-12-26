## Introduction
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

## Virtual Router
First things first, let's open VirtualBox and create the server
