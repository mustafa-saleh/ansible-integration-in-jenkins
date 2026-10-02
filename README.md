# 🚀 Ansible Integration in Jenkins

![Jenkins](https://img.shields.io/badge/Jenkins-D24939?style=for-the-badge&logo=jenkins&logoColor=white)
![Ansible](https://img.shields.io/badge/Ansible-EE0000?style=for-the-badge&logo=ansible&logoColor=white)
![DigitalOcean](https://img.shields.io/badge/DigitalOcean-0080FF?style=for-the-badge&logo=digitalocean&logoColor=white)
![AWS](https://img.shields.io/badge/AWS-232F3E?style=for-the-badge&logo=amazonaws&logoColor=white)
![Boto3](https://img.shields.io/badge/Boto3-FF9900?style=for-the-badge&logo=amazonaws&logoColor=white)
![Docker](https://img.shields.io/badge/Docker-2496ED?style=for-the-badge&logo=docker&logoColor=white)
![Java](https://img.shields.io/badge/Java-ED8B00?style=for-the-badge&logo=openjdk&logoColor=white)
![Maven](https://img.shields.io/badge/Maven-C71A36?style=for-the-badge&logo=apachemaven&logoColor=white)
![Linux](https://img.shields.io/badge/Linux-FCC624?style=for-the-badge&logo=linux&logoColor=black)
![Git](https://img.shields.io/badge/Git-F05032?style=for-the-badge&logo=git&logoColor=white)

**Jenkins** is an open-source automation server that enables developers to reliably build, test and deploy their software through **Continuous Integration and Continuous Delivery (CI/CD) pipelines**. It is written in Java, has an enormous plugin ecosystem, and can be extended to orchestrate virtually any step of a software delivery process — from compiling source code, to running automated tests, to calling out to external configuration-management tools like Ansible as part of a fully automated release pipeline.

**Ansible Automation** is an open-source, agentless IT automation engine that automates provisioning, configuration management, application deployment, and orchestration across large fleets of servers. It describes the *desired state* of a system using simple, human-readable YAML files called **playbooks**, and pushes those changes to managed nodes over standard **SSH** — without requiring any agent, daemon, or extra software installed on the target machines. Because every module is designed to be **idempotent**, running the same automation twice produces the same end state without unintended side effects, which makes Ansible a safe, repeatable and auditable way to manage infrastructure at scale.

## 📖 Overview

This project demonstrates how to wire **Ansible** into a **Jenkins CI/CD pipeline**, so that infrastructure configuration becomes just another automated stage of software delivery instead of a manual, out-of-band task. A dedicated **Ansible Control Node** droplet is provisioned on **DigitalOcean**, completely separate from the **Jenkins** server, and two **AWS EC2** instances are stood up as the **Ansible Managed Nodes** that will be configured.

Rather than running `ansible-playbook` by hand from a laptop, the **Jenkinsfile** drives the entire hand-off: it securely copies the Ansible playbook, configuration and SSH credentials from the Jenkins server onto the remote Control Node over SSH, bootstraps that Control Node with Ansible/Python3/Boto3, and finally triggers the playbook run remotely — which in turn uses Ansible's **dynamic AWS EC2 inventory plugin** to discover and configure the two Managed Node EC2 instances with **Docker** and **Docker Compose**. The result is a single `Build Now` click in Jenkins that takes infrastructure all the way from "unconfigured EC2 instances" to "Docker-ready servers," with every step logged and reproducible.

## Demo Project

Ansible Integration in Jenkins

## Technologies used

- Ansible
- Jenkins
- DigitalOcean
- AWS
- Boto3
- Docker
- Java
- Maven
- Linux
- Git

## Project Description

- Create and configure a dedicated server for Jenkins
- Create and configure a dedicated server for Ansible Control Node
- Write Ansible Playbook, which configures 2 EC2 Instances
- Add ssh key file credentials in Jenkins for Ansible Control Node server and Ansible Managed Node servers
- Configure Jenkins to execute the Ansible Playbook on remote Ansible Control Node server as part of the CI/CD pipeline
- So the Jenkinsfile configuration will do the following:
    - a. Connect to the remote Ansible Control Node server
    - b. Copy Ansible playbook and configuration files to the remote Ansible Control Node server
    - c. Copy the ssh keys for the Ansible Managed Node servers to the Ansible Control Node server
    - d. Install Ansible, Python3 and Boto3 on the Ansible Control Node server
    - e. With everything installed and copied to the remote Ansible Control Node server, execute the playbook remotely on that Control Node that will configure the 2 EC2 Managed Nodes

## 📁 Repository structure

```text
ansible-integration-in-jenkins/
├── NOTES.md                        # Personal study notes covering the full Ansible learning path
├── README.md                       # This file 📄
├── .gitignore                      # Excludes build output, *.pem private keys and local script.groovy
├── Jenkinsfile                     # Declarative pipeline - copies files to the Control Node & runs the playbook
├── prepare-ansible-server.sh       # Bootstrap script - installs ansible & python3-boto3 on the Control Node
├── pom.xml                          # Maven build descriptor for the sample Java/Spring Boot app being shipped
├── ansible-jenkins.pem              # Local copy of the EC2 Managed Node key-pair (git-ignored, *.pem)
├── ansible/                        # Everything shipped to the remote Ansible Control Node
│   ├── ansible.cfg                  # Points Ansible at the dynamic AWS EC2 inventory + SSH key
│   ├── inventory_aws_ec2.yaml       # aws_ec2 dynamic inventory plugin configuration
│   └── my-playbook.yaml            # Playbook - installs Docker & Docker Compose on both EC2 Managed Nodes
├── src/                             # Sample Java application source (built via Maven in the wider pipeline)
└── images/                         # Screenshots captured while running the demo
    ├── ansible-server-terminal.png
    └── jenkins-pipeline-success.png
```

## 🏗️ Architecture overview

```mermaid
flowchart TB
    Dev["💻 Developer<br/>git push / Build Now"] --> Jenkins

    subgraph Jenkins["🧰 Jenkins Server (DigitalOcean Droplet)"]
        direction TB
        S1["Stage: Declarative Checkout SCM"] --> S2["Stage: copy files to ansible server<br/>sshagent + scp"]
        S2 --> S3["Stage: execute ansible playbook<br/>sshScript + sshCommand"]
    end

    S2 -- "scp ansible/* + ssh-key.pem<br/>(SSH, credential: ansible-server-key)" --> CN

    subgraph CN["🎛️ Ansible Control Node (DigitalOcean Droplet)"]
        direction TB
        Prep["prepare-ansible-server.sh<br/>installs ansible + python3-boto3"] --> PB["ansible-playbook my-playbook.yaml"]
    end

    S3 -. "sshScript / sshCommand<br/>(SSH, credential: ansible-server-key)" .-> CN

    PB -- "aws_ec2 dynamic inventory<br/>(boto3 + AWS credentials)" --> AWS

    subgraph AWS["☁️ AWS EC2 — Ansible Managed Nodes"]
        direction LR
        EC1["EC2 Instance 1<br/>Docker + Docker Compose"]
        EC2n["EC2 Instance 2<br/>Docker + Docker Compose"]
    end

    PB -- "SSH (ssh-key.pem,<br/>credential: ec2-server-key)" --> EC1
    PB -- "SSH (ssh-key.pem,<br/>credential: ec2-server-key)" --> EC2n
```

- The **Jenkins server** and the **Ansible Control Node** are two intentionally separate **DigitalOcean** droplets — Jenkins only orchestrates the pipeline, it never runs Ansible itself
- Jenkins authenticates to the Control Node using a stored **"SSH Username with private key"** credential (`ansible-server-key`), wrapped in the **`sshagent`** pipeline step so the private key never touches disk in plaintext on the Jenkins workspace
- The **`ec2-server-key`** credential (the `ansible-jenkins.pem` key-pair) is copied onto the Control Node as `ssh-key.pem`, so that the Control Node — not Jenkins — is the one that ultimately SSHes into the two EC2 **Managed Nodes**
- On the Control Node, Ansible's **`amazon.aws.aws_ec2`** dynamic inventory plugin (backed by the **Boto3**/botocore Python SDK and AWS credentials) discovers the EC2 Managed Nodes automatically at playbook run-time, instead of relying on a hardcoded, static inventory file
- The playbook itself installs **Docker** and the **Docker Compose** CLI plugin on both Managed Nodes, making them ready to run containerized workloads
- A sample **Java/Maven (Spring Boot)** application (built via [pom.xml](pom.xml)) represents the kind of application that this Docker-ready infrastructure would ultimately host once the pipeline is extended with build/deploy stages

## 🧭 Implementation Guide

### 1. Prerequisites

Before running this automation, make sure you have the following in place:

- ✅ A **Jenkins server** up and running, with the required plugins installed. This demo reuses the Jenkins setup from [demo-module-8-build-automation-and-ci-cd-with-jenkins](https://github.com/mustafa-saleh/demo-module-8-build-automation-and-ci-cd-with-jenkins), plus two additional plugins:
  - **SSH Agent Plugin** — lets a pipeline stage use a stored SSH credential to authenticate outbound `scp`/`ssh` commands, see [SSH Agent Plugin docs](https://plugins.jenkins.io/ssh-agent/)
  - **SSH Pipeline Steps Plugin** — provides the `sshScript`/`sshCommand` pipeline steps used to run commands and scripts on a remote host over SSH, see [SSH Pipeline Steps Plugin docs](https://plugins.jenkins.io/ssh-steps/)
- ✅ A second **DigitalOcean droplet** to act as the dedicated **Ansible Control Node**, reachable over SSH from the Jenkins server
- ✅ An **AWS account** with programmatic (access key / secret key) credentials that have permission to describe and manage EC2 instances
- ✅ **2 AWS EC2 instances** created from the AWS Console using a new key-pair (`ansible-jenkins`) — these are the **Ansible Managed Nodes** that the playbook will configure
- ✅ Jenkins credentials configured ahead of time, under **Manage Jenkins → Credentials**, of type **"SSH Username with private key"**:
  - `ansible-server-key` — username `root`, private key of the Ansible Control Node droplet
  - `ec2-server-key` — username `ec2-user`, private key of the EC2 instances (`ansible-jenkins.pem`)

  > ⚠️ If a private key starts with `-----BEGIN OPENSSH PRIVATE KEY-----`, Jenkins' credential store expects the older **PEM** format. Convert it locally before pasting it in:
  >
  > ```bash
  > ssh-keygen -p -m PEM -f ~/.ssh/id_ed25519 -P "" -N ""
  > ```

### 2. Prepare the Ansible Control Node

SSH onto the Control Node droplet and install Ansible and the Python dependencies it needs to talk to the AWS API:

```bash
apt update
apt install ansible-core -y

# below will install python3-botocore as a dependency
apt install python3-boto3 -y
```

Ansible's `aws_ec2` inventory plugin authenticates to AWS using the standard AWS credentials file, so it needs to be created on the Control Node at `~/.aws/credentials`:

```ini
[default]
aws_access_key_id = <your-access-key-id>
aws_secret_access_key = <your-secret-access-key>
```

This exact bootstrap sequence is codified in [prepare-ansible-server.sh](prepare-ansible-server.sh), so the Jenkins pipeline can re-run it idempotently on every build instead of relying on the Control Node being pre-configured by hand:

```bash
#!/usr/bin/env bash

apt update
apt install ansible -y
apt install python3-boto3
```

### 3. Configure Ansible's dynamic AWS EC2 inventory

Instead of hardcoding the two EC2 Managed Nodes' IP addresses in a static inventory file, this project uses Ansible's **`amazon.aws.aws_ec2`** inventory plugin to discover them dynamically at runtime. [ansible/inventory_aws_ec2.yaml](ansible/inventory_aws_ec2.yaml) configures the plugin to scan the `eu-central-1` region and group discovered hosts by tag and instance type:

```yaml
---
plugin: aws_ec2
regions: 
  - eu-central-1
keyed_groups:
  - key: tags
    prefix: tag
  - key: instance_type
    prefix: instance_type
```

[ansible/ansible.cfg](ansible/ansible.cfg) wires this inventory file, the plugin, and the SSH connection details into Ansible's defaults, so every `ansible-playbook` invocation on the Control Node automatically targets the live EC2 fleet over SSH using the copied-over key:

```ini
[defaults]
host_key_checking = False
inventory = inventory_aws_ec2.yaml

enable_plugins = aws_ec2

remote_user = ec2-user
private_key_file = ~/ssh-key.pem
```

### 4. Write the Ansible Playbook to configure the EC2 Managed Nodes

[ansible/my-playbook.yaml](ansible/my-playbook.yaml) is the automation that actually configures the two EC2 instances — installing **Docker** and the **Docker Compose** CLI plugin so they're ready to run containerized workloads:

```yaml
- name: Install Docker
  hosts: all
  become: yes
  tasks:
    - name: Install Docker
      yum:
        name: docker
        update_cache: yes
        state: present
    - name: Start docker daemon
      systemd:
        name: docker
        state: started

- name: Install Docker-compose
  hosts: all
  tasks:
    - name: Create docker-compose directory
      file:
        path: ~/.docker/cli-plugins
        state: directory
    - name: Get architecture of remote machine
      shell: uname -m
      register: remote_arch
    - name: Install docker-compose
      get_url:
        url: "https://github.com/docker/compose/releases/latest/download/docker-compose-linux-{{ remote_arch.stdout }}"
        dest: ~/.docker/cli-plugins/docker-compose
        mode: +x
```

- The first play uses the **`yum`** module (the EC2 instances run Amazon Linux) to install Docker, then the **`systemd`** module to ensure the Docker daemon is started
- The second play detects the EC2 instance's CPU architecture with `uname -m` and uses the **`get_url`** module to download the matching Docker Compose binary straight into Docker's CLI-plugins directory, making `docker compose` available as a native Docker subcommand
- Both plays target `hosts: all`, which — thanks to the dynamic `aws_ec2` inventory — resolves to both EC2 Managed Nodes automatically, with no IP addresses hardcoded anywhere in the playbook

### 5. Write the Jenkinsfile to orchestrate the whole workflow

[Jenkinsfile](Jenkinsfile) is a **declarative pipeline** with two stages that bridge Jenkins, the Ansible Control Node, and the EC2 Managed Nodes together:

```groovy
pipeline {   
  agent any
  environment {
    ANSIBLE_SERVER = "165.232.189.172"
  }
  stages {
    stage("copy files to ansible server") {
      steps {
        script {
          echo "copying all neccessary files to ansible control node"
          sshagent(['ansible-server-key']) {
            sh "scp -o StrictHostKeyChecking=no ansible/* root@${ANSIBLE_SERVER}:/root"

            withCredentials([sshUserPrivateKey(credentialsId: 'ec2-server-key', keyFileVariable: 'keyfile', usernameVariable: 'user')]) {
              sh 'scp $keyfile root@$ANSIBLE_SERVER:/root/ssh-key.pem'
            }
          }
        }
      }
    }
    stage ("execute ansible playbook") {
      steps {
        script {
          echo "calling ansible playbook to configure ec2 instances"
          def remote = [:]
          remote.name = "ansible-server"
          remote.host = ANSIBLE_SERVER
          remote.allowAnyHosts = true

          withCredentials([sshUserPrivateKey(credentialsId: 'ansible-server-key', keyFileVariable: 'keyfile', usernameVariable: 'user')]) {
            remote.user = user
            remote.identityFile = keyfile
            sshScript remote: remote, script: "prepare-ansible-server.sh"
            sshCommand remote: remote, command: "ansible-playbook my-playbook.yaml"
          }
        }
      }
    }      
  }
} 
```

- **Stage 1 — "copy files to ansible server"**: the **`sshagent`** step loads the `ansible-server-key` credential into an SSH agent for the duration of the block, allowing the plain `scp` command to authenticate to the Control Node without ever writing the private key to the workspace. It copies everything under [ansible/](ansible/) (the playbook, `ansible.cfg` and the dynamic inventory config) to `/root` on the Control Node. A second, nested `withCredentials` block then retrieves the `ec2-server-key` private key as a temporary file and `scp`s it to the Control Node as `ssh-key.pem` — the exact path referenced by `private_key_file` in `ansible.cfg`
- **Stage 2 — "execute ansible playbook"**: a `remote` connection map is built targeting the same Control Node, then `withCredentials` again injects the `ansible-server-key` credential — this time to drive the **SSH Pipeline Steps** plugin's `sshScript` (which uploads and runs [prepare-ansible-server.sh](prepare-ansible-server.sh) remotely to install Ansible/Boto3) followed by `sshCommand`, which simply runs `ansible-playbook my-playbook.yaml` on the Control Node itself
- Because both Jenkins credentials are referenced **by ID** (`ansible-server-key`, `ec2-server-key`) rather than embedded private keys, the exact same `Jenkinsfile` can be pointed at a different Control Node or EC2 fleet just by updating the `ANSIBLE_SERVER` environment variable and the stored credentials — no pipeline code changes required

### 6. Create and run the Jenkins pipeline

A new **Pipeline** job named `ansible-pipeline` was created in Jenkins, pointed at this repository's `Jenkinsfile`, and triggered with **Build Now**.

### 7. Verify the deployment

Connecting to the Ansible Control Node confirms that the pipeline's first stage successfully copied over the playbook, Ansible config, dynamic inventory file and the EC2 SSH key:

![Files copied to the Ansible Control Node by the Jenkins pipeline](images/ansible-server-terminal.png)

The Jenkins **Stage View** confirms the full pipeline completed successfully end-to-end — checking out the Jenkinsfile from source control, copying the files to the Ansible Control Node in a couple of seconds, and then running the full Ansible playbook against both EC2 Managed Nodes in about a minute and a half:

![Jenkins pipeline stage view showing all stages completed successfully](images/jenkins-pipeline-success.png)

## ✅ Final result

By the end of this demo:

- 🧰 A dedicated **Jenkins server** and a dedicated **Ansible Control Node** were provisioned as two separate **DigitalOcean** droplets, keeping CI/CD orchestration cleanly separated from configuration-management execution
- 🔑 SSH credentials for both the Control Node and the EC2 Managed Nodes were securely stored in Jenkins' credential store and never hardcoded in the pipeline or committed to source control
- 📦 The **Jenkinsfile** automated the full hand-off — copying the Ansible playbook, configuration and SSH key to the remote Control Node, bootstrapping it with Ansible and Boto3, and triggering the playbook run — all from a single **Build Now** click
- 🔍 Ansible's **dynamic `aws_ec2` inventory plugin** discovered the two EC2 Managed Nodes automatically at runtime, with no static IPs hardcoded anywhere
- 🐳 Both EC2 Managed Nodes were configured idempotently with **Docker** and the **Docker Compose** CLI plugin, ready to run containerized workloads
- ✅ The pipeline's Stage View and the Control Node's filesystem both confirmed a fully successful, repeatable, end-to-end run

This project demonstrates how configuration management and CI/CD orchestration can be cleanly decoupled yet fully automated together — Jenkins handles *when* and *how* a deployment is triggered, while Ansible (running on its own dedicated Control Node) handles *what* actually gets configured on the target infrastructure, with AWS credentials, SSH keys and dynamic inventory discovery all flowing securely through Jenkins' credential store.

## 📚 References

- [Jenkins Documentation](https://www.jenkins.io/doc/)
- [Jenkins Pipeline Syntax](https://www.jenkins.io/doc/book/pipeline/syntax/)
- [SSH Agent Plugin](https://plugins.jenkins.io/ssh-agent/)
- [SSH Pipeline Steps Plugin](https://plugins.jenkins.io/ssh-steps/)
- [Jenkins Credentials Binding Plugin (`withCredentials`)](https://plugins.jenkins.io/credentials-binding/)
- [Ansible Documentation](https://docs.ansible.com/)
- [Ansible `amazon.aws.aws_ec2` Inventory Plugin](https://docs.ansible.com/ansible/latest/collections/amazon/aws/aws_ec2_inventory.html)
- [Ansible `yum` module](https://docs.ansible.com/ansible/latest/collections/ansible/builtin/yum_module.html)
- [Ansible `get_url` module](https://docs.ansible.com/ansible/latest/collections/ansible/builtin/get_url_module.html)
- [Boto3 (AWS SDK for Python) Documentation](https://boto3.amazonaws.com/v1/documentation/api/latest/index.html)
- [Docker Compose CLI plugin installation](https://docs.docker.com/compose/install/linux/)
- [Amazon EC2 Documentation](https://docs.aws.amazon.com/ec2/)
- [DigitalOcean Droplets Documentation](https://docs.digitalocean.com/products/droplets/)
