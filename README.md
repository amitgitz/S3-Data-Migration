# S3-Data-Migration
S3 Data Migration from one AWS Account to Another AWS Account using DataSync

## Step 1: Create Two S3 Bucket - One in Source Account & One in Destination Account
  - Destination Bucket
![image](https://github.com/amitgitz/S3-Data-Migration/assets/88843810/51b5d161-9073-4c89-8b79-b2258bbe6a37)
  - Source Buekcet
![image](https://github.com/amitgitz/S3-Data-Migration/assets/88843810/9d2572dd-0d12-4ce4-b74b-d250e156bd3d)

Note: Upload some data in the source bucket (at least 1GB)
    
## Step 2: Create an IAM User with Administrative Access
![image](https://github.com/amitgitz/S3-Data-Migration/assets/88843810/9b852a91-ba44-4f07-b045-0a470e054ba3)

## Step 3: Create IAM Role with the following permission and inline policy
![image](https://github.com/amitgitz/S3-Data-Migration/assets/88843810/f4ab838e-47a6-4ac2-81e5-6e2f1a96affc)
- Inline Policy
```
{
	"Version": "2012-10-17",
	"Statement": [
		{
			"Action": [
				"s3:GetBucketLocation",
				"s3:ListBucket",
				"s3:ListBucketMultipartUploads"
			],
			"Effect": "Allow",
			"Resource": "arn:aws:s3:::dest-bucket47"
		},
		{
			"Action": [
				"s3:AbortMultipartUpload",
				"s3:DeleteObject",
				"s3:GetObject",
				"s3:ListMultipartUploadParts",
				"s3:PutObject",
				"s3:GetObjectTagging",
				"s3:PutObjectTagging"
			],
			"Effect": "Allow",
			"Resource": "arn:aws:s3:::dest-bucket47/*"
		}
	]
}
```

## Step 4: Update the S3 bucket policy in Destination Account
  - In S3 Bucket navigate to the permission section and change the s3 policy as per the following
    ```
    {
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "DataSyncS3Access",
            "Effect": "Allow",
            "Principal": {
                "AWS": "arn:aws:iam::184199584420:role/s3-data-sync"
            },
            "Action": [
                "s3:GetBucketLocation",
                "s3:ListBucket",
                "s3:AbortMultipartUpload",
                "s3:DeleteObject",
                "s3:GetObject",
                "s3:ListMultipartUploadParts",
                "s3:PutObject",
                "s3:GetObjectTagging",
                "s3:PutObjectTagging"
            ],
            "Resource": [
                "arn:aws:s3:::dest-bucket47",
                "arn:aws:s3:::dest-bucket47/*"
            ]
        },
        {
            "Sid": "DataSyncS3ListBucket",
            "Effect": "Allow",
            "Principal": {
                "AWS": "arn:aws:iam::184199584420:user/kops-connect"
            },
            "Action": "s3:ListBucket",
            "Resource": "arn:aws:s3:::dest-bucket47"
        }
    ]}

## Step 5: Create a DataSync destination location for the S3 bucket
  - Create a terminal and configure the IAM role with Access Key ID & Secret Key
  - Give the following command
```
 aws datasync create-location-s3   --s3-bucket-arn arn:aws:s3:::dest-bucket47   --s3-config '{"BucketAccessRoleArn":"arn:aws:iam::184199584420:role/s3-data-sync"}'
```
Expected Output
```
{
    "LocationArn": "arn:aws:datasync:eu-north-1:184199584420:location/loc-0804a70617fed10c0"
}

```
## Step 6: Navigate to the DataSync in AWS Console and create the Tasks
  - Configure source and destination location
  - Give the task name
  - Select CloudWatch log group
  - Create the task

Now click on the start with defaults and your data will be migrated to the destination s3 account.






