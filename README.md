<img width="929" height="205" alt="image" src="https://github.com/user-attachments/assets/0309981d-9662-4e98-a786-aed49b86040a" /># AWS-AWS-Cloud-deployment
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

<img width="949" height="350" alt="image" src="https://github.com/user-attachments/assets/f36e5a22-7aee-4475-9d9e-7bbedcc40f1a" />


Click Next, after that, you'll reach the Target page.
Select:
Target type: AWS service
Service: Lambda
Function: EBSSnapshotManager
Then click Next, we will reach Target Page.
Target type : AWS service


<img width="949" height="341" alt="image" src="https://github.com/user-attachments/assets/79b19b28-419d-414b-9f74-65309609feba" />


<img width="935" height="407" alt="image" src="https://github.com/user-attachments/assets/ab5c5210-2fce-47ce-84d1-d4eb818598b4" />

=======================
Notes:
What is a Target?
A Target is the AWS resource that EventBridge will execute when the schedule is triggered.

========================= 

<img width="218" height="208" alt="image" src="https://github.com/user-attachments/assets/03bb599d-d823-43de-8005-18370b8564d8" />

So, the target is your Lambda function.
Select the AWS Service.
Select Target: Lambda Function
Select the Target Location: Target in this account
Function : EBSSnapshotManager 

<img width="922" height="295" alt="image" src="https://github.com/user-attachments/assets/81009b52-7de4-41b3-bc93-4a07ab87cb64" />

Why?
EventBridge needs permission to invoke (start) your Lambda function. AWS will automatically handle this permission for you.

=============== 
Notes:
Why?
Your Lambda function (EBSSnapshotManager) is in the same AWS account, so EventBridge should invoke it directly.

================== 

Permissions: Use execution role (recommended)
Execution Role: Create a new role for this specific resource 
AWS has generated a role name such as: Amazon_EventBridge_Invoke_Lambda_1015727220

<img width="930" height="347" alt="image" src="https://github.com/user-attachments/assets/6d4aea86-6620-410f-9c7f-dbb36d81e15f" />

=========== 
Notes:
What is this role used for?
This role allows EventBridge → Lambda communication.
Remember, this is different from the LambdaEBSSnapshotRole.

<img width="529" height="128" alt="image" src="https://github.com/user-attachments/assets/0088c77d-41b7-47fa-88de-2abe62a85387" />

================= 

Leave everything at the default values.
Click Next, we will see configure tags

Notes:
What are Tags?
A tag is a label that helps you organize AWS resources.
A tag consists of:
. Key
. Value

<img width="379" height="169" alt="image" src="https://github.com/user-attachments/assets/80211b4d-1c96-48e6-bf9b-1b4269a928c4" />


Tags are mainly used for:

Organizing resources
Searching for resources
Cost tracking
Resource management

================ 

This assignment does not require EventBridge rule tags. Leave all default as it is. Click Next.

We will see Review and Create Page. Click the orange Create rule button.

<img width="931" height="347" alt="image" src="https://github.com/user-attachments/assets/bb158da1-f3a4-4267-872f-200bb1d521ff" />

<img width="572" height="339" alt="image" src="https://github.com/user-attachments/assets/75277440-c3de-4073-a822-002a3ecdd6ab" />

<img width="531" height="267" alt="image" src="https://github.com/user-attachments/assets/23c3d27b-c5b8-4282-a11b-a49652040a99" />


 <img width="545" height="234" alt="image" src="https://github.com/user-attachments/assets/9fb7ebb8-e9f3-4f33-a65d-87d59f59e892" />


Rule is created successfully:

<img width="935" height="407" alt="image" src="https://github.com/user-attachments/assets/aba914b5-6055-4f84-b6b3-f4d24c310ef2" />

<img width="437" height="314" alt="image" src="https://github.com/user-attachments/assets/ba6000a5-3e1e-4784-92ce-7c8a26bca311" />


=============== 

Final Verification

After the rule is created:
Go to Amazon EventBridge.
Click Rules.

<img width="188" height="261" alt="image" src="https://github.com/user-attachments/assets/d293a059-02de-445c-aae9-67887efc295c" />


Click the Scheduled Rules tab.

<img width="715" height="325" alt="image" src="https://github.com/user-attachments/assets/54622985-0c3b-4ee2-a1e9-2aaa7f87b50a" />


You should see your rule with:
Status: Enabled
Target: EBSSnapshotManager 

