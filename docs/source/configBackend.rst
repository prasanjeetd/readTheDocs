Config_Backend
===============


**Backend Deployment**

The backend is deployed on Amazon EC2. Here are the instructions to deploy a Node.js server on EC2 using Nginx and PM2:

1. **Create an EC2 Instance**

   - Go to the **EC2** service in the AWS Management Console.

   - Click on **Launch Instance**.

   - Choose an Amazon Machine Image (AMI) (e.g., **Ubuntu Server 20.04 LTS**).

   - Select an instance type (e.g., **t2.micro** for basic use).

   - Create an key pair to connect for SSH access.

   - Configure security group rules to allow SSH (port 22), HTTP (port 80), and HTTPS (port 443) traffic.

   - Review and launch the instance.

   - Download the key pair (.pem file) for SSH access.



2. **Connect to the EC2 Instance**

  To connect to the instance, go to the **Connect** tab in the AWS Management Console. You can connect using either EC2 Instance Connect or an SSH client.

**For Ubuntu or macOS Users:**
  
  1. Open an SSH client.
  
  2. Locate your private key file (e.g., `awskey.pem`).
  
  3. Run the following command to ensure your key is not publicly viewable:
  
     .. code-block:: console
  
        $ chmod 400 "awskey.pem"
  
  4. Connect to your instance using its Public DNS:
  
     .. code-block:: console
  
        $ ssh -i "awskey.pem" ubuntu@ec2-your-public-dns

**For Windows Users (Using PowerShell):**

  1. Open PowerShell.
  
  2. Navigate to the directory where your key file (`awskey.pem`) is stored.
  
  3. Run the following commands to configure the key permissions:
  
     .. code-block:: powershell
  
        $ awskey.pem /reset
        $ awskey.pem /grant:r "$($env:username):(r)"
        $ awskey.pem /inheritance:r
  
  4. Connect to your instance using its Public DNS:
  
     .. code-block:: powershell
  
        ssh -i "awskey.pem" ubuntu@ec2-your-public-dns



2. **Connect to the EC2 Instance**

1. Configure UFW Firewall


Ubuntu 20.04 servers can use the UFW (Uncomplicated Firewall) to ensure only connections to certain services are allowed. Here’s how to set up a basic firewall using UFW:

   .. note::
   
       If your servers are running on DigitalOcean, you can optionally use DigitalOcean Cloud Firewalls instead of UFW. It is recommended to use only one     firewall at a time to avoid conflicting rules that may be difficult to debug.

  1. **Check Available Applications**
  
     Applications can register their profiles with UFW upon installation. These profiles allow UFW to manage these applications by name. OpenSSH, the service allowing you to connect to your server, has a profile registered with UFW. To see this, type:
  
     .. code-block:: console
  
        $ ufw app list
  
     **Output:**
  
     .. code-block::
  
        Available applications:
          OpenSSH
  
  2. **Allow SSH Connections**
  
     To ensure that the firewall allows SSH connections so you can log back in next time, allow these connections by typing:
  
     .. code-block:: console
  
        $ ufw allow OpenSSH
  
  3. **Enable the Firewall**
  
     Enable the firewall by typing:
  
     .. code-block:: console
  
        $ ufw enable
  
     Type `y` and press **ENTER** to proceed.
  
  4. **Verify Firewall Status**
  
     To confirm that SSH connections are still allowed and check the firewall status, type:
  
     .. code-block:: console
  
        $ ufw status
  
     **Output:**
  
     .. code-block::
  
        Status: active
  
        To                         Action      From
        --                         ------      ----
        OpenSSH                    ALLOW       Anywhere
        OpenSSH (v6)               ALLOW       Anywhere (v6)
  














