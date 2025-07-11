# Azure VM Web Server Deployment with Terraform, Ansible, and Docker

[![Terraform](https://img.shields.io/badge/Terraform-%23623CE4.svg?logo=terraform&logoColor=white)](https://www.terraform.io)
[![Ansible](https://img.shields.io/badge/Ansible-%231A1918.svg?logo=ansible&logoColor=white)](https://www.ansible.com)
[![Docker](https://img.shields.io/badge/Docker-%232496ED.svg?logo=docker&logoColor=white)](https://www.docker.com)

This project uses Terraform, Ansible, and Docker to deploy a containerised Apache HTTPD web server with a comprehensive monitoring stack on Azure VMs. Terraform provisions two VMs and networking resources declaratively, Ansible automates Docker installation and container setup via SSH, and Docker runs both the web server and monitoring components (Prometheus, Node Exporter, cAdvisor) in a consistent, isolated environment. Grafana provides real-time visualisation of system and container metrics. The pipeline enables automated, repeatable, and scalable infrastructure deployment with built-in observability.

## Project Structure

```
azure-ansible-docker/
├── ansible/                    # Ansible configuration and playbooks
│   ├── inventory.ini.template # Template for Ansible inventory
│   ├── inventory.ini         # Your configured inventory file (gitignored)
│   ├── playbooks/            # Ansible playbooks
│   │   ├── main.yml         # Main playbook for Docker setup
│   │   ├── prometheus.yml   # Prometheus deployment
│   │   ├── node_exporter.yml # Node Exporter deployment
│   │   ├── cadvisor.yml     # cAdvisor deployment
│   │   └── test.yml         # Test playbook for verification
│   ├── files/               # Files to be deployed
│   │   └── index.html      # Custom web page
│   └── host_vars/          # Host-specific variables
│       ├── servervars.yml.template  # Template for server variables
│       └── servervars.yml          # Your configured variables (gitignored)
├── terraform/               # Terraform configuration
│   ├── main.tf             # Main Terraform configuration
│   │   ├── Main VM setup   # Web server and monitoring components
│   │   └── Grafana VM setup # Visualisation dashboard
│   ├── variables.tf        # Input variables for Terraform
│   ├── output.tf           # Output values from Terraform
└── README.md               # Project documentation
```

## What currently get deployed

1 Main VM + 1 Grafana VM → Containers → 1 Website + Monitoring Stack

### Infrastructure
- [x] **Main VM**: Ubuntu 22.04 LTS instance (172.167.66.20)
  - Running web server and monitoring components
  - Multiple containers: Apache, Prometheus, Node Exporter, cAdvisor
- [x] **Grafana VM**: Ubuntu 22.04 LTS instance (4.234.136.220)
  - Running Grafana for metrics visualisation
- [x] **Network Security Group**: Restricted ports (22/8080/9090/9100/8081/3000)
- [x] **Virtual Network**: Isolated subnet configuration

### Application Stack
- [x] **Docker**: Container runtime installed on both VMs
- [x] **Apache HTTPD**: Containerised web server
- [x] **Static Website**: Served on port 8080
- [x] **Persistent Storage**: Volume-mounted web content

### Security Features
- [x] **SSH Access**: Key-based auth only (port 22)
- [x] **Network Policies**: HTTP restricted to port 8080
- [x] **Container Isolation**: Process sandboxing
- [x] **Self-Healing**: Auto-restart policy

### Monitoring Stack

- [x] **Prometheus**: Scrapes metrics from Node Exporter and cAdvisor (port 9090)
- [x] **Node Exporter**: Collects CPU, memory, disk, and network metrics (port 9100)
- [x] **cAdvisor**: Collects container resource usage metrics (port 8081)
- [x] **Grafana**: Visualisations with preloaded dashboards (ID 1860, 14282) (port 3000)


## What scalability could look like

100 VMs → Multiple Containers per VM → Microservices (Web, DB Cache) → Load Balanced Website + Customised Monitoring Stack

### Scaling
- [ ] **Horizontal**: VM Scale Sets + Azure Load Balancer  
- [ ] **Vertical**: Upgrade to Dv3-series VMs with tuned memory limits  
- [ ] **Hybrid**: Auto-scaling based on CPU/memory metrics  

### Architecture 
- [ ] **Static Assets**: Azure Blob Storage + CDN
- [ ] **Reverse Proxy**: Nginx/Traefik for routing
- [ ] **Microservices**: Decoupled DB/cache layers

### Orchestration
- [ ] **Container Management**: Docker Swarm/AKS
- [ ] **Auto-Scaling**: Pod/node autoscaling
- [ ] **CI/CD Pipeline**: Automated rolling updates

## Troubleshooting

- **Check Docker Installation**
  ```bash
  ssh adminuser@your-vm-ip 'docker --version'
- **Verify Apache Container status**
   ```bash
   ssh adminuser@your-vm-ip 'docker ps -a'
- **Confirm that port 8080 is open in the Network Security Group (NSG):**

   Check Docker status
   ```bash
   az network nsg rule list -g your-resource-group --nsg-name your-nsg-name
   ```
   View container logs
   ```bash
   ssh adminuser@your-vm-ip 'docker logs apache-httpd'
   ```

## Roadmap (short-term)

- [ ] Custom Docker Image & Nginx Reverse Proxy
- [ ] Configure alerting for critical metrics
- [ ] Implement Azure Key Vault for secrets

## Contributing

1. Fork the repository
2. Create a feature branch
3. Make your changes
4. Submit a pull request

## 🔄 Internal vs External Ports in Docker

### **External Port (Host Port)**
- **What it is**: The port on your **host machine** (your VM)
- **Example**: `-p 8080:80` means port `8080` on your VM
- **Access**: `http://172.167.66.20:8080` (from outside)
- **Purpose**: How the outside world reaches your container

### **Internal Port (Container Port)**
- **What it is**: The port **inside the container** where the app runs
- **Example**: `-p 8080:80` means port `80` inside the container
- **Access**: `http://localhost:80` (from inside the container)
- **Purpose**: Where your application actually listens

## 🚨 Port Issues You Encountered in This Project

### **1. cAdvisor Port Mismatch**
```yaml
# Your cAdvisor container:
-p 8081:8080  # External:Internal

# Your Prometheus config (WRONG):
- targets: ['cadvisor:8080']  # Should be internal port 8080
```

**The Problem**: Prometheus was trying to reach cAdvisor on the wrong port within the Docker network.

### **2. Docker Network Communication**
```yaml
# Some containers had --network monitoring
# Others didn't, causing communication failures
```

**The Problem**: Containers couldn't talk to each other because they weren't on the same network.

### **3. Port Conflicts**
Your setup had these ports:
- **Apache**: `8080:80` (web server)
- **Prometheus**: `9090:9090` (monitoring)
- **Node Exporter**: `9100:9100` (metrics)
- **cAdvisor**: `8081:8080` (container metrics)
- **Grafana**: `3000:3000` (dashboard)

**The Problem**: If any two containers tried to use the same external port, Docker would fail to start them.

### **4. Prometheus Target Configuration**
The biggest issue was in your `prometheus.yml`:
```yaml
<code_block_to_apply_changes_from>
```

## 💡 Key Lessons from Your Project

1. **Always use internal ports** when containers communicate within Docker networks
2. **Use external ports** when accessing from outside (browser, SSH)
3. **Keep containers on the same network** for internal communication
4. **Double-check port mappings** - they're the most common Docker issue
5. **Test connectivity** between containers before assuming they work

This is why you had "a lot of issues with different ports" - Docker networking can be tricky when you mix internal and external port references!