<img width="729" height="201" alt="image" src="https://github.com/user-attachments/assets/8e30835e-2272-4512-9e2f-ce55b6184b0c" />

========================================== 
=======================================================================================
======================================================================================
====================================================================================== 

3. Auto-Tagging EC2 Instances on Launch
Objective: Automatically tag newly launched EC2 instances for resource tracking, ownership, and cost allocation.

Instructions:

Lambda IAM Role: Inline policy with ec2:CreateTags and ec2:DescribeInstances.

Lambda Function (Boto3):

Extract the instance ID from the EventBridge event (detail.instance-id).

Tag the instance with LaunchDate=<current date> and a custom tag (e.g., Owner or Environment).

Print a confirmation message.

EventBridge Rule: Create a rule matching event pattern — source aws.ec2, detail-type EC2 Instance State-change Notification, state running — with the Lambda as target.

Testing: Launch a new instance; after a short delay, confirm the tags appear.

Bonus: Extract the launching IAM user from CloudTrail events and add an Owner tag automatically — this is a popular interview scenario.

======================================= 
======================================

Assignment 3.

=========================== 

Architecture:

Launch EC2 Instance
        │
        ▼
EC2 changes state → Running
        │
        ▼
EventBridge Rule detects event
        │
        ▼
Lambda Function executes
        │
        ▼
Lambda adds tags to EC2 Instance 



<img width="465" height="257" alt="image" src="https://github.com/user-attachments/assets/c704f377-72be-4f39-bc4f-814b2fe9bdb9" />

========================== 

Steps:

Step 1: Create an IAM Role for Lambda

===================================
Notes:
The Lambda function needs permission to:
. Read EC2 instance details
. Add tags to EC2 instances
============================= 

   . Open IAM (Identity and Access Management )
   . Go to
   . AWS Console → IAM → Roles
   . Click Create Role 

   <img width="959" height="204" alt="image" src="https://github.com/user-attachments/assets/04358b08-6927-4cb8-8f28-7ed0b3c8ff54" />


   Trusted Entity:
      Choose: AWS Service
      Select: Lambda
      Click Next 

   <img width="842" height="352" alt="image" src="https://github.com/user-attachments/assets/e6f90711-3465-496f-8122-fbdc81c457db" />  

   <img width="717" height="275" alt="image" src="https://github.com/user-attachments/assets/69eabe68-2947-4fd3-a626-f03a72aa2f53" />


   In Add permissions:
   Click Create Inline Policy.
   In Jason Code Paste the below code:

   {
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "ec2:CreateTags",
        "ec2:DescribeInstances"
      ],
      "Resource": "*"
    }
  ]
}


   <img width="432" height="322" alt="image" src="https://github.com/user-attachments/assets/1691ed39-ce93-479e-92bc-4e75a7e8d124" /> 

   
   Click Set Permissions Boundary.
   Role Name: EC2AutoTagRole


   <img width="658" height="283" alt="image" src="https://github.com/user-attachments/assets/59b0b5ae-4d2b-4571-9077-5459963de87d" /> 

   Select Trusted entities, edit this.
   Select Trusted Entity Type: AWS Service.
   Use Case: Lambda.
   Click next.
   We reach the previous page, again click Next.
   Add Permissions:
   Inline Policy Name: EC2AutoTagPolicy
   We already see Service: EC2
   Access level: Limited: List, Tagging
   Resource: All resources
   Click Create Role.

   <img width="693" height="337" alt="image" src="https://github.com/user-attachments/assets/cab6395d-a95f-43d9-bf2e-bdb1f14d51f9" /> 

   <img width="776" height="206" alt="image" src="https://github.com/user-attachments/assets/a8743ea9-d92c-414b-a494-4108d13028e0" />

=========================== 

Step 2: Create the Lambda Function

   Search Lambda and Click Lamda.
   Click Create New Function.
   Select Author from scratch.
   Function Name: EC2AutoTagFunction
   Select Runtime: Python 3.12
   Leave Architecture as: x86_64
   Click enable Custom Execution Role
   Execution Role: Use an existing Execution role
   From Drop down select: EC2AutoTagRole

   <img width="387" height="196" alt="image" src="https://github.com/user-attachments/assets/6a8430ea-c941-41d6-a28a-df08b3e0f6df" />  

   <img width="353" height="332" alt="image" src="https://github.com/user-attachments/assets/9d9d0135-419a-4451-a2b4-e84cd5d65444" />

   Click Create Function.

