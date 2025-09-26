````markdown
## CIT 4640 — Week 4 Lab: Terraform + cloud-init

This repo provisions a **Debian** EC2 instance in **us-west-2** (AZ **us-west-2a**) using Terraform.  
Bootstrapping is done with **cloud-init** at `scripts/cloud-config.yaml` to:
- Create a `web` user
- Install **nginx** and **nmap**
- Enable/start nginx

The Terraform config also creates:
- A dedicated **VPC** (`10.0.0.0/16`) with a **public subnet** (`10.0.1.0/24`)
- An **Internet Gateway** + route to `0.0.0.0/0`
- A **Security Group** opening **SSH (22/tcp)** and **HTTP (80/tcp)**
- An **AWS Key Pair** named **`lab4_key`** from your local `~/.ssh/lab4_key.pub` (note: the instance login relies on cloud-init’s `web` user key)

---

## 1) Prerequisites (General Setup) — Previously on Lab 3

- Linux dev environment with Bash and OpenSSH client
- Terraform **v1.5+** (provider pinned to `hashicorp/aws ~> 6.0`)
- AWS credentials configured locally:
  ```bash
  aws configure
  # or environment variables:
  export AWS_ACCESS_KEY_ID=...
  export AWS_SECRET_ACCESS_KEY=...
  export AWS_DEFAULT_REGION=us-west-2
````

* Git

> If you need to install Terraform on Ubuntu/Debian:

```bash
sudo apt-get update -y
sudo apt-get install -y gnupg software-properties-common curl
curl -fsSL https://apt.releases.hashicorp.com/gpg | sudo gpg --dearmor -o /usr/share/keyrings/hashicorp-archive-keyring.gpg
echo "deb [signed-by=/usr/share/keyrings/hashicorp-archive-keyring.gpg] https://apt.releases.hashicorp.com $(lsb_release -cs) main" | sudo tee /etc/apt/sources.list.d/hashicorp.list
sudo apt-get update -y && sudo apt-get install -y terraform
terraform -version
```

---

## 2) Get the Starter & Directory Layout

```bash
git clone https://gitlab.com/cit_4640/4640-w4-lab-start-w25.git
cd 4640-w4-lab-start-w25
mkdir -p scripts
```

---

## 3) Create a New SSH Key Pair (Task 1)

> Your `main.tf` expects an AWS Key Pair resource named **`lab4_key`** that reads `~/.ssh/lab4_key.pub`.
> We also use the **same public key** in `cloud-config.yaml` for the **`web`** user.

```bash
ssh-keygen -t ed25519 -C "lab4_key" -f ~/.ssh/lab4_key
cat ~/.ssh/lab4_key.pub
chmod 600 ~/.ssh/lab4_key
```

---

## 4) Cloud-Init Configuration (Task 1)

Create/edit `scripts/cloud-config.yaml` and paste your **public key** (from above) into `ssh_authorized_keys`:

```yaml
#cloud-config
users:
  - name: web
    primary_group: web
    groups: sudo
    shell: /bin/bash
    sudo: ['ALL=(ALL) NOPASSWD:ALL']
    ssh_authorized_keys:
      - ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIMffqKze8r+q6jFwWFGyJJo77W3GDjRdGPga+AWnseYp lab4

packages:
  - nginx
  - nmap
```

---

## 5) Terraform Commands (Task 2)

From the repo root:

```bash
terraform init
terraform fmt
terraform validate
terraform plan -out plan
terraform apply plan
```

**Outputs (IP & DNS):**

```bash
terraform output instance_ip_addr
{
  "dns_name" = "ec2-54-186-120-15.us-west-2.compute.amazonaws.com"
  "public_ip" = "54.186.120.15"
}
```

---

## 6) Connect & Verify (Task 3)

Cloud-init created the **`web`** user with your SSH key:

```bash
ssh -i ~/.ssh/lab4_key web@ec2-54-186-120-15.us-west-2.compute.amazonaws.com
```
