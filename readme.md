# Wazuh Alert System for Failed Login Attempts

**Tags**: #SIEM #Wazuh #Cybersecurity #SOC #BlueTeam

> [!NOTE]  
> This project sets up Wazuh, an open-source SIEM, to monitor failed login attempts on an Ubuntu VM, creating custom alerts and visualizing them in a dashboard. It’s designed for your GitHub portfolio to land SOC Tier 1 or Analyst roles in Malaysia.


## Step-by-Step Guide

> [!TIP]  
> Follow these steps in order. After each step, check progress and ask if you need clarification. I’ll troubleshoot any issues.

### Step 1: Set Up the Ubuntu VM

**Why**: The VM hosts Wazuh and generates login logs, simulating a SOC system.

1. **Open VMware Workstation Pro** on your host system.
2. **Create a New VM**:
    - Select “New Virtual Machine” > Typical > “Installer disc image file (iso)”.
    - Use the Ubuntu 22.04 LTS ISO.
    - Allocate:
        - RAM: 4GB
        - CPU: 2 cores
        - Disk: 40GB (store on SSD)
    - Network: NAT (for internet).
    - Name: “Wazuh-Ubuntu”.
3. **Install Ubuntu**:
    - Follow the installer, select “Minimal installation”.
    - Set:
        - Username: `user`
        - Password: `password123`
    - Enable SSH (check “Install OpenSSH server”).
4. **Update Ubuntu**:
    - Log in to the VM.
    - Open terminal (Ctrl+Alt+T).
    - Run:
        
        ```bash
        sudo apt update && sudo apt upgrade -y
        ```
        
    - **Why**: Keeps the system ready for Wazuh.

> [!NOTE]  
> If the ISO download is slow, try a mirror from ubuntu.com or let me know for alternatives.

### Step 2: Install Wazuh All-in-One Deployment

**Why**: Wazuh’s all-in-one setup simplifies SIEM deployment with server, agent, and dashboard.

1. **Check Requirements**:
    - In Ubuntu terminal, verify resources:
        
        ```bash
        free -h
        df -h
        ```
        
    - **Why**: Confirms 4GB RAM and 20GB disk for Wazuh.
2. The All-in-One command where: 

- Wazuh Manager
    
- Wazuh Indexer
    
- Wazuh Dashboard

Step 1: Download the installation script
Code:
```shell
curl -sO https://packages.wazuh.com/4.7/wazuh-install.sh

```

Step 2: Run the installation script (all-in-one)
```shell
bash wazuh-install.sh -a

```

If step 2 does not works then use this

```shell
sudo bash wazuh-install.sh -a --overwrite

```

After giving this command it will take some moment and in last there will be username and password. You need to save those credentials for login in Wazuh.

![alt text](image.png)

This will:

- Remove previously installed components (like Filebeat)
    
- Clean up old configs
    
- Do a fresh install of:
    
    - Wazuh Manager
        
    - Wazuh Indexer
        
    - Wazuh Dashboard

After that we need to do 
```shell
ip a
```

For checking which ip is working. It will show like this

![alt text](image-1.png)

Need to copy the inet 192.168.246.140 and use this in a browser like this

```https
https://192.168.246.140/
```

Now we need to see how failed attempts work:

### **Step 3: Install and Configure Wazuh Manager (If Not Already Done)**

You already have **Wazuh Manager** and **Wazuh Dashboard** installed, but let’s confirm everything is set up:

1. Verify the Wazuh Manager is Running: Run the following command to ensure that Wazuh Manager is running:
```bash
sudo systemctl status wazuh-manager
```

It should show as active (running). If it is not running, you can start it with:

```bash
sudo systemctl start wazuh-manager

```

2. **Check the Wazuh Dashboard**: Ensure that the **Wazuh Dashboard** is running by checking:
```bash
sudo systemctl status wazuh-dashboard

```

You can access the Wazuh Dashboard by opening your browser and going to:

```http
http://<your_server_ip>:5601

```

### **Step 4: Configure Wazuh Manager to Monitor Logs**

Since you’re not using the Wazuh Agent, you will configure the **Wazuh Manager** to directly monitor log files on your **Ubuntu system**.

1. **Edit `ossec.conf`** to Monitor `/var/log/auth.log` (failed login attempts):
    
    Open the **ossec.conf** file in the Wazuh Manager’s configuration directory:
    ```bash
    sudo nano /var/ossec/etc/ossec.conf

```

2. **Add Log Monitoring Entry**: Scroll down to find the `<localfile>` section, or add the following block to the end of the file:

```bash
<localfile>
  <log_format>syslog</log_format>
  <location>/var/log/auth.log</location>
</localfile>

```

this need to add in the end of the code in the localfile section
This configuration tells Wazuh to monitor **/var/log/auth.log**, where Ubuntu logs authentication (login) attempts, including failed logins.
    
3. **Save and Exit**:
    
    - After editing the file, press **Ctrl+O** to save and **Ctrl+X** to exit.
        
4. **Restart Wazuh Manager**: Restart Wazuh Manager for the changes to take effect:

```bash
sudo systemctl restart wazuh-manager

```

### Step 5: Simulate Failed Login Attempts (For Testing)

#### 1. **Ensure SSH Server is Installed and Running**

Make sure `OpenSSH` is installed and active:

```bash
sudo apt update
sudo apt install openssh-server -y
sudo systemctl start ssh
sudo systemctl enable ssh

```

#### 2. **Find Your Machine’s IP Address**

Run:
```bash
ip a
```

Look for your IP address (e.g., `192.168.1.100`)

#### 3. **Attempt SSH Login as a Fake User**

Now, from the **same machine** or another computer (or even within the same terminal), run:

```bash

ssh fakeuser@localhost
```

After entering we need to give wrong password or just enter like this

![alt text](image-2.png)

### 🧪 Step 4: Check If Logs Were Created

Now run:

```bash
sudo tail -f /var/log/auth.log

```


and by doing that it will show in the Wazuh Dashboard

we need to go in 

![alt text](image-3.png)

after clicking Security Events it will show like this:

![alt text](image-4.png)

there we can use filter for knowing which users do what like this

![alt text](image-5.png)

after selecter fakeuser it will show how attempts they do

![alt text](image-6.png)


I will update this after some times. But this will help to progress in further analysis.