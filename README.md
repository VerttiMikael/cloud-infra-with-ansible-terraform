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

## Setting up terraform 

### Step 1: creating providers.tf and main.tf files
```bash
touch providers.tf 
touch nano.tf 
```
### Step 2: Configuring the providers.tf file 
```bash
nano providers.tf  

# providers.tf
terraform { 
required_version = ">= 0.14.0" 
  required_providers { 
    openstack = { 
      source  = "terraform-provider-openstack/openstack" 
      version = "~> 1.53.0" 
    } 
  } 
}

provider "openstack" { 
  user_name   =  
  tenant_name = "project_2011826" 
  password    =  
  auth_url    = "https://pouta.csc.fi:5001/v3" 
  region      = "regionOne" 
  user_domain_name = "Default" 
} 
```

### Step 3: Initializing terraform
```bash
terraform init
```

### Step 4: Configuring the main.tf file
```bash
nano main.tf

# main.tf

resource "openstack_compute_keypair_v2" "my-cloud-key" {  
  name       = "project1-key"  
  public_key = "ssh-ed25519 AAA....”  
}  

# Create the first virtual machine   
resource "openstack_compute_instance_v2" "VM1" {  
  name          = "VM 1"  
  image_id      = "be795267-2cbb-44d8-aa6f-4411d0df7df9"  
  flavor_id     = "d4a2cb9c-99da-4e0f-82d7-3313cca2b2c2"  
  network {  
    name = "project_2011826"  
  }  
  security_groups = [openstack_compute_secgroup_v2.secgroup_1.name]  
  key_pair ="${ openstack_compute_keypair_v2.my-cloud-key.name}"  
}  

# Create the rest of the virtual machines  
resource "openstack_compute_instance_v2" "VM2-4" {  
  count         = 3  
  name          = "VM ${count.index+2}"  
  image_id      = "be795267-2cbb-44d8-aa6f-4411d0df7df9"  
  flavor_id     = "d4a2cb9c-99da-4e0f-82d7-3313cca2b2c2"  
  network {  
    name = "project_2011826"  
  }  
  security_groups = [openstack_compute_secgroup_v2.secgroup_2.name]  
  key_pair ="${ openstack_compute_keypair_v2.my-cloud-key.name}"  
}  

# Implement the floating ip to VM 1  
resource "openstack_networking_floatingip_v2" "fip_1" {  
  pool = "public"  
}  

resource "openstack_compute_floatingip_associate_v2" "fip_1" {  
  floating_ip = openstack_networking_floatingip_v2.fip_1.address  
  instance_id = openstack_compute_instance_v2.VM1.id  
}  

# Creating the first sec group for VM1  
resource "openstack_compute_secgroup_v2" "secgroup_1" {  
  name        = "secgroup_1"  
  description = "security group for VM1"  

  rule {  
    from_port   = 22  
    to_port     = 22  
    ip_protocol = "tcp"  
    cidr        = "0.0.0.0/0"  
  }  

  rule {  
    from_port   = 80  
    to_port     = 80  
    ip_protocol = "tcp"  
    cidr        = "0.0.0.0/0"  
  }  

  rule {  
    from_port   = 1  
    to_port     = 65535  
    ip_protocol = "tcp"  
    cidr        = "192.168.1.0/24"  
  }  
}  

# Creating the second sec group for VM2-4  
resource "openstack_compute_secgroup_v2" "secgroup_2" {  
  name        = "secgroup_2"  
  description = "security group for VM2-4"  

  rule {  
    from_port   = 1  
    to_port     = 65535  
    ip_protocol = "tcp"  
    cidr        = "192.168.1.0/24"  
  }  
} 
```

### Step 5: Save and deploy changes 
```bash
terraform apply 
```


### Links:
- Setting up Terraform
https://greenwebpage.com/community/how-to-install-terraform-on-ubuntu-24-04/
https://training.galaxyproject.org/training-material/topics/admin/tutorials/terraform/tutorial.html

- Setting up .tf files
https://registry.terraform.io/providers/terraform-provider-openstack/openstack/2.1.0/docs/resources/compute_floatingip_associate_v2
https://registry.terraform.io/providers/terraform-provider-openstack/openstack/2.1.0/docs/resources/compute_secgroup_v2
