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
- **Enter current password for root (enter for none):** Press Enter. A fresh MariaDB installation does not assign a root password yet, so leaving this blank is expected.
- **Switch to unix_socket authentication? [Y/n]:** Type n. This keeps the traditional password-based authentication instead of tying the root login to the system user via the Unix socket. Using a password gives you more flexibility, especially in labs, VMs, or remote environments—because you can log in as root without depending on the Linux user account.
- **Change the root password? [Y/n]:** Type Y and set a strong and memorable root password. This step is crucial: the root user has full control over all databases, so setting a password ensures that only authorized users can access administrative features.
- **Remove anonymous users? [Y/n]:** Type Y. Anonymous accounts allow anyone on the system to log into the database without a username. This is convenient for testing but extremely insecure. Removing them closes a common attack vector.
- **Disallow root login remotely? [Y/n]:** Type Y. Preventing remote root logins ensures that the most privileged account can only be used locally. This reduces the risk of brute-force attempts or credential theft through the network.
- **Remove test database and access to it? [Y/n]:** Type Y. MariaDB ships with a test database that is accessible to all users by default. Removing it prevents low-privileged users from storing or executing unwanted code in an uncontrolled area.
- **Reload privilege tables now? [Y/n]:** Type Y. This reloads all privilege rules so your changes take effect immediately without restarting the database server.

Now our database is more secure and reliable.

<br/>

## PHP installation
Now that we have Apache and MariaDB installed, the next step is to add PHP to complete our LAMP stack. But instead of using the old mod_php module (which is considered outdated and less secure), we will use PHP-FPM, a faster and more efficient way for Apache to process PHP files.

First of all, let's install php:

      # apt install php-fpm php-mysql -y

<br/>

This command installs two essential components. First, php-fpm, the FastCGI Process Manager, runs PHP as a separate background service. When a user requests a PHP page, Apache forwards that request to PHP-FPM, which processes the PHP code and returns the result. This separation makes the server faster, more stable, and more secure. Second, php-mysql is the PHP extension that allows PHP to communicate with MariaDB or MySQL databases. Without this extension, PHP would not be able to access or manipulate the database.

After this, let's enable two important modules for apache:

      # a2enmod proxy_fcgi setenvif

<br/>

Apache uses modules to extend its functionality, and these two are necessary for PHP-FPM to work. The proxy_fcgi module allows Apache to forward PHP requests to the PHP-FPM service. In other words, when a user opens a .php page, Apache passes it to PHP-FPM for processing. The setenvif module lets Apache set environment variables based on specific conditions, which PHP-FPM uses internally to work correctly. The a2enmod command simply activates these modules.

Now we are going to enable the up-to-date version for php, because we need the 8.4 version (if there a more recent and established version you should use that instead of this one I'm going to use for my lamp). After changing the version we have to reload the server to apply the changes:

      # a2enconf php8.4-fpm
      # systemctl reload apache2

<br/>

Once this is complete, Apache and PHP-FPM are fully integrated, and your server is ready to run dynamic PHP websites like WordPress, phpMyAdmin, or your own web applications.

<br/>

## Final checks
### Checking apache
First, we need to make sure Apache is accessible from your host machine. Since the server is not directly accessible from your PC’s network, the router acts as a bridge. We need to forward a port on the router so that traffic from your PC can reach the server by configuring an iptables rule:

      # iptables -t nat -A PREROUTING -i enp0s3 -p tcp --dport 8080 -j DNAT --to-destination 10.0.0.60:80
      # netfilter-persistent save
      
<br/>

This rule will make our PC reach the web server and after this we just have to put this on the web browser to see the apache configuration file:

      http://192.168.1.100:8080

![]()

