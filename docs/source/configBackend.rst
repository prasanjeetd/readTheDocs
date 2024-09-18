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
  
          chmod 400 "awskey.pem"
  
  4. Connect to your instance using its Public DNS:
  
     .. code-block:: console
  
          ssh -i "awskey.pem" ubuntu@ec2-your-public-dns

**For Windows Users (Using PowerShell):**

  1. Open PowerShell.
  
  2. Navigate to the directory where your key file (`awskey.pem`) is stored.
  
  3. Run the following commands to configure the key permissions:
  
     .. code-block:: powershell
  
          icacls.exe awskey.pem /reset
          icacls.exe awskey.pem /grant:r "$($env:username):(r)"
          icacls.exe awskey.pem /inheritance:r
  
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
      
             sudo ufw app list
      
        **Output:**
      
        .. code-block::
      
           Available applications:
             OpenSSH
      
      2. Allow SSH Connections
      
        To ensure that the firewall allows SSH connections so you can log back in next time, allow these connections by typing:
      
        .. code-block:: console
      
             sudo ufw allow OpenSSH
      
      3. Enable the Firewall
      
        Enable the firewall by typing:
      
        .. code-block:: console
      
             sudo ufw enable
      
        Type `y` and press **ENTER** to proceed.
      
      4. Verify Firewall Status
      
        To confirm that SSH connections are still allowed and check the firewall status, type:
      
        .. code-block:: console
      
             sudo ufw status
      
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
      
              sudo apt update
      
      2. Install Nginx:
      
         .. code-block:: console
      
              sudo apt install nginx
      
         After accepting the procedure, `apt` will install Nginx and any required dependencies to your server.
      
      *Step 2 – Adjusting the Firewall*
      
      Before testing Nginx, adjust the firewall software to allow access to the service. Nginx registers itself as a service with UFW upon installation, making it straightforward to allow Nginx access.
      
      1. List the application configurations that UFW knows how to work with:
      
         .. code-block:: console
      
              sudo ufw app list
      
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
      
              sudo ufw allow 'Nginx HTTP'
      
      3. Verify the change by typing:
      
         .. code-block:: console
      
              sudo ufw status
      
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
      
              systemctl status nginx
      
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
      
              curl -4 icanhazip.com
      
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
      
              sudo systemctl stop nginx
      
      2. Start the Web Server
      
         To start the web server when it is stopped, type:
      
         .. code-block:: console
      
              sudo systemctl start nginx
      
      3. Restart the Web Server
      
         To stop and then start the service again, type:
      
         .. code-block:: console
      
              sudo systemctl restart nginx
      
      4. Reload the Configuration
      
         If you are only making configuration changes, Nginx can often reload without dropping connections. To reload Nginx, type:
      
         .. code-block:: console
      
              sudo systemctl reload nginx
      
      5. Disable Automatic Start at Boot
      
         By default, Nginx is configured to start automatically when the server boots. If you do not want this behavior, you can disable it by typing:
      
         .. code-block:: console
      
              sudo systemctl disable nginx
      
      6. Re-enable Automatic Start at Boot
      
         To re-enable the service to start up at boot, type:
      
         .. code-block:: console
      
              sudo systemctl enable nginx
      
      
    3. **Installing Node.js**
   
     
   1. **Install Node.js*
      1. *Install NodeSource PPA*
      
         First, install the NodeSource PPA to access its contents. Make sure you’re in your home directory, and use `curl` to retrieve the installation script for the most recent LTS version of Node.js from its archives:
      
         .. code-block:: console
      
              cd ~
              curl -sL https://deb.nodesource.com/setup_14.x -o nodesource_setup.sh
      
         You can inspect the contents of this script with `nano` or your preferred text editor:
      
         .. code-block:: console
      
              nano nodesource_setup.sh
      
      2. *Run the Installation Script*
      
         After inspecting the script, run it under `sudo`:
      
         .. code-block:: console
      
              sudo bash nodesource_setup.sh
      
         The PPA will be added to your configuration, and your local package cache will be updated automatically.
      
      3. *Install Node.js*
      
         After running the setup script from NodeSource, install the Node.js package:
      
         .. code-block:: console
      
            $ sudo apt install nodejs
      
      4. *Verify Node.js Installation*
      
         To check which version of Node.js you have installed after these initial steps, type:
      
         .. code-block:: console
      
              node -v
            Output: v14.4.0
      
         **Note**: When installing from the NodeSource PPA, the Node.js executable is called `nodejs`, rather than `node`.
      
      5. *Verify npm Installation*
      
         The `nodejs` package contains the Node.js binary as well as `npm`, a package manager for Node modules, so you don’t need to install `npm` separately. Execute this command to verify that `npm` is installed and to create the configuration file:
      
         .. code-block:: console
      
              npm -v
            Output: 6.14.5
      
      6. *Install Build Tools*
      
         In order for some `npm` packages to work (those that require compiling code from source, for example), install the `build-essential` package:
      
         .. code-block:: console
      
              sudo apt install build-essential
      
      You now have the necessary tools to work with `npm` packages that require compiling code from source.
      
      With the Node.js runtime installed, let’s move on to writing a Node.js application.

   1. **Creating a Sample Application or Cloning a Project*


