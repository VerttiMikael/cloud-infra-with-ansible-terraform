# cloud-infra-with-ansible-terraform
This project was made for course work and it involves using Terraform to create a set of virtual machines, then configuring a jumphost and aa HTTP load balancer using Ansible. 

## Prerequisites:
- Linux operating machine or  local Ubuntu vm
- Python virtual enviroment installed
- ssh keypair for the vm instances
---

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
---
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

In this projects we will use ubuntu-24.04 as the image and the flavor standard.small
Rest of the images and flavors can be found with the commands

```bash
openstack image list
openstack flavor list
```

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
![Projects_Pics](/ProjectPictures/pic1.png)

We can now see in the OpenStack dashboard that our vm instances are online 

![Projects_Pics](/ProjectPictures/pic2.png)

Lets now run  
```bash
terraform graph -type=plan | dot -Tpng >graph.png  
```
To see our current architecture 

![Projects_Pics](/ProjectPictures/pic3.png)

---
## Ansible setup
 Now that we have deployed our virtual machines with terraform, we can start setting up our jumphost and HTTP load balancer with Ansible

### Step 1: Making a virtual enviroment for ansible 

```bash
verlep@verlep-VirtualBox:~$ python3 -m venv ansible-2.9.1 
verlep@verlep-VirtualBox:~$ source ansible-2.9.1/bin/activate 
(ansible-2.9.1) verlep@verlep-VirtualBox:~$ pip install ansible  
```

### Step 2: Make a myansible directory and add all the necessary files

after that the file hierarchy should look like this:
```plaintext
myansible/
├── ansible.cfg
├── files/
│   ├── nginx.conf
│   └── nginx_proxy.conf
├── inventory/
│   └── csc.ini
├── templates/
│   └── index.html.j2
└── webservers.yml
```

### Step 3. Modify ansible.cfg file
```bash
[defaults]
inventory = inventory/csc.ini
host_key_checking = False
stdout_callback = yaml
callback_enabled = timer
interpreter_python = auto_silent 
```

### Step 4. Modify the ssh config file
```bash
Host *  
  ForwardAgent yes  

Host VM1_host 
  Hostname 195.148.21.XX 
  User ubuntu  
  Port 22  
  IdentityFile /home/lepver/.ssh/id_ed25519  

Host VM_2 
  Hostname 192.168.1.XX 
  User ubuntu 
  ProxyJump VM1_host 

Host VM_3 
  Hostname 192.168.1.XX 
  User ubuntu 
  ProxyJump VM1_host

Host VM_4 
  Hostname 192.168.1.XX 
  User ubuntu 
  ProxyJump VM1_host 
```

### Step 5. Modify the csc.ini file
```bash
[webservers] 
testserver ansible_port=22  

[webservers:vars] 
ansible_host=195.148.22.XXX 
ansible_user=ubuntu 
ansible_private_key_file=/home/verlep/.ssh/id_ed25519 

[csc_proxy] 
VM1_host   

[csc_vms] 
VM_2 
VM_3 
VM_4 
```
lets test the connections to the virtual machines using these commands

```bash
ansible -m ping csc_proxy
ansible -m ping csc_vms
```
and we should get results like this
![Projects_Pics](/ProjectPictures/pic6.png)

### Step 6. Modify webservers.yml, nginx.conf and index.html.j2

webservers.yml
```bash
- name: Configure HTTP load balancer with nginx 
  hosts: csc_proxy 
  become: yes 
  tasks: 
    - name: Install nginx 
      apt: 
        name: nginx 
        state: present 
        update_cache: yes 
  
    - name: Copy nginx proxy configuration to load balancer 
      copy: 
        src: files/nginx_proxy.conf 
        dest: /etc/nginx/nginx.conf 
        owner: root 
        group: root 
        mode: '0644' 

    - name: Enable and start nginx on load balancer 
      service: 
        name: nginx 
        state: started 
        enabled: yes 

- name: Configure web servers with nginx 
  hosts: csc_vms 
  become: yes 
  tasks: 
    - name: Install nginx 
      apt: 
        name: nginx 
        state: present 
        update_cache: yes 
  
    - name: Copy nginx web server configuration 
      copy: 
        src: files/nginx.conf 
        dest: /etc/nginx/nginx.conf 
        owner: root 
        group: root 
        mode: '0644' 

    - name: Copy index.html template to web server document root 
      template: 
        src: templates/index.html.j2 
        dest: /usr/share/nginx/html/index.html 
        owner: www-data 
        group: www-data 
        mode: '0644' 

    - name: Enable and start nginx on web servers 
      service: 
        name: nginx 
        state: started 
        enabled: yes 
```
nginx.conf
```bash
server { 
        listen 80 default_server; 
        listen [::]:80 default_server ipv6only=on; 

        root /usr/share/nginx/html; 
        index index.html index.htm;   

        server_name localhost;   
        location / { 
                try_files $uri $uri/ =404; 
        } 
} 
```
index.html.j2
```bash
<html> 
  <head> 
    <title>Welcome to ansible</title> 
  </head> 
  <body> 
  <h1>Nginx, configured by Ansible</h1> 
  <p>If you can see this, Ansible successfully installed nginx.</p> 
  <p>Running on {{ inventory_hostname }}</p> 
  </body> 
</html>  
```

### Step 6. Modify the nginx_proxy.conf file
```bash
# Load ngx_stream_module  
load_module /usr/lib/nginx/modules/ngx_stream_module.so; 

events { 
    worker_connections 8192; 
} 

stream { 
    # Log file settings  
    log_format dns '$remote_addr - - [$time_local] $protocol $status $bytes_sent $bytes_received $sess> 
    access_log /var/log/nginx/access.log dns; 
    error_log /var/log/nginx/error.log; 

    upstream myapp1 { 
        server 192.168.1.XX:80; # Private IP address of the vm2 and port number 
        server 192.168.1.XX:80; # Private IP address of the vm3 and port number 
        server 192.168.1.XX:80; # Private IP address of the vm4 and port number 
    } 
    server { 
        listen 80;             # Nginx load balancer listen port 80 
        proxy_pass myapp1;     # The server group myapp1 
    } 
}  
```

### Step 7. Deploy the playbook
```bash
ansible-playbook webservers.yml
```
---
### Result

We should now have a working envionment consisting of 1 jumphost and HTTP load balancer and 3 web servers
![Projects_Pics](/ProjectPictures/pic8.PNG)

### Sources:
- Setting up Terraform
https://greenwebpage.com/community/how-to-install-terraform-on-ubuntu-24-04/
https://training.galaxyproject.org/training-material/topics/admin/tutorials/terraform/tutorial.html

- Setting up .tf files
https://registry.terraform.io/providers/terraform-provider-openstack/openstack/2.1.0/docs/resources/compute_floatingip_associate_v2
https://registry.terraform.io/providers/terraform-provider-openstack/openstack/2.1.0/docs/resources/compute_secgroup_v2

- Setting up Ansible
https://www.oreilly.com/library/view/ansible-up-and/9781098109141/ch03.html#creating_a_group
