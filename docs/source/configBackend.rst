Config_Backend
===============


**Backend Deployment**
-----------------------


The backend is deployed on Amazon EC2. Here are the instructions to deploy a Node.js server on EC2 using Nginx and PM2:

1. **Create an EC2 Instance**
----------------------------------

   - Go to the **EC2** service in the AWS Management Console.

   - Click on **Launch Instance**.

   - Choose an Amazon Machine Image (AMI) (e.g., **Ubuntu Server 20.04 LTS**).

   - Select an instance type (e.g., **t2.micro** for basic use).

   - Create an key pair to connect for SSH access.

   - Configure security group rules to allow SSH (port 22), HTTP (port 80), and HTTPS (port 443) traffic.

   - Review and launch the instance.

   - Download the key pair (.pem file) for SSH access.



2. **Connect to the EC2 Instance**
-------------------------------------

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



3. **Set-Up your Server**
----------------------------------

   1. **Configure UFW Firewall**
  
   
   
      Ubuntu 20.04 servers can use the UFW (Uncomplicated Firewall) to ensure only connections to certain services are allowed. Here’s how to set up a basic firewall using UFW:
   
               .. note::
               
                   If your servers are running on DigitalOcean, you can optionally use DigitalOcean Cloud Firewalls instead of UFW. It is recommended to use only one    firewall at a time to avoid conflicting rules that may be difficult to debug.
   
      1. Check Available Applications
      
        Applications can register their profiles with UFW upon installation. These profiles allow UFW to manage these applications by name. OpenSSH, the service allowing you to connect to your server, has a profile registered with UFW. To see this, type:
      
        .. code-block:: console
      
           $ ufw app list
      
        **Output:**
      
        .. code-block::
      
           Available applications:
             OpenSSH
      
      2. Allow SSH Connections
      
        To ensure that the firewall allows SSH connections so you can log back in next time, allow these connections by typing:
      
        .. code-block:: console
      
           $ ufw allow OpenSSH
      
      3. Enable the Firewall
      
        Enable the firewall by typing:
      
        .. code-block:: console
      
           $ ufw enable
      
        Type `y` and press **ENTER** to proceed.
      
      4. Verify Firewall Status
      
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
   
   
   2. **Install and Configure Nginx**
   
   
   
   *Step 1 – Installing Nginx*
   
   Because Nginx is available in Ubuntu’s default repositories, you can install it using the `apt` packaging system. 
   
   1. Update the local package index to ensure you have the most recent package listings:
   
      .. code-block:: console
   
         $ sudo apt update
   
   2. Install Nginx:
   
      .. code-block:: console
   
         $ sudo apt install nginx
   
      After accepting the procedure, `apt` will install Nginx and any required dependencies to your server.
   
   *Step 2 – Adjusting the Firewall*
   
   Before testing Nginx, adjust the firewall software to allow access to the service. Nginx registers itself as a service with UFW upon installation, making it straightforward to allow Nginx access.
   
   1. List the application configurations that UFW knows how to work with:
   
      .. code-block:: console
   
         $ sudo ufw app list
   
      **Output:**
   
      .. code-block::
   
         Available applications:
           Nginx Full
           Nginx HTTP
           Nginx HTTPS
           OpenSSH
   
      There are three profiles available for Nginx:
   
      - **Nginx Full**: Opens both port 80 (normal, unencrypted web traffic) and port 443 (TLS/SSL encrypted traffic).
      - **Nginx HTTP**: Opens only port 80 (normal, unencrypted web traffic).
      - **Nginx HTTPS**: Opens only port 443 (TLS/SSL encrypted traffic).
   
      It is recommended to enable the most restrictive profile that will still allow the traffic you’ve configured. For now, we will only need to allow traffic on port 80.
   
   2. Allow HTTP traffic by typing:
   
      .. code-block:: console
   
         $ sudo ufw allow 'Nginx HTTP'
   
   3. Verify the change by typing:
   
      .. code-block:: console
   
         $ sudo ufw status
   
      **Output:**
   
      .. code-block::
   
         Status: active
   
         To                         Action      From
         --                         ------      ----
         OpenSSH                    ALLOW       Anywhere                  
         Nginx HTTP                 ALLOW       Anywhere                  
         OpenSSH (v6)               ALLOW       Anywhere (v6)             
         Nginx HTTP (v6)            ALLOW       Anywhere (v6)
   
   *Step 3 – Checking Your Web Server*
   
   At the end of the installation process, Ubuntu 20.04 starts Nginx. The web server should already be up and running.
   
   1. Check with the `systemd` init system to make sure the service is running:
   
      .. code-block:: console
   
         $ systemctl status nginx
   
      **Output:**
   
      .. code-block::
   
         ● nginx.service - A high performance web server and a reverse proxy server
            Loaded: loaded (/lib/systemd/system/nginx.service; enabled; vendor preset: enabled)
            Active: active (running) since Fri 2020-04-20 16:08:19 UTC; 3 days ago
              Docs: man:nginx(8)
          Main PID: 2369 (nginx)
             Tasks: 2 (limit: 1153)
            Memory: 3.5M
            CGroup: /system.slice/nginx.service
                    ├─2369 nginx: master process /usr/sbin/nginx -g daemon on; master_process on;
                    └─2380 nginx: worker process
   
      This confirms that the service has started successfully.
   
   2. Test Nginx by requesting a page:
   
      Access the default Nginx landing page by navigating to your server’s IP address. If you do not know your server’s IP address, you can find it using the following command:
   
      .. code-block:: console
   
         $ curl -4 icanhazip.com
   
      When you have your server’s IP address, enter it into your browser’s address bar:
   
      .. code-block::
   
         http://your_server_ip
   
      You should receive the default Nginx landing page.
   
      .. image:: images/nginx.png
         :alt: Description of the image
         :width: 800px
         :height: 150px
         :align: center

   *Step 4 – Managing the Nginx Process*


Now that you have your web server up and running, let’s review some basic management commands.

1. Stop the Web Server

   To stop your web server, type:

   .. code-block:: console

      $ sudo systemctl stop nginx

2. Start the Web Server

   To start the web server when it is stopped, type:

   .. code-block:: console

      $ sudo systemctl start nginx

3. Restart the Web Server

   To stop and then start the service again, type:

   .. code-block:: console

      $ sudo systemctl restart nginx

4. Reload the Configuration

   If you are only making configuration changes, Nginx can often reload without dropping connections. To reload Nginx, type:

   .. code-block:: console

      $ sudo systemctl reload nginx

5. Disable Automatic Start at Boot

   By default, Nginx is configured to start automatically when the server boots. If you do not want this behavior, you can disable it by typing:

   .. code-block:: console

      $ sudo systemctl disable nginx

6. Re-enable Automatic Start at Boot

   To re-enable the service to start up at boot, type:

   .. code-block:: console

      $ sudo systemctl enable nginx


 3. **Installing Node.js**


















