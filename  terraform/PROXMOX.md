



module pod frappe framework z erpnext, lms i raven chatem oraz instalacyjne pliki pod budowę web stron internetowych builder:
https://codewithkarani.com/2023/12/31/how-to-install-erpnext-version-15-in-ubuntu-a-step-by-step-guide/ pliki instalacyjne pod instalację frappe frameworku na Ubuntu 22.04 używając frappe bench i python3.10 venv

https://github.com/gavindsouza/awesome-frappe?tab=readme-ov-file  - module plugins for frappe framework (15)

https://github.com/frappe/builder - frappe web builder for web page design




           
##  ERPNext Version 15 on Ubuntu 22.04 using Terraform 
                
To install ERPNext Version 15 on Ubuntu 22.04 using Terraform within a Proxmox environment, you'll need to follow a series of steps to create the infrastructure and configure the VM via cloud-init. Let's split it into sections: Terraform configuration, Proxmox setup, and post-deployment steps.

### Prerequisites
1. Proxmox set up and running.
2. Terraform installed locally.
3. Proxmox credentials ready.
4. Proxmox CLI tool (`pve_iso`) and API access configured.

Assuming you have all these prerequisites in place, let's proceed.

### Step 1: Create Terraform Configuration Files

#### 1.1. Provider Configuration
Create a file called `main.tf` and start by defining the Proxmox provider and configuration.


```hcl
provider "proxmox" {
  pm_api_url      = "https://YOUR_PROXMOX_HOST:8006/api2/json"
  pm_user         = "root@pam"
  pm_password     = "YOUR_PROXMOX_PASSWORD"
  pm_tls_insecure = true
}
```

#### 1.2. Proxmox VM Configuration
Next, define the VM configuration.

```hcl
resource "proxmox_vm_qemu" "ubuntu_erpnext" {
  name        = "ubuntu-erpnext"
  target_node = "YOUR_PROXMOX_NODE" # e.g., pve

  # VM hardware
  vmid     = 100 # You may choose an ID based on your setup
  memory   = 4096
  cores    = 2

  # VM disk configuration
  disk {
    size           = "32G"
    storage        = "local-lvm" # Confirm your Proxmox storage name
    type           = "scsi"
    iothread       = true  # Enabling IO threads may improve disk performance
  }

  # Network configuration
  network {
    model   = "virtio"
    bridge  = "vmbr0" # Confirm your Proxmox network bridge
  }

  # Cloud-init configuration
  cloud_init {
    target_user    = "ubuntu"
    ssh_keys = <<EOF
ssh-rsa YOUR_SSH_PUBLIC_KEY
EOF
    user_data = <<EOF
#cloud-config
packages:
  - ca-certificates
  - curl
  - gnupg
  - lsb-release
  - docker.io

runcmd:
  - sudo systemctl start docker
  - sudo systemctl enable docker
  - docker run -d --name erpnext --restart always -p 80:80 frappe/erpnext:v15.0.0 # Replace this with the actual image tag once released
EOF
  }
  
  # Base image configuration
  clone {
    base_vm_id = "YOUR_TEMPLATE_VMID"  # Provide the template VM ID
  }
}
```

Make sure you replace the placeholders with your actual Proxmox details like host, password, node name, storage, bridge, and VM template ID.

#### 1.3. Optional Variables and Outputs
To keep configurations modular:

##### variables.tf
```hcl
variable "pm_api_url" {
  description = "URL for the Proxmox API"
  default     = "https://YOUR_PROXMOX_HOST:8006/api2/json"
}

variable "pm_user" {
  description = "Proxmox user"
  default     = "root@pam"
}

variable "pm_password" {
  description = "Proxmox password"
}

variable "pm_tls_insecure" {
  description = "Ignore TLS certificate verification"
  default     = true
}

variable "vmid" {
  description = "VM ID"
  default     = 100
}

variable "memory" {
  description = "Memory size for VM"
  default     = 4096
}

variable "cores" {
  description = "Number of CPU cores for VM"
  default     = 2
}

variable "disk_size" {
  description = "Disk size for VM"
  default     = "32G"
}

variable "storage" {
  description = "Storage location for VM disk"
  default     = "local-lvm"
}

variable "bridge" {
  description = "Network bridge for VM"
  default     = "vmbr0"
}
```

##### outputs.tf
```hcl
output "vm_id" {
  description = "ID of the created VM"
  value       = proxmox_vm_qemu.ubuntu_erpnext.vmid
}

output "vm_name" {
  description = "Name of the created VM"
  value       = proxmox_vm_qemu.ubuntu_erpnext.name
}

output "vm_ip" {
  description = "IP address of the created VM"
  value       = proxmox_vm_qemu.ubuntu_erpnext.network.0.ip
}
```

### Step 2: Deploy with Terraform

After ensuring the Terraform files have the correct details, initialize and apply the configuration.

```sh
terraform init
terraform plan
terraform apply
```

When prompted, confirm the deployment by typing `yes`.

### Step 3: Post-Deployment

1. **Accessing the VM:**
    - SSH into the VM using your SSH key:
      ```sh
      ssh -i YOUR_SSH_PRIVATE_KEY ubuntu@VM_IP_ADDRESS
      ```
2. **Verifying Docker and ERPNext:**
    - Check if the Docker container is running:
      ```sh
      docker ps
      ```

3. **Accessing  - Access ERPNext by opening the VM's IP address in a web browser:
     ```
     http://VM_IP_ADDRESS
     ```

### Additional Notes
1. **Template Preparation on Proxmox:**
    - Ensure you have a VM template with Ubuntu 22.04 and cloud-init installed.
    - Convert your provisioned VM to a template. For example:
      ```sh
      qm template YOUR_TEMPLATE_VMID
      ```

2. **Security Considerations:**
    - Ensure that your Proxmox setup and Terraform apply are secured and have appropriate firewall rules and SSH key management.
    - Always use strong passwords and rotate them regularly.

3. **ERPNext Version and Docker Image:**
    - The Docker image tag `frappe/erpnext:v15.0.0` is a placeholder. Replace it with the actual tag once ERPNext Version 15 is available.

This guide should get your ERPNext environment set up on Proxmox using Terraform. Adjustments may be required based on specific needs and environment configurations. Happy deploying!
