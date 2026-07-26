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

<img width="214" height="230" alt="image" src="https://github.com/user-attachments/assets/0f25830f-bdac-4425-a428-8577d21a8410" />


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

==================
Notes:
What is Lambda?
AWS Lambda is a serverless compute service. It runs your Python code without you managing any servers.
In this project, Lambda will:
Read all files in the S3 bucket.
Check how old each file is.
Delete files older than the specified time.
Write the names of deleted files to CloudWatch Logs.

Why?
Lambda cannot access S3 unless AWS grants permission.
Permissions are given through an IAM Role.

=======================

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
Note: LambdaS3CleanupRole only has the AWSLambdaBasicExecutionRole policy attached. This policy only allows Lambda to write logs to CloudWatch, it does not allow access to your S3 bucket.

Step 4: Create inline policy
        1. Open the created role : 
        2. Click on add permissions on the right side
        3. click on inline policy

 <img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/76fb8a2a-9259-4487-a2c1-2a925e5ebdf4" />


 <img width="929" height="322" alt="image" src="https://github.com/user-attachments/assets/26ba643d-88c9-40f0-93e8-8de35a0e6416" />

   4. You'll see two tabs:
           Visual
           JSON
           Click JSON.

  Replace the Json text with the below text:
  Important: Replaced YOUR_BUCKET_NAME with the S3 bucket name: sandy-cleanup-bucket-2026

{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": "s3:ListBucket",
      "Resource": "arn:aws:s3:::sandy-cleanup-bucket-2026"
    },
    {
      "Effect": "Allow",
      "Action": "s3:DeleteObject",
      "Resource": "arn:aws:s3:::sandy-cleanup-bucket-2026/*"
    }
  ]
}

Save the policy
Click Next.
Give the policy a name: DeleteOldObjectsPolicy

<img width="925" height="391" alt="image" src="https://github.com/user-attachments/assets/932bbcf3-33ee-47d2-9223-3f345f402b97" />

Click Create policy. We will have 2 policies now.

<img width="704" height="214" alt="image" src="https://github.com/user-attachments/assets/f513ca65-0202-4cf2-87ee-8d065dead210" />

================================================ 
Note:
A Lambda function has no permissions by default.
This role gives Lambda permission to:
Write logs to CloudWatch.
List all files in your S3 bucket.
Delete files from your S3 bucket.
Without these permissions, the Lambda function would fail with an AccessDenied error.

Step 5: Create the Lambda Function
   
   1. Open Lambda
      In the AWS Console.
      Search for Lambda.
      Click Lambda.
      We will see the Lambda dashboard.

   2. Create a Function:

   Click: Create function
   

   <img width="938" height="412" alt="image" src="https://github.com/user-attachments/assets/c76f8eec-acd3-49b0-9ed2-6b7b97f82676" />

  3. Select "Author from scratch"

   Choose: Author from scratch, as we are writing our own Python code.

  4. Enter Function Details

     Field         | Value               |
   | ------------- | ------------------- |
   | Function Name | `S3CleanupFunction` |
   | Runtime       | `Python 3.12`       |
   | Architecture  | `x86_64` (default)  |

<img width="937" height="398" alt="image" src="https://github.com/user-attachments/assets/d44bafcf-0005-4870-b6a4-749c21520786" /> 

<img width="468" height="86" alt="image" src="https://github.com/user-attachments/assets/e57f0a07-8ce6-4dd6-b772-3b9844a867ec" /> 

5. Configure Permissions
   
 
   choose Custom execution role under Additional settings.
   Enable the Custom Execution Role option.
   <img width="316" height="25" alt="image" src="https://github.com/user-attachments/assets/459e59bb-a88c-4dd4-bf10-1b84a0144aca" />

   <img width="931" height="290" alt="image" src="https://github.com/user-attachments/assets/2e7e48ee-ae46-4fee-a924-d57e806d8389" />

   Change default execution role.
   Click the down arrow to expand.
   Select: LambdaS3CleanupRole
   Click Save and then click Create Function
   
   Note: This tells Lambda to use the IAM role you created earlier, which gives it permission to access your S3 bucket and write logs.

    AWS creates the Lambda function.
    After a few seconds, you'll be taken to the function page.

   <img width="917" height="374" alt="image" src="https://github.com/user-attachments/assets/20f6eeb1-3dc6-4a7d-abfd-863e8124c3c6" />

   We will see lambda_function.py in the funcation page.
   Select everything inside lambda_function.py and delete it.
   
   Paste the following code:

   import boto3
