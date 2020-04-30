# AWS_Serverless_CCS

## Initial Setup
0. unzip the zip file.

1. Create an IAM user, **aws-user** and give programatic access. You will need to store the credentials as these will be used to interact with AWS. which will assume different roles and interact with AWS. Attach the following identity-based policy to allow it to interact with S3, EC2, Elastic Transcoder, Kinesis and IAM services:
```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "VisualEditor0",
            "Effect": "Allow",
            "Action": "sts:AssumeRole",
            "Resource": [
                "arn:aws:iam::{accountId}:role/setup-role",
                "arn:aws:iam::{accountId}:role/execute-role"
            ]
        }
    ]
}
```

2.  Create an IAM role, **setup-role**, which will be used to create initial resources necessary. Attach the following *trust policy* under Trust relationships tab to enable the **aws-user** to assume this role.

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": {
        "AWS": "arn:aws:iam::{accountId}:user/aws-user"
      },
      "Action": "sts:AssumeRole"
    }
  ]
}
```
Also attach the following idenity-based policy to **setup-role**. This allows it to attach and detach the **execute-policy**, **setup-env-policy** to the **execute-role**.
```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": [
                "iam:CreatePolicyVersion",
                "iam:DeletePolicyVersion",
                "iam:ListPolicyVersions"
            ],
            "Resource": [
                "arn:aws:iam::{accountId}:policy/execute-policy",
                "arn:aws:iam::{accountId}:policy/setup-env-policy"
            ]
        },
        {
            "Effect": "Allow",
            "Action": [
                "iam:DetachRolePolicy",
                "iam:AttachRolePolicy",
                "iam:ListAttachedRolePolicies"
            ],
            "Resource": "arn:aws:iam::{accountId}:role/execute-role"
        }
    ]
}
```

3.  Update the *setup-role-arn* and *setup-role-name* properties in *config.properties* file with the **ARN**  and **name** of the **setup-role** created in step 1 respectively.
4.	Create an IAM role, **execute-role** which will responsible for executing the various APIs. Attach the following *trust policy* under Trust relationships tab to enable the **aws-user** to assume this role.

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": {
        "AWS": "arn:aws:iam::{accountId}:user/aws-user"
      },
      "Action": "sts:AssumeRole"
    }
  ]
}
```
5.	Update the *execution-role-name* property property in *config.properties* file with the **name** of the **execute-role** created in step 4.
6.  Attach the following IAM identity-based policy to the **setup-role**. This allows setup role to attach or detach policies from the **execute-role**.
7.  Create an IAM idenity-based policy, **setup-env-policy**, which is used to setup the needed resources before running the tests.
8.  Update the *setup-env-policy-arn* property in *config.properties* file with the **ARN** of the **setup-env-policy** policy created in step 6.
9.  Create an IAM idenity-based policy, **execute-policy**, which will posses the permissions for executing the APIs.
10.  Update the *execute-policy-arn* property in *config.properties* file with the **ARN** of the **setup-env** policy created in step 8.


## Common Step
1.  Update the **accountId** for all policies in *s3/iamPolicies, kinesis/iamPolicies, ec2/iamPolicies* with the your AWS account id.

## For Testing s3.getObjectAsBytes():
1. Update the policies in *s3/iamPolicies* with the correct resources (bucket, object) if the **bucket, key** properties in *config.properties* are changed.

## For Testing kinesisClient.startStreamEncryption():
1. By default, use one of the AWS managed keys, **aws/kinesis** for encrypting the AWS Kinesis data stream. The **aws/kinesis** key can be obtained from the AWS Key Management Service (KMS). Note: A custom key can also be created but we avoid it for simplicity.

2. Update the **key-id** property in *config.properties* file with the key Id of the  **aws/kinesis**.
3. Change the *accountId* in the *kinesis/iamPolicies*.
```json
{
  "Version": "2012-10-17",
  "Statement": {
    "Effect": "Allow",
    "Action": [
      "kinesis:CreateStream",
      "kinesis:DeleteStream"
    ],
    "Resource": "arn:aws:kinesis:us-east-1:{accountId}:stream/test-stream-5457"
  }
}
```


**Note:** If the **kinesis-data-stream-name** property in *config.properties* is changed, change the resource in *kinesis/iamPolicies*.

## For Testing EC2.exportImage()
1. Setup a new EC2 instance.

2. Update the **instance-id** property in *config.properties* file with the instance id from the instance created in Step 1.

3. Create an IAM role, **ec2-service-role**. This is a service role that is assumed by the EC2 service to perform tasks on the service's behalf.

Attach the following *trust policy* to the **ec2-service-role**:
```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": {
        "Service": "vmie.amazonaws.com"
      },
      "Action": "sts:AssumeRole"
    }
  ]
}
```

4. Create an IAM idenity-based policy, **ec2-service-role-policy**, which is used to give access to the **ec2-service-role**.

5. Update the **ec2-service-role-policy-arn** property in config.properties file with the ARN of the *ec2-service-role-policy* policy created in step 4.

6. Update the policy attached to  **setup-role**. This allows it to attach and detach the **execute-policy**, **setup-env-policy** to the **execute-role** and **ec2-service-role-policy** to **ec2-service-role**

```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": [
                "iam:CreatePolicyVersion",
                "iam:DeletePolicyVersion",
                "iam:ListPolicyVersions"
            ],
            "Resource": [
                "arn:aws:iam::{accountId}:policy/execute-policy",
                "arn:aws:iam::{accountId}:policy/setup-env-policy",
                "arn:aws:iam::{accountId}:policy/ec2-service-role-policy"
            ]
        },
        {
            "Effect": "Allow",
            "Action": [
                "iam:DetachRolePolicy",
                "iam:AttachRolePolicy",
                "iam:ListAttachedRolePolicies"
            ],
            "Resource": [
                "arn:aws:iam::{accountId}:role/execute-role",
                "arn:aws:iam::{accountId}:role/ec2-service-role"
            ]
        }
    ]
}
```
**Extra:** Following is the policy obtained from the [VM Import/Export User Guide](https://docs.aws.amazon.com/vm-import/latest/userguide/vm-import-ug.pdf), which is more privileged than the least-privileged policy provided in *ec2/iamPolicies*. You can try it out yourself.
```json
{
	"Version": "2012-10-17",
	"Statement": [{
			"Effect": "Allow",
			"Action": [
				"s3:GetBucketLocation",
				"s3:GetObject",
				"s3:ListBucket"
			],
			"Resource": [
				"arn:aws:s3:::disk-image-file-bucket",
				"arn:aws:s3:::disk-image-file-bucket/*"
			]
		},
		{
			"Effect": "Allow",
			"Action": [
				"s3:GetBucketLocation",
				"s3:GetObject",
				"s3:ListBucket",
				"s3:PutObject",
				"s3:GetBucketAcl"
			],
			"Resource": [
				"arn:aws:s3:::export-bucket",
				"arn:aws:s3:::export-bucket/*"
			]
		},
		{
			"Effect": "Allow",
			"Action": [
				"ec2:ModifySnapshotAttribute",
				"ec2:CopySnapshot",
				"ec2:RegisterImage",
				"ec2:Describe*"
			],
			"Resource": "*"
		}
	]
}
```

## Running the code:
We have created three files *S3Main.java, KinesisMain.java, EC2Main.java* which execute API methods.

We provide sample actual output in the *s3/output, kinesis/output, ec2/output* folders.
