# Amazon Linux 2023 on Hyper-V

Launch Amazon Linux 2023 locally on Hyper-V using `cloud-init`. Downloads of preconfigured images are available via private link in the advanced setup folders. 

## Quick Start

```bash
git clone https://github.com/hayeseoin/al2023-hyperv
cd al2023-hyperv
cp user-data.template user-data
# Insert your SSH key into user-data
genisoimage -output seed.iso -volid cidata -joliet -rock user-data meta-data
```

## Overview

This guide covers:
- Generating `seed.iso` with cloud-init data
- Creating a Hyper-V VM
- Logging in via SSH

Use the following directories for advanced setups:
- [`vagrant-box`](vagrant-box) — Reusable Vagrant box
- [`hyper-v-template`](hyper-v-template) — Importable Hyper-V template

## Requirements

- WSL2 (Debian or similar) or some other Linux terminal to generate the seed.iso
- Hyper-V with Default Switch
- [`genisoimage`](https://linux.die.net/man/1/genisoimage)
- AL2023 Hyper-V image: [Download here](https://cdn.amazonlinux.com/al2023/os-images/latest/)


## Steps

### Step 1. Create user-data file
Copy user-data.template to user-data and replace the SSH key with your actual ssh pubic key. Snippet below:

```sh
...
    ssh_authorized_keys:
      - ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABgQCuSpmtQkeIjxliaQn3gfeoWWAYbNXLT2cwJS+jZnTQu ...
    lock_passwd: false
    # Set password as password
    passwd: "$6$/EcnIbxiVa3zpvGi$ivoX7m/BL85E3SULpq2PHIqr2MLl0IaubOPAdpCheIZ1KF4W6618YlaLng.ve2r6lUlP5v.qqBOCcasL4ATpd1"
hostname: al2023
...
```
If you don't already have an SSH key, generate one:
```sh
ssh-keygen -t rsa -b 4096
```
The user-data above configures the username and password as ec2-user/password. Passwords should be avoided for SSH access, so I reccomend not including this line and using only your SSH key.

Optional: You can configure the meta-data file too to whatever you want, the config in this repo works fine for testing.

### Step 2. Generate seed.iso file
Run this command (this only works in Linux, alternate steps exist for Windows)

```sh
genisoimage -output seed.iso -volid cidata -joliet -rock user-data meta-data
```

### Step 3. Download and prepare AL2023 image
Create a directory for your seed.iso file and the Hyper V template from Amazon e.g. C:\Users\<your_user>\hyperv\al2023-base

Copy the seed.iso file to this location.

Download the Hyper V disk and save it and extract it to here. The format should be VHDX. It may be in a nester folder.

### Step 4: Create and configure Hyper-V VM:
- New -> Virtual Machine
- Name: al2023-base
- Generation 2
- Assign memory: 512 MB
- Configure Networking: Default Switch
- Use an existing Virtual hard disk: Choose the VHDX file you extracted in step 4.
- Create virtual machine

Once the new VM is created, don't start it yet. Navigate to the VM's settings. 
- Virtual CPUs: Set to 1 or 2 
- Security: Uncheck 'Enable secure boot' (AL2023 does not support Hyper V secure boot)
- SCSI Controller: 
    - Add a DVD drive
    - Image file: The seed.iso file you created i.e C:\Users\<your_user>\hyperv\al2023-base.local\seed.iso
    - Optional: Disable checkpoints if you don't want them.

### Step 5: Launch and SSH in
Start the VM in Hyper V.

The IP address should show up in the Networking tab of the VM in Hyper V. You can add an entry to your Windows `hosts` file to access the VM via hostname. This is not static, it will reset when the computer resets.

![alt text](images/image.png)

I recommend using [PowerToys Hosts File Editor](https://learn.microsoft.com/en-us/windows/powertoys/hosts-file-editor) or the [Hyper V Hosts Manager](https://github.com/hayeseoin/hyper-v-hosts-manager) to simplify this process.

![alt text](images/image-1.png)

SSH into the VM in Powershell or WSL

```sh
ssh ec2-user@al2023-base.local
```
Notes: If you're using WSL, you will need to run the following Powershell command to reach Hyper V from WSL. This is not persistent config and will need to be run every time you restart your computer.
```ps
Get-NetIPInterface | where {$_.InterfaceAlias -eq 'vEthernet (WSL (Hyper-V firewall))' -or $_.InterfaceAlias -eq 'Default Switch'} | Set-NetIPInterface -Forwarding Enabled -Verbose
``` 

## 🔐 Security Warning

> This lab setup is not hardened. Avoid using for exposed/public environments.
