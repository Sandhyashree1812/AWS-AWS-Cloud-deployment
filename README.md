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


=======================================================================================
=======================================================================================
=======================================================================================

Assignment 2.
==================== 
2.Automated EBS Snapshot Creation and Cleanup
Objective: Automate EBS volume backups and delete snapshots older than a retention period.

Instructions:

EBS Setup: Identify or create an EBS volume; note the volume ID.

Lambda IAM Role: Inline policy with ec2:CreateSnapshot, ec2:DescribeSnapshots, ec2:DeleteSnapshot, ec2:CreateTags.

Lambda Function (Boto3):

Create a snapshot of the specified volume and tag it (e.g., CreatedBy=Lambda-Backup).

List snapshots with that tag (owned by your account) and delete those older than 30 days.

Print IDs of created and deleted snapshots.

EventBridge: Schedule the function weekly.

Testing: Trigger manually; confirm snapshot creation and cleanup in the EC2 console.

Discussion point: AWS Data Lifecycle Manager (DLM) does this natively. Note in your documentation when Lambda is still the better choice (custom retention logic, cross-account copies, notifications).


================================== 

 Architecture:

   EventBridge (Weekly Schedule)
                │
                ▼
         Lambda Function
                │
    ┌───────────┴────────────┐
    │                        │
    ▼                        ▼
Create Snapshot          Delete Old Snapshots
    │                        │
    ▼                        ▼
 EBS Volume          Older than 30 Days

 <img width="322" height="183" alt="image" src="https://github.com/user-attachments/assets/31c8e948-efbf-47b0-810d-d7686028ac35" /> 

 =========================== 

 Steps:

 Step 1: Create an EC2 instance

Note: An EBS (Elastic Block Store) volume is attached to an EC2 instance. We need an EBS volume because Lambda will create snapshots (backups) of that volume.

  1. search for EC2 in AWS console

     <img width="784" height="164" alt="image" src="https://github.com/user-attachments/assets/1121971c-8e98-4ca2-942d-70cbd46bca71" />

     Click Launch instance.

      Give Name: SnapshotDemo

     <img width="606" height="167" alt="image" src="https://github.com/user-attachments/assets/13fbcd7f-0016-4886-bdad-966dbe4d7ac2" />
     <img width="98" height="275" alt="image" src="https://github.com/user-attachments/assets/e3f105f2-ff04-4147-bf92-8dddbcdb6533" />
     <img width="586" height="85" alt="image" src="https://github.com/user-attachments/assets/58a3d376-ac96-475f-b5d4-486ebb25fe1a" />
     <img width="277" height="36" alt="image" src="https://github.com/user-attachments/assets/9e2ae1af-85f9-4ef1-899b-4b39595c1911" />

     <img width="569" height="141" alt="image" src="https://github.com/user-attachments/assets/ddbd192b-75bb-44ef-9225-05fd1fa522c8" />

     <img width="442" height="322" alt="image" src="https://github.com/user-attachments/assets/297fa9d9-6371-4078-91da-13ea38f35756" />

     The .pem file will be downloaded.
     Keep everything else default.
     Launch instance.

     <img width="928" height="136" alt="image" src="https://github.com/user-attachments/assets/84b8f11b-13cd-430e-bab4-ead42bb6cdda" />

Step 2. Find the EBS Volume ID

   Open the EC2 Dashboard
   In the AWS Console, click EC2 (top-left breadcrumb or search for EC2).
   In the left navigation pane, scroll down until you see: Elastic Block Store, Click Volumes. 

<img width="914" height="242" alt="image" src="https://github.com/user-attachments/assets/989be1f1-d9ca-459e-a8f2-7501cff5085c" /> 

  Copy the Volume ID

Step 3: Create an IAM Role for Lambda  
        By default, a Lambda function cannot create or delete EBS snapshots.
        We need to give Lambda permission by creating an IAM Role. 
        Without this role, Lambda will return an AccessDenied error.

   1. Open IAM :

       <img width="803" height="154" alt="image" src="https://github.com/user-attachments/assets/8bac22bc-b4b1-4bbd-a98f-d0ac8c0a5b6a" />

       Open Roles, Create a New Role:
        Click the Create role button.
        Select Trusted Entity:
          Trusted entity type: AWS service
          Use case: Lambda.
      Attach Basic Lambda Policy
      In the search box, type: AWSLambdaBasicExecutionRole

 
      <img width="830" height="226" alt="image" src="https://github.com/user-attachments/assets/266e870b-33eb-4ac7-a4c9-d41249065ef9" />


       <img width="730" height="284" alt="image" src="https://github.com/user-attachments/assets/b650ebf0-246b-40dd-a0dc-54bfac4defc6" />

      Give Name : LambdaEBSSnapshotRole, and click create role.

      ==================

      Step 4: Add an Inline Policy to the IAM Role

      As Currently, your role can only write logs to CloudWatch.

   1. Open the Role
   2. Add an Inline Policy:
       Click Add permissions
        Create inline policy
        Select the JSON Editor, delete the code and the below code :
      {
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "ec2:CreateSnapshot",
        "ec2:DescribeSnapshots",
        "ec2:DeleteSnapshot",
        "ec2:CreateTags"
      ],
      "Resource": "*"
    }
  ]
}


