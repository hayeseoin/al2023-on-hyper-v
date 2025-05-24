# AL2023 on Hyper V

AL2023 base image can be found at: https://cdn.amazonlinux.com/al2023/os-images/latest/

Use the following steps to generate a seed iso for Hyper V bootable version of AL2023

1. Copy user-data.template to user-data and replace __SSH_KEY__ with your actual ssh pubic key.

To set username and password, uncomment out the passwd line in user-data


2. Run this command

```sh
genisoimage -output seed.iso -volid cidata -joliet -rock user-data meta-data
```

Username and password: ec2-user/password

- Optional

You can automatically add your own SSH key with these commands

```sh
export SSH_KEY="$(cat ~/.ssh/id_rsa.pub)"

sed \
  -e "s|__SSH_KEY__|$SSH_KEY|" \
  user-data.template > user-data

genisoimage -output seed.iso -volid cidata -joliet -rock user-data meta-data

```

4. Create a directory for your seed.iso file and the Hyper V template from Amazon e.g. C:\Users\<your_user>\hyperv\al2023-base

Copy the seed.iso file to this location.

Download the Hyper V disk and save it and extract it to here. The format should be VHDX. It may be in a nester folder.

5. First we need to create the VM, then we must configure Hyper V settings:
In Hyper V Create the VM:
- New -> Virtual Machine
- Name: al2023-base
- Generation 2
- Assign memory: 2048 MB
- Configure Networking: Default Switch
- Use an existing Virtual hard disk: Choose the VHDX file you extracted in step 4.
- Create virtual machine

6. Configure Hyper V
Navigate to the VM's settings. 
- Security: Uncheck 'Enable secure boot' (AL2023 does not support Hyper V secure boot)
- SCSI Controller: 
    - Add a DVD drive
    - Image file: The seed.iso file you created i.e C:\Users\<your_user>\hyperv\al2023-base.local\seed.iso
    - Optional: Disable checkpoints if you don't want them.

7. Start the VM. The IP address should show up in the Networking tab of the VM in Hyper V. You can update your hosts file with this. This is not static, it will reset when the computer resets.
![alt text](images/image.png)
I reccomend using Windows PowerToys GUI for the hosts file 
![alt text](images/image-1.png)

8. SSH into the machine
```sh
ssh ec2-user@al2023-base.local
```
Notes: If you're using WSL, you will need to run the following Powershell command to reach Hyper V from WSL. This is not persistent config and will need to be run every time you restart your computer.
```ps
Get-NetIPInterface | where {$_.InterfaceAlias -eq 'vEthernet (WSL (Hyper-V firewall))' -or $_.InterfaceAlias -eq 'Default Switch'} | Set-NetIPInterface -Forwarding Enabled -Verbose
``