from datetime import datetime, timezone, timedelta

# Create an S3 client
s3 = boto3.client("s3")

# Your bucket name
BUCKET_NAME = "sandy-cleanup-bucket-2026"

# For testing: delete files older than 5 minutes
AGE_LIMIT = timedelta(minutes=5)

# For production, change to:
# AGE_LIMIT = timedelta(days=30)

def lambda_handler(event, context):

   # Get the current UTC time
   now = datetime.now(timezone.utc)

   # Create a paginator to list all objects
   paginator = s3.get_paginator("list_objects_v2")

   # Iterate through every page of objects
   for page in paginator.paginate(Bucket=BUCKET_NAME):

   # Skip if the bucket is empty
   if "Contents" not in page:
           continue

   # Check each object
   for obj in page["Contents"]:

   key = obj["Key"]
          last_modified = obj["LastModified"]

   # Calculate object age
   age = now - last_modified

   # Delete if older than the configured age
   if age > AGE_LIMIT:

   s3.delete_object(
   Bucket=BUCKET_NAME,
   Key=key
                )

   print(f"Deleted: {key}")

   return {
        "statusCode": 200,
        "body": "Cleanup completed."
    }

  ===================== 

  Notes:
  We need two libraries:
  boto3 → AWS SDK for Python. It lets the Lambda function communicate with S3.
  datetime → Used to calculate how old each file is.
  1. Therefore importing them, code has the below:
  
   import boto3
   from datetime import datetime, timezone, timedelta
  
  2. Create an S3 Client :
      s3 = boto3.client("s3")

       This creates a connection to the Amazon S3 service.
       Without this, Lambda cannot list or delete files.

   3. Bucket Name:
        BUCKET_NAME = "sandy-cleanup-bucket-2026"

        A Lambda function can access many AWS services.
        This tells it exactly which bucket to clean.

   4. Age Limit:
         AGE_LIMIT = timedelta(minutes=5)

      For testing, we don't want to wait 30 days.
      Later we can change to 30 Days as below:
         AGE_LIMIT = timedelta(days=30)

   5. Lambda Handler
          def lambda_handler(event, context):

        This is the entry point of the Lambda function.
        Whenever Lambda is triggered, AWS starts execution here.

   6. Get Current Time:
        now = datetime.now(timezone.utc)

      S3 stores the LastModified timestamp in UTC.
      Using UTC for both values ensures the comparison is correct.

   7. Create a Paginator:
         paginator = s3.get_paginator("list_objects_v2")

      Why use a paginator?
      S3 returns up to 1,000 objects per request.
      A paginator automatically fetches additional pages until all objects are processed, so no files are missed.

   8. Loop Through Every Page:
        for page in paginator.paginate(Bucket=BUCKET_NAME):

      This processes each page returned by S3.

   9. Check if the Bucket Is Empty:
         if "Contents" not in page:
         continue

       If the bucket has no objects, there won't be a "Contents" key. This avoids errors.

   10. Process Each Object:
           for obj in page["Contents"]:

       Each obj contains metadata such as:
         . Object name (Key)
         . Last modified date
         . Size

   11. Get the File Name:
         key = obj["Key"]

       eg: file1.txt

   12. Get the Last Modified Time:
         last_modified = obj["LastModified"]

       eg: 2026-07-26 16:30 UTC

   13. Calculate the File Age:
          age = now - last_modified

       If the current time is 6:40 PM and the file was modified at 6:30 PM, then:
          Age = 10 minutes

   14. Delete Old Files:
          if age > AGE_LIMIT:

       If the file is older than the configured limit (5 minutes for testing), it is deleted.

   15. Delete the Object:
           s3.delete_object(
           Bucket=BUCKET_NAME,
           Key=key
        )

   This removes the object from the bucket.

   16. Print the Deleted File:
          print(f"Deleted: {key}")

         This writes a message to CloudWatch Logs, for example:
              Deleted: file1.txt
              Deleted: report.pdf

   17. Return Success:
           return {
          "statusCode": 200,
          "body": "Cleanup completed."
      }

   This tells AWS that the function finished successfully.

=================================

 Step 8: Deploy the Code

   After pasting the code:
   Click Deploy, on left side of the code.

   <img width="464" height="352" alt="image" src="https://github.com/user-attachments/assets/ba81a3f8-eea9-4125-a7f9-72d5af05439c" />

