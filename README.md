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

### Monitoring Stack

- [x] **Prometheus**: Scrapes metrics from Node Exporter and cAdvisor (port 9090)
- [x] **Node Exporter**: Collects CPU, memory, disk, and network metrics (port 9100)
- [x] **cAdvisor**: Collects container resource usage metrics (port 8081)
- [x] **Grafana**: Visualisations with preloaded dashboards (ID 1860, 14282) (port 3000)


## What scalability could look like

100 VMs → Container Orchestration → Microservices (Web, DB Cache) → Load Balanced Website + Customised Monitoring Stack

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
- [ ] **CI/CD Pipeline**: Contionous deployments

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

## Contributing

1. Fork the repository
2. Create a feature branch
3. Make your changes
4. Submit a pull request