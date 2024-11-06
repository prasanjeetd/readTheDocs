Enabling HTTPS with Load Balancer and Route 53
=============================================

After deploying your Next.js application, you can set up HTTPS using a load balancer and Route 53 for a secure connection with a custom domain.

Step 1: Create a Load Balancer Listener for HTTPS
-------------------------------------------------
   1. Go to **EC2** in the AWS Console.
   2. Under **Load Balancers**, select the load balancer associated with your ECS service.
   3. Choose **Listeners** and click **Add listener**.
   4. Select **HTTPS** as the protocol and specify **443** as the port.
   5. Choose an **SSL certificate** from the dropdown (if you don’t have one, create an SSL certificate through **AWS Certificate Manager** for your domain).
   6. Set the **Default actions** to forward to the target group used by your ECS service.
   7. Save your changes to enable HTTPS.

Step 2: Configure Route 53 to Use Your Load Balancer with HTTPS
--------------------------------------------------------------
   1. In the **Route 53** console, go to **Hosted zones** and select your domain.
   2. Click **Create Record** and set up an **A record**:
      - **Alias:** Yes
      - **Alias target:** Select your load balancer from the dropdown list.
      - **Region:** Choose the region where your ECS cluster is deployed.
   3. Save the record.

Your application should now be accessible over HTTPS via your custom domain.
