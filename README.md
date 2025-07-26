
# OpenStack Automated Deployment Scripts

A collection of Bash scripts to automate the installation and initial configuration of an OpenStack Rocky environment on CentOS/RHEL-based systems. These scripts streamline prerequisites, network setup database provisioning, and Packstack-based deployment for a quick, repeatable OpenStack installation.

## Table of Contents

- [Features](#features)
- [Prerequisites](#prerequisites)
- [Repository Structure](#repository-structure)
- [Quick Start](#quick-start)
- [Detailed Installation Steps](#detailed-installation-steps)
    - [1. Pre-installation Setup](#1-pre-installation-setup)
    - [2. Network Configuration](#2-network-configuration)
    - [3. Database Installation](#3-database-installation)
    - [4. Memcached Configuration](#4-memcached-configuration)
    - [5. Packstack Deployment](#5-packstack-deployment)
- [Post-Deployment](#post-deployment)
- [Troubleshooting](#troubleshooting)
- [Contributing](#contributing)
- [License](#license)


## Features

- **Chrony time synchronization**
- **SELinux disabled** and **firewall/network manager** turned off
- **Custom network bridge** set up via `br-ex`
- **MariaDB** installation and secure configuration
- **Memcached** installation for OpenStack services
- **Packstack** answer file generation and automated deployment
- **One-command** “all.sh” orchestrator for end-to-end setup


## Prerequisites

- A CentOS/RHEL 7 or 8 server with at least:
    - 4 GB RAM (8 GB+ recommended)
    - 2 CPU cores
    - Two network interfaces (e.g., `ens33`, `ens34`)
    - 20 GB disk available
- Root or sudo privileges
- Access to the Internet for package repositories


## Repository Structure

```text
openstack/
├── all.sh              # Master orchestrator
├── pre-install.sh      # Installs dependencies, Chrony, repos, disables SELinux & firewall
├── ipconfigs.sh        # Prompts for IPs and generates bridge/network scripts
├── mysql.sh            # Installs and secures MariaDB for OpenStack
├── memcached.sh        # Installs and configures Memcached (see file)
├── all-script.txt      # Alternate master script variant
├── IP-s.txt            # Sample IP mapping for interfaces
└── README.md           # Project documentation (this file)
```


## Quick Start

1. Clone the repository:

```bash
git clone https://github.com/volomi/openstack.git
cd openstack
chmod +x *.sh
```

2. Run the orchestrator:

```bash
sudo ./all.sh
```

3. Follow prompts for network IPs, gateway, DNS, and secure database password.
4. After Packstack completes, source the admin credentials:

```bash
source /root/keystonerc_admin
```


## Detailed Installation Steps

### 1. Pre-installation Setup

Script: `pre-install.sh`

- Installs Chrony time service and configures it to sync with the management interface.
- Enables the OpenStack Rocky repository and updates the system.
- Installs `python-openstackclient` and `openstack-selinux`.
- Disables SELinux enforcement, FirewallD, and NetworkManager.


### 2. Network Configuration

Script: `ipconfigs.sh`

- Prompts for:
    - Management interface IP (`ens33`)
    - External interface IP (`ens34`)
    - Gateway
    - DNS
- Generates:
    - `ifcfg-br-ex` (external bridge)
    - `ifcfg-ens33`, `ifcfg-ens34`
    - Updates `/etc/sysctl.conf` for IPv4 forwarding


### 3. Database Installation

Script: `mysql.sh`

- Installs `mariadb`, `mariadb-server`, and Python PyMySQL driver.
- Enables and starts MariaDB service.
- Runs `mysql_secure_installation` for root password, remote root access, and sample database cleanup.
- Creates `/etc/my.cnf.d/openstack.cnf` with recommended InnoDB settings.


### 4. Memcached Configuration

Script: `memcached.sh`

- Installs and configures Memcached for Keystone token caching.
- Binds Memcached to the management interface for security.
- Enables and starts the service.


### 5. Packstack Deployment

- Installs the `openstack-packstack` package.
- Generates an answer file:

```bash
sudo packstack --gen-answer-file /root/answers.txt
```

- Edit `/root/answers.txt` to customize parameters (network, passwords, services).
- Run Packstack with the answer file:

```bash
sudo packstack --answer-file /root/answers.txt
```

- On completion, source the admin credentials:

```bash
source /root/keystonerc_admin
```


## Post-Deployment

- Verify services:

```bash
openstack service list
openstack network list
```

- Access Horizon dashboard at `http://<controller-ip>/dashboard` using `admin` credentials.


## Troubleshooting

- **Chrony unsynchronized**: Check `/var/log/chrony/chrony.log` and verify interface IP in `/etc/chrony.conf`.
- **Packstack errors**: Review `/var/log/packstack/packstack*.log` for detailed failure information.
- **Database connection refused**: Ensure MariaDB is listening on the management network and firewall is disabled.


## Contributing

1. Fork this repository.
2. Create a feature branch: `git checkout -b feature/your-feature`.
3. Commit changes with descriptive messages.
4. Submit a pull request detailing improvements.

## License

This project is licensed under the **Apache 2.0 License**. See [LICENSE](LICENSE) for details.

<div style="text-align: center">⁂</div>

[^1]: all.sh

[^2]: ipconfigs.sh

[^3]: memcached.sh

[^4]: mysql.sh

[^5]: pre-install.sh

[^6]: all-script.txt

[^7]: IP-s.txt

