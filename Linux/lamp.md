# Sections
- [Back to Linux page](linux_intro.md)
- [Back to the main page](../index.md)

<br/>

# Creation of a LAMP server
Hello guys we are here again and I'm going to teach you about LAMP servers and how to make one yourself.
But as you're probably wondering, what is a LAMP server? Well, A LAMP stack is basically a bundle of four different tools that work together to run a website. It is the most popular way to set up a web server because it is free, reliable, and very easy to use. Each letter in the name stands for a specific part of the system.

- The L stands for Linux, which is the "brain" or the operating system. It manages everything on the server, just like Windows or macOS does on a laptop. Most people use Debian or Ubuntu for this because they are very stable.
- The A is for Apache, the web server. Think of it as a waiter in a restaurant. When you type a website address in your browser, Apache "takes your order" and brings the website's files to your screen so you can see them.
- The M is for MariaDB (or MySQL), which is the database. This is where all the "memory" of the site is kept, like user accounts, passwords, and posts. It keeps everything organized in tables so the server can find information quickly.
- Finally, the P stands for PHP. This is the language that makes the website do things. It connects the database with the web server to create "dynamic" pages—like when you log into a site and it shows your specific profile instead of a generic page.

After reading all this text let's begin with the LAMP. We just require the virtual router we created on the FreeIPA section and a server with any distro you want. My recomendation is to use Ubuntu server, since it's easy and reliable.

I won't stop at the installation of the server but this are the network config. that I will be using on this server:
      
      --> Subnet: 10.0.0.0/24
  
      --> Address: 10.0.0.60
  
      --> Gateway: 10.0.0.1
  
      --> Name servers: 8.8.8.8, 8.8.4.4

<br/>

First, you should configure the virtual machine in VirtualBox to include it in the same LAN as the IPA server. Alternatively, you can keep the machine on your physical network; either setup will work.
Now, first thing we are going to do is an update and upgrade on the server just to get everything up to date:

      # apt update -y; apt upgrade -y

<br/>

## Apache installation
It's time to start the installations, so the first thing being installed is going to be apache2. It is the web server responsible for handling incoming requests and serving our site's content over the internet.

      # apt install apache2 -y

We should check that the service is running:

      # systemctl status apache2

<br/>

## Mariadb Installation
Now that our Apache web server is up and running, we need a reliable place to store and organize our data. This is where MariaDB comes in, a powerful, scalable, and open-source relational database management system (a popular "fork" of MySQL). First, we need to download and install the server package from the official repositories. We hav to run the following command:

      # apt install mariadb-server -y

<br/>

By default, a fresh MariaDB installation includes settings that are helpful for testing but insecure for a real environment. To fix this, MariaDB provides a security script that we should always run:

      # mariadb-secure-installation

<br/>

Once you run the script, an interactive assistant will guide you through several prompts. Here is what you should choose and why:
