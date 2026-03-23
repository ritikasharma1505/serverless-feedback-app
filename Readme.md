## Serverless Feedback Application built using AWS Lambda, API Gateway, DynamoDB, SES, S3, and CloudFront.

### 1. Project Goal - Build a feedback web application where users can submit feedback and receive an email confirmation.

Key flow

- User opens a web page
- Submits feedback form
- API Gateway receives request
- Lambda processes it
- Feedback stored in DynamoDB
- Email sent via SES
- Static frontend hosted on S3 + CloudFront
- CI/CD handled with GitHub Actions

### 2. Architecture Overview

```
User
  ↓
CloudFront
  ↓
S3 (Static Website)
  ↓
API Gateway (REST API)
  ↓
Lambda Function
  ↓
DynamoDB (store feedback)
  ↓
SES (send confirmation email)

```

### 3. AWS Services Used

```
| Service            | Purpose                  |
| ------------------ | ------------------------ |
| **S3**             | Host static frontend     |
| **CloudFront**     | CDN + HTTPS              |
| **API Gateway**    | Expose REST API          |
| **Lambda**         | Backend logic            |
| **DynamoDB**       | Store feedback           |
| **SES**            | Send confirmation emails |
| **GitHub Actions** | CI/CD deployment         |
```

### 4. Project Folder Structure

Example GitHub repo:

```
serverless-feedback-app
│
├── frontend
│   └── index.html
│
├── lambda
│   └── feedback-handler.py
│
└── .github
    └── workflows
        └── deploy.yml

```

### Step-by-Step Implementation

1️⃣ Create DynamoDB Table

Open DynamoDB.

Click Create table.

Fill:

```
Table name: FeedbackTable
Partition key: feedback_id
Type: String
```
Leave all other settings default.

Click Create Table.

2️⃣ Create S3 Bucket for Attachments

Open S3 → Create bucket.

Example name:
```
feedback-app-attachments-12345
```
Settings:
```
Block Public Access: ON
Versioning: OFF
```
Click Create bucket.

This bucket stores uploaded PDF files.

3️⃣ Verify Email in SES

Open SES.

Go to:

Verified identities

Click:

Create Identity

Choose:

Identity type: Email address

Enter your email.

Example:
```
your-email@gmail.com
```
AWS sends a verification email.

Click the verification link.

4️⃣ Create Lambda Function

Open Lambda.

Click:

Create Function

Choose:

Author from scratch

Settings:
```
Function name: feedback-handler
Runtime: Python 3.13
Architecture: x86_64
```
Click Create Function.

5️⃣ Add Environment Variables

Inside Lambda:

Configuration → Environment variables

Add:
```
TABLE_NAME = FeedbackTable
BUCKET_NAME = your-attachment-bucket
ADMIN_EMAIL = your-email@gmail.com
```
Click Save.

6️⃣ Add Lambda Permissions

Inside Lambda:

Configuration → Permissions

Click the Execution Role.

Attach policies:
```
AmazonDynamoDBFullAccess
AmazonS3FullAccess
AmazonSESFullAccess
```


7️⃣ Add Lambda Code

Open the Code tab.

Paste your final Python Lambda code.

Click:

Deploy

Backend logic is ready.

8️⃣ Create API Gateway (REST API)

Open API Gateway.

Click:

Create API

Choose:

REST API → Build

Configure:
```
API name: FeedbackAPI
Endpoint type: Regional
```
Click Create API.

9️⃣ Create Resource

Inside the API:

Go to Resources.

Click Create Resource.

Fill:
```

Resource path: /
Resource name: feedback
```
Click Create Resource.

🔟 Create POST Method

Select the /feedback resource.

Click Create Method.

Choose:
```
Method type: POST
Integration type: Lambda Function
```
Enable:

Lambda Proxy Integration

Select your function:

feedback-handler

Click Save.

Allow API Gateway to invoke Lambda.

1️⃣1️⃣ Enable CORS

Select /feedback resource.

Click:

Actions → Enable CORS

Configure:
```
Access-Control-Allow-Origin: *
Access-Control-Allow-Headers: Content-Type
Access-Control-Allow-Methods: POST,OPTIONS
```
Click:

