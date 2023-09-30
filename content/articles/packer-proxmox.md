---
title: "Creating a RHEL Template with Packer on Proxmox"
date: 2023-09-29
draft: false
---
HashiCorp's Packer automates the creation of machine images across various platforms. I have recently been learning a bit about Packer, and in this post, I will show an example of using Packer to generate a Red Hat Enterprise Linux (RHEL) template on Proxmox. I still need to learn more of the Packer intricacies, and this article will be pretty high-level, but I hope it helps you at least start using Packer.

### Prerequisites:

- A Proxmox environment set up and running.
- A RHEL ISO, we will use RHEL8 in this demonstration.
- Packer installed on your local machine; see references for docs on installing Packer.

### Step 1: Create the Packer Configuration File

Configuration file named `rhel-template.pkr.hcl`:

```hcl {linenos=true}
packer {
  required_plugins {
    proxmox = {
      version = ">= 1.1.3"
      source  = "github.com/hashicorp/proxmox"
    }
  }
}

variable "proxmox_host" {
    type = string
}

variable "proxmox_node" {
    type = string
}

variable "iso_file" {
    type = string
}

variable "proxmox_user" {
    type = string
}

variable "proxmox_pass" {
    type = string
}

variable "vm_name" {
    type = string
}

variable "ssh_user" {
    type = string
}

variable "ssh_pass" {
    type = string
}

source "proxmox-iso" "rhel-tpl" {

    proxmox_url = "https://${var.proxmox_host}/api2/json"
    insecure_skip_tls_verify = true
    node = var.proxmox_node
    iso_file = var.iso_file
    vm_name = var.vm_name
    username = var.proxmox_user
    password = var.proxmox_pass
    cpu_type = "Nehalem"
    cores = "2"
    memory = "1024"
    disks {
        disk_size         = "60G"
        storage_pool      = "local-lvm"
        type              = "virtio"
    }
    network_adapters {
        bridge = "vmbr0"
    }
    ssh_username = var.ssh_user
    ssh_password = var.ssh_pass
    ssh_timeout = "30m"
    ssh_handshake_attempts = "100"
    boot_command = ["<esc><wait>", "vmlinuz initrd=initrd.img ", "inst.ks=http://{{ .HTTPIP }}:{{ .HTTPPort }}/ks.cfg", "<enter>"]
    http_directory = "http"
}

build {
    sources = ["source.proxmox-iso.rhel-tpl"]
}
```

There are four parts of the above config that I want to call out.

- The top section, lines 1-8, is how we load the plugin for proxmox.
- The following section , lines 10-40, defines the Packer variables. It doesn't assign them; it just defines them—more on assigning later.
- In the next section, lines 42-68, you define what Packer needs for connection and setting up the template on proxmox. Things like proxmox url, iso file to use, CPU/memory, disk, etc., are all assigned here.
- The last section, lines 70-72, is the build section, where you can do some cool things like run shell scripts or an Ansible playbook (among other tools) to finish the template setup at the end of the kickstart run. In this example, though, I am just letting the template finish building with no additional configuration steps.

### Step 2: Install the Packer Proxmox Plugin

With the above configuration file we can do a:

```bash
packer init .
```

The `packer init .` command will install the Packer proxmox plugin.

### Step 3: Set Up a Kickstart Configuration

The configuration's file boot_command above references a Kickstart file (`ks.cfg`). Create this in a directory named `http`:

```bash
mkdir http
```

Here's a basic `ks.cfg` to get started:

```cfg
#version=RHEL8
lang en_US.UTF-8
keyboard us
timezone America/Los_Angeles --isUtc
rootpw your-rhel-root-password
reboot
text
cdrom
bootloader --append="rhgb quiet crashkernel=1G-4G:192M,4G-64G:256M,64G-:512M"
zerombr
clearpart --all --initlabel
autopart
skipx
firstboot --disable
selinux --enforcing
%packages
@^minimal-environment
%end
```
Replace `your-rhel-root-password` with your desired password.

### Step 4: Create a vars file
This file will be assigning values to the variables you set in the first file we created. Here is an example:

### rhel_vars.pkrvars.hcl

```hcl
proxmox_host = your-proxmox-host
proxmox_node = "pve01"
iso_file = "isos:iso/rhel-8.6-x86_64-dvd.iso"
proxmox_user = your-promox-user
proxmox_pass = your-proxmox-user-password
ssh_user = "root"
ssh_pass = your-rhel-root-password
vm_name = "rhel-tpl"

Replace the values that start with "your-" with your values.

### Step 5: Run Packer

With configurations set, initiate the Packer build:

```bash
packer build -var-file=rhel_vars.pkrvars.hcl .
```

Monitor Packer's output. It will create the VM, provision it, and convert it into a Proxmox template.

Once Packer completes, log into the Proxmox web interface. You should see the new RHEL template. Clone this template whenever you need a new RHEL instance.

---

With Packer, you can automate the creation of standardized machine images for Proxmox, ensuring that every instance starts from a known configuration. 

>NOTE: This article is as much for me as anyone else, like most of my articles. I will continue to update this article as I discover new things and refinements for Packer. 

### References and further reading:

Packer:
https://www.packer.io/

Packer install docs:
https://developer.hashicorp.com/packer/tutorials/docker-get-started/get-started-install-cli

Proxmox:
https://www.proxmox.com/en/

Developer subscription for RHEL:
https://developers.redhat.com/articles/faqs-no-cost-red-hat-enterprise-linux
