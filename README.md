# Mini-Project-Single-End-Point-Architecture-For-EC2-And-S3-Services-



Mini Project: Single Endpoint Architecture for EC2 and S3 Services
Project Title: Architecting a Scalable Web Infrastructure: Integrating AWS EC2 Compute with S3 Storage for Zappy e-Bank
Role: Cloud Solutions Architect / DevOps Engineer
Core Skills: AWS S3 Management, IAM Roles & Policies, EC2 Instance Connectivity, CLI File Synchronization, Public Endpoint Configuration.
Environment: AWS Management Console (EC2, S3, IAM) + Local / Cloud9 Terminal.


Part 1: Theoretical Foundation – The Philosophy of Compute & Storage
1.1 The "Zappy e-Bank" Scenario
In our previous project, we secured the identities of our employees (John and Mary). Now, Zappy e-Bank is ready to build its infrastructure. The company needs a scalable, secure architecture to deliver its financial services.

The Compute Layer: EC2 (Elastic Compute Cloud) will host the web application and backend code.

The Storage Layer: S3 (Simple Storage Service) will store static assets (images, user-uploaded documents, HTML files).

The Challenge: The EC2 instance needs access to S3 to fetch these files and serve them to customers. We cannot store secrets or passwords on the EC2 instance—that is a massive security vulnerability.

1.2 What is a "Single Endpoint" Architecture?
A "Single Endpoint" architecture refers to a design where your application (running on EC2) accesses your storage (S3) via a single, unified, secure route—typically via a VPC Endpoint or through the AWS Public Internet using IAM Roles.
In this project, we will focus on the Compute-to-Storage integration: configuring an EC2 instance to securely read from and write to an S3 bucket without hardcoding any AWS Access Keys.

Part 2: Project Prerequisites & Architecture Overview
2.1 Prerequisites
An active AWS Account with EC2 and S3 permissions.

A running EC2 Linux Instance (Ubuntu 20.04 LTS) with SSH access (created in previous AWS projects).

Basic understanding of the Linux terminal.

2.2 Architecture Diagram (Mental Model)
text
[Internet User]  --->  [EC2 Web Server]  <--- (Pulls Files) <---  [AWS S3 Bucket]
                                      |
                                [IAM Role] (Securely authenticates EC2 to S3)
Part 3: Step-by-Step Project Implementation
Task 1 – Creating the Storage Backbone (S3 Bucket)
We need a place to store the static files for Zappy e-Bank's website.

Step 1: Navigate to S3

Log in to the AWS Management Console.

In the top search bar, type "S3" and select "S3" (Simple Storage Service).

Step 2: Create a Bucket

Click the orange "Create bucket" button.

Bucket name: Enter a globally unique name. Example: zappy-ebank-assets-[your-name]. (Note: S3 bucket names must be globally unique across all of AWS).

AWS Region: Select a region close to you (e.g., us-east-1).

Block Public Access settings: Keep the default (Block all public access). This is secure. The EC2 instance will access this bucket using an IAM Role, not by making it public to the internet.

Click "Create bucket".

Task 2 – Creating the Compute Identity (IAM Role for EC2)
This is the most critical security step of this project. To allow the EC2 instance to talk to S3 without using a password, we create an IAM Role and attach it to the EC2 server.

Step 1: Navigate to IAM

In the search bar, type "IAM" and select Identity and Access Management.

On the left menu, click "Roles".

Click the orange "Create role" button.

Step 2: Select the Trusted Entity

Trusted entity type: Select "AWS service" (this tells AWS: "I want to create a role for another AWS service to assume").

Use case: Select "EC2" (this tells AWS: "This role will be used by an EC2 instance").

Click "Next".

Step 3: Attach Permissions Policies

Search for AmazonS3FullAccess in the search bar.

Check the box next to AmazonS3FullAccess.

(Note: In a production environment, you would use a custom policy with read-only access or specific bucket access. For this lab, full access simplifies the learning process).

Click "Next".

Step 4: Name the Role

