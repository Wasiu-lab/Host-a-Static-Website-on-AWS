# Host a Static Website on AWS

## Project Overview

This project demonstrates how to host a static website on Amazon Web Services (AWS) using a scalable, secure, and production-ready architecture. The deployment includes EC2 instances, load balancing, and automated content delivery from S3.

## Reference Architecture

![Reference Architecture](reference%20architecture.png)

## Architecture Components

### Network Infrastructure
- **VPC (Virtual Private Cloud)**: Isolated network environment
- **Public Subnet**: Hosts the bastion host for secure access
- **Private Subnet**: Contains web servers for enhanced security
- **Internet Gateway**: Enables internet connectivity
- **NAT Gateway**: Allows outbound traffic from private resources

### Compute Resources
- **EC2 Instances**: 
  - Bastion Host (Public Subnet): Jump server for SSH access to private instances
  - Web Servers (Private Subnet): Host the static website content
- **Auto Scaling Group**: Automatically scales instances based on demand
- **Application Load Balancer (ALB)**: Distributes traffic across web servers

### Storage
- **Amazon S3 Bucket**: Stores the website files (jupiter.zip)
- **EC2 Root Volume**: Stores deployed website content

### Security
- **Security Groups**: 
  - Control inbound/outbound traffic for bastion and web servers
  - ALB security group allows HTTP/HTTPS from the internet
  - Web server security group allows traffic only from ALB

## Deployment Steps

### 1. Prepare Your SSH Key

**Windows PowerShell:**

```powershell
# Move the key to the Home directory
move "$HOME\Downloads\dev-keypair.pem" "$HOME\dev-keypair.pem"

# Verify if the key exists in the Home directory
Test-Path "$HOME\dev-keypair.pem"
```

### 2. Deploy Infrastructure

Run the deployment script to set up all AWS resources:

```bash
./deployment script.sh
```

### 3. Access Your Website

#### Connect to EC2 in Public Subnet (Bastion Host)

```powershell
ssh -i C:\path\to\your-key.pem ec2-user@<public-ipv4-of-bastion-host>
```

#### Connect to EC2 in Private Subnet via Bastion

**Step 1: Configure SSH Agent (Windows PowerShell)**

```powershell
# Check if ssh-agent is running
Get-Service ssh-agent

# If the ssh-agent service is not running, set it to automatic and start it
Set-Service -Name ssh-agent -StartupType Automatic
Start-Service ssh-agent

# Load your key file into the ssh-agent
ssh-add C:\path\to\your-key.pem
```

**Step 2: SSH to Bastion with Agent Forwarding**

```powershell
ssh -A -i C:\path\to\your-key.pem ec2-user@<public-ipv4-of-bastion-host>
```

**Step 3: From Bastion, SSH to Private Web Server**

```bash
ssh ec2-user@<private-ipv4-of-the-web-server>
```

## Deployment Automation

The deployment script (`deployment script.sh`) automates the following tasks:

```bash
#!/bin/bash

# Create an environment variable for the S3 zip file
export S3_URI="s3://dev-abdul-app-webfiles-11-12-25-14-25/jupiter.zip"

# Update the packages on the EC2 instance
sudo yum update -y

# Install the Apache HTTP Server
sudo yum install -y httpd

# Change to the Apache web root directory
cd /var/www/html

# Remove any existing files
sudo rm -rf *

# Download the zip file from the S3 bucket
sudo aws s3 cp "$S3_URI" .

# Unzip the downloaded file
sudo unzip jupiter.zip

# Copy the contents to the html directory
sudo cp -R jupiter/. .

# Clean up zip file and extracted folder
sudo rm -rf jupiter jupiter.zip

# Enable and start Apache service
sudo systemctl enable httpd
sudo systemctl start httpd
```

### Script Workflow:
1. Updates system packages
2. Installs Apache HTTP Server (httpd)
3. Downloads website files from S3 bucket
4. Extracts and deploys files to web root
5. Starts Apache service

## Key Features

✅ **High Availability**: Multi-AZ deployment with load balancing  
✅ **Security**: Private subnets for web servers, bastion host for access  
✅ **Scalability**: Auto Scaling Group handles traffic spikes  
✅ **Automated Deployment**: One-click deployment script  
✅ **Content Distribution**: S3-based file storage and delivery  

## Configuration Variables

- **S3 URI**: `s3://dev-abdul-app-webfiles-11-12-25-14-25/jupiter.zip`
- **Website Package**: `jupiter.zip`
- **Web Server Port**: 80 (HTTP)
- **EC2 User**: `ec2-user`
- **Web Root Directory**: `/var/www/html`

## Prerequisites

- AWS Account with appropriate IAM permissions
- SSH key pair created in AWS (`.pem` file)
- Windows PowerShell or Linux terminal
- AWS CLI configured on EC2 instances

## Troubleshooting

### Cannot Connect to Private EC2
- Ensure SSH agent forwarding is enabled (`-A` flag)
- Verify security groups allow SSH (port 22) from bastion to web server
- Check NAT Gateway routing in private subnet

### Website Not Loading
- Verify S3 bucket permissions and file availability
- Check Apache service status: `sudo systemctl status httpd`
- Review deployment script logs in `/var/log/cloud-init-output.log`

### SSH Connection Timeout
- Confirm bastion host security group allows inbound SSH (port 22)
- Verify internet connectivity and route tables
- Check if public IP address is correctly assigned

## Files in This Repository

- `README.md` - Project documentation
- `deployment script.sh` - Automated deployment script
- `reference architecture.png` - Architecture diagram
- `ssh-commands (windows powershell).txt` - SSH connection examples
- `move-ssh-key (windows powershell).md` - SSH key setup guide
- `jupiter.zip` - Website content archive
- `Images/` - Supporting screenshots and diagrams
