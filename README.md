# Ansible environment deployment on Proxmox

This repository provides instructions and scripts for deploying Ansible environment by Proxmox Virtual Environment (Proxmox VE) on a bare metal machine. Ansible environment contains 1 control node virtual machine that sends request to hosts & 2 target host machines that executes ansible commands. Proxmox VE is an open-source virtualization platform that combines two virtualization technologies: KVM (Kernel-based Virtual Machine) for virtual machines and LXC (Linux Containers) for lightweight container-based virtualization.

## Prerequisites

Before starting the deployment, make sure your bare metal machine meets the following requirements:

- A 64-bit processor with virtualization support (Intel VT-x or AMD-V)
- Sufficient RAM for your virtualization needs
- Adequate storage for hosting 3 virtual machines

My machine is HP T630 with:

- 6GB RAM
- 512 GB NVME disk
- CPU AMD GX-420GI 4x2.0 Ghz

![image](https://github.com/damian-andrzej/ansible_env_by_proxmox/assets/102800704/f6f1f2ee-d5e1-4a5c-b44b-07a5079883b5)


## Proxmox Installation Steps

Follow these steps to deploy Proxmox on your bare metal machine:

1. **Download Proxmox VE ISO:**
   - Visit the [Proxmox download page](https://www.proxmox.com/proxmox-ve) and download the latest Proxmox VE ISO.

2. **Create Bootable USB or CD:**
   - Use a tool like Rufus (for Windows) or dd (for Linux) to create a bootable USB or burn the ISO to a CD.

3. **Boot from Installation Media:**
   - Insert the bootable USB or CD into your bare metal machine.
   - Boot from the installation media.

4. **Proxmox VE Installation:**
   - Connect server to your LAN via Ethernet cable.
     Its most reliable connection way, by default proxmox prohibit wirelles WiFi connections
   - Follow the on-screen instructions to install Proxmox VE.
   - This time storage and network settings may be default
     Depending on your LAN settings, Proxmox IP must be corresponding to your subnet. In most cases its 192.168.1.0 network.
     My addess is 192.168.1.21/24
     Try to not set the same IP with another machine working your home, it will disconnect one of them or even both devices from network.

5. **Access Proxmox Web Interface:**
   - After the installation, access the Proxmox web interface by navigating to `https://<your-server-ip>:8006` in a web browser.
   - Log in with the credentials you set during the installation.

6. **Configuration and Virtualization:**
   - Configure additional settings based on your requirements.
   - Start creating virtual machines (VMs) and containers to meet your virtualization needs.
     
     6.1 Deploying Ansible controler
     
       Choose your own OS that corresponds your preferences. I choose Debian 10
       - Right click on your Proxmox controller then from menu select "Create VM"
     ![image](https://github.com/damian-andrzej/ansible_env_by_proxmox/assets/102800704/c3ebb767-bb6a-4a9d-9cbe-d5412e7cf5c6)
      - Choose OS depending on your preferences : Iso may be downloaded locally or by Proxmox images feature,
        more information here - https://www.proxmox.com/en/downloads/proxmox-virtual-environment/documentation

     - Disc space shouldnt be less than 32 GB for ansible control node. 2GB RAM is minimum, 4 GB may be consider. Theorethically its going to work but we'll feel a lot of limitation when our environment increase
       ![image](https://github.com/damian-andrzej/ansible_env_by_proxmox/assets/102800704/9fc05821-88ab-4751-8d2f-d298fb88e978)

     6.2 Deploying target hosts

     Procedure is simmilar for target hosts, but our requirements decreases. 1 GB Ram and 20 G of storage are sufficient to answer on control node requests.
     By default Proxmox creates machines in the same virtual network so they will be visable for each other. Below we see my setup for 2 target nodes
  
     Control node : 192.168.1.51
     
     Target host 1: 192.168.1.52
     
     Target host 2: 192.168.1.54 (not .53 because its reserved by another project already)
     
     ![image](https://github.com/damian-andrzej/ansible_env_by_proxmox/assets/102800704/da4574c0-16fe-4f2b-9007-87e60fbe6416)


7. **Ansible configuration on control node:**

Step 0: Create ansible user. Specify your name and home path fpr your user that gonna operates  as ansible user 
useradd -m -d /home/ansible/ ansible

Step 1: Install Ansible on the Control Node
On your control node, execute the following commands:

 Update package list
sudo apt update

 Install Ansible
sudo apt install ansible

Step 2: Generate SSH Key Pair on the Control Node
Generate an SSH key pair on the control node:

 Generate SSH key pair
ssh-keygen
Follow the prompts, or press Enter to accept the defaults.

Step 3: Copy SSH Public Key to Target Hosts
Copy the SSH public key to both target hosts:

 Replace USER and HOST with your target host's username and IP address
ssh-copy-id USER@HOST
Repeat this step for the second target host.

Step 4: Test SSH Connectivity
Ensure you can SSH from the control node to both target hosts without entering a password:
If you got any issues connecting via SSH, check /etc/ssh/sshd_config file, here are most prone lines you should check.

for password auth
![image](https://github.com/damian-andrzej/ansible_env_by_proxmox/assets/102800704/7bc41149-7c49-493e-9866-e479861892ef)

for public key auth
![image](https://github.com/damian-andrzej/ansible_env_by_proxmox/assets/102800704/fea5d0c3-f22c-4b38-af54-92c0ab208578)

for executing ansible commands as root (not recommended)
![image](https://github.com/damian-andrzej/ansible_env_by_proxmox/assets/102800704/bace89e0-75cc-4cd2-b69f-65fcbdbe988d)




 Replace USER and HOST with your target host's username and IP address
ssh USER@HOST
Step 5: Create Ansible Inventory
Create an Ansible inventory file (e.g., inventory.ini) to define your hosts. Replace the placeholder values with your actual host information:

[targets]
host1 ansible_host=IP_ADDRESS_1
host2 ansible_host=IP_ADDRESS_2

[targets:vars]
ansible_user=YOUR_SSH_USERNAME

here my example of inventory file

![image](https://github.com/damian-andrzej/ansible_env_by_proxmox/assets/102800704/bf5dfe78-ea7d-4f49-90f5-a664c018a02e)


Step 6: Test Ansible Connectivity
Test Ansible connectivity to your hosts:

ansible -i inventory.ini all -m ping

![image](https://github.com/damian-andrzej/ansible_env_by_proxmox/assets/102800704/f12a82cc-0b5a-460b-9a80-f2aac3bc35c9)

it pings and check if ansible can connect to all machines declared into inventory file

If you authenticate by password instead of keygen (not recommended) 
 

    

## Customization

Feel free to customize the deployment based on your specific requirements. You may want to:

- Automate the installation process using configuration management tools like Ansible.
- Secure your Proxmox installation by configuring firewall rules and using HTTPS.
- Explore Proxmox documentation for advanced configurations and features.

## What next?

The purpose of this lab is create bare metal training environment for ansible. Its the simplest config to run our playbooks. In next chapter we will focus on imporving security and usability aspects. 

