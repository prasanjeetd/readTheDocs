Converting EC2 HTTP to HTTPS via API Gateway
============================================

This guide provides a step-by-step process for converting an HTTP endpoint hosted on an EC2 instance to an HTTPS endpoint using Amazon API Gateway. The API Gateway acts as a secure proxy to forward traffic to the EC2 instance.

**Prerequisites**
-----------------
Before proceeding, ensure you have the following:
- **AWS account** with necessary permissions for EC2 and API Gateway.
- **EC2 instance** with a publicly accessible HTTP endpoint.
- **AWS CLI** installed and configured with your credentials.

Step 1: **Set Up the EC2 HTTP Endpoint**
------------------------------------------
Ensure that your EC2 instance is running and hosting your application over HTTP. You should have the **Public DNS** of your EC2 instance ready (e.g., `http://<ec2-public-dns>`).

Step 2: **Create a REST API in API Gateway**
--------------------------------------------
1. Log in to the **AWS Console** and navigate to **API Gateway**.
2. Select **Create API**.
3. Choose **REST API** as the API type.
4. Give your API a **name** (e.g., `ec2-https-proxy`) and a **description**.
5. Select **Edge optimized** for the Endpoint Type.
6. Click **Create API**.

Step 3: **Create a Proxy Resource**
-----------------------------------
1. In the API Gateway dashboard, select the newly created API.
2. Under **Resources**, click **Actions** and select **Create Resource**.
3. Enable the **Configure as proxy resource** option.
4. In the **Resource Name** field, enter `{proxy+}` to allow all paths to be routed to your EC2 instance.
5. Set **Resource Path** to `/ {proxy+}` (this allows all URL patterns to be proxied).
6. Click **Create Resource**.

Step 4: **Create an ANY Method**
-----------------------------------
1. Select the newly created resource `{proxy+}` under **Resources**.
2. Click **Actions** and select **Create Method**.
3. From the dropdown, choose `ANY` as the method and click the checkmark to proceed.
4. For **Integration Type**, choose **HTTP**.
5. In the **Endpoint URL** field, enter your EC2 DNS (e.g., `http://<ec2-public-dns>/{proxy}`).
   - Make sure to add `/{proxy}` at the end of the EC2 DNS URL. This enables API Gateway to forward all URL paths to the EC2 instance.
6. Leave the other settings as default.
7. Click **Save**.
8. In the **Method Execution** page, click **Method Request** and set **API Key Required** to `false` (unless you want to restrict access).

Step 5: **Deploy the API to a Stage**
--------------------------------------
1. After creating the method, navigate to **Actions** and select **Deploy API**.
2. Choose **New Stage** and give the stage a name (e.g., `prod`).
3. Click **Deploy**.

Step 6: **Test the HTTPS API Endpoint**
----------------------------------------
1. After deploying the API, the **Invoke URL** for your API will be displayed (e.g., `https://<api-id>.execute-api.<region>.amazonaws.com/prod`).
2. Test the HTTPS endpoint by appending any path that matches your EC2 service (e.g., `https://<api-id>.execute-api.<region>.amazonaws.com/prod`).

Step 7: **Verify HTTPS Proxy**
-------------------------------
1. Open the **Invoke URL** in your browser or use tools like `curl` or Postman to make a request to the endpoint.
   - Example: `https://<api-id>.execute-api.<region>.amazonaws.com/prod/{your-api-path}`.
2. Ensure that your HTTP traffic is successfully forwarded to the EC2 instance, but is now accessible over HTTPS through API Gateway.


**Conclusion**
-----------
Your EC2 HTTP endpoint is now accessible over HTTPS through API Gateway. API Gateway handles the HTTPS layer, providing encryption, while forwarding traffic securely to your EC2 instance. You can further configure your API for security, monitoring, and scaling as needed.
