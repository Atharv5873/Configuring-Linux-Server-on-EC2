# Configuring-Linux-Server-on-EC2

## Steps Performed:

### Step 1: Lanch EC2 Instance
- **AMI Used**: Ubuntu Server 24.04 LTS
- **Instance Type**: t3.micro
- **Key Pair**: Used my own SSH key for authentication, insted of renerating a new AWS Key Pair, stored in `~/.ssh/id_rsa`
- **Security Group Rules**:
  - SSH (port 22) open to my IP
  - HTTP (port 80) open to world
- **Updates & Upgrades**:
  ```bash
  sudo apt update && sudo apt upgrade -y
  sudo passwd #set password for your user


  