====================================== 
Step 3: Write the Lambda Function Code
   We will see a Code Tab opened. 
   In the file: lambda_function.py delete the code and paste the below Code. 

import json
import boto3
from datetime import datetime

# Create an EC2 client
ec2 = boto3.client('ec2')

def lambda_handler(event, context):

   # Print the incoming event to CloudWatch Logs
   print(json.dumps(event))

   # Get the EC2 Instance ID from the EventBridge event
   instance_id = event['detail']['instance-id']

   # Get today's date
   launch_date = datetime.utcnow().strftime("%Y-%m-%d")

   # Add tags to the EC2 instance
   ec2.create_tags(
        Resources=[instance_id],
        Tags=[
            {
                'Key': 'LaunchDate',
                'Value': launch_date
            },
            {
                'Key': 'Environment',
                'Value': 'Dev'
            }
        ]
    )

   print(f"Successfully tagged instance: {instance_id}")

   return {
        "statusCode": 200,
        "body": "Tags added successfully."
    }




================================ 

Step 4: Deploy the Function 

   Click the Deploy button near the top of the page. 

   <img width="947" height="286" alt="image" src="https://github.com/user-attachments/assets/711dd36a-c34b-40b8-a552-a9ac9cf4b0ce" />

==================== 

Step 7: Create an EventBridge Rule

   Open Amazon Event Bridge.
   Click Rules.
   Event Bus: Default.
   Click Create Rule.
   
   Select Advanced builder.
   Give Rule Name: EC2AutoTagRule
   Description: Automatically tags EC2 instances when they enter running state.
   Event Bus Name: default 

============ 

   Notes:
   Why default?
   AWS services like EC2 send events to the default EventBridge bus.
   We are listening for EC2 state changes.

   ============= 
   
   Click Next

<img width="931" height="372" alt="image" src="https://github.com/user-attachments/assets/3764ebe1-47f3-4a26-95af-9daa4105f6c0" />

   ======================== 
   
   Step 8: Create Event Pattern

   This step tells EventBridge which events should trigger Lambda.
   
   In Build Event Pattern: 
   
   Event Source: AWS events or EventBridge partner events 

   <img width="719" height="359" alt="image" src="https://github.com/user-attachments/assets/5919afa4-289a-480a-8412-6f4ebff7709e" />



   Event Pattern: USe Pattern Form 

   <img width="518" height="244" alt="image" src="https://github.com/user-attachments/assets/6f7ebff8-7e71-4e83-bf2a-03ba04fe746e" />


   Event Source: AWS services
   AWS Service: EC2

   <img width="194" height="224" alt="image" src="https://github.com/user-attachments/assets/9e5818a7-9775-4c62-82e8-4a85c4f61f05" />


   Event Type: EC2 Instance State-change Notification
   Event Type Specification 1:
   Specific state(s):
   running 

   <img width="213" height="287" alt="image" src="https://github.com/user-attachments/assets/e19f5448-7df3-4b19-aca7-4530e4dd5c8d" />


Click Next.
We will see Targets Page.
1. Target type: AWS Services
2. Select a target: : Lambda function
3. Function: EC2AutoTagFunction
4. Click Next




   <img width="738" height="273" alt="image" src="https://github.com/user-attachments/assets/ff53c8d1-afc5-4c71-a3b6-d839d72a63e5" />


<img width="518" height="357" alt="image" src="https://github.com/user-attachments/assets/22489acd-3476-409d-afee-b6d838cbb4ae" />


<img width="735" height="369" alt="image" src="https://github.com/user-attachments/assets/ba7a9c7b-dffc-4117-b739-a24e7ab776e0" />


<img width="940" height="414" alt="image" src="https://github.com/user-attachments/assets/8e7e9902-563b-4c41-ae28-c1116d1d40e1" />


Now we will be in tags Page, Click Next.
We will see Review and Crate Page.
Click Create Role.

<img width="700" height="296" alt="image" src="https://github.com/user-attachments/assets/5e1bf034-58cc-4cb4-b82c-bc9c050d66bc" />


<img width="515" height="248" alt="image" src="https://github.com/user-attachments/assets/dd14817f-c6e2-4a5f-82f4-b70d5855c84d" />


<img width="527" height="262" alt="image" src="https://github.com/user-attachments/assets/52344dc2-bf1e-4521-af29-2ece000b02a5" />


<img width="719" height="361" alt="image" src="https://github.com/user-attachments/assets/3f239931-a503-47c3-a6fd-8761c7afdd1c" />