Enable CORS and replace headers

This creates an OPTIONS method.

1️⃣2️⃣ Deploy API

Click:

Actions → Deploy API

Choose:
```
Deployment stage: New Stage
Stage name: prod
```
Click Deploy.

1️⃣3️⃣ Copy API Endpoint

You will see something like:
```
https://abc123xyz.execute-api.us-east-1.amazonaws.com/prod
```
Your final endpoint becomes:
```
https://abc123xyz.execute-api.us-east-1.amazonaws.com/prod/feedback
```
Copy this.

1️⃣4️⃣ Update Frontend API URL

Open index.html

Replace:
```
const API_URL = "YOUR_API_GATEWAY_URL";
```
with:
```
const API_URL = "https://abc123xyz.execute-api.us-east-1.amazonaws.com/prod/feedback";
```
Save the file.

1️⃣5️⃣ Create S3 Bucket for Frontend

Create another S3 bucket.

Example:
```
feedback-app-frontend-12345
```
After creation:

Go to:

Properties → Static website hosting

Enable it.

Set:

Index document: index.html

1️⃣6️⃣ Upload Frontend Files

Upload these files:

```
index.html
```
to the frontend bucket.

1️⃣7️⃣ Make Frontend Bucket Public

Go to:

Permissions → Bucket Policy

Add:
```
{
"Version":"2012-10-17",
"Statement":[
{
"Effect":"Allow",
"Principal":"*",
"Action":"s3:GetObject",
"Resource":"arn:aws:s3:::YOUR_BUCKET_NAME/*"
}
]
}
```
Replace YOUR_BUCKET_NAME.

Save.

1️⃣8️⃣ Create CloudFront Distribution

Open CloudFront.

Click:

Create Distribution

Origin domain:

Select your frontend S3 bucket

Configure:
```
Viewer protocol policy: Redirect HTTP to HTTPS
Default root object: index.html
```
Leave most settings default.

Click Create Distribution.

Deployment takes about 5–10 minutes.

1️⃣9️⃣ Access Your Website

CloudFront provides a URL like:
```
https://d123abcxyz.cloudfront.net
```
Open it.

Your website now loads through CloudFront CDN.

2️⃣0️⃣ Test the System

Open the site.

Submit feedback.

Check:

• DynamoDB → new record stored
• S3 → attachment uploaded
• Email → SES notification received

If these work, the system is fully deployed.

=======================
### What You Built
=======================

A complete serverless feedback application using:

- Amazon CloudFront for CDN delivery

- Amazon S3 for static hosting and file storage

- Amazon API Gateway for REST API endpoints

- AWS Lambda for backend logic

- Amazon DynamoDB for NoSQL storage

- Amazon Simple Email Service for email notifications


==================================================
### GitHub Secrets/Env Variables Setup : Important
==================================================

1️⃣ GitHub Secrets (for CI/CD)

These are used by GitHub Actions to authenticate with AWS and deploy your frontend.

Go to your repository:

Settings → Secrets and variables → Actions → New repository secret

Create these secrets.
```
Required Secrets
AWS_ACCESS_KEY_ID
AWS_SECRET_ACCESS_KEY
AWS_REGION
S3_BUCKET
CLOUDFRONT_DIST_ID
```
Example values:
```
AWS_ACCESS_KEY_ID = AKIA...
AWS_SECRET_ACCESS_KEY = ********
AWS_REGION = us-east-1
S3_BUCKET = feedback-app-frontend
CLOUDFRONT_DIST_ID = E3ABCXYZ123
```
These are referenced in your workflow:
```
${{ secrets.AWS_ACCESS_KEY_ID }}
${{ secrets.S3_BUCKET }}
```

2️⃣ Lambda Environment Variables

These are used inside AWS Lambda when your code runs.

Go to:

Lambda → Configuration → Environment variables

Add these variables.

Lambda Variables
```
TABLE_NAME
BUCKET_NAME
ADMIN_EMAIL
```
Example:
```
TABLE_NAME = FeedbackTable
BUCKET_NAME = feedback-app-attachments-12345
ADMIN_EMAIL = your-email@gmail.com
```

3️⃣ IAM Policy for GitHub CI/CD User