Click Next and give a policy Name : EBSSnapshotPolicy

<img width="942" height="221" alt="image" src="https://github.com/user-attachments/assets/41d38291-5ff6-46e9-9fdf-151afb2e677e" /> 

<img width="933" height="322" alt="image" src="https://github.com/user-attachments/assets/7863f08c-00b1-49c0-af13-11fa94c9f270" /> 

Step 5: Create the Lambda Function 

  notes: Why?
      The Lambda function is the program that will:
      Create a snapshot of your EBS volume.
      Tag the snapshot.
      Find old snapshots.
      Delete snapshots older than 30 days.

   Instead of creating an EC2 instance to run your Python code, AWS runs the code for you only when it is needed.

   <img width="295" height="139" alt="image" src="https://github.com/user-attachments/assets/0463402a-64a3-4767-9335-ab70af60e6f6" />


<img width="815" height="149" alt="image" src="https://github.com/user-attachments/assets/f14c49c7-d373-4562-9ecb-d905498f2e3a" />

   In AWS Console searh for Lambda and click it.
   Click Create a Function.
   Select Author from Scratch.
   Give Function Name: EBSSnapshotManager 

   <img width="557" height="307" alt="image" src="https://github.com/user-attachments/assets/b92e1f3a-4cf5-41bf-9a80-e8d61e478891" />

   Runtime : Python 3.12
   Click Additional Settings.
   Keep the default architecture:  x86_64 (default)

   <img width="545" height="274" alt="image" src="https://github.com/user-attachments/assets/3087f8fe-fef4-4dae-9bb7-12b4334fb653" />

  
   Click Custom execution role, turn it on.
   Use an existing role: LambdaEBSSnapshotRole and click save.

   <img width="364" height="286" alt="image" src="https://github.com/user-attachments/assets/2ec9641f-76a7-4b67-ba7b-bbf1252b0793" /> 


   Click Create Function. 

   <img width="933" height="337" alt="Screenshot 2026-07-27 000653" src="https://github.com/user-attachments/assets/dab0910e-4682-4be0-8cdc-9f4a8132a65b" />

   
============================
   Notes: Why?

   This role contains the permissions that allow Lambda to:

   Create snapshots
   Delete snapshots
   Describe snapshots
   Add tags
   Write logs to CloudWatch
   Without this role, Lambda would receive an AccessDenied error.
================================

   Step 6: Replace the Default Code 

   De;ete the code in Json and paste the below Code:

import boto3
from datetime import datetime, timezone, timedelta

# Create EC2 client
ec2 = boto3.client('ec2')

# Replace this with your Volume ID
VOLUME_ID = "vol-051c35e5359192101"

# Retention period
RETENTION_DAYS = 30


def lambda_handler(event, context):

   # Create snapshot
   response = ec2.create_snapshot(
       VolumeId=VOLUME_ID,
        Description="Automated Lambda Backup"
    )

   snapshot_id = response["SnapshotId"]

 # Add tag
   ec2.create_tags(
        Resources=[snapshot_id],
        Tags=[
            {
                "Key": "CreatedBy",
                "Value": "Lambda-Backup"
            }
        ]
    )

   print(f"Created Snapshot: {snapshot_id}")

   # Find snapshots created by Lambda
   snapshots = ec2.describe_snapshots(
        OwnerIds=["self"],
        Filters=[
            {
                "Name": "tag:CreatedBy",
                "Values": ["Lambda-Backup"]
            }
        ]
    )

   # Calculate cutoff date
   cutoff = datetime.now(timezone.utc) - timedelta(days=RETENTION_DAYS)

   deleted = []

   # Delete old snapshots
   for snap in snapshots["Snapshots"]:

   if snap["StartTime"] < cutoff:

   ec2.delete_snapshot(
                SnapshotId=snap["SnapshotId"]
            )

   deleted.append(snap["SnapshotId"])

   print("Deleted Snapshots:", deleted)

   return {
        "created_snapshot": snapshot_id,
        "deleted_snapshots": deleted
    }
     
   

====================== 

Step 7: Deploy the Code

After pasting the code:

Click the Deploy button.
Wait for the success message: Successfully updated the function. 

Step 8: Test the Lambda Function

  Click Invoke.
  Click Create a Test Event.
  Event Name: TestSnapshot

  <img width="902" height="193" alt="Screenshot 2026-07-27 010716" src="https://github.com/user-attachments/assets/aecea0c9-7640-4e90-87e7-c1f0a6831e85" />

  Change the code to : {} 

  <img width="311" height="173" alt="Screenshot 2026-07-27 010822" src="https://github.com/user-attachments/assets/50f03119-7053-4ca8-9b3c-a2682fd1d894" />

  Click save. 
  Click Test/Invoke again.
