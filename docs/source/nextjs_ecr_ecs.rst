Deploying a Next.js App to ECS Using ECR
=========================================

This guide provides a step-by-step process to deploy a Next.js application to Amazon Elastic Container Service (ECS) using Amazon Elastic Container Registry (ECR).

**Prerequisites**
------------------
Before proceeding, ensure you have the following:
- **AWS account** with necessary permissions for ECS, ECR, and IAM.
- **AWS CLI** installed and configured with your credentials.
- **Docker** installed on your local machine.
- **Next.js application** to containerize and deploy.

Step 1: **Set Up ECR (Elastic Container Registry)**
-----------------------------------------------------
   Create a repository in ECR to store your Docker images.
   
   1. Log in to the AWS Console.
   2. Navigate to **ECR** (Elastic Container Registry).
   3. Select **Create Repository**.
   4. Give your repository a name (e.g., `nextjs-app`) and configure other settings as needed.
   5. Click **Create Repository**.
   6. Now open the repository and press ** view push commands** follow the commands after you done with the step 2 part.

Step 2: **Create Dockerfile for Next.js Application**
-----------------------------------------------------
   To containerize your Next.js application, you need a `Dockerfile`. Create this file at the root of your Next.js project.
   
   Here is a sample `Dockerfile`:
   
   .. code-block:: docker
   
      # Use the official Node.js 16 image as the base image
      FROM node:16-alpine AS builder
   
      # Set the working directory
      WORKDIR /app
   
      # Copy package.json and package-lock.json
      COPY package*.json ./
   
      # Install dependencies
      RUN npm install
   
      # Copy the rest of the application code
      COPY . .
   
      # Build the Next.js app
      RUN npm run build
   
      # Stage 2: Production Environment
      FROM node:16-alpine
   
      # Set the working directory
      WORKDIR /app
   
      # Copy built app from Stage 1
      COPY --from=builder /app ./
   
      # Install only production dependencies
      RUN npm ci --production
   
      # Expose port 3000
      EXPOSE 3000
   
      # Start the application
      CMD ["npm", "start"]

Step 3: **run all commands that gets popup after pressing view push commands**
------------------------------------------------------------------------------

Step 4: **Set Up ECS Cluster**
-------------------------------
   1. Log in to the AWS Console.
   2. Navigate to **ECS** (Elastic Container Service).
   3. Select **Create Cluster** and give cluster a name.
   4. Choose the  option ( Fargate or EC2 if preferred) .
   5. Click **Create**.

Step 6: **Create ECS Task Definition**
--------------------------------------
   1. In the ECS Console, navigate to **Task Definitions**.
   2. Select **Create new Task Definition** Specify a unique task definition family name.
   3. Choose either **EC2** or **Fargate** depending on your setup.
   4. Provide the required configurations if needed can set Environment variables also.
   5. Click **Create**.

Step 7: **Create ECS Service**
------------------------------
   1. Navigate to **ECS Services** and click **Create Service**.
   2. Select your ECS cluster and **Task Definition**.
   3. Set up **Service Name** and the desired number of tasks (instances of your app).
   4. Configure the **Load Balancer** if required, or choose to use a **Network Configuration** for public access.
   5. Click **Create Service**.

Step 8: **Access Your Application**
-----------------------------------
   Once the service is up and running, you can access your Next.js application.
   
   1. Navigate to the **EC2** section in the AWS Console.
   2. Find the **Public IP** of your ECS instance (if using EC2) or the Load Balancer URL (if using Fargate).
   3. Open the IP or URL in your browser to see your deployed Next.js app.

**Conclusion**
--------------
   Your Next.js application is now deployed on AWS ECS using ECR for container management. You can further monitor, scale, and manage the application through the ECS console.