1. **Create a Sample Application**

   To create a simple Node.js application, first navigate to your home directory and then create a new file using `nano` or your preferred text editor:

   .. code-block:: console

        cd ~
        nano hello.js

   Add the following code to `hello.js`:

   .. code-block:: javascript

      console.log('Hello, World!');

   Save and close the file. You now have a basic Node.js application.

2. **Clone an Existing Project**

   Alternatively, you can clone an existing project from a repository using `git`. Replace `httpsurl` with the actual URL of the repository:

   .. code-block:: console

        git clone https://github.com/username/repository.git

   This will download the project files into a new directory.

      
      
      Step 3 — Installing PM2
========================

Next, let’s install PM2, a process manager for Node.js applications. PM2 makes it possible to daemonize applications so that they will run in the background as a service.

1. **Install PM2**

   Use `npm` to install the latest version of PM2 on your server:

   .. code-block:: console

      sudo npm install pm2@latest -g

   The `-g` option tells npm to install the module globally, so that it’s available system-wide.

2. **Run Your Application with PM2**

   Let’s use the `pm2 start` command to run your application, `hello.js`, in the background:

   .. code-block:: console

      pm2 start hello.js

   This also adds your application to PM2’s process list, which is outputted every time you start an application:

   .. code-block:: console

      Output
      ...
      [PM2] Spawning PM2 daemon with pm2_home=/home/sammy/.pm2
      [PM2] PM2 Successfully daemonized
      [PM2] Starting /home/sammy/hello.js in fork_mode (1 instance)
      [PM2] Done.
      ┌────┬────────────────────┬──────────┬──────┬───────────┬──────────┬──────────┐
      │ id │ name               │ mode     │ ↺    │ status    │ cpu      │ memory   │
      ├────┼────────────────────┼──────────┼──────┼───────────┼──────────┼──────────┤
      │ 0  │ hello              │ fork     │ 0    │ online    │ 0%       │ 25.2mb   │
      └────┴────────────────────┴──────────┴──────┴───────────┴──────────┴──────────┘

   As indicated above, PM2 automatically assigns an App name (based on the filename, without the `.js` extension) and a PM2 id. PM2 also maintains other information, such as the PID of the process, its current status, and memory usage.

3. **Set Up PM2 to Start on Boot**

   Applications running under PM2 will be restarted automatically if the application crashes or is killed. However, to ensure the application launches on system startup, use the `startup` subcommand:

   .. code-block:: console

      pm2 startup systemd

   Run the command from the output, with your username in place of `sammy`:

   .. code-block:: console

      sudo env PATH=$PATH:/usr/bin /usr/lib/node_modules/pm2/bin/pm2 startup systemd -u sammy --hp /home/sammy

4. **Save the PM2 Process List**

   Save the PM2 process list and corresponding environments:

   .. code-block:: console

      pm2 save

   You have now created a systemd unit that runs PM2 for your user on boot. This PM2 instance, in turn, runs `hello.js`.

5. **Start the PM2 Service**

   Start the service with `systemctl`:

   .. code-block:: console

      sudo systemctl start pm2-sammy

   Check the status of the systemd unit:

   .. code-block:: console

      systemctl status pm2-sammy

   PM2 provides many subcommands that allow you to manage or look up information about your applications:

   - Stop an application:

     .. code-block:: console

        pm2 stop app_name_or_id

   - Restart an application:

     .. code-block:: console

        pm2 restart app_name_or_id

   - List the applications currently managed by PM2:

     .. code-block:: console

        pm2 list

   - Get information about a specific application using its App name:

     .. code-block:: console

        pm2 info app_name

   - Monitor application status, CPU, and memory usage:

     .. code-block:: console

        pm2 monit

Step 4 — Setting Up Nginx as a Reverse Proxy Server
===================================================

Your application is running and listening on `localhost`, but you need to set up a way for your users to access it. We will set up the Nginx web server as a reverse proxy for this purpose.

1. **Edit Nginx Configuration**

   Open your Nginx configuration file for editing:

   .. code-block:: console

      sudo nano /etc/nginx/sites-available/example.com

   Within the `server` block, you should have an existing `location /` block. Replace the contents of that block with the following configuration:

   .. code-block:: nginx

      server {
      ...
          location / {
              proxy_pass http://localhost:3000;
              proxy_http_version 1.1;
              proxy_set_header Upgrade $http_upgrade;
              proxy_set_header Connection 'upgrade';
              proxy_set_header Host $host;
              proxy_cache_bypass $http_upgrade;
          }
      ...
      }

   This configures the server to respond to requests at its root. Assuming our server is available at `example.com`, accessing `https://example.com/` via a web browser would send the request to `hello.js`, listening on port `3000` at `localhost`.

2. **Optional: Add Additional Location Blocks**

   You can add additional `location` blocks to the same `server` block to provide access to other applications on the same server:

   .. code-block:: nginx

      server {
      ...
          location /app2 {
              proxy_pass http://localhost:3001;
              proxy_http_version 1.1;
              proxy_set_header Upgrade $http_upgrade;
              proxy_set_header Connection 'upgrade';
              proxy_set_header Host $host;
              proxy_cache_bypass $http_upgrade;
          }
      ...
      }

   This block allows access to another application running on port `3001` via `https://example.com/app2`.

3. **Test Nginx Configuration**

   Make sure you didn’t introduce any syntax errors by typing:

   .. code-block:: console

      sudo nginx -t

4. **Restart Nginx**

   Restart Nginx to apply the changes:

   .. code-block:: console

      sudo systemctl restart nginx











