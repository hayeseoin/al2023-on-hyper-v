# Create a Hyper-V Template for Amazon Linux 2023

These steps create a Hyper-V template which can be imported as a new VM. This repo includes a ready made seed.iso file for launching an Amazon Linux 2023 (AL2023) instance on Hyper-V. The Hyper-V image can be downloaded here https://cdn.amazonlinux.com/al2023/os-images/latest/

To do this, we will boot the VM with the provided seed.iso file (which adds the insecure key), then export the VM to a template. The steps below cover the proess.

When the template is deployed, it can be accessed with the `ec2-insecure-access.pem` key. Users of the template should delete this key after the template is deployed (or better yet, deploy this as a vagrant box).

## 1. Download Hyper-V image
Download the Hyper-V image from Amazon, and the seed.iso file from this folder `hyper-v-template/seed.iso`.

## 2. Create VM
In Hyper-V, create a new VM: Actions >> New >> Virtual Machine.  
Create the VM with the following settings, but *don't* start it when done.
 - Name: al2023-hyperv-template
    - Location doesn't matter - we'll be exporting it anyway.
 - Specify Generation: Generation 2
 - Assign Memory: 1024 MB  
    - Dynamic memory: optional
 - Configure Networking: Default Switch
 - Connect Virtual Hard Disk:  
 Use an existing virtual disk: `"C:\path\to\al2023-hyperv-2023.7.20250512.0-kernel-6.1-x86_64.xfs.gpt.vhdx"`

## 3. Configure VM settings
Before starting the VM, right click and press Settings and set:
 - Security: Disable Secure Boot
 - Processor: Set to 1 or 2 if not already there
 - Checkpoints: Disable checkpoints
Apply changes, but stay in settings.

## 4. Attach `seed.iso`
While still in settings, attach the seed.iso:
 - SCSI Controller:
    - DVD Drive - Add
 - DVD Drive: 
    - Media: Image File: `C:\path\to\seed.iso`
Apply changes. Exit settings.

## 5. Start the VM
Start the VM and confirm access. To confirm access, use the insecure key in this repo `ec2-insecure-access.pem`
```
$ ssh -i ec2-insecure-access.pem ec2-user@172.28.71.70

The authenticity of host '172.28.71.70 (172.28.71.70)' can't be established.
ED25519 key fingerprint is SHA256:as8aSBFFYJYhojgl0DVT/B6Un0hMIicgRA6teQOzZQA.
This key is not known by any other names.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes

Warning: Permanently added '172.28.71.70' (ED25519) to the list of known hosts.
   ,     #_
   ~\_  ####_        Amazon Linux 2023
  ~~  \_#####\
  ~~     \###|
  ~~       \#/ ___   https://aws.amazon.com/linux/amazon-linux-2023
   ~~       V~' '->
    ~~~         /
      ~~._.   _/
         _/ _/
       _/m/'
[ec2-user@localhost ~]$ 
```
## 6. Turn off and Export the VM
Turn off the VM (right click - Turn off).

**Important**: Before exporting the VM, remove the DVD drive in the settings.

Right click the VM and click Export..., then save it somewhere on your computer. e.g. `C:\Users\<your_user>>\hyperv\`

## 7. Confirm template can be deployed
You should be able to import this by selecting the folder you just exported. 
 - First, create a new directory to host the new VM, e.g. `C:\Users\<your_user>\hyperv\al2023-template-test`.
 - In Hyper-V, go to Actions > New > Import Virtual Machine
    - Create a new directory to host the new VM, e.g. `C:\Users\<your_user>\hyperv\al2023-template-test`.
    - Locate Folder: `C:\path\to\your\template\folder`  
        - e.g. for me this is `C:\Users\eoaha\hyperv\al2023-hyperv-template\`
    - Select Virtual Machine: Select the VM we just created.
    - Choose Import Type: **Copy** the virtual machine.

You must use copy. Registering in place will only 

You may have to store the VM in a different location. Hyper-V doesn't like when virtual hard drives hav ethe same name, and this is exactly what happens when we import a machine and copy it like this. If the defautls don't allow you to save the new hard drive try something like, e.g.
 - Virtual machine configuration folder: `C:\Users\eoaha\hyperv\al2023-template-test`
 - Smart Paging file store: `C:\Users\eoaha\hyperv\al2023-template-test`
 - Checkpoint folder: `C:\Users\eoaha\hyperv\al2023-template-test`
 - Virtual hard disk destination folder: `C:\Users\eoaha\hyperv\al2023-template-test\Virtual Hard Disks\`

## 8. Confirm access
Confirm new VM can be accessed.
```
$ ssh -i ec2-insecure-access.pem ec2-user@172.28.77.50
...
Last login: Sun Jun  1 20:01:35 2025 from 172.30.203.156
[ec2-user@localhost ~]$ 
```
## 9. Turn off the VMs
Turn off the VMs in Hyper-V

## 10. Zip up your template
Compress your template and call it something like `Golden-AL2023-Hyper-V-Base.zip`.

# Using the template
Now that you have a zip file with the golden template saved, you can deploy this template by unzipping, and importing. Note: you will still need to create a new directory to copy the VM into due to Hyper-V requiring the new VM have a unique ID. 

After importing the VM, you should rename it to something identifiable, and replace the insecure key with your own public SSH key.