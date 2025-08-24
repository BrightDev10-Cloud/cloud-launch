## CloudLaunch AWS Deployment Guide

A practical walkthrough for deploying a secure static site and private document storage using AWS S3, IAM, and VPC.

## Overview

This guide details the CloudLaunch project setup — a lightweight web platform deployed with AWS Free Tier resources. It features:

- A publicly accessible static site for the company.

- Secure internal file storage with strict permission controls.

- A future-proof VPC architecture for expansion.

Tasks covered:

- Static Website Hosting on S3 (with tailored IAM permissions)

- VPC Network Design for secure resource separation

### Submision Links

S3 Site Link:
http://cloudlaunch-site-bucket.s3-website-<region>.amazonaws.com

CloudFront URL (if configured):
https://<distribution-id>.cloudfront.net

## Task 1: Static Website Hosting and Secure File Storage

### Step 1. Create Your S3 Buckets

a. cloudlaunch-site-bucket
Purpose: Hosts your public static site (HTML/CSS/JS).

Setup:

- Create bucket named cloudlaunch-site-bucket in S3.

- Enable “Static website hosting” under bucket properties.

- Upload your site files (index.html, etc.).

Permissions:

- Public read only: Attach this bucket policy:

```JSON
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "PublicReadGetObject",
      "Effect": "Allow",
      "Principal": "*",
      "Action": "s3:GetObject",
      "Resource": "arn:aws:s3:::cloudlaunch-site-bucket/*"
    }
  ]
}

```

- Bonus: Configure a CloudFront distribution for HTTPS and global caching.

b. cloudlaunch-private-bucket
Purpose: Private document storage.

Setup:

- Bucket name: cloudlaunch-private-bucket

- Block all public access (default).

Permissions:

- Only accessible by a designated IAM user (see below).

- Allow GetObject and PutObject; deny DeleteObject.

c. cloudlaunch-visible-only-bucket
Purpose: IAM user can see bucket name (list), but not open files.

Setup:

- Bucket name: cloudlaunch-visible-only-bucket

- Block all public access.

Permissions:

- IAM user can use ListBucket (see bucket name) only.

### Step 2. Create and Configure IAM User

- Go to AWS IAM > Users > Add user.

- Username: cloudlaunch-user

- Grant Programmatic access only.

- Custom IAM Policy Example
  - - Attach the following JSON policy to the new user for strict resource access:

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
        "arn:aws:s3:::cloudlaunch-site-bucket",
        "arn:aws:s3:::cloudlaunch-private-bucket",
        "arn:aws:s3:::cloudlaunch-visible-only-bucket"
      ]
    },
    {
      "Action": [
        "s3:GetObject"
      ],
      "Effect": "Allow",
      "Resource": "arn:aws:s3:::cloudlaunch-site-bucket/*"
    },
    {
      "Action": [
        "s3:GetObject",
        "s3:PutObject"
      ],
      "Effect": "Allow",
      "Resource": "arn:aws:s3:::cloudlaunch-private-bucket/*"
    }
  ]
}

```

- No DeleteObject permissions.

- No access to cloudlaunch-visible-only-bucket contents.

### Step 3. (Optional) Set Up CloudFront for Static Site

- Go to CloudFront > Create Distribution.

- Set the Origin to your S3 bucket (cloudlaunch-site-bucket).

- Enable HTTPS and default caching.

- Save the CloudFront distribution URL.

## Task 2: Secure VPC Network Design

### Step 1. Create the VPC

- Go to VPC > Create VPC.

- Name: cloudlaunch-vpc

- CIDR block: 10.0.0.0/16

### Step 2. Create Three Subnets

- cloudlaunch-public-subnet | 10.0.1.0/24 (For public-facing services)

- cloudlaunch-app-subnet | 10.0.2.0/24 (For application servers)

- cloudlaunch-db-subnet | 10.0.3.0/28 For databases (RDS etc.)

### Step 3. Setup Internet Gateway

- Create an Internet Gateway named cloudlaunch-igw.

- Attach it to your VPC.

### Step 4. Route Tables

a. Public Route Table (cloudlaunch-public-rt)

- Associate with cloudlaunch-public-subnet.

Add route:

- Destination: 0.0.0.0/0

- Target: cloudlaunch-igw

b. App Route Table (cloudlaunch-app-rt)

- Associate with cloudlaunch-app-subnet.

- Do not add public routes.

c. DB Route Table (cloudlaunch-db-rt)
Associate with cloudlaunch-db-subnet.

- Do not add public routes.

### Step 5. Create Security Groups

a. cloudlaunch-app-sg

- Allows HTTP (port 80) within the VPC only.

- Inbound: port 80, source 10.0.0.0/16

b. cloudlaunch-db-sg

- Allows MySQL (port 3306) from the App subnet only.

- Inbound: port 3306, source 10.0.2.0/24

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

Thank You!
