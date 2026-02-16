# Sections
- [Back to Linux page](linux_intro.md)
- [Back to the main page](../index.md)

<br/>

# IAM architecture within Linux infraestructure
### Introduction
In this project, the goal is to move beyond the basic deployment of a FreeIPA server and implement a fully functional Identity and Access Management (IAM) architecture within a Linux-based infrastructure. While installing FreeIPA allows for centralized authentication through LDAP and Kerberos, its true value lies in its ability to enforce access control policies across multiple systems in a unified domain environment.

To demonstrate this, a simulated enterprise infrastructure will be deployed inside a private laboratory network. This environment will include several servers representing different operational roles commonly found in production environments, such as application servers, database servers, reverse proxies, backup nodes, and monitoring systems. Each of these machines will be enrolled into the FreeIPA domain and governed by centralized identity policies.

Rather than managing users locally on each system, identities will be defined once within the directory service and then assigned permissions based on their functional role. By organizing users and hosts into logical groups, it becomes possible to implement Role-Based Access Control (RBAC) mechanisms that determine who can access specific services on specific hosts. This allows administrators to enforce the principle of least privilege, ensuring that users are granted only the permissions required to perform their tasks.

Additionally, privileged command execution will be centrally managed using FreeIPA’s sudo policy framework, eliminating the need to manually configure access rules on individual machines. This approach not only simplifies system administration but also enhances security by providing consistent policy enforcement across the entire infrastructure.

By the end of this project, the FreeIPA deployment will evolve from a simple authentication service into a centralized access control system capable of managing user identities, host access, and administrative privileges across a multi-host Linux environment.

</br>

### Creation of a FreeIPA Client Template Machine
Before creating all the servers that will be part of the infrastructure, we first need to set up a base virtual machine that will act as a template for the rest of the nodes.

When working with FreeIPA, every server that joins the domain needs some specific configuration in order to authenticate correctly, such as proper DNS resolution, time synchronization with the IPA server, and the installation of the FreeIPA client packages for LDAP and Kerberos integration.

If we had to configure all of this manually on every server, we would be repeating the same process multiple times, which increases deployment time and the chances of making mistakes. To avoid this, we will create a template machine that is already integrated into the FreeIPA domain. Later on, this template will be cloned to generate the different servers of the infrastructure, ensuring that all of them share the same base configuration.
