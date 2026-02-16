# Sections
- [Back to Linux page](linux_intro.md)
- [Back to the main page](../index.md)

<br/>

# IAM architecture within Linux infraestructure
### Introduction
## Introduction
In this project, the goal is to move beyond the basic deployment of a FreeIPA server and implement a fully functional Identity and Access Management (IAM) architecture within a Linux-based infrastructure. While installing FreeIPA allows for centralized authentication through LDAP and Kerberos, its true value lies in its ability to enforce access control policies across multiple systems in a unified domain environment.

To demonstrate this, a simulated enterprise infrastructure will be deployed inside a private laboratory network. This environment will include several servers representing different operational roles commonly found in production environments, such as application servers, database servers, reverse proxies, backup nodes, and monitoring systems. Each of these machines will be enrolled into the FreeIPA domain and governed by centralized identity policies.

Rather than managing users locally on each system, identities will be defined once within the directory service and then assigned permissions based on their functional role. By organizing users and hosts into logical groups, it becomes possible to implement Role-Based Access Control (RBAC) mechanisms that determine who can access specific services on specific hosts. This allows administrators to enforce the principle of least privilege, ensuring that users are granted only the permissions required to perform their tasks.

Additionally, privileged command execution will be centrally managed using FreeIPA’s sudo policy framework, eliminating the need to manually configure access rules on individual machines. This approach not only simplifies system administration but also enhances security by providing consistent policy enforcement across the entire infrastructure.

By the end of this project, the FreeIPA deployment will evolve from a simple authentication service into a centralized access control system capable of managing user identities, host access, and administrative privileges across a multi-host Linux environment.