<img width="932" height="136" alt="image" src="https://github.com/user-attachments/assets/42614c4d-7093-4ca5-8dbb-8bd56ff185d1" />

   Wait until you see a message like:  
     Successfully updated S3CleanupFunction 
     
===================================

 Step 9: Test the Lambda Function
         On the left side, you'll see the blue button: 
         
   <img width="493" height="400" alt="image" src="https://github.com/user-attachments/assets/a39c120d-0e31-4e48-97c7-57121569502d" /> 

   Click Test.
If AWS asks you to create a test event, You'll see a page similar to this: Create new test event.

<img width="327" height="337" alt="image" src="https://github.com/user-attachments/assets/470676e0-ee01-42db-b930-d980f7c0cc6e" />


Event Name: S3CleanupTest

<img width="371" height="170" alt="image" src="https://github.com/user-attachments/assets/480253de-a1a4-4eea-8d21-9b7d0e8c17d5" />

Event JSON
Leave it as: {}

<img width="318" height="102" alt="image" src="https://github.com/user-attachments/assets/ec1da941-1db1-4bc5-8a5e-8bcf185fd003" />

 Notes: Why is it empty?
        Our Lambda function doesn't require any input. It already knows:
           . which bucket to check
           . how old files should be
           . what to delete
       So an empty JSON object is enough.

  Click Save.
  Click Invoke

  <img width="314" height="104" alt="image" src="https://github.com/user-attachments/assets/19dbbc8f-946a-445c-9779-38187adf75bb" />

<img width="290" height="86" alt="image" src="https://github.com/user-attachments/assets/5515b226-2cda-4fc5-aaac-3a238620fbdf" /> 

============================== 

  Step 10:
       After Lambda finishes.
        You'll see something like:
           {
              "statusCode": 200,
              "body": "Cleanup completed."
           }


<img width="350" height="103" alt="image" src="https://github.com/user-attachments/assets/4c90fad0-2daa-4481-9972-8fea311e4044" /> 

In the Function Logs, we can see:
Deleted: file1.txt
Deleted: notes.docx.docx


<img width="444" height="147" alt="image" src="https://github.com/user-attachments/assets/4d5494bf-eae0-4a2e-b54b-834de4f37e84" />

========================== 

Step 11: Verify the Bucket

   Go back to:
   Amazon S3 → sandy-cleanup-bucket-2026
   Refresh the bucket.
   If some files were older than 5 minutes, they should no longer be listed.

   <img width="935" height="431" alt="image" src="https://github.com/user-attachments/assets/778c6101-6ca0-4e32-b050-93fa85dcc4c5" /> 

   Objects (0) means there are no files left in the bucket. The Bucket is empty now.
   This confirms that your Lambda function successfully deleted all the objects that matched the age condition.


========================= 

Step 12: Check CloudWatch Logs

If you want to see which files were deleted:
   Open CloudWatch.

   <img width="805" height="181" alt="image" src="https://github.com/user-attachments/assets/6ca210fd-b317-4927-b92c-b31040ac1534" />

   Click Log and then click Log Management.

   <img width="306" height="336" alt="image" src="https://github.com/user-attachments/assets/20396822-aab2-4395-9711-0ef8651a03af" /> 

   <img width="889" height="368" alt="image" src="https://github.com/user-attachments/assets/e9df1fc5-2f87-401a-8413-04e4f1f3901f" /> 

   From the listed Log Groups click this: /aws/lambda/S3CleanupFunction 
   Open the Log Stream
   Inside the log group you'll see one or more Log streams.
   Example: 2026/07/26/[$LATEST]xxxxxxxxxxxxxxxx
   Click the latest log stream. 

   <img width="906" height="324" alt="image" src="https://github.com/user-attachments/assets/cb05529f-24e1-44e5-98bf-a58ef90dda0f" /> 

View the Logs
You'll see the log entries.

<img width="732" height="304" alt="image" src="https://github.com/user-attachments/assets/8340f479-ec25-4177-94f9-8435bca8afdc" />


============================================= 
Step 13: Final Code Change:

   For testing, we used 5 minutes older files:
      AGE_LIMIT = timedelta(minutes=5)
   Now we will change it to 30 Days:   
      AGE_LIMIT = timedelta(days=30)


  Click Deploy after making the change. 

  <img width="943" height="404" alt="image" src="https://github.com/user-attachments/assets/9bb6bd57-54b7-4b98-9758-dffc33a4b779" /> 

      
     

   