Role name: ec2-s3-admin-role.

Description: Allows EC2 instances to read and write files to S3 buckets.

Click "Create role".

Task 3 – Attaching the Role to the EC2 Instance
Now that the identity exists, we must assign it to our server.

Step 1: Navigate to EC2

Go to the EC2 Dashboard and click on "Instances".

Select the running EC2 instance you want to use (e.g., the one you created in the previous AWS project).

Step 2: Modify the IAM Role

At the top, click on "Actions" -> "Security" -> "Modify IAM role".

Under "IAM role", click the dropdown and select the role we just created: ec2-s3-admin-role.

Click "Update IAM role".

(Verification: You will see a green banner saying "Successfully updated IAM role").

Task 4 – Connecting to EC2 and Syncing Data
Now we will prove that the instance can talk to S3 without a password.

Step 1: SSH into the EC2 Instance

bash
ssh -i "your-key.pem" ubuntu@<your-ec2-public-ip>
(You are now inside the server).

Step 2: Verify the IAM Role is Active
Run the following command to check what permissions the EC2 instance currently has:

bash
aws sts get-caller-identity
(If the IAM role is attached correctly, this command will return the ARN of the ec2-s3-admin-role you created, proving the server securely assumed the identity).

Step 3: Upload a Test File to S3
Let's create a test file on the server and upload it to our S3 bucket.

bash
# Create a sample HTML file
echo "<h1>Welcome to Zappy e-Bank</h1>" > index.html

