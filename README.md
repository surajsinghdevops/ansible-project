# 🚀 Ansible CI/CD Infrastructure Automation

Ansible-based infrastructure and Docker application deployment project designed to automate the configuration of an AWS EC2 application server.

The project uses **Ansible Roles** to install and configure Docker and Nginx, and provides a reusable application deployment role for running Docker containers from Docker Hub.

---

## 📌 Project Overview

This project demonstrates how Ansible can be used to automate application server configuration and Docker-based application deployment.

Instead of manually connecting to an EC2 server and installing/configuring services, Ansible performs the configuration automatically.

### Current Architecture

```text
                    Ansible Controller
                         EC2
                          |
                          | SSH
                          |
                          v
                 +------------------+
                 |   Application EC2 |
                 |                  |
                 |     Docker       |
                 |        |         |
                 |        v         |
                 |   Application    |
                 |                  |
                 |      Nginx        |
                 +------------------+
                          |
                          v
                     End User
```

---

# 🛠️ Technologies Used

* AWS EC2
* Amazon Linux 2023
* Ansible
* Ansible Roles
* Docker
* Nginx
* Docker Hub
* SSH
* Git / GitHub

---

# 📂 Project Structure

```text
ansible-cicd/
│
├── ansible.cfg
│
├── inventory/
│   └── hosts
│
├── playbooks/
│   ├── setup.yml
│   └── deploy.yml
│
├── roles/
│   │
│   ├── docker/
│   │   ├── defaults/
│   │   ├── handlers/
│   │   ├── tasks/
│   │   │   └── main.yml
│   │   ├── templates/
│   │   └── vars/
│   │
│   ├── nginx/
│   │   ├── defaults/
│   │   ├── handlers/
│   │   ├── tasks/
│   │   │   └── main.yml
│   │   ├── templates/
│   │   └── vars/
│   │
│   └── application/
│       ├── defaults/
│       │   └── main.yml
│       ├── handlers/
│       ├── tasks/
│       │   └── main.yml
│       ├── templates/
│       └── vars/
│
├── Dockerfile
├── app/
│   └── index.html
│
└── README.md
```

---

# ⚙️ Ansible Configuration

The project uses an `ansible.cfg` file to define the inventory and role location.

```ini
[defaults]
inventory = inventory/hosts
roles_path = ./roles
host_key_checking = False
```

This allows Ansible to automatically locate the project's roles.

---

# 🖥️ Inventory

The application EC2 server is defined in the Ansible inventory.

Example:

```ini
[app_servers]
app1 ansible_host=<APP_PRIVATE_IP> ansible_user=ec2-user
```

Replace:

```text
<APP_PRIVATE_IP>
```

with the private IP address of the application server.

---

# 🐳 Docker Role

The Docker role is responsible for installing and starting Docker on the application server.

Example tasks:

```yaml
- name: Install Docker
  ansible.builtin.dnf:
    name: docker
    state: present

- name: Start Docker service
  ansible.builtin.service:
    name: docker
    state: started
    enabled: true
```

The role ensures that Docker is installed and running.

---

# 🌐 Nginx Role

The Nginx role installs and starts Nginx on the application server.

Example:

```yaml
- name: Install Nginx
  ansible.builtin.dnf:
    name: nginx
    state: present

- name: Start Nginx
  ansible.builtin.service:
    name: nginx
    state: started
    enabled: true
```

Nginx will eventually act as the reverse proxy in front of the Docker application.

---

# 📦 Application Role

The application role provides a reusable way to deploy Docker images.

Default variables are defined in:

```text
roles/application/defaults/main.yml
```

Example:

```yaml
docker_image: "nginx"
docker_tag: "latest"
container_name: "myapp"
container_port: 80
host_port: 8080
```

Because these are default variables, the deployment playbook can be executed without passing extra variables.

```bash
ansible-playbook playbooks/deploy.yml
```

Ansible will use the values defined in `defaults/main.yml`.

---

# 🚀 Application Deployment

The application deployment playbook uses the application role.

Example:

```yaml
---
- name: Deploy application
  hosts: app_servers
  become: true

  roles:
    - application
```

Run:

```bash
ansible-playbook playbooks/deploy.yml
```

The application role will:

```text
Docker Image
     ↓
Pull Image
     ↓
Stop Existing Container
     ↓
Remove Existing Container
     ↓
Start New Container
     ↓
Application Running
```

