# cloud-infra-with-ansible-terraform
This project involves using Terraform to create a set of virtual machines, then configuring a jump host and an HTTP load balancer using Ansible.

# Installing Terraform on a local Ubuntu virtual machine
Updated and Upgraded System Packages
sudo apt update
sudo apt upgrade -y
-------------------------
Install Required Packages
sudo apt install gnupg software-properties-common -y
-------------------------------------------
Download the HashiCorp GPG key for Terraform
wget -O- https://apt.releases.hashicorp.com/gpg | sudo gpg --dearmor -o /usr/share/keyrings/hashicorp-archive-keyring.gpg
-----------------------------------------------------
 Add the HashiCorp APT Repository
echo "deb [signed-by=/usr/share/keyrings/hashicorp-archive-keyring.gpg] https://apt.releases.hashicorp.com $(lsb_release -cs) main" | sudo tee /etc/apt/sources.list.d/hashicorp.list
----------------------------------------------------
Update Ubuntu Package List
udo apt update
----------------------------------------------
Install Terraform via APT
sudo apt install terraform -y

Links:
- Setting up Terraform:
https://greenwebpage.com/community/how-to-install-terraform-on-ubuntu-24-04/
https://training.galaxyproject.org/training-material/topics/admin/tutorials/terraform/tutorial.html
