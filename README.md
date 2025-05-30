# Cloud Instance Initialization

This project demonstrates comprehensive cloud instance initialization using Vagrant and cloud-init for automated Ubuntu 22.04 system provisioning. The setup automates Docker and Docker Compose installation, user management, network configuration, and deploys a complete Graylog logging stack with MongoDB and OpenSearch, all with a single `vagrant up` command.

## Table of Contents

- [Overview](#overview)
- [Prerequisites](#prerequisites)
- [Project Structure](#project-structure)
- [Quick Start](#quick-start)
- [Features](#features)
- [Architecture](#architecture)
- [Cloud-Init Configuration](#cloud-init-configuration)
- [Graylog Stack Deployment](#graylog-stack-deployment)
- [Testing](#testing)
- [Troubleshooting](#troubleshooting)
- [Daily Operations](#daily-operations)
- [Technical Details](#technical-details)

## Overview

This project provides an autonomous cloud-init configuration that provisions Ubuntu 22.04 virtual machines with the following capabilities:
- **Cloud-Init Automation**: Complete system initialization without manual intervention
- **Docker Engine & Compose**: Official Docker installation with Compose v2 plugin
- **User Management**: Automated creation of users (Mallory, Eve) with Docker group membership
- **Network Optimization**: MTU configuration (1442) for Docker daemon
- **Graylog Stack**: Complete logging infrastructure with MongoDB, OpenSearch, and Graylog
- **Web Interface**: Accessible Graylog UI on port 9000
- **Zero Interaction**: Fully autonomous execution after `vagrant up`

## Prerequisites

Before running the project, ensure you have:

- **Vagrant** (2.3.4 or later) for VM management
- **VirtualBox** (6.1 or later) as the virtualization provider
- **Host System**: Linux, macOS, or Windows with WSL2
- **Network Access**: For downloading packages and Docker images
- **Sufficient Resources**: At least 4GB RAM and 15GB disk space

### System Requirements

| Component | Minimum | Recommended |
|-----------|---------|-------------|
| **RAM** | 4GB | 8GB |
| **CPU** | 2 cores | 4 cores |
| **Disk Space** | 10GB | 20GB |
| **Network** | Broadband | Broadband |

## Project Structure

```
exercise-03-cloud-init/
├── README.md --------------------------------> # This file
├── Vagrantfile ------------------------------> # VM configuration with cloud-init
├── config.cfg -------------------------------> # Cloud-init configuration file
├── .vagrant/ --------------------------------> # Vagrant runtime data (auto-generated)
│   └── machines/ ----------------------------> # VM-specific configurations
│       └── default/ -------------------------> # Default VM state
├── report.pdf -------------------------------> # Project documentation
└── logs/ ------------------------------------> # Runtime logs (generated)
    ├── cloud-init.log -----------------------> # Cloud-init execution log
    ├── docker.log ---------------------------> # Docker installation log
    └── graylog.log --------------------------> # Graylog stack logs

# Generated inside VM at /home/vagrant/:
├── docker-compose.yml -----------------------> # Graylog stack definition (cloud-init generated)
```

## Quick Start

### Automated Setup

1. **Clone and navigate to the project:**
   ```bash
   git clone <repository-url>
   cd exercise-03-cloud-init
   ```

2. **Start the virtual machine:**
   ```bash
   vagrant up
   ```

3. **Access Graylog web interface:**
   ```bash
   # Open browser and navigate to:
   http://localhost:9000
   
   # Default credentials:
   # Username: admin
   # Password: admin
   ```

4. **Verify installation:**
   ```bash
   # SSH into the VM
   vagrant ssh
   
   # Check Docker installation
   docker --version
   docker compose version
   
   # Verify users and groups
   id mallory
   id eve
   getent group docker
   
   # Check running containers
   docker ps
   ```

### Manual Step-by-Step

1. **Initialize Vagrant environment:**
   ```bash
   # Create new directory
   mkdir my-cloud-init-project
   cd my-cloud-init-project
   
   # Initialize with Ubuntu 22.04
   vagrant init ubuntu/jammy64 --box-version 20241002.0.0
   ```

2. **Configure cloud-init provisioning:**
   ```bash
   # Copy the provided Vagrantfile and config.cfg
   cp /path/to/Vagrantfile .
   cp /path/to/config.cfg .
   ```

3. **Launch the VM:**
   ```bash
   vagrant up
   ```

## Features

### Core Functionality

| Feature | Description | Implementation |
|---------|-------------|----------------|
| **Cloud-Init Provisioning** | Automated system initialization | YAML configuration file |
| **Docker Installation** | Official Docker Engine + Compose v2 | APT repository setup |
| **User Management** | Create users with Docker group access | Cloud-init users directive |
| **Network Tuning** | MTU optimization for containers | Docker daemon configuration |
| **Package Management** | System updates and utility installation | APT package management |
| **Container Orchestration** | Multi-service deployment | Docker Compose |

### Advanced Features

- **Idempotent Operations**: Safe to run multiple times without side effects
- **Resource Optimization**: VM configured with 4GB RAM and 4 CPU cores
- **Port Forwarding**: Automatic host port 9000 → guest port 9000 mapping
- **Health Monitoring**: Container health checks and restart policies
- **Logging Integration**: Centralized log collection with Graylog
- **Security Configuration**: Non-root user setup with sudo privileges

## Architecture

### Architecture Overview

The system architecture follows the cloud-init provisioning flow as designed in the project documentation. The complete workflow demonstrates how Vagrant initializes the Ubuntu 22.04 VM, injects the cloud-init configuration file, and orchestrates the entire deployment process from system updates through Graylog stack deployment.

<img width="900" alt="Exercise 03 - Cloud instance initialization (6)" src="https://github.com/user-attachments/assets/3e627587-345f-4629-9450-aeb75d8ecf2d" />

*Figure 1: The diagram displaying the Cloud-init Provisioning flow from Vagrant initialization to full Graylog stack deployment.*

### Key Architecture Components

1. **Initialization Layer**: Vagrant + VirtualBox VM management
2. **Provisioning Layer**: Cloud-init configuration and execution
3. **Infrastructure Layer**: Docker Engine + Compose plugin
4. **Application Layer**: MongoDB, OpenSearch, and Graylog containers
5. **Access Layer**: Web UI accessible on port 9000

### Service Dependencies

- **MongoDB**: Provides document storage for Graylog configuration and metadata
- **OpenSearch**: Handles log indexing and search functionality
- **Graylog**: Central logging platform that coordinates between MongoDB and OpenSearch
- **Docker Network**: Bridge network enabling inter-container communication

## Cloud-Init Configuration

### Configuration Overview

The `config.cfg` file handles all system provisioning through cloud-init directives:

#### Boot Commands
```yaml
bootcmd:
  - install -m0755 -d /etc/apt/keyrings
  - curl -fsSL https://download.docker.com/linux/ubuntu/gpg | gpg --dearmor -o /etc/apt/keyrings/docker.gpg
  - echo "deb [arch=amd64 signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu jammy stable" > /etc/apt/sources.list.d/docker.list
  - sysctl -w vm.max_map_count=262144
```

#### Package Management
```yaml
package_update: true
package_upgrade: true

packages:
  - ca-certificates
  - curl
  - gnupg
  - docker-ce
  - docker-ce-cli
  - containerd.io
  - docker-compose-plugin
  - neofetch
  - htop
```

#### User Configuration
```yaml
users:
  - name: mallory
    groups: [docker]
    shell: /bin/bash
    sudo: ['ALL=(ALL) NOPASSWD:ALL']
  - name: eve
    groups: [docker]
    shell: /bin/bash
    sudo: ['ALL=(ALL) NOPASSWD:ALL']
```

#### File Creation
```yaml
write_files:
  - path: /etc/docker/daemon.json
    permissions: '0644'
    content: |
      {
        "mtu": 1442
      }
  
  - path: /home/vagrant/docker-compose.yml
    owner: vagrant:vagrant
    permissions: '0644'
    content: |
      version: '3'
      services:
        # Graylog stack definition
```

#### Runtime Commands
```yaml
runcmd:
  - systemctl restart docker
  - chown vagrant:vagrant /home/vagrant/docker-compose.yml
  - docker compose -f /home/vagrant/docker-compose.yml up -d
  - usermod -aG docker vagrant
  - usermod -aG docker mallory
  - usermod -aG docker eve
  - systemctl restart cloud-final.service
```

## Graylog Stack Deployment

### Service Configuration

#### MongoDB
```yaml
mongo:
  image: mongo:5.0
  networks:
    - graylog
  restart: always
```

#### OpenSearch
```yaml
opensearch:
  image: opensearchproject/opensearch:2
  environment:
    - "OPENSEARCH_JAVA_OPTS=-Xms1g -Xmx1g"
    - "bootstrap.memory_lock=true"
    - "discovery.type=single-node"
    - "action.auto_create_index=false"
    - "plugins.security.ssl.http.enabled=false"
    - "plugins.security.disabled=true"
  networks:
    - graylog
  restart: always
```

#### Graylog
```yaml
graylog:
  image: graylog/graylog:6.2
  environment:
    - GRAYLOG_PASSWORD_SECRET=somepasswordpepper
    - GRAYLOG_ROOT_PASSWORD_SHA2=8c6976e5b5410415bde908bd4dee15dfb167a9c873fc4bb8a81f6f2ab448a918
    - GRAYLOG_HTTP_EXTERNAL_URI=http://127.0.0.1:9000/
    - GRAYLOG_ELASTICSEARCH_HOSTS=http://opensearch:9200
  networks:
    - graylog
  restart: always
  ports:
    - 9000:9000
    - 1514:1514
    - 1514:1514/udp
    - 12201:12201
    - 12201:12201/udp
  depends_on:
    - mongo
    - opensearch
```

### Network Configuration

```yaml
networks:
  graylog:
    driver: bridge
```

## Testing

### Automated Testing

Create a test script to verify all components:

```bash
#!/bin/bash
# test_deployment.sh

echo "=== Testing Cloud-Init Deployment ==="

# Test 1: Check VM status
echo "1. Checking VM status..."
vagrant status

# Test 2: Verify system packages
echo "2. Testing installed packages..."
vagrant ssh -c "neofetch --version"
vagrant ssh -c "htop --version"

# Test 3: Check Docker installation
echo "3. Verifying Docker installation..."
vagrant ssh -c "docker --version"
vagrant ssh -c "docker compose version"

# Test 4: Verify user creation and group membership
echo "4. Testing user management..."
vagrant ssh -c "id mallory"
vagrant ssh -c "id eve"
vagrant ssh -c "getent group docker"

# Test 5: Check Docker MTU configuration
echo "5. Verifying Docker MTU configuration..."
vagrant ssh -c "cat /etc/docker/daemon.json"

# Test 6: Verify container status
echo "6. Checking Graylog stack..."
vagrant ssh -c "docker ps"

# Test 7: Test Graylog web interface
echo "7. Testing Graylog web interface..."
curl -f http://localhost:9000/ || echo "Graylog UI not accessible"

echo "=== Testing Complete ==="
```

### Manual Testing

#### System Verification
```bash
# SSH into the VM
vagrant ssh

# Check system information
neofetch
htop

# Verify cloud-init completed successfully
sudo cat /var/log/cloud-init.log | grep -i error
sudo cloud-init status
```

#### Docker Testing
```bash
# Check Docker service status
sudo systemctl status docker

# Verify Docker MTU configuration
cat /etc/docker/daemon.json

# Test Docker functionality
docker run hello-world

# Check Compose plugin
docker compose version
```

#### User Management Testing
```bash
# Switch to created users
sudo su - mallory
docker ps  # Should work without sudo

sudo su - eve
docker images  # Should work without sudo

# Check group membership
getent group docker
```

#### Graylog Stack Testing
```bash
# Check all containers are running
docker ps

# Check container logs
docker logs graylog
docker logs opensearch
docker logs mongo

# Test container networking
docker exec graylog ping opensearch
docker exec graylog ping mongo
```

### Test Results Validation

| Test Case | Expected Result | Status |
|-----------|----------------|--------|
| VM Boot | Cloud-init completes successfully | ✅ |
| Package Installation | neofetch, htop, Docker installed | ✅ |
| User Creation | mallory, eve users exist | ✅ |
| Docker Group | Users in docker group | ✅ |
| Docker MTU | MTU set to 1442 | ✅ |
| Container Status | 3 containers running | ✅ |
| Graylog UI | Web interface accessible | ✅ |
| Service Communication | Containers can communicate | ✅ |

## Troubleshooting

### Common Issues

| Issue | Symptoms | Solution |
|-------|----------|----------|
| **VM fails to start** | Vagrant up errors | Check VirtualBox installation and VM resources |
| **Cloud-init fails** | Boot process hangs | Check `config.cfg` syntax and cloud-init logs |
| **Docker installation fails** | Package not found errors | Verify network connectivity and APT sources |
| **Graylog containers not starting** | Docker compose errors | Check container logs and dependencies |
| **Web UI inaccessible** | Connection refused on port 9000 | Verify port forwarding and container status |
| **Users cannot use Docker** | Permission denied errors | Check Docker group membership |

### Diagnostic Commands

#### Cloud-Init Diagnostics
```bash
# Check cloud-init status
sudo cloud-init status

# View cloud-init logs
sudo cat /var/log/cloud-init.log
sudo cat /var/log/cloud-init-output.log

# Validate cloud-init configuration
sudo cloud-init schema --config-file config.cfg

# Debug cloud-init modules
sudo cloud-init collect-logs
```

#### Docker Diagnostics
```bash
# Check Docker service
sudo systemctl status docker
sudo journalctl -u docker.service

# Verify Docker daemon configuration
sudo cat /etc/docker/daemon.json
docker info | grep -i mtu

# Test Docker networking
docker network ls
docker network inspect bridge
```

#### Container Diagnostics
```bash
# Check container status and health
docker ps -a
docker stats

# View container logs
docker logs --tail 50 graylog
docker logs --tail 50 opensearch
docker logs --tail 50 mongo

# Inspect container configuration
docker inspect graylog
docker inspect opensearch
docker inspect mongo

# Test inter-container connectivity
docker exec graylog curl http://opensearch:9200
docker exec graylog ping mongo
```

### Error Resolution

#### Cloud-Init Configuration Error
```bash
# Problem: Syntax error in config.cfg
# Solution: Validate YAML syntax
python3 -c "import yaml; yaml.safe_load(open('config.cfg'))"

# Problem: Missing permissions
# Solution: Check file ownership and permissions
ls -la config.cfg
```

#### Docker Service Issues
```bash
# Problem: Docker service won't start
# Solution: Check system compatibility and logs
sudo systemctl restart docker
sudo journalctl -u docker.service --no-pager

# Problem: MTU configuration not applied
# Solution: Restart Docker after configuration change
sudo systemctl restart docker
docker info | grep -i mtu
```

#### Graylog Stack Issues
```bash
# Problem: OpenSearch connection refused
# Solution: Check container networking and environment variables
docker exec graylog env | grep ELASTICSEARCH
docker network inspect graylog_default

# Problem: Containers restart repeatedly
# Solution: Check resource limits and container logs
docker stats
docker logs --tail 100 graylog
```

## Daily Operations

### Development Workflow

1. **Start development environment:**
   ```bash
   # Start the VM
   vagrant up
   
   # Check VM status
   vagrant status
   
   # Monitor cloud-init progress
   vagrant ssh -c "sudo tail -f /var/log/cloud-init-output.log"
   ```

2. **Modify configuration:**
   ```bash
   # Edit cloud-init configuration
   nano config.cfg
   
   # Validate configuration
   python3 -c "import yaml; yaml.safe_load(open('config.cfg'))"
   
   # Reload VM with new configuration
   vagrant reload
   ```

3. **Test changes:**
   ```bash
   # SSH into VM
   vagrant ssh
   
   # Check cloud-init status
   sudo cloud-init status
   
   # Verify services
   docker ps
   curl -f http://localhost:9000/
   ```

### Production Usage

1. **Deploy to cloud provider:**
   ```bash
   # For AWS EC2
   aws ec2 run-instances \
     --image-id ami-xxxxxxxx \
     --user-data file://config.cfg \
     --instance-type t3.medium
   
   # For Azure
   az vm create \
     --custom-data config.cfg \
     --size Standard_B2s
   
   # For Google Cloud
   gcloud compute instances create graylog-instance \
     --metadata-from-file user-data=config.cfg \
     --machine-type e2-medium
   ```

2. **Monitor deployment:**
   ```bash
   # Check cloud-init status on target system
   ssh user@instance "sudo cloud-init status"
   
   # Verify Graylog accessibility
   curl -f http://instance-ip:9000/
   ```

### Maintenance Tasks

#### VM Management
```bash
# Start VM
vagrant up

# Stop VM
vagrant halt

# Restart VM
vagrant reload

# Destroy and recreate
vagrant destroy -f
vagrant up

# SSH into VM
vagrant ssh

# Check VM status
vagrant status

# Update Vagrant box
vagrant box update
```

#### Container Management
```bash
# View running containers
docker ps

# Stop all containers
docker compose -f /home/vagrant/docker-compose.yml down

# Start containers
docker compose -f /home/vagrant/docker-compose.yml up -d

# Update container images
docker compose -f /home/vagrant/docker-compose.yml pull
docker compose -f /home/vagrant/docker-compose.yml up -d

# View container logs
docker logs -f graylog
```

#### Log Management
```bash
# View cloud-init logs
sudo tail -f /var/log/cloud-init.log

# Rotate Docker logs
docker system prune -f

# Archive system logs
sudo tar -czf logs-$(date +%Y%m%d).tar.gz /var/log/

# Clean up old logs
sudo find /var/log -name "*.log" -mtime +30 -delete
```

## Technical Details

### Cloud-Init Best Practices

- **Idempotency**: All operations are safe to run multiple times
- **Error Handling**: Proper error checking and graceful failure handling
- **Validation**: YAML syntax validation before deployment
- **Logging**: Comprehensive logging for debugging and monitoring
- **Security**: Non-root user creation with minimal privileges

### Performance Optimization

- **Resource Allocation**: VM configured with adequate RAM and CPU
- **Container Optimization**: Optimized container images and resource limits
- **Network Tuning**: MTU configuration for optimal network performance
- **Storage Efficiency**: Efficient container storage and log rotation

### Security Considerations

- **User Privileges**: Limited sudo access for created users
- **Container Security**: Non-root container execution where possible
- **Network Security**: Isolated container network with bridge driver
- **Secret Management**: Secure handling of passwords and keys
- **Access Control**: Docker group membership for container access

### Compatibility Matrix

| Component | Version | Compatibility | Notes |
|-----------|---------|---------------|-------|
| **Ubuntu** | 22.04 LTS | Full | Long-term support |
| **Cloud-Init** | 20.4+ | Full | Native Ubuntu support |
| **Docker** | 24.0+ | Full | Official repository |
| **Docker Compose** | 2.0+ | Full | Plugin version |
| **MongoDB** | 5.0 | Full | Graylog compatible |
| **OpenSearch** | 2.x | Full | Elasticsearch alternative |
| **Graylog** | 6.2 | Full | Latest stable release |

### Extension Possibilities

- **Multi-VM Deployment**: Scale to multiple VMs with load balancing
- **SSL/TLS Configuration**: Add HTTPS support for web interfaces
- **Backup Integration**: Automated backup configuration for data persistence
- **Monitoring Stack**: Add Prometheus and Grafana for system monitoring
- **Log Forwarding**: Configure log forwarding from external systems
- **Authentication Integration**: LDAP/Active Directory integration for Graylog

---

**Project**: Exercise 03 - Cloud Instance Initialization  
**Course**: IKT114 - IT Orchestration  
**Institution**: University of Agder  
**Authors**: Yusef Said & Eirik André Lindseth

## Version History

- **v1.0**: Initial cloud-init configuration with Docker and Graylog stack
- **v1.1**: Added user management and network optimization
- **v1.2**: Enhanced error handling and documentation
- **v1.3**: Improved container orchestration and monitoring