---

# 🔧 Server Setup

The setup playbook configures the application server.

Example:

```yaml
---
- name: Configure application servers
  hosts: app_servers
  become: true

  roles:
    - docker
    - nginx
```

Run:

```bash
ansible-playbook playbooks/setup.yml
```

This configures:

```text
Application EC2
       |
       ├── Docker
       |
       └── Nginx
```

---

# 🧪 Verify Ansible Connectivity

Before running the playbooks, test connectivity:

```bash
ansible app_servers -m ping
```

Expected output:

```text
app1 | SUCCESS => {
    "changed": false,
    "ping": "pong"
}
```

---

# 🐳 Verify Docker

After running the setup playbook:

```bash
ansible app_servers -m shell -a "docker --version"
```

Check the Docker service:

```bash
ansible app_servers -m shell -a "systemctl is-active docker"
```

Expected:

```text
active
```

---

# 🌐 Verify Nginx

Check the Nginx service:

```bash
ansible app_servers -m shell -a "systemctl is-active nginx"
```

Expected:

```text
active
```

You can also test locally on the application server:

```bash
curl localhost
```

---

# 🐳 Docker Application

The repository can contain a simple Docker application.

Example:

```dockerfile
FROM nginx:alpine

COPY app/index.html /usr/share/nginx/html/index.html

EXPOSE 80
```

Build the image:

```bash
docker build -t <dockerhub-username>/myapp:1.0 .
```

Login to Docker Hub:

```bash
docker login
```

Push the image:

```bash
docker push <dockerhub-username>/myapp:1.0
```

The application role can then deploy the image on the application EC2.

---

# 🔄 Custom Docker Image Deployment

The application role uses defaults, but the variables can be overridden when required.

For example:

```bash
ansible-playbook playbooks/deploy.yml \
-e "docker_image=<dockerhub-username>/myapp docker_tag=1.0"
```

However, passing extra variables is optional.

Without `-e`:

```bash
ansible-playbook playbooks/deploy.yml
```

the values from:

```text
roles/application/defaults/main.yml
```

are used.

---

# 🔐 Security Considerations

For a real production environment:

* Do not commit SSH private keys to Git.
* Do not commit Docker Hub passwords or tokens.
* Use Jenkins Credentials or Ansible Vault for secrets.
* Restrict SSH access using AWS Security Groups.
* Prefer private subnets for application servers.
* Use IAM roles instead of hardcoded AWS credentials.
* Pin Docker image versions instead of always using `latest`.

---

# 📈 Future Improvements

This project is designed to evolve into a complete CI/CD platform.

Planned improvements include:

```text
GitHub
   ↓
Jenkins
   ↓
Build & Test
   ↓
Docker Build
   ↓
Docker Hub / Amazon ECR
   ↓
Ansible
   ↓
Application EC2
   ↓
Health Check
   ↓
Deployment
```

Future features:

* Jenkins CI/CD pipeline
* GitHub webhook integration
* Automated Docker image builds
* Docker Hub integration
* Amazon ECR integration
* Versioned Docker images
* Jenkins credentials management
* Ansible Vault
* Application health checks
* Automatic rollback
* Multiple application servers
* Load balancing
* Monitoring with Prometheus and Grafana

---

# 🎯 Learning Objectives

This project demonstrates practical knowledge of:

### Ansible

* Inventory
* Playbooks
* Roles
* Tasks
* Variables
* Default variables
* Idempotency
* SSH-based automation
* Modular infrastructure automation

### Docker

* Dockerfiles
* Images
* Containers
* Docker Hub
* Container deployment

### AWS

* EC2
* Security Groups
* Private networking
* Server automation

### DevOps

* Infrastructure automation
* Configuration management
* Application deployment
* CI/CD concepts
* Containerized application delivery

---

# 👨‍💻 Author

**Suraj Singh**

DevOps / Cloud Engineering Learner

GitHub:

```text
https://github.com/surajsinghdevops
```

---

# ⭐ Project Goal

The ultimate goal of this project is to create a reusable DevOps deployment platform where a developer can push application code to GitHub and the CI/CD system automatically builds, packages, and deploys the application using Docker, Jenkins, and Ansible.

```text
Code
 ↓
GitHub
 ↓
Jenkins
 ↓
Docker
 ↓
Ansible
 ↓
AWS EC2
 ↓
Application
```

---