<img width="929" height="205" alt="image" src="https://github.com/user-attachments/assets/0cd88b15-db63-4ba5-8b59-9c245f6c21fd" />


<img width="191" height="175" alt="image" src="https://github.com/user-attachments/assets/e19369bd-84ad-4d4e-873a-46f60758c641" />

Check for Policy:
In Lambda, Click EC2AutoTagFunction, Click configuration, click permissions. Click on vie Policy. If policy is not added, then follow below steps:
Click add permissions.
Choose AWS Service.
Statement ID: AllowEventBridgeInvoke
Principal: events.amazonaws.com
Action: lambda:InvokeFunction
Click save.


<img width="928" height="350" alt="image" src="https://github.com/user-attachments/assets/77045881-f02f-49ce-b3f8-0cab79f11851" />

<img width="929" height="362" alt="image" src="https://github.com/user-attachments/assets/3fbd5856-7291-437d-8b0b-5522550260b4" />


<img width="626" height="272" alt="image" src="https://github.com/user-attachments/assets/850bd7ce-ba19-4292-8365-7b04305b191e" />


Check the policy details:

<img width="914" height="312" alt="image" src="https://github.com/user-attachments/assets/53ef07f0-cd89-4463-b681-1f2cabf400d2" />

Step 9:
Launch a new EC2 instance:
Click EC2.
Click Instances.
Click the orange Launch instances button.
Name: Auto-Tag-Test
Keep everything as default. 
Create a new Key Pair if prompted:
Keypair Name: Auto Tag New Key Pair
Key pair type: RSA
Private key file format: .pem
Click Create key pair.



<img width="434" height="287" alt="image" src="https://github.com/user-attachments/assets/c17dbf7e-8ec1-4543-8118-4335e673afa7" />

Click Launch instance.

<img width="899" height="260" alt="image" src="https://github.com/user-attachments/assets/2ddbfd2c-1491-45b2-8824-fe1859a1f8f8" />


Click instance and wait till instance state shows: Running 
Status Check: Check passed

<img width="739" height="313" alt="image" src="https://github.com/user-attachments/assets/9feae8ea-fa5a-4c40-858e-a9fa1c52acda" />


Add Another Policy:

In IAM. Roles. EC2AutoTagRole, Permissions, Click add Permissions, click attach policy.
Search for: AWSLambdaBasicExecutionRole 
Click Attach.

<img width="935" height="283" alt="image" src="https://github.com/user-attachments/assets/d0465541-e553-46e8-97ef-7c09903625b1" />


Launch a new EC2 Instance:

<img width="890" height="250" alt="image" src="https://github.com/user-attachments/assets/9eb015b9-2979-4157-9d85-aafc14204f65" />

Instance status shows: Running


<img width="710" height="265" alt="image" src="https://github.com/user-attachments/assets/4997e727-7175-440d-836a-dfa0f89a523b" />  

Select the instance and click on tags tag


Key                        Value

LaunchDate               2026-08-07
Name                 Auto tag Test New
Environment              Production
Owner                      DevOps

<img width="742" height="56" alt="image" src="https://github.com/user-attachments/assets/26897759-dc37-4128-89ef-afd9178d0ccc" />


To Check Whether Lambda Was Triggered:
Go to AWS Console, click CloudWatch.
On the left-hand side, under Logs click Log Management, select Log Groups tab. Click the /aws/lambda/EC2AutoTagFunction function. 
Click Log STreams.
Click the latest log.

<img width="953" height="367" alt="image" src="https://github.com/user-attachments/assets/1dd508d3-4537-4294-912a-e55e54ebd72e" />


<img width="740" height="323" alt="image" src="https://github.com/user-attachments/assets/b60f140c-3252-4064-84e0-cab0d3fdcc57" /> 

<img width="377" height="261" alt="image" src="https://github.com/user-attachments/assets/d70a79a3-baa3-4a2b-bbe9-ea7aba322cae" /> 

==================== 

What you've successfully completed
. Created the Lambda function
. Created the IAM execution role
. Added EC2 permissions (DescribeInstances and CreateTags)
. Added the AWSLambdaBasicExecutionRole policy for CloudWatch logging
. Created the EventBridge rule
. Configured the EC2 Running event pattern
. Connected EventBridge to the Lambda function
. Updated and deployed the Lambda code
. Launched an EC2 instance
. Verified automatic tagging
. Verified CloudWatch logs
============================================ 
============================= 
==============================================









