# Exporting a DynamoDB Table to a cross-account s3 bucket

## Preparing your bucket

First is to create an s3 bucket where you plan to export your DynamoDB data.

After creating the bucket you should modify the bucket policies to allow your origin account to be able to write to the s3 bucket in your intended account.


Replaces the vales  **/\*AWS Account ID\*/** and **/\*Target Bucket Name\*/** in the following policy.

    {
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "AllowDynamoDBToWriteToS3",
            "Effect": "Allow",
            "Principal": {
                "AWS": "/*AWS Account ID*/"
            },
            "Action": [
                "s3:ListBucket",
                "s3:PutObject"
            ],
            "Resource": [
                "arn:aws:s3:::/*Target Bucket Name*/",
                "arn:aws:s3:::/*Target Bucket Name*//*"
            ],
            "Condition": {
                "ForAnyValue:StringEquals": {
                    "aws:CalledVia": "dynamodb.amazonaws.com"
                }
            }
        }
	 ]
	}

The above policy does the following

 - Allows only the required actions used to export to the bucket **s3:ListBucket** and **s3:PutObject**
 - Allows actions only through the specified principal which in our case is the AWS account ID we specify.
 - Checks if the s3 bucket actions was initiated by DynamoDB using the **aws:CalledVia** condition key



## Exporting your DynamoDB Table

Go to your table and go to **Exports and Streams**

Click on the **Export to S3** button, which bring you to the export details wizard

Exporting to S3 requires you to enable point-in-time recovery (PITR) so just enable it in the panel. You can disable it after exporting the table.

Enter the bucket name, you can add a path to denote a folder, following the s3://bucket/prefix format

>Example:  s3://mybucket/myfolder


For the S3 bucket owner select **A different AWS account** and enter the Account ID where your bucket .

For additional settings you can set the point in time, file format and encryption key type.

Note that as of writing even though DynamoDB is able to import CSV files it is only able to export JSON and Amazon Ion formats.

After modifying the wizard. You can proceed by clicking the export button.


# Additional
## Importing the data in S3 to DynamoDB

Go to DynamoDB and in the main menu go to **Import from S3**

Once in the Import from S3 panel click on the Import from S3 button

Enter your bucket URL or click on the **Browse S3** button and select your bucket from there.
Note that the exported files will be stored in a folder named AWSDynamoDB and within that a folder with the export ARN as its name.

> BucketName/AWSDynamoDB/exportARN

Do not import the exportARN folder but rather the data folder inside. This is because the export folder also contains manifest files that DynamoDB cannot parse and does not require anyway.

> BucketName/AWSDynamoDB/exportARN/data

Use the default selection **This AWS account**

And for Import file compression select **GZIP** this that is what DynamoDB defaults to.

Select your import file type and click next.

In the next panel configure the DynamoDB settings.
Enter the table name and the partition key and sort key values

**Take note that the partition key and sort key is case sensitive and must match your previous DynamoDB values.**

Finally configure table settings to your liking and proceed with reviewing your changes and initiating the import.




## AWS Documentation 
[DynamoDB cross account migration alternatives](https://aws.amazon.com/premiumsupport/knowledge-center/dynamodb-cross-account-migration/)

[Policy Condition Keys](https://docs.aws.amazon.com/IAM/latest/UserGuide/reference_policies_condition-keys.html)

[DynamoDB export output format](https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/S3DataExport.Output.html)

[Policy Principals](https://docs.aws.amazon.com/IAM/latest/UserGuide/reference_policies_elements_principal.html)
