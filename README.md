# CloudLaunch Project – Task 1 & Task 2

This documentation describes how CloudLaunch, a platform that showcases a basic company website and stores some internal private documents was deployed using AWS core services.


## Bacground  
This project was part of my Cloud 3rd semester assignment at AltSchool where I got hands-on practice with Amazon S3, IAM, and VPC design. The goal was to combine storage, access management, and networking fundamentals into a working environment that could scale for future projects.  

I had two main tasks:  
1. Static Website Hosting using S3 + IAM user with limited permissions**  
2. VPC Design with logical separation of Subnets, Route Tables, and Security Groups**  



## Task 1: Static Website Hosting Using S3 + IAM User with Limited Permissions  


### How I went about it 
- I created **three S3 buckets**:  
  1. cloudlaunch-site-bucket-rita→ This bucket hosted a very simple static website (just HTML/CSS). I enabled static website hosting here and made the files publicly readable so that anyone with the URL could access the site.  I inserted my name at the end because it turns out site bucket has to have a unique name, as the originally suggested name had been taken elsewhere. 
  2. cloudlaunch-private-bucket-rita → This was strictly private. I made this bucket to be accessible only through an IAM user with limited permissions, for **GetObject** and **PutObject** operations. No deletes allowed.  
  3. cloudlaunch-visible-only-bucket-rita→ I created this so the IAM user could **see** that the bucket existed but could not actually read or upload any files inside it.  

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
        "arn:aws:s3:::cloudlaunch-site-bucket",
        "arn:aws:s3:::cloudlaunch-private-bucket",
"arn:aws:s3:::cloudlaunch-visible-only-bucket"
      ]
    },
    {
      "Sid": "AllowReadSiteBucket",
      "Effect": "Allow",
      "Action": "s3:GetObject",
      "Resource": "arn:aws:s3:::cloudlaunch-site-bucket/*"
    },
    {
      "Sid": "AllowPrivateBucketReadWrite",
      "Effect": "Allow",
      "Action": [
        "s3:GetObject",
        "s3:PutObject"
      ],
      "Resource": "arn:aws:s3:::cloudlaunch-private-bucket/*"
    }
  ]
}



