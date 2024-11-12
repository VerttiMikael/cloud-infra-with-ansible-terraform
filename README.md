# cloud-infra-with-ansible-terraform
This project involves using Terraform to create a set of virtual machines, then configuring a jump host and an HTTP load balancer using Ansible.

## Installing Terraform on a Local Ubuntu Virtual Machine

### Step 1: Update and Upgrade System Packages
```bash
sudo apt update
sudo apt upgrade -y
```

### Step 2: Install Required Packages
```bash
sudo apt install gnupg software-properties-common -y
```

### Step 3: Download the HashiCorp GPG Key for Terraform
```bash
wget -O- https://apt.releases.hashicorp.com/gpg |
sudo gpg --dearmor -o /usr/share/keyrings/hashicorp-archive-keyring.gpg
```

### Step 4: Add the HashiCorp APT Repository
```bash
echo "deb [signed-by=/usr/share/keyrings/hashicorp-archive-keyring.gpg]
https://apt.releases.hashicorp.com $(lsb_release -cs) main" |
sudo tee /etc/apt/sources.list.d/hashicorp.list
```

### Step 5: Update Ubuntu Package List and install Terraform via APT
```bash
sudo apt update
sudo apt install terraform -y
```


### Links:
- Setting up Terraform
https://greenwebpage.com/community/how-to-install-terraform-on-ubuntu-24-04/
https://training.galaxyproject.org/training-material/topics/admin/tutorials/terraform/tutorial.html

- Setting up .tf files
https://registry.terraform.io/providers/terraform-provider-openstack/openstack/2.1.0/docs/resources/compute_floatingip_associate_v2
