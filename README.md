# AWS-AWS-Cloud-deployment
1.Automated S3 Bucket Cleanup (Objects Older Than 30 Days)
Objective: Automate deletion of stale objects in an S3 bucket.
Task: Delete files older than 30 days in a specific bucket.
Instructions:
S3 Setup: Create a bucket and upload several files. (Since you can't easily create "old" objects, temporarily lower the age threshold to minutes for testing — then set it back to 30 days in the final code.)

Lambda IAM Role: Inline policy with s3:ListBucket and s3:DeleteObject scoped to your bucket.

Lambda Function (Python 3.12+, Boto3):

List objects in the bucket (use the paginator — never assume one page of results).

Compare each object's LastModified (timezone-aware) with the current UTC time.

Delete objects older than 30 days.

Print the names of deleted objects.

Testing: Manually trigger and confirm only newer files remain.

Discussion point (include in your documentation): In production, S3 Lifecycle Rules handle this natively with zero code. Explain in 2–3 lines when you'd use Lambda instead (e.g., conditional logic, naming patterns, cross-service actions).

========================================== 

Assignment 1 : Automated S3 Bucket Cleanup (Objects Older Than 30 Days)

=====================================

1. Architecture:

   Upload Files
        │
        ▼
   Amazon S3 Bucket
        │
        ▼
   AWS Lambda Function
        │
   Lists all objects
        │
Checks LastModified Date
        │
  Deletes old objects
        │
  Prints deleted files

==========================================

Steps
Step 1: Create an S3 Bucket
   Why?:
   S3 (Simple Storage Service) stores files (called objects).
   Our Lambda function will delete old files from this bucket.

   1.Login to AWS account.
   2. Search for S3
   3. Click Amazon S3
   4. Click Create bucket
   5. Choose the region : AWS Region as below
   US East (N. Virginia) us-east-1 , Leave remaining settings as default.
=================================== 
Step 2. Open your bucket. Click Upload
    1. Upload 3–5 files :
        file1.txt
        world.jpg
        report.pdf
        notes.docx
==================================        
        
Step 3: Create IAM Role for Lambda
Why?
Lambda cannot access S3 unless AWS grants permission.
Permissions are given through an IAM Role.

  1. Open IAM
  2. Click Roles
       Choose AWS Service
       Choose Lambda
     <img width="830" height="401" alt="image" src="https://github.com/user-attachments/assets/03006b6c-60eb-4fab-933c-c825e78be91d" />

     <img width="941" height="386" alt="image" src="https://github.com/user-attachments/assets/8ddcf8f5-1337-4077-9b68-21683598bfaf" />

       Click Next
  3. In Add permissions, add policy : AWSLambdaBasicExecutionRole
     
     <img width="935" height="374" alt="image" src="https://github.com/user-attachments/assets/869b9145-8413-4191-a207-fd51962b45e8" />

       Click Next
  4. Give a Role name : LambdaS3CleanupRole
  5. Create Role

Step 4: Create inline policy
        1. Open the created role : 
        2. Click on add permissions on the right side
        3. click on inline policy
        4. You'll see two tabs:
           Visual
           JSON
           Click JSON.

  Replace the Json text with the below text:

   {
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": "s3:ListBucket",
      "Resource": "arn:aws:s3:::YOUR_BUCKET_NAME"
    },
    {
      "Effect": "Allow",
      "Action": "s3:DeleteObject",
      "Resource": "arn:aws:s3:::YOUR_BUCKET_NAME/*"
    }
  ]
}

   <img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/76fb8a2a-9259-4487-a2c1-2a925e5ebdf4" />


 <img width="929" height="322" alt="image" src="https://github.com/user-attachments/assets/26ba643d-88c9-40f0-93e8-8de35a0e6416" />


   
