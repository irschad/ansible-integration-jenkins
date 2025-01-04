# Ansible Integration in Jenkins

## Project Overview
This project demonstrates the integration of Ansible with Jenkins to automate the configuration of AWS EC2 instances as part of a CI/CD pipeline. The primary objective is to leverage Jenkins pipelines to execute Ansible playbooks on a dedicated Ansible Control Node, which in turn configures multiple Managed Nodes.

## Technologies Used
- **Ansible**: For configuration management and automation.
- **Jenkins**: For continuous integration and continuous delivery (CI/CD).
- **AWS**: Cloud platform hosting EC2 instances.
- **Boto3**: Python library for AWS automation.
- **Docker**: For containerization of the application.
- **Java**: Backend application language.
- **Maven**: Java build automation tool.
- **Linux**: Operating system for servers.
- **Git**: Version control system.

---

## Project Steps
### Step 1: Create and Configure Jenkins Server
1. Launch a dedicated EC2 instance to host Jenkins.
2. Install Jenkins on the server and configure required plugins (e.g., SSH Agent, SSH Pipeline Steps).
3. Secure Jenkins with SSH key-based authentication and configure credentials.

### Step 2: Create and Configure Ansible Control Node
1. Launch a dedicated EC2 instance for the Ansible Control Node.
2. Install the following on the server:
   - Ansible
   - Python3
   - Boto3
3. Configure AWS credentials on the Control Node for dynamic inventory support.

### Step 3: Write Ansible Playbooks
1. Create an `ansible/` folder containing the following files:
   - **`ansible.cfg`**: Configuration for Ansible, including remote user and private key.
   - **`inventory_aws_ec2.yaml`**: Dynamic inventory plugin configuration for AWS.
   - **`ec2-playbook.yaml`**: Playbook to install Docker, Docker Compose, and other dependencies on EC2 Managed Nodes.

#### Example `ansible.cfg`
```ini
[defaults]
inventory = ./inventory_aws_ec2.yaml
remote_user = ec2-user
private_key_file = /path/to/private/key.pem
host_key_checking = False
```

#### Example `ec2-playbook.yaml`
```yaml
---
- name: Configure EC2 Instances
  hosts: all
  become: yes
  tasks:
    - name: Install Docker
      yum:
        name: docker
        state: present
    - name: Start Docker service
      service:
        name: docker
        state: started
        enabled: yes
    - name: Install Docker Compose
      get_url:
        url: https://github.com/docker/compose/releases/download/1.29.2/docker-compose-Linux-x86_64
        dest: /usr/local/bin/docker-compose
        mode: '0755'
    - name: Verify Docker Compose installation
      command: docker-compose --version
```

### Step 4: Jenkins Pipeline Configuration
1. Create a Jenkins pipeline with the following stages:
   - **Copy Files to Ansible Server**:
     - Use `scp` to copy playbooks and SSH keys to the Ansible Control Node.
   - **Execute Ansible Playbook**:
     - Connect to the Control Node and execute the playbook remotely.
2. Key file: `Jenkinsfile`

#### Example `Jenkinsfile`
```groovy
pipeline {
    agent any
    stages {
        stage('Copy Files') {
            steps {
                sh '''
                scp -i /path/to/private/key.pem ansible/* ec2-user@<Ansible-Control-Node-IP>:~/ansible/
                '''
            }
        }
        stage('Run Playbook') {
            steps {
                sh '''
                ssh -i /path/to/private/key.pem ec2-user@<Ansible-Control-Node-IP> "ansible-playbook ~/ansible/ec2-playbook.yaml"
                '''
            }
        }
    }
}
```

### Step 5: Automate CI/CD Pipeline
1. Use the Jenkins pipeline to:
   - Build the Java Maven application.
   - Containerize the application with Docker.
   - Deploy the application by configuring EC2 instances using Ansible.

---

## File Structure
```plaintext
├── Dockerfile
├── Jenkinsfile
├── pom.xml
├── ansible/
│   ├── ansible.cfg
│   ├── ec2-playbook.yaml
│   ├── inventory_aws_ec2.yaml
├── prepare-ansible-server.sh
├── script.groovy
```

### Key Files
- **`Dockerfile`**: Defines the base image and entry point for the Dockerized application.
- **`Jenkinsfile`**: Jenkins pipeline script to orchestrate build, deploy, and Ansible integration.
- **`pom.xml`**: Maven configuration for building the Java application.
- **`prepare-ansible-server.sh`**: Shell script to set up the Ansible Control Node.
- **Ansible Playbooks**:
  - `ansible.cfg`: Configures Ansible settings.
  - `ec2-playbook.yaml`: Automates the setup of Docker and Docker Compose on EC2 instances.
  - `inventory_aws_ec2.yaml`: Dynamic inventory configuration for AWS.

---

## Troubleshooting
- **Jenkins SSH Issues**:
  - Verify SSH key permissions and ensure the private key is in PEM format.
- **Ansible Playbook Errors**:
  - Use `-vvv` for verbose logs.
  - Check dynamic inventory configuration.
- **Pipeline Failures**:
  - Check Jenkins console logs for detailed error messages.

---