<img width="934" height="356" alt="Screenshot 2026-07-27 001342" src="https://github.com/user-attachments/assets/403dc8cd-d11a-4243-8960-fe12a9e4bc71" /> 

<img width="328" height="65" alt="Screenshot 2026-07-27 010953" src="https://github.com/user-attachments/assets/b65a5d05-0eaa-46f4-84fa-493454e182fe" />

<img width="340" height="137" alt="Screenshot 2026-07-27 011024" src="https://github.com/user-attachments/assets/9adf4d9f-fc82-4664-99f8-cc922e7f187a" /> 

<img width="614" height="134" alt="Screenshot 2026-07-27 011113" src="https://github.com/user-attachments/assets/9b2a1309-fbbe-4b39-82ba-3fbc5c807c8d" /> 

This is the new snapshot created by Lambda. 
deleted_snapshots : [] , this is empty as this is the first time the fuction is running. An empty list means there were no snapshots older than 30 days to delete. This is normal on the first run. 

==================== 

Notes:
When we clicked Test, the Lambda function:

. Connected to EC2 using Boto3.
. Found your EBS volume
. Created a snapshot.
. Tagged it with:
  CreatedBy = Lambda-Backup(Description shows this)

  =========================== 

Step 10: Verify the Tag 

   Let's make sure the snapshot has the correct tag.
   Go to EC2.
   Click Snapshots.
   Click on the snapshot that Lambda created.
   Scroll down to the Tags section.

<img width="906" height="350" alt="Screenshot 2026-07-27 011945" src="https://github.com/user-attachments/assets/2d22002d-babb-45f6-8a1f-96ea7081d063" /> 

<img width="932" height="344" alt="Screenshot 2026-07-27 012623" src="https://github.com/user-attachments/assets/5a41b3ca-313f-4016-9a85-6337f3db416d" />

Lambda-Backup: This ensures it only deletes snapshots created by your Lambda, not any other snapshots in your account.

=============== 

Step 11: Test the Cleanup Feature

Since waiting 30 days isn't practical, we can temporarily change the retention period.
In the Lambda code, change: RETENTION_DAYS = 30 to RETENTION_DAYS = 0 

<img width="569" height="250" alt="Screenshot 2026-07-27 013536" src="https://github.com/user-attachments/assets/0ab6e859-f220-449b-a07c-2bd1c132a789" />


   Then:
   . Click Deploy. 
   . Click Test again. 
   This allows the function to delete snapshots that are older than the current execution time.

   <img width="312" height="70" alt="Screenshot 2026-07-27 013714" src="https://github.com/user-attachments/assets/e00eeaaa-513d-4777-a5f8-ffb2d20108fb" /> 

   <img width="301" height="112" alt="Screenshot 2026-07-27 013747" src="https://github.com/user-attachments/assets/33b6c23e-8f87-44f2-a2f8-de9a2220dd75" /> 

   <img width="632" height="164" alt="Screenshot 2026-07-27 013903" src="https://github.com/user-attachments/assets/1fee2d8b-890f-4bba-9d50-5f9b0450c134" /> 

   Click on cloudwatch, go to Log management then in Log Streams, click the latest log to see the details

<img width="919" height="278" alt="Screenshot 2026-07-27 014210" src="https://github.com/user-attachments/assets/fd9e7636-742f-4c48-9ea2-fa63c8dc77c7" /> 

<img width="922" height="323" alt="Screenshot 2026-07-27 014342" src="https://github.com/user-attachments/assets/ea56a377-95bb-4750-b318-d7a57fe12e75" /> 

<img width="734" height="340" alt="Screenshot 2026-07-27 014417" src="https://github.com/user-attachments/assets/7e598e49-d58e-4211-88aa-6b987b7505e8" />

========================================== 

Step 12: Schedule the Lambda with EventBridge.

===============================
   notes:

   What is EventBridge?
   Amazon EventBridge is a scheduling service.
   It automatically runs your Lambda function at a specific time.
   Instead of clicking the Test button every week, EventBridge will do it automatically.

 =================================
 
   Now let's automate it so you don't have to click Test manually.

   . In AWS Console, search for EventBridge and open Amazon EventBridge.
   . click Rules, then click Create Rule. 
   . Keep Event Bus as Default.

   <img width="946" height="368" alt="image" src="https://github.com/user-attachments/assets/327abd42-3b5a-4ea2-b2d4-bcd2a30f6ce1" />

  . Click the tab Sceduled rules.
  . Click go to Sceduled Rules
  . Click Crate Schedule Rule

  Rule name: WeeklyEBSSnapshot
  Description (Optional): Creates an EBS snapshot every week.
  Leave the other settings as their default values.
  Click Next.

  <img width="952" height="238" alt="image" src="https://github.com/user-attachments/assets/f465750e-0ae2-4f9c-89d7-bd6b952d28af" />

. Select the Schedule Pattern: A fine-grained schedule that runs at a specific time, such as 8:00 a.m. PST on the first Monday of every month.

. In Cron Expression, enter : 0 0 ? * SUN *





 
















     


     

   
