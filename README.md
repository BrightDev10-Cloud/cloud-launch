## CloudLaunch AWS Deployment Guide

![alt text](image-30.png)

A practical walkthrough for deploying a secure static site and using AWS S3, IAM, Cloudfront and VPC.

## Overview

This guide details the CloudLaunch project setup — a lightweight web platform deployed with AWS Free Tier resources. It features:

- A publicly accessible static site for the company.

- Secure internal file storage with strict permission controls.

- A future-proof VPC architecture for expansion.

Tasks covered:

- Static Website Hosting on S3 (with tailored IAM permissions)

- VPC Network Design for secure resource separation

## Submision Links

- S3 Site Link:
  [s3://abdul-cloud-launch-site-bucket/CloudLaunch/](s3://abdul-cloud-launch-site-bucket/CloudLaunch/)

  ```bash
      s3://abdul-cloud-launch-site-bucket/CloudLaunch/
  ```

- CloudFront URL:
  [d54miv63u75lb.cloudfront.net](d54miv63u75lb.cloudfront.net)

  ```bash
    d54miv63u75lb.cloudfront.net
  ```

## Task 1: Static Website Hosting and Secure File Storage

### Step 1. Create S3 Buckets

- Log in to AWS Management Console. Go to “S3” -> “Create bucket”

  ![alt text](image.png)

  > [!TIP]
  > Create three buckets.

### a. cloudlaunch-site-bucket (Purpose: Hosts your public static site (HTML/CSS/JS)).

Setup:

- Create bucket named anything you like in my case i use "abdul-cloud-launch-site-bucket" in S3 and make it publicly accessible.

  - ![alt text](image-4.png)

  - ![alt text](image-2.png)

  - ![alt text](image-5.png)

  - ![alt text](image-10.png)

- Enable “Static website hosting” under bucket properties after creating the bucket in the step before.

  - ![alt text](image-6.png)

  - ![alt text](image-7.png)

- Upload your site files from your computer to the s3 bucket (index.html, etc.).

  - ![alt text](image-9.png)

  - ![alt text](image-8.png)

Permissions:

- Public read only: Attach this bucket policy to allow public read-only access to the s3 bucket:

```JSON
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "PublicReadGetObject",
      "Effect": "Allow",
      "Principal": "*",
      "Action": "s3:GetObject",
      "Resource": "arn:aws:s3:::abdul-cloud-launch-site-bucket/*"
    }
  ]
}

```

- ![alt text](image-11.png)
- ![alt text](image-12.png)
- ![alt text](image-13.png)

### b. cloudlaunch-private-bucket (Purpose: Private document storage).

Setup: (Please complete step 2 to create and configure the IAM user permissions for the steps below).

- Bucket name: abdul-cloud-launch-private-bucket

- Block all public access (default).

  ![alt text](image-14.png)

Permissions:

- Only accessible by a designated IAM user.

- Allow GetObject and PutObject; deny DeleteObject.

### c. cloudlaunch-visible-only-bucket (Purpose: IAM user can see bucket name (list), but not open files.)

Setup:

- Bucket name: abdul-cloudlaunch-visible-only-bucket

- Block all public access.

- ![alt text](image-15.png)

Permissions:

- IAM user can use ListBucket only.

### Step 2. Create and Configure IAM User

- Go to AWS IAM > Users > Create user.

  - ![alt text](image-16.png)

- Username: cloudlaunch-user

- Grant Programmatic access only.

  - ![alt text](image-17.png)

- Custom IAM Policy Example

  - Attach the following JSON policy to the new user for strict resource access:

```JSON
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Action": [
        "s3:ListBucket"
      ],
      "Effect": "Allow",
      "Resource": [
        "arn:aws:s3:::abdul-cloud-launch-site-bucket",
        "arn:aws:s3:::abdul-cloud-launch-private-bucket",
        "arn:aws:s3:::abdul-cloud-launch-visible-only-bucket"
      ]
    },
    {
      "Action": [
        "s3:GetObject"
      ],
      "Effect": "Allow",
      "Resource": "arn:aws:s3:::abdul-cloud-launch-site-bucket/*"
    },
    {
      "Action": [
        "s3:GetObject",
        "s3:PutObject"
      ],
      "Effect": "Allow",
      "Resource": "arn:aws:s3:::abdul-cloud-launch-private-bucket/*"
    }
  ]
}

```

- No DeleteObject permissions.

- No access to cloudlaunch-visible-only-bucket contents.

  - ![alt text](image-18.png)

  - ![alt text](image-19.png)

  - ![alt text](image-20.png)

### Step 3. (Optional) Set Up CloudFront.

- Go to CloudFront > Create a Cloudfront Distribution.

- ![alt text](image-21.png)

- Set the Origin to your S3 bucket (abdul-cloud-launch-site-bucket.s3-website-us-east-1.amazonaws.com).

- ![alt text](image-22.png)

- ![alt text](image-23.png)

### Enable HTTPS

- - Edit Default Cache Behaviour : Go to the "Behaviors" tab and select the "Default (\*)" cache behavior, then choose "Edit."

    - ![alt text](image-24.png)

- - Configure Viewer Protocol Policy: Under "Viewer Protocol Policy," select one of the following options to enforce HTTPS:

    - - Redirect HTTP to HTTPS: This redirects all HTTP requests to HTTPS, ensuring all traffic uses a secure connection.

    - - HTTPS Only: This only accepts HTTPS requests, rejecting any HTTP requests.

        - ![alt text](image-25.png)

- - Save Changes: Apply the changes to the default cache behavior.

    - ![alt text](image-26.png)

### Enable Default Caching

- - Edit Default Cache Behavior: Go to the "Behaviors" tab, select the "Default (\*)" cache behavior, and choose "Edit."

- - Configure Cache Policy: Under "Viewer Cache Policy"
    - - Managed Cache Policies: CloudFront provides pre-configured managed cache policies (e.g., "CachingOptimized," "CachingDisabled"). You can select one that aligns with your caching needs.
    - ![alt text](image-27.png)
- Custom Cache Policy: If a managed policy doesn't meet your requirements, you can create a custom cache policy.

- Save Changes: Apply the changes to the default cache behavior.

- ![alt text](image-28.png)

- Save the CloudFront distribution URL.

- ![alt text](image-29.png)

## Task 2: Secure VPC Network Design

### Step 1. Create the VPC

- Go to VPC > Create VPC.

  - ![alt text](image-31.png)

- Name: cloudlaunch-vpc

- CIDR block: 10.0.0.0/16

- ![alt text](image-32.png)
- ![alt text](image-33.png)

### Step 2. Create Three Subnets

- Navigate to the "Subnets" tab on the VPC dashboard

- ![alt text](image-34.png)

- cloudlaunch-public-subnet | 10.0.1.0/24 (For public-facing services)

- ![alt text](image-35.png)
- ![alt text](image-36.png)

- cloudlaunch-app-subnet | 10.0.2.0/24 (For application servers)

- ![alt text](image-37.png)

- cloudlaunch-db-subnet | 10.0.3.0/28 For databases (RDS etc.)

- ![alt text](image-38.png)

### Step 3. Setup Internet Gateway

- Create an Internet Gateway named cloudlaunch-igw.

- - Navigate to "Internet Gateways" tab on the VPC dashboard on AWS

    - ![alt text](image-39.png)

- - Click on "Create Internet Gateway" Button

    - ![alt text](image-40.png)

    - ![alt text](image-41.png)

- Attach it to your VPC.

- ![alt text](image-42.png)

- ![alt text](image-43.png)

### Step 4. Create Route Tables for all three Subnets

### a. Public Route Table (cloudlaunch-public-rt)

- Navigate to "Route tables" tab on the VPC dashboard on AWS

- - ![alt text](image-44.png)

- Click on the " Create route table" button

- Name your route table and attach to the cloudlaunch-vpc

- ![alt text](image-45.png)

- Create route table

- ![alt text](image-46.png)

- Associate with cloudlaunch-public-subnet.

- - go back to the "Subnet" tab on the VPC dashboard and double click the cloudlaunch-public-subnet

- - Click on the "Edit route association" button

  - ![alt text](image-47.png)

- - Associate the subnet with the route table

  - ![alt text](image-48.png)

- Add route that sends all internet-bound traffic (0.0.0.0/0) to the Internet Gateway cloudlaunch-igw:

- Go to "Route tables" tab on the VPC dashboard, click on "add route" button and configure it with the details below:

  - - Destination: 0.0.0.0/0

  - - Target: cloudlaunch-igw

      ![alt text](image-49.png)

### b. App Route Table (cloudlaunch-app-rt)

- repeat the (a) steps above to create a new route table for the application layer

- Associate with cloudlaunch-app-subnet.

- Do not add public routes.

- ![alt text](image-50.png)

### c. DB Route Table (cloudlaunch-db-rt)

- Associate with cloudlaunch-db-subnet.

- Do not add public routes.

- ![alt text](image-52.png)

### Step 5. Create Security Groups

a. cloudlaunch-app-sg

- Navigate to the "Security groups" tab on the VPC dashboard.

- ![alt text](image-53.png)

- Click on the "Create security group button"

- Name the Security group "cloudlaunch-app-sg"

- Attach the SG to the cloudlaunch-vpc.

- Add inbound rule to allow HTTP (port 80) within the VPC only.

- Inbound: port 80, source 10.0.0.0/16

- ![alt text](image-54.png)

b. cloudlaunch-db-sg

- Click on the "Create security group button"

- Name the Security group "cloudlaunch-db-sg"

- Attach the SG to the cloudlaunch-vpc.

- Allows MySQL (port 3306) from the App subnet only.

- Inbound: port 3306, source 10.0.2.0/24

- ![alt text](image-55.png)

### Step 6. IAM Permissions for VPC Read-Only

- Attach the following policy for the IAM user cloudlaunch-user:

```JSON
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "ec2:DescribeVpcs",
        "ec2:DescribeSubnets",
        "ec2:DescribeRouteTables",
        "ec2:DescribeInternetGateways",
        "ec2:DescribeSecurityGroups"
      ],
      "Resource": "*"
    }
  ]
}

```

- Navigate to the IAM dashboard and select "User groups" tab to "Create user group".

- Create a new policy named "cloudlaunch-vpc-access-policy

  - - ![alt text](image-56.png)

- Create a new group "cloudlaunch-user-group" and add the "cloudlaunch-user" to the the new cloudlaunch-user-group.

  - - ![alt text](image-57.png)

- Attach the "cloudlaunch-vpc-access-policy" and the c"loudlaunch-policy" to the cloudlaunch-user-group

  - - ![alt text](image-58.png)

Thank You!
