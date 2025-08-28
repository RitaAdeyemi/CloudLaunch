CloudLaunch Project – Task 1  

This project documents how I hosted a **static website** on Amazon S3, created multiple buckets for different use cases, and configured an IAM user with **restricted permissions** to access those buckets.  

This is an assignment I completed at AltSchool Africa as part of semester assigments in my Cloud Engineering track.

Objectives  
- Host a static website using S3.  
- Configure multiple buckets for different access patterns.  
- Create an IAM user with limited permissions to manage those buckets securely.  

 How I Did It  

### 1. Setting Up S3 and Hosting a Static Website  

- I logged into AWS Console as the **root user** and navigated to the **S3 service**.  
- I created **three buckets**:  
  1. **`cloudlaunch-site-bucket-rita`** – for hosting the static website. (The “rita” suffix was necessary because S3 bucket names must be globally unique).  
  2. **`cloudlaunch-private-bucket-rita`** – intended for private file storage.  
  3. **`cloudlaunch-visible-only-bucket-rita`** – for visibility only (no object access).  

- For the **cloudlaunch-site-bucket-rita**, I:  
  - Clicked into the bucket → **Upload** → uploaded my `index.html`.  
  - When I tested the object URL, it gave an error because it was private.  
  - To fix this:  
    - I went into the **Permissions tab** → Disabled “Block all public access”.  
    - Then, I created a **Bucket Policy** using AWS’s Policy Generator:  
      - Type = S3 Bucket Policy  
      - Effect = Allow  
      - Principal = `*` (anyone)  
      - Action = `s3:GetObject`  
      - Resource = the bucket’s ARN with `/*`  
    - I pasted the generated JSON into the bucket policy editor and saved.  

- Now the static site was **publicly available (read-only)**. Testing the object URL in a browser loaded my webpage successfully. 


### 2. Creating the Other Buckets  

- **Private bucket (`cloudlaunch-private-bucket-rita`)**: left defaults → non-public.  
- **Visible-only bucket (`cloudlaunch-visible-only-bucket-rita`)**: left defaults → non-public.  


### 3. IAM User with Limited Permissions  

- In **IAM**, I created a new user called **`cloudlaunch-user`**.  
  - Username = `cloudlaunch-user`.  
  - Auth = **auto-generated password** + forced password reset at first login.  
  - I downloaded the **.csv file** containing login details.  

- I wanted this user to have:  
  - `ListBucket` access to **all three buckets**.  
  - `GetObject` and `PutObject` on the **cloudlaunch-private-bucket-rita only**.  
  - `GetObject` on the **cloudlaunch-site-bucket-rita only**.  
  - **No access** to objects in the cloudlaunch-visible-only-bucket-rita.  
  - **No DeleteObject permissions** anywhere.  

- For security, I created a new IAM user named **`cloudlaunch-user`**. This user only has permissions to interact with the above three buckets strictly.  

- To enforce this, I wrote and attached a **custom JSON IAM policy** that defined exactly what this user could and couldn’t do.  

### JSON IAM Policy  
Here’s the policy I created and attached to the `cloudlaunch-user` account:  

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "AllowListBuckets",
      "Effect": "Allow",
      "Action": "s3:ListAllMyBuckets",
      "Resource": "arn:aws:s3:::*"
    },
    {
      "Sid": "AllowListSpecificBuckets",
      "Effect": "Allow",
      "Action": "s3:ListBucket",
      "Resource": [
        "arn:aws:s3:::cloudlaunch-site-bucket-rita",
        "arn:aws:s3:::cloudlaunch-private-bucket-rita",
"arn:aws:s3:::cloudlaunch-visible-only-bucket-rita"
      ]
    },
    {
      "Sid": "AllowReadSiteBucket",
      "Effect": "Allow",
      "Action": "s3:GetObject",
      "Resource": "arn:aws:s3:::cloudlaunch-site-bucket-rita/*"
    },
    {
      "Sid": "AllowPrivateBucketReadWrite",
      "Effect": "Allow",
      "Action": [
        "s3:GetObject",
        "s3:PutObject"
      ],
      "Resource": "arn:aws:s3:::cloudlaunch-private-bucket-rita/*"
    }
  ]
}

```
Morevoer, I created access credentials for the cloudlaunch user that would allow them sign-in as an IAM user that would require resetting their password on logging in for the  first time.

Task 2: VPC Design for a Secure and Logical Network
The second part didn’t require the launch of EC2 instance or NAT Gateway.

Steps I Took
I started by creating a VPC named CloudLaunch-VPC with a CIDR block of 10.0.0.0/16.


Inside the VPC, I created three subnets:


Public Subnet (10.0.1.0/24) – Intended for load balancers or future public-facing services.
Application Subnet (10.0.2.0/24) – Intended for app servers (private).
Database Subnet (10.0.3.0/28) – Intended for RDS-like services (private).


Then I set up route tables:


cloudlaunch-public-rt-this public route table was associated with the public subnet and had a default route pointing to an Internet Gateway.


Cloudlaunch-app-rt and cloudlaunch-db-rt- these private route tables were associated with the application Subnet and database subnet respectively and had no internet access (yet).


Next was to attach security groups:
loudlaunch-app-sg: that allows HTTP (port 80) access within the VPC only.
cloudlaunch-db-sg: that allows MySQL (port 3306) access from app subnet only.

Finally here's the link to access the s3 static site: https://cloudlaunch-site-bucket-rita.s3.eu-north-1.amazonaws.com/index.html
Also, the console url/Account ID/Account alias along with the user credentials can be accessed here: https://drive.google.com/file/d/1ldZJ0AAnISRloRORJN87yaa6H0i8dbgg/view?usp=sharing









