# High Availability Web Cluster Automation with Ansible

## Project Overview

This repository contains an Ansible automation project designed to deploy a fully redundant, High Availability (HA) web infrastructure. The project orchestrates the configuration of a Load Balancer, a Virtual IP (VIP) failover mechanism, and Web Servers across multiple nodes.

A distinct feature of this infrastructure is its secure network topology: the worker nodes operate without direct internet access. All outbound traffic (e.g., package installation via YUM) is tunneled through a Squid Proxy server running on the Ansible Controller node.

## Network Architecture

The architecture consists of three main components:

1.  **Controller Node:** Hosting Ansible and acting as a Squid Proxy Gateway.
2.  **Worker Nodes (x2):** Hosting the Web Stack (Apache), Load Balancer (HAProxy), and Failover Logic (Keepalived).
3.  **Virtual IP (VIP):** A floating IP address managed by Keepalived to ensure single-point-of-entry and redundancy.

![System Architecture](./diagrams/architecture_diagram.png)

## Directory Structure

The project follows the standard Ansible Roles structure for modularity and maintainability.

```text
.
├── ansible.cfg             # Main Ansible configuration file
├── inventory               # Host inventory file (Groups definition)
├── playbook001.yaml        # Main playbook entry point
├── diagrams/               # Architecture diagrams
├── roles/                  # Automation logic split by function
│   ├── haproxy             # Installs and configures HAProxy Load Balancer
│   ├── httpd               # Installs Apache Web Server and deploys custom HTML
│   ├── keepalived          # Configures VRRP for Virtual IP (High Availability)
│   ├── squid_client        # Configures nodes to use the Controller as an HTTP Proxy
│   └── squid_controller    # Installs and secures Squid Proxy on localhost
└── test_tasks/             # Prototype tasks (Legacy/Pre-Refactoring)
    ├── install_haproxy.yaml
    ├── install_httpd.yaml
    └── install_keepalived.yaml

```

## Features

* **High Availability:** Utilizes Keepalived (VRRP) to float a Virtual IP between nodes, ensuring zero downtime in case of node failure.
* **Load Balancing:** HAProxy is configured to distribute traffic across the backend web servers using the Round Robin algorithm.
* **Proxy Tunneling:** Worker nodes are configured with global environment variables (`http_proxy`, `https_proxy`) to route traffic through the Controller node, enabling secure package management without direct internet exposure.
* **Dynamic Configuration:** Jinja2 templates are used for all configuration files to ensure portability across different environments (IPs and Ports are variable-driven).
* **System Hardening:** Automated handling of SELinux booleans and Firewalld rules for custom ports.

## Role Descriptions

### 1. squid_controller

Runs on the Controller (localhost). Installs Squid Proxy and configures Access Control Lists (ACLs) to allow connections strictly from the internal network subnet.

### 2. squid_client

Runs on worker nodes. Modifies `/etc/environment` to set proxy variables, allowing system tools (like `yum` and `curl`) to reach the internet via the Controller.

### 3. httpd

Installs the Apache Web Server, configures custom listening ports, and deploys a dynamic `index.html` page that identifies the specific host serving the request for testing purposes.

### 4. haproxy

Configures the frontend (VIP listener) and backend (Round Robin pool) to distribute incoming traffic.

### 5. keepalived

Sets up the VRRP instance, defining the MASTER/BACKUP state, priority levels, and the Virtual IP address.

## Usage

### 1. Clone the repository

```bash
git clone https://github.com/AdhamAyad/ansible-ha-web-cluster.git
cd ansible-ha-web-cluster

```

### 2. Update Inventory

Modify the `inventory` file to match your target node IP addresses.

### 3. Configure Variables

Review role `defaults` or `playbook001.yaml` vars to adjust:

* `virtual_ip`
* `proxy_port`
* `http_port`

### 4. Run the Playbook

```bash
ansible-playbook playbook001.yaml

```

## Development Notes

The directory `test_tasks/` contains the initial monolithic task files created during the prototyping phase. These were later refactored into the modular roles structure present in the root directory to follow Ansible best practices.

```

```
