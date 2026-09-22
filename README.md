_This project has been created as part of the 42 curriculum by adede._

# Born2beRoot

> This document is a System Administration related exercise.

## Description

**Born2beRoot** is a system administration project focused on virtualization of a Linux server configuration. The goal is to create and configure a minimal server inside a Virtual Machine (VM) while following strict requirements.

With this project, we become able to set up our own operating system while implementing basic server configurations, like: storage, authentication, user management, firewall configuration, SSH access, and system monitoring.

### General Guidelines

- We should use **VirtualBox** or **UTM** as the virtualizor.
- We choose the latest stable version of either **Debian** or **Rocky** as our operating system (no testing/unstable).
- It is **forbidden** to install a graphical interface, like: _X.org_, _Wayland_ or any other equivalent graphics server. So we should install a minimal set of services.
- We create a `signature.txt` at the project root to ensure that the VM we created is not modified between evaluations.
- There should be **no snapshots** at the beginning of the evaluations.

### Project Description

For this project, we chose **Debian** as the operating system. It's recommended for beginners in system administration because of its relatively simple installation and configuration process, extensive documentation, large community, and mature package management system.

But it's main disadvantages are that some software versions can be older than those other more frequently updated distributions, and certain enterprise-oriented tools are less central to the default Debian experience.

#### Debian vs Rocky Linux

| Debian                                          | Rocky Linux                                                  |
| ----------------------------------------------- | ------------------------------------------------------------ |
| Community-driven Linux distribution.            | Enterprise-oriented Linux distribution.                      |
| Uses `apt` and `dpkg` for package management.   | Uses `dnf` and RPM for package management.                   |
| Generally simpler to approach for beginners.    | Provides an environment close to _Red Hat Enterprise Linux_. |
| Uses **AppArmor** for mandatory access control. | Uses **SELinux** for mandatory access control.               |
| Large community and extensive documentation.    | Strong focus on stability and compatibility.                 |

#### Main System Design

1. The VM is configured without a graphical environment because the project requires a minimal server. The system uses encrypted partitions managed with **LVM**, providing logical storage management while protecting the contents of the encrypted partitions.
2. `sudo` is configured with additional security restrictions. Sudo activity is logged under `/var/log/sudo/`.
3. A regular user is created and assigned to the `user42` and `sudo` groups.
4. A strong password policy is enforced via `libpam-pwquality` authentication module.
5. The server is configured with **SSH** on port `4242`.
6. The firewall is configured with **UFW**, with port `4242` allowed for SSH while other incoming connections remain blocked.
7. Finally, a Bash `monitoring.sh` script periodically broadcasts system information to all terminals. It reports crutial system information.

#### AppArmor vs SELinux

AppArmor and SELinux are Linux security mechanisms that implement mandatory access control.

**AppArmor**
: uses application profiles to define which files and system capabilities a program can access. Its profile-based approach is generally easier to understand and configure.

**SELinux**
: uses security labels and policies to control access between processes, files, users, and other system resources. It provides very fine-grained access control, but its policy model can be more complex.

Since Debian was selected for this project, **AppArmor** is used and configured to start automatically with the system.

#### UFW vs firewalld

**UFW (Uncomplicated Firewall)**
: is a simplified firewall management interface commonly used on Debian-based systems. It provides an easy way to create and manage firewall rules using straightforward commands.

**firewalld**
: is a dynamic firewall management solution commonly associated with Red Hat-based distributions such as Rocky Linux. It supports zones and allows firewall configuration to be changed without restarting the entire firewall service.

Because the project uses Debian, **UFW** was selected to implement the required firewall configuration.

#### VirtualBox vs UTM

**VirtualBox**
: is a general-purpose virtualization platform that supports running VMs on multiple operating systems and architectures.

**UTM**
: is an alternative virtualization application, particularly useful on Apple Silicon Macs where virtualization requirements can differ.

The project uses **VirtualBox** as the primary virtualization solution. UTM can be used as an alternative when VirtualBox cannot be used.

## Instructions

The following section provides an overview of the operations. It is up to you to check if the requirements are met and understand why things are done the way they are.

### Choosing the Linux Distribution

> **WARNING**
> If you're part of the _42 Network_, you might consider different storage options for your files that best suits for your campus workstations, like using the `sgoinfre/` folder or an external drive.

First, we need to get the latest "amd64" ISO image of Debian from [Debian's official website](https://www.debian.org/distrib/netinst).

Then in VirtualBox; create a new VM, name it, specify the downloaded ISO image, and select a storage location. Make sure to check "Skip Unattended Installation".

Now you can allocate more resources for the machine by increasing the 'Base memory' and 'Processor' fields. Try not to overdo and crash your host computer. Check "Pre-allocate Full Size".

### Operating System Installation

Now that our VM is ready, the next step is to install the operating system.

1. Start your VM.
2. Select "Install".
3. Continue with the instructions until you see the network configurations.
4. Set the hostname as specified by the project guidelines. You can skip domain name configuration.

> **INFO**
> If you're part of the _42 Network_, it is recommended **not to use** real passwords that you use in real life, so you can freely discuss it with your peers while evaluating the upcoming password management section.

5. Set a password for the 'root'.
6. Configure a user with the login and password information.

#### Disk Partitioning

> **INFO**
> For disk partitioning, we will fulfill **bonus** requirements. Just a heads up.

1. Select "Guided - use entire disk and set up encrypted LVM".
2. Select "Separate /home, /var and /tmp partitions".
3. Set an encryption passphrase.
4. Adjust the volume group size in guided partitioning to carve out space for a couple more logical volumes.
5. Make two logical volumes: "srv" and "var-log".
   1. Head over to "Configure the Logical Volume Manager"
   2. Create both logical volumes
   3. Allocate just enough space to distribute the remaining space to both of them
6. Mount the two volumes
   1. Select them from the LV list
   2. Set them both as "Ext4 journaling file system"
   3. Mount them respectively to `/srv` and `/var/log`.

#### Tidying up

1. Avoid scanning for extra installation media.
2. Select the default Debian archive mirror and leave the proxy settings blank.
3. Choose not to participate in the package usage survey. Let's not pollute the survey results.
4. Ensure to install the GRUB loader.
5. For software selection, only select "SSH server" and "Standard system utilities".

### SSH Setup

We can now log into the server, but what if we want to access it remotely? It's time to set up SSH.

Since we have no `sudo` configured yet, log in as root:

```bash
su -
```

and then, check the SSH service status:

```bash
systemctl status ssh
```

#### SSH Server Configuration File

To change the SSH daemon configuration, edit the file located at `/etc/ssh/sshd_config`.

```ini
#Port 22
Port 4242

#PermitRootLogin prohibit-password
PermitRootLogin no
```

Once done, restart SSH service:

```bash
systemctl restart ssh
```

#### VirtualBox Port Forwarding

Back in VirtualBox:

1. Go to your VM's **Settings (Expert) > Network > Port Forwarding**.
2. Add a new rule to redirect an available "Host Port" to the "Guest Port (4242)".

## Resources
