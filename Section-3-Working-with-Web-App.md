# Working with a WebApp - AWS KMS best practices

In this section, using a Web App, we are going to implement best practices for AWS KMS.
These best practices are based on the Whitepaper "**[AWS Key Management Service Best Practices](https://d0.awsstatic.com/whitepapers/aws-kms-best-practices.pdf)**"

this section has the following parts:
* [Installing the Web App](https://github.com/aws-samples/aws-kms-workshop/blob/master/Section-3-Working-with-Web-App.md#installing-the-web-app)
* [Adding Encryption to the Web App](https://github.com/aws-samples/aws-kms-workshop/blob/master/Section-3-Working-with-Web-App.md#adding-encryption-to-the-web-app)
* [Working with Key Policies](https://github.com/aws-samples/aws-kms-workshop/blob/master/Section-3-Working-with-Web-App.md#working-with-key-policies)
* [Key Policies and VPC Private Endpoint](https://github.com/aws-samples/aws-kms-workshop/blob/master/Section-3-Working-with-Web-App.md#vpc-endpoints-and-key-policies)
* [AWS KMS key tagging](https://github.com/aws-samples/aws-kms-workshop/blob/master/Section-3-Working-with-Web-App.md#key-tagging)
---

### Installing the Web App

The Web App is very simple python web server that works as a shared file server, for internal employees for example. It allows to upload and download files to/from  S3. For downloads the Web App keeps a local file in the instance where Web App is running, prefixing the file with "localfile-". Remenber, our instance has a role with a policy attached to it that allow to read/write from S3.

Let's make a working directory for a our sample Web App and install the boto3 AWS Pyhon library library: Check we are in our home directory first.

```
$ pwd
/home/ec2-user
```

Make the new directory and install the needed boto3 python library (if not present already).

```
$ sudo mkdir SampleWebApp
$ sudo pip install boto3
```

Now, get into the directory and download the sample WebApp with wget as stated below:

```
$ cd SampleWebApp
$ sudo wget  https://raw.githubusercontent.com/aws-samples/aws-kms-workshop/master/WebApp.py
```

You have downloaded a python application, named "**WebApp.py**", that will be our test Web App.

We will need to obtain the instance IP to connect to it from the Internet. We will get it from the metadata of the instance. If you need more information about instance metadata, please look into this [section of the AWS documentation](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ec2-instance-metadata.html).

```
$ sudo curl http://169.254.169.254/latest/meta-data/public-ipv4/
  X.8.X.17
```
We have the public IP of our instance now, write down as we will use to connect later. Let's run the web server with the following command:

```
$ sudo python WebApp.py 80
  
  Serving HTTP on 0.0.0.0 port 80 ...

```

Go to your browser now and navigate to the Web App in the IP you obtained in the previous step:  http://X.8.X.17


**Note:** if you run into issues with reaching the server, it may be worthy to recheck the security group associated with the server, ensure HTTP traffic is allowed. Use [this link to the Security Groups documentation](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/using-network-security.html). 

![alt text](/res/S3F1.png)
<**Figure-1**>

The WebApp has the file uploader, a browser of present objects in our S3 bucket and the local files.
In your laptop create a text file with a sample text, and provide it a name like "**SampleFile-KMS.txt**"
Upload it through the WebApp.




You should get to a page informing that the operation was successul. 
Now, go back ("**press back link in Success page**)  and check that it is the file is showing up in the main server's page. You can now click on it, to download it and display it.

If you refresh the page in your browser, you will notice the same file appears now as a local file with prefix "localfile". The Web App is designed to create also a further local cache.


---

### Adding Encryption to the Web App


The S3 bucket with its corresponding files is well protected under Bucket Policies and IAM policies. Currently, the role we have set in the working instance, has read and write access to the S3 bucket. 
However, for some reason, other instances or users may need read access to the bucket.
It might be desirable that we encrypt the files with the CMK we have created importing our key material, to add protection for our files in bucket. 

In order to do that, the Web App is using boto3 python S3 APIs to upload the files. We need to use the appropriate API to upload the files using Server Side Encryption "**(SSE)**" with AWS KMS and the CMK we created. 

The API as stated in the [Amazon S3 documentation](https://boto3.amazonaws.com/v1/documentation/api/latest/reference/services/s3.html#id217), has this structure:

```
s3.put_object(Bucket=BUCKET,
              Key='encrypt-key',
              Body=b'foobar',
              ServerSideEncryption='aws:kms',
              # Optional: SSEKMSKeyId
              SSEKMSKeyId=keyid)
```

We could very easily modify the code in our App to include the Server Side Encryption (SSE). 
However, it is more clear if we download a version of the WebApp with the changes already implemented in the code, and hence that provides Server Side Encryption using one of our CMKs.

Stop the server from running with CTRL+C (maybe twice). 
Download the version of the Web App that **adds Server Side Encryption** and run the server again:

```
$  sudo wget https://raw.githubusercontent.com/aws-samples/aws-kms-workshop/master/WebAppEncSSE.py
```

We are going to need the KeyId of the CMK we pretend to use for the encryption of the files. The CMK we want to use is the one generated with our import material and which alias was "**ImportedCMK**".

Issue the following command to display your working keys and identify the KeyId of "ImportedCMK", in case you don´t find it in your notes.

```
$ aws kms list-aliases

{
    "Aliases": [
        {
            "AliasArn": "arn:aws:kms:eu-west-1:your-account-id:alias/ImportedCMK", 
            "AliasName": "alias/ImportedCMK", 
            "TargetKeyId": "your-key-id"
        }, 

```
**NOTE:** If you run into trouble with this command and get errors complaining about "import awscli.clidriver", a version mismatch with AWS CLI installed by boto3, then use the following command to go back to normal:

```
$ sudo yum downgrade aws-cli.noarch python27-botocore -y
```


Copy the value under **TargerKeyId**, it is the KeyID of our CMK and we are going to need when we start our file upload server. Keep it handy, we will use several times in this section of the workshop.
Now it is time to start the server with encryption included in the code:

```
$ sudo python WebAppEncSSE.py 80
```

Enter the KeyId of the CMK we have obtained in the previous step when asked. Make sure it is the right one, otherwise the upload will fail. 

Once the server is running on port 80, go back again into your laptop's browser and refresh the file uploader server page. Try to upload again the text file you created before, "SampleFile-KMS.txt".

If the upload is successful, let's check that the file was in fact encrypted. We will use the AWS console.
Open the AWS console. Navigate to the Amazon S3 service and locate our working bucket "**kmsworkshop-accountid**".  The file you have just uploaded will be there.  Click on its name to open its properties.

![alt text](/res/S3F3.png)
<**Figure-3**>


As you can see, the files metadata specifies that is under Server Side Encryption and displays the ARN of the KMS key used to encrypt. Any accidental access to the bucket now will not be able to display the contents of the file, as they are encrypted.

From the file browser Web App, you can download and display the file we have upload and encrypted. Remember that when using Server Side Encryption with KMS,  you don´t need to provide any additional information for getting the object; S3 is able to know how to decrypt the object from the metadata. 

Amazon S3 Server Side Encryption works with envelope encryption in a similar way as what we described in the previous section for Amazon EBS. For more details check [this S3 documentation link](https://docs.aws.amazon.com/AmazonS3/latest/dev/UsingKMSEncryption.html).

You can go back to the terminal and stop the server before entering in Part 3.

---

### Working with Key Policies

Key policies are the primary resource for controlling "who" has access to do "what" with your CMKs.
You have a full description about them in the following [AWS KMS link](https://docs.aws.amazon.com/kms/latest/developerguide/key-policies.html), in case you want to go deeper - and for the importance of the topic,  you should. 
We are going to work with some practical examples. 

Up to now, the assigned IAM  role ("**KMSWorkshop-InstanceInitRole**") of our working instance allows us to perform many things in AWS KMS.  Following best practices like "**Least Privilege**" and "**Separation of Duties**", it can be, for example, that our instance is meant to be used only for uploading data with server side encryption, but not decrypt it and download it.
Maybe the download and decrypt operation needs to be done from another instance with more specific security constraints. 

How can we comply with these requirements? We will use two main resources:
* **IAM roles** and their policies are helpful to control access to CMKs. 
* **Key policies** are resource based policies that we can use to fine grain the access to the CMKs and tweak it to our needs.

Each key that is generated in AWS KMS, has an initial policy attached. Let's looks at the initial policy set for the key we created with our import material, alias "**ImportedCMK**". Type the following command in your instance terminal (replace "your-key-id" by the "**ImportedCMK**" KeyId or ARN, **Note:** the Alias will not work for this operation):

```
$ aws kms list-key-policies --key-id your-key-id

{
    "PolicyNames": [
        "default"
    ]
}
```
**NOTE:** Remember that if you run into trouble with this command and get errors complaining about "import awscli.clidriver", a versions mismatch with AWS CLI installed by boto3, then use the following command to go back to normal:

```
$ sudo yum downgrade aws-cli.noarch python27-botocore -y
```

Execute the command again and this time it should work. If you don´t have a copy of the KeyId, just issue again the list-aliases command  identify the KeyId of "ImportedCMK". 

```

$ aws kms list-aliases

{
    "Aliases": [
        {
            "AliasArn": "arn:aws:kms:eu-west-1:your-account-id:alias/ImportedCMK", 
            "AliasName": "alias/ImportedCMK", 
            "TargetKeyId": "your-key-id"
        }, 
```

Note that we are able to list key policies because the IAM role assigned of our instance allows this operation.  As you can see, we have a key policy attached to the key, its name is "Default". Let's see what it contains:

```
$ aws kms get-key-policy --key-id your-key-id --policy-name default
{
    "Policy": "{\n  \"Version\" : \"2012-10-17\",\n  \"Id\" : \"key-default-1\",\n  \"Statement\" : [ {\n    \"Sid\" : \"Enable IAM User Permissions\",\n    \"Effect\" : \"Allow\",\n    \"Principal\" : {\n      \"AWS\" : \"arn:aws:iam::your-account-id:root\"\n    },\n    \"Action\" : \"kms:*\",\n    \"Resource\" : \"*\"\n  } ]\n}"
}

```

The default key policy gives the AWS account (root user) that owns the CMK full access to the CMK.
This has policy has two important effects (more information in [this link](https://docs.aws.amazon.com/kms/latest/developerguide/key-policies.html#key-policy-default)):

* Reduces the risk of the CMK becoming unmanageable. You cannot delete your AWS account's root user, so allowing access to this user reduces the risk of the CMK becoming unmanageable.
  
* Enables IAM policies to allow access to the CMK. Giving the AWS account full access to the CMK does this; it enables you to use IAM policies to give IAM users and roles in the account access to the CMK. It does not by itself give any IAM users or roles access to the CMK, but it enables you to use IAM policies to do so.

Let's modify the permission of the role assigned to the instance, **allowing it only to encrypt, but not decrypt**.
Let's use the console. Open the AWS Console. Navigate to IAM service, left column "Roles" and search for the role currently assigned to the instance: **KMSWorkshop-InstanceInitRole**. 


Within the role, locate the policy we attached when working in the second section of the workshop, we named it "**KMSWorkshop-AditionalPermissions**". Click the button "**Edit Policy**". Expand the Actions-Access Level-Write section and remove the check box on "**Decrypt**". Review policy and save it.

Let's run the server once more. Please remember you need the KeyID again:

```
$ sudo python WebAppEncSSE.py 80
```

Now try to upload a new file to S3. It will succeed it. Now try and download it. You can also try to download the previous text file we created "SampleFile-KMS.txt". Both operations will fail, and the logs on your instance display show have something like:

```
"ClientError: An error occurred (AccessDenied) when calling the GetObject operation: Access Denied"
```
**NOTE: **  "kms:Encrypt" needs also the capability to generate data keys if it is meant to be used though AWS services (envelope encryption!) This is important when fine-graining key policies.

#### Least Privilege - Access only from the account

Now this role is able to encrypt but not to decrypt. Furthermore, we want to enforce "**Least Privilege**" access and ensure that the encryption Role, providing capability to encrypt, is only used from our account, and not subject to Cross-Account Role access policies that could grant access to the CMK. For that, we will use a handy Key policy.

We need to identify our current role "**KMSWorkshop-InstanceInitRole**" ARN in order to link it to the key policy. Go back to the console, IAM service, click Roles. Search for the role currently assigned to the instance: **KMSWorkshop-InstanceInitRole** and click on it.  In the upper part of the screen you have the associated ARN. As part of the Role ARN you have your account Id. This is the generic structure: 

```
arn:aws:iam::ACCOUNT-ID-WITHOUT-HYPHENS:role/ROLE-NAME
```
Copy the account Id and write it down, we are going to use it now. 


![alt text](/res/S3F5.png)
<**Figure-5**>


Go back to the left column click "**Encryption Keys**". 
Identify our CMK whose Alias is "**Imported CMK**" and click on it. You will land on a page with more details about the key itself. Scroll down until you see "**Key Policy**".

Modify the current policy with this one:

```
{
  "Version": "2012-10-17",
  "Id": "key-default-1",
  "Statement": [
    {
      "Sid": "Enable IAM User Permissions",
      "Effect": "Allow",
      "Principal": {
        "AWS": "arn:aws:iam::your-acount-id:root"
      },
      "Action": "kms:*",
      "Resource": "*"
    },
    {
      "Sid": "Allow for Use only within our Account",
      "Effect": "Allow",
      "Principal": {
        "AWS": "arn:aws:iam::your-acount-id:role/KMSWorkshop-InstanceInitRole"
      },
      "Action": "kms:*",
      "Resource": "*",
      "Condition": {
        "StringEquals": {
          "kms:CallerAccount": "your-account-id"
        }
      }
    }
  ]
}
```

Make sure you replace "**your-account-id**" for the real values of your account in all the places where it is needed.  With this policy you have restricted the key to be used only within your account.

#### Least Privilege - Access only from a Role

Another type of key policy that can become very useful when enforcing "**Least Privilege**" is the capability to also ensure that this CMK is only called by our role and other roles or user can´t use the key for encryption or decryption. For that, you can use a policy like the one below.

With this policy we will ensure that only instances that have the appropriate role attached are able to use the key. It can become handy to control the instances/services tha can use the CMKs, only allowing it through a specific role. This policy is set as an example of what can be done. **Optionally you can enforce it, but it is not required in the workshop**.


```

{
  "Version": "2012-10-17",
  "Id": "key-default-1",
  "Statement": [
    {
      "Sid": "Enable IAM User Permissions",
      "Effect": "Allow",
      "Principal": {
        "AWS": "arn:aws:iam::your-acount-id:root"
      },
      "Action": "kms:*",
      "Resource": "*"
    },
    {
      "Sid": "Allow for Use only within our Account",
      "Effect": "Deny",
      "NotPrincipal": { 
        "AWS": [ "arn:aws:iam::your-acount-id:role/KMSWorkshop-InstanceInitRole", "arn:aws:iam::your-acount-id:root"]
      },
      "Action": "kms:*",
      "Resource": "*",
      "Condition": {
        "StringEquals": {
          "kms:CallerAccount": "your-account-id"
        }
      }
    }
  ]
}

```


#### Key Policy - Including conditions

Furthermore,  you can also include conditions over key policies to help you fine tune access and link to several other parameters. 
As an example, conditions can work with **encryption context** to be able to restrict the operations for this KMS. Amazon S3 when calling AWS KMS to generate a Data Key and perform the envelope encryption process, it passes and encryption context to AWS KMS, see below a log from a "**GenerateDataKey**" event:
```
"eventName": "GenerateDataKey",
…
"requestParameters": {
        "keySpec": "AES_256",
        "encryptionContext": {
            "aws:s3:arn": "arn:aws:s3:::kms-workshop/SampleFile-KMS.txt"
        },
```

We could use that to add a condition in AWS KMS Key Policy like "**kms:EncryptionContextKey**". 
There is a full example in the [documentation here](https://docs.aws.amazon.com/kms/latest/developerguide/policy-conditions.html#conditions-kms-encryption-context-keys). Make sure you click the link and check that you understand how the key policy is enforced using encryption context as condition.

Finally, let's try to add another layer of security via **MFA**. In the key policy we might request that the users that are going to use the CMK have passed through a MFA process. 
MFA is enforced through a condition as seen below:

```
{
  "Version": "2012-10-17",
  "Id": "key-default-1",
  "Statement": [
    {
      "Sid": "Enable IAM User Permissions",
      "Effect": "Allow",
      "Principal": {
        "AWS": "arn:aws:iam::your-acount-id:root"
      },
      "Action": "kms:*",
      "Resource": "*"
    },
    {
      "Sid": "Allow for Use only within our Account",
      "Effect": "Allow",
      "Principal": { 
        "AWS": [ "arn:aws:iam::your-acount-id:user/userA",]
      },
      "Action":  [
    "kms:DeleteAlias",
    "kms:DeleteImportedKeyMaterial",
    "kms:PutKeyPolicy",
    "kms:ScheduleKeyDeletion"
     ], 
      "Resource": "*",
      "Condition": {
        "NumericLessThan":{"aws: MultiFactorAuthAge":"300"}       }
    }
  ]
}
```

In this key policy, we don´t allow certain very sensitive operations to take place, unless the user or the role has gone through a MFA authentication process in the last 5 minutes (300 seconds). Make sure you understand the policy. After the examples seen by now, you should be able to understand how it works. We will not enforce as in the workshop we are working with roles and not with users. However the overall concept is the same.

---


### VPC Endpoints and Key Policies

Resources can communicate with AWS KMS through a VPC private endpoint.
A VPC endpoint enables you to privately connect your VPC to supported AWS services and VPC endpoint services powered by PrivateLink without requiring an internet gateway, NAT device, VPN connection, or AWS Direct Connect connection. 

When you use a VPC endpoint, **communication between your VPC and AWS KMS is conducted entirely within the AWS network**.
You can even specify the VPC endpoint in [AWS KMS API operations](https://docs.aws.amazon.com/kms/latest/APIReference/) and [AWS CLI commands](https://docs.aws.amazon.com/kms/latest/APIReference/).
In order not to make the workshop too extensive **it is not required to set up the VPC Endpoint**. We will mention it, so you know what you can do with it.

As a reference: the VPC endpoint can  easily be created from the console, you can follow the steps in this [link to the documentation](https://docs.aws.amazon.com/kms/latest/developerguide/kms-vpc-endpoint.html).
Once an endpoint is created you can enforce communication through it with commands as seen below, where parameter "**--endpoint-url**" is added. 

```
$ aws kms list-keys --endpoint-url https://vpce-xxxxxxxxa-xxxxxx.kms.your-region.vpce.amazonaws.com
```


VPC endpoints would allows us to establish more restrictive conditions to the operations we allow on AWS KMS and its CMKs.

For example, change the key policy of a CMK to allow only certain operations from the VPC endpoint.
A sample policy would be like the one below. Please ensure you understand the conditions applied before continuing.

```
{
  "Version": "2012-10-17",
  "Id": "key-default-1",
  "Statement": [
    {
      "Sid": "Enable IAM User Permissions",
      "Effect": "Allow",
      "Principal": {
        "AWS": "arn:aws:iam::your-account-id:root"
      },
      "Action": "kms:*",
      "Resource": "*"
    },
    {
      "Sid": "Allow for Use only within our VPC",
      "Effect": "Deny",
      "Principal": {
        "AWS": "arn:aws:iam::your-account-id:role/KMSWorkshop-InstanceInitRole"
      },
      "Action": [
        "kms:Encrypt",
        "kms:Decrypt",
        "kms:ReEncrypt*",
        "kms:GenerateDataKey*"
      ],
      "Resource": "*",
      "Condition": {
        "StringNotEquals": {
          "aws:sourceVpce": "vpce-1xxxxxxxx1"
        }
      }
    }
  ]
}
```

With that key policy, an extra layer of protection the key would be established, making AWS KMS only available internally within AWS network for certain sensitive operations.


---



### Key Tagging

Tagging is an important strategy for managing CMKs in AWS KMS.
You can add, change, and delete tags for customer managed CMKs. Each tag consists of a tag key and a tag value that you define.
You can add tags to a CMK when you first create them. Then, add, edit, and delete tags at any time. 

To add a tag to the CMK we have been working with, you can use the console or the CLI. Let's tag our CMK "**ImportedCMK**", with a project it may belong to, just an a example.

```
$ aws kms tag-resource --key-id your-key-id --tags TagKey=project,TagValue=kmsworkshop
```

The other usual operations with tags are also available: list tags for resource and untag the resource.
Let's list the resource tags:

```
$ aws kms list-resource-tags --key-id your-key-id
```

Add a few more tags to the CMK and try to remove the tags (untag) with commad "untag-resource".
For information on the command, please use [this section of the AWS KMS documentation](https://docs.aws.amazon.com/kms/latest/developerguide/tagging-keys.html).



This section of the Workshop is now completed. Please, navigate to [the next Section, Monitoring AWS KMS](https://github.com/aws-samples/aws-kms-workshop/blob/master/Section-4-Monitoring-AWS-KMS.md).



