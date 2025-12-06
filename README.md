# terraform-ansible-jenkins-deploy

✅ Ansible Jinja2 template for index.html
✅ Updated playbook using template module
✅ Terraform example with DigitalOcean/GCP switch + local_file inventory generation
✅ Docker-based Jenkins setup

---

# ✅ **GitHub Repository Structure**

```
ansible-terraform-webapp/
├── ansible/
│   ├── inventory
│   ├── playbook.yml
│   ├── templates/
│   │   └── index.html.j2
│   └── roles/
│       └── web/
│           ├── tasks/main.yml
│           ├── templates/
│           │   └── nginx.conf.j2
│           └── files/
├── terraform/
│   ├── main.tf
│   ├── variables.tf
│   ├── outputs.tf
│   └── generate_inventory.tf
├── jenkins/
│   ├── Dockerfile
│   ├── docker-compose.yml
│   └── Jenkinsfile
└── README.md
```

---

# ✅ **1. Jinja2 Template (index.html.j2)**

`ansible/templates/index.html.j2`

```html
<!DOCTYPE html>
<html>
<head>
    <title>Ansible Web App</title>
</head>
<body>
    <h1>Welcome to the Ansible Deployed App</h1>
    <p>Backend IP: {{ backend_ip }}</p>
</body>
</html>
```

---

# ✅ **2. Updated Ansible Playbook Using Jinja Template**

`ansible/playbook.yml`

```yaml
---
- name: Deploy Web App
  hosts: webservers
  become: yes

  vars:
    backend_ip: "{{ lookup('env','BACKEND_IP') | default('127.0.0.1', true) }}"

  tasks:
    - name: Install NGINX
      ansible.builtin.package:
        name: nginx
        state: present

    - name: Deploy index.html from template
      ansible.builtin.template:
        src: templates/index.html.j2
        dest: /usr/share/nginx/html/index.html
        owner: root
        group: root
        mode: '0644'

    - name: Start and enable nginx
      ansible.builtin.service:
        name: nginx
        state: started
        enabled: true
```

---

# ✅ **3. Terraform Module (DigitalOcean Example)**

`terraform/main.tf`

```hcl
terraform {
  required_providers {
    digitalocean = {
      source = "digitalocean/digitalocean"
      version = "~> 2.0"
    }
  }
}

provider "digitalocean" {
  token = var.do_token
}

resource "digitalocean_droplet" "web" {
  name   = "ansible-web"
  region = "blr1"
  size   = "s-1vcpu-1gb"
  image  = "ubuntu-22-04-x64"

  ssh_keys = [var.ssh_key_id]
}

output "web_ip" {
  value = digitalocean_droplet.web.ipv4_address
}
```

---

# ✅ **4. Generate Ansible Inventory Automatically**

`terraform/generate_inventory.tf`

```hcl
resource "local_file" "ansible_inventory" {
  content = <<EOT
[webservers]
${digitalocean_droplet.web.ipv4_address} ansible_user=root ansible_ssh_private_key_file=~/.ssh/id_rsa
EOT

  filename = "../ansible/inventory"
}
```

---

# ✅ **5. Switch Provider to GCP (Optional)**

Replace the DO provider block with this:

```hcl
terraform {
  required_providers {
    google = {
      source = "hashicorp/google"
      version = "~> 4.0"
    }
  }
}

provider "google" {
  project = var.gcp_project
  region  = var.gcp_region
  zone    = var.gcp_zone
}

resource "google_compute_instance" "vm" {
  name         = "ansible-web"
  machine_type = "e2-micro"
  zone         = var.gcp_zone

  boot_disk {
    initialize_params {
      image = "ubuntu-os-cloud/ubuntu-2204-lts"
    }
  }

  network_interface {
    network       = "default"
    access_config {}
  }
}

output "web_ip" {
  value = google_compute_instance.vm.network_interface[0].access_config[0].nat_ip
}
```

---

# ✅ **6. Docker-based Jenkins Setup**

`jenkins/Dockerfile`

```dockerfile
FROM jenkins/jenkins:lts

USER root
RUN apt-get update && apt-get install -y docker.io
USER jenkins
```

---

`jenkins/docker-compose.yml`

```yaml
version: '3.8'

services:
  jenkins:
    build: .
    ports:
      - "8080:8080"
      - "50000:50000"
    volumes:
      - jenkins_home:/var/jenkins_home
      - /var/run/docker.sock:/var/run/docker.sock

volumes:
  jenkins_home:
```

---

`jenkins/Jenkinsfile`

```groovy
pipeline {
    agent any

    stages {
        stage('Build') {
            steps {
                sh 'echo Building App'
            }
        }

        stage('Docker Build') {
            steps {
                sh '''
                docker build -t myapp:latest .
                '''
            }
        }

        stage('Push to DockerHub') {
            steps {
                sh '''
                echo "$DOCKER_PASS" | docker login -u "$DOCKER_USER" --password-stdin
                docker tag myapp:latest $DOCKER_USER/myapp:latest
                docker push $DOCKER_USER/myapp:latest
                '''
            }
        }
    }
}
```

---

# ✅ **README.md**

`README.md`

````markdown
# Ansible + Terraform WebApp Deployment

This repository contains:

✔ Ansible playbook using Jinja2 templates  
✔ Terraform infrastructure for DigitalOcean or GCP  
✔ Auto-generated Ansible inventory  
✔ Docker-based Jenkins CI/CD pipeline  

---

## 1. Deploy Infrastructure (Terraform)

```bash
cd terraform
terraform init
terraform apply -auto-approve
````

This creates:

* A VM instance (DigitalOcean or GCP)
* A generated Ansible inventory file at `ansible/inventory`

---

## 2. Run Ansible Playbook

```bash
cd ../ansible
ansible-playbook playbook.yml
```

This deploys:

* NGINX
* Dynamic index.html with backend IP injected

---

## 3. Start Jenkins (Docker)

```bash
cd jenkins
docker-compose up -d
```

Access Jenkins at:

```
http://localhost:8080
```

---

## 4. CI/CD Pipeline

The Jenkinsfile builds and pushes Docker images to DockerHub.

---

## 5. Switching Cloud Providers

* To use **DigitalOcean**, keep `digitalocean` provider.
* To use **GCP**, replace with `google` provider and update variables.

---

## 6. Auto-Generated Inventory

Terraform writes the VM IP to:

```
ansible/inventory
```

So Ansible always deploys to the correct server.

---

## Author

Md Gayasuddin – DevOps Engineer

```

---
