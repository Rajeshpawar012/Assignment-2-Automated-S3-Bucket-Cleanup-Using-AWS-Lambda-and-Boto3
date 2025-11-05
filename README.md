## Assignment-2-Automated-S3-Bucket-Cleanup-Using-AWS-Lambda-and-Boto3 ##

### 1.1 Create a Bucket ###
• 	Go to AWS Console > S3 > Create bucket
• 	Name it (e.g. my-cleanup-bucket, )
• 	Choose region and settings as needed
• 	Click Create bucket
---
### 1.2 Upload Files ###
• 	Upload multiple files via Upload > Add files
• 	To simulate older files:
• 	Use files with old timestamps (from another system)
• 	Or use AWS CLI to upload files with --metadata and backdated timestamps (for testing only)
---
### Step 2: Create IAM Role for Lambda ###
2.1 Go to IAM Console
• 	Navigate to IAM > Roles > Create Role
• 	Choose AWS service > Lambda
---
### 2.2 Attach Policies ###
• 	Attach AmazonS3FullAccess (for simplicity in testing)
• 	For production, use a custom policy scoped to your bucket
---
### 2.3 Name and Create ###
• 	Name the role (e.g. LambdaS3CleanupRole, )
• 	Click Create role
---
### Step 3: Create Lambda Function ###
3.1 Go to Lambda Console
• 	Navigate to AWS Lambda > Create function
• 	Choose Author from scratch
• 	Name it (e.g. S3CleanupFunction, )
• 	Runtime: Python 3.10
• 	Assign the IAM role created above
---
### 3.2 Add Python Code ###
Paste this code into the Lambda function editor:
import boto3
import datetime

# Configuration
BUCKET_NAME = 'my-cleanup-bucket'
DAYS_THRESHOLD = 30

def lambda_handler(event, context):
    s3 = boto3.client('s3')
    deleted_files = []
    
    # Calculate threshold date
    threshold_date = datetime.datetime.now(datetime.timezone.utc) - datetime.timedelta(days=DAYS_THRESHOLD)
    
    # List objects in bucket
    response = s3.list_objects_v2(Bucket=BUCKET_NAME)
    for obj in response.get('Contents', []):
        if obj['LastModified'] < threshold_date:
            s3.delete_object(Bucket=BUCKET_NAME, Key=obj['Key'])
            deleted_files.append(obj['Key'])
    
    # Log deleted files
    print(f"Deleted {len(deleted_files)} files:")
    for key in deleted_files:
        print(f" - {key}")
    
    return {
        'statusCode': 200,
        'body': f"Deleted {len(deleted_files)} files older than {DAYS_THRESHOLD} days."
    }

---
### 3.3 Save and Deploy ###
• 	Click Deploy to save the function
---
### Step 4: Manual Invocation and Verification ###
4.1 Test the Function
• 	Go to your Lambda function > Test
• 	Create a test event (use default settings)
• 	Click Test
---
### 4.2 Verify Cleanup ###
• 	Go to S3 > your bucket
• 	Confirm that files older than 30 days are deleted
• 	Check CloudWatch Logs for deleted file name