# Upload the file to your S3 bucket using the AWS CLI
# Note: Replace 'zappy-ebank-assets-[your-name]' with your actual bucket name!
aws s3 cp index.html s3://zappy-ebank-assets-[your-name]/
(Output: You will see upload: ./index.html to s3://zappy-ebank-assets-[your-name]/index.html).

Step 4: Verify the File is in S3

Go back to the S3 Dashboard in your browser.

Click on your bucket (zappy-ebank-assets-[your-name]).

You will see the index.html file sitting in the bucket.

Success! Your EC2 instance successfully communicated with S3 securely using an IAM Role, without ever exposing a password!

Task 5 – Making the File Publicly Accessible (The Single Endpoint)
To enable the "Single Endpoint" for the public, we need to make the file accessible over the internet.

Step 1: Change the Bucket Permissions

In the S3 Dashboard, click on your bucket.

Click the "Permissions" tab.

Scroll down to "Block public access (bucket settings)".

Click "Edit".

Uncheck "Block all public access". (AWS will warn you that this makes the bucket public; this is fine for this learning lab).

Click "Save changes".

Step 2: Make the File Public

Go back to the "Objects" tab of your bucket.

Click on the index.html file.

Go to the "Permissions" tab.

Scroll down to "Access control list (ACL)".

Click "Edit".

Under the "Everyone (public access)" section, check "Read" for "List objects" and "Read bucket permissions".

Click "Save changes".

Step 3: Access the Single Endpoint

In the S3 Overview for the index.html file, click the "Properties" tab.

Scroll down to "Object overview". Copy the "Object URL" (e.g., https://zappy-ebank-assets-[your-name].s3.amazonaws.com/index.html).

Open a new browser tab and paste the URL.

Success: You will see the "Welcome to Zappy e-Bank" HTML page served directly from the S3 bucket!

(Note: Depending on the region, the browser may take up to 1-2 minutes to propagate the public access permissions. If you get a 403 Forbidden, wait 60 seconds and refresh).

Part 4: Expert Insights & Pro-Tips (The Senior Engineer's Perspective)
To guarantee a "top-notch" grade and demonstrate true cloud engineering knowledge, include these advanced concepts in your submission:

4.1 Why IAM Roles are Superior to Access Keys
Imagine if you hardcoded an AWS_ACCESS_KEY and AWS_SECRET_ACCESS_KEY into a file on your EC2 instance.

The Risk: If a hacker breaches your EC2 instance, they can steal those keys and gain full access to your S3 bucket, possibly deleting all your data.

The IAM Role Solution: IAM Roles generate temporary, rotating credentials automatically. The EC2 instance requests these credentials from the AWS metadata service every few minutes. There is no secret key stored on the server itself. This is the absolute gold standard for cloud security.

4.2 The aws s3 sync Command
In a real-world scenario, a developer wouldn't just upload one file; they would sync an entire website folder. The command to do this is:

bash
aws s3 sync /local/web/folder/ s3://zappy-ebank-assets-[your-name]/
This command checks the difference between your local folder and the bucket, and only uploads new or modified files. This is critical for efficient CI/CD pipelines.

4.3 AWS CLI Installation
If you are using a brand-new EC2 instance, the AWS CLI might not be installed. If you run aws s3 cp and get a command not found error, run this first:

bash
sudo apt update
sudo apt install awscli
4.4 Cleaning Up to Avoid Costs (The "Bill Shock" Safety)
After completing this project, do not forget to delete your resources to avoid incurring charges.

Delete the files in your S3 bucket by selecting them and clicking Delete.

Delete the S3 bucket itself.

Crucial Step: Ensure you stop or terminate your EC2 instance!

(Optional but recommended) Delete the ec2-s3-admin-role in the IAM dashboard.

Part 5: Project Reflection (The Academic Deliverable)
To complete your submission, include this written reflection to demonstrate your deep conceptual understanding:

1. Explain the Role of EC2 and S3 in AWS:
EC2 (Elastic Compute Cloud) provides resizable, secure compute capacity in the cloud. It is the "brain" of your infrastructure, running your operating system and application code. S3 (Simple Storage Service) is a highly durable, scalable, serverless object storage service. It is designed to store and retrieve any amount of data from anywhere. Together, EC2 and S3 form the backbone of modern web applications: EC2 does the processing, and S3 handles the static assets.

2. How does IAM enhance security between EC2 and S3?
Instead of storing long-term, static credentials (Access Keys) inside an EC2 instance (which is a massive security risk), IAM Roles allow EC2 instances to assume a temporary identity. The instance retrieves ephemeral, rotating credentials from the internal AWS metadata service. If an attacker compromises the EC2 instance, they cannot steal permanent keys, and their access is strictly limited to the permissions defined in the attached IAM Role. This eliminates hard-coded secrets.

3. Discuss the concept of a "Single Endpoint" in the context of this architecture:
A "Single Endpoint" architecture implies a unified path for data access. In this project, we secured a direct, authenticated pipeline from the EC2 compute instance to the S3 storage bucket via an IAM Role. By synchronizing files from the EC2 instance to the S3 bucket using the aws s3 sync command, and then serving those files publicly via the S3 Object URL, we created a streamlined workflow where developers push files from their servers directly to the public facing storage tier without complex network routing.

4. Why is it important to understand cost implications in the cloud?
One of the greatest advantages of cloud computing is operational expenditure (OpEx) instead of capital expenditure (CapEx). However, this also means you pay for what you use. Large VM sizes (t2.large) cost more than small ones (t2.micro). S3 storage costs depend on the amount of data stored and how often it is accessed (Standard vs. Standard-IA tiers). Understanding these pricing models allows a DevOps engineer to architect solutions that are not only functional but also cost-optimized, saving the company thousands of dollars annually.

Conclusion: You Are Now an AWS Solutions Architect
This project successfully transformed you from a cloud user into an AWS Solutions Architect.

You have achieved:

Secure Architecture: You set up an IAM Role, bridging the gap between EC2 and S3 using zero hardcoded passwords.

Storage Management: You created an S3 bucket, uploaded files via the command line, and configured public access.

Compute-to-Storage Integration: You used the aws s3 CLI tool to securely synchronize files between your operating system and the cloud.

Real-World Deployment: You successfully deployed a publicly accessible HTML endpoint, creating the foundational architecture for Zappy e-Bank's future web presence.


You are no longer just clicking buttons in the AWS Console; you are designing scalable, secure, and cost-aware cloud infrastructures!