The IAM user used by GitHub should only deploy frontend files and invalidate CloudFront.

Attach a policy like this:
```
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "s3:ListBucket",
        "s3:PutObject",
        "s3:DeleteObject"
      ],
      "Resource": [
        "arn:aws:s3:::feedback-app-frontend",
        "arn:aws:s3:::feedback-app-frontend/*"
      ]
    },
    {
      "Effect": "Allow",
      "Action": [
        "cloudfront:CreateInvalidation"
      ],
      "Resource": "*"
    }
  ]
}

```
This allows:

GitHub → upload frontend to S3
GitHub → invalidate CloudFront cache


```
| Location                     | Purpose                            |
| ---------------------------- | ---------------------------------- |
| GitHub Secrets               | CI/CD authentication               |
| Lambda Environment Variables | runtime configuration              |
| IAM Policies                 | permissions to access AWS services |

```


============
### Test ###
============

1️⃣ Test Lambda Directly

First confirm your AWS Lambda function works independently.

Open your Lambda → Test tab.

Create a test event like this:
```
{
  "body": "{\"name\":\"Test User\",\"email\":\"test@example.com\",\"message\":\"Testing feedback submission\"}"
}
```
Click Test.

Expected result:
```
Status code 200
```
Message:

Feedback submitted successfully

If Lambda fails, check CloudWatch logs inside Lambda → Monitor → Logs.

2️⃣ Verify DynamoDB Storage

Now check your Amazon DynamoDB table.

Open:

DynamoDB → Tables → FeedbackTable → Explore table items

You should see a record like:
```
feedback_id : uuid
name        : Test User
email       : test@example.com
message     : Testing feedback submission
created_at  : timestamp
```
If this exists, DynamoDB integration works.

3️⃣ Test S3 File Upload (Optional Attachment)

Next test attachment upload to Amazon S3.

Create a new Lambda test event including a small base64 file.

Example (shortened):
```
{
  "body": "{\"name\":\"Test User\",\"email\":\"test@example.com\",\"message\":\"Testing file upload\",\"file_base64\":\"data:application/pdf;base64,JVBERi0xLjQK...\"}"
}
```
Run the test again.

Then open your S3 bucket:

S3 → feedback-app-attachments → Objects

You should see a file like:
```
uuid.pdf
```
If yes, S3 upload works.

4️⃣ Verify Email Sending

Now check Amazon Simple Email Service.

After running Lambda tests, check your inbox.

You should receive a message like:
```
Subject: New Feedback Submission
```
If not:

Common causes:

• SES email not verified
• SES sandbox restrictions
• Wrong ADMIN_EMAIL value

5️⃣ Test API Gateway Endpoint

Now test your Amazon API Gateway REST endpoint.

Use Postman or curl.

Example curl command:
```
curl -X POST https://abc123.execute-api.us-east-1.amazonaws.com/prod/feedback \
-H "Content-Type: application/json" \
-d '{"name":"API Test","email":"test@example.com","message":"Testing API"}'
```
Expected response:
```
{
  "message": "Feedback submitted successfully"
}
```
If this works, API Gateway → Lambda integration works.

6️⃣ Test Frontend from S3

Now open your S3 website endpoint.

Example:
```
http://feedback-app-frontend.s3-website-us-east-1.amazonaws.com
```
Fill the form.

Submit feedback.

Expected:
• success message appears
• DynamoDB record created
• email notification received

If submission fails, open browser Developer Tools → Console to check errors.

7️⃣ Test CloudFront Distribution

Finally test the CDN through Amazon CloudFront.

Open CloudFront.

Copy the distribution URL:
```
https://d123abcxyz.cloudfront.net
```
Open it in the browser.

Submit feedback again.

Everything should work exactly the same.

If the site doesn’t update after changes, clear CloudFront cache:

CloudFront → Invalidations → Create Invalidation

Use:
```
/*
```
8️⃣ Final End-to-End Test

Now verify the entire system:

1️⃣ Open CloudFront URL
2️⃣ Submit feedback with optional PDF
3️⃣ Confirm:
  - DynamoDB record created
  - S3 file uploaded
  -  Email received

If all three happen, your system is fully operational.