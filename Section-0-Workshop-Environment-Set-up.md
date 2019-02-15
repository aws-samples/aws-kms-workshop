# Getting things ready to kick-off 

This is the first section of the AWS KMS Workshop where we will get things ready to start with the workshop.
Please ensure you have read and understood the [prerequisites for the workshop](https://github.com/DanBerr/aws-kms-workshop#pre---requisites). 

Especially, though AWS KMS prior knowledge is not really needed, Workshop is more meaningful if you take a look at this introduction to AWS KMS:

* [What is Key Management Service?](https://docs.aws.amazon.com/kms/latest/developerguide/overview.html)
* [Getting Started with AWS KMS](https://docs.aws.amazon.com/kms/latest/developerguide/getting-started.html)

---

When you are ready, please follow the following steps to create all the artifacts we will be using during the workshop:


1. Login into your AWS account and navigate to the region you want work in. 



2. Download the Workshop's **CloudFormation template on the (Save Link As) [following link](https://raw.githubusercontent.com/DanBerr/aws-kms-workshop/master/res/cf-workshoptemplate.txt)**. This template will create a Role named "**KMSWorkshop-InstanceInitRole**" and an Amazon S3 bucket named "**kmsworkshop-accountid**", where accountid is the identifier of your account.


   Go to the AWS Console, navigate to "**CloudFormation**" Service and select "**Create Stack**" as you can see in figure below:
   
   
   
![alt text](/res/S0F1.png)
   
   
3. Then, in the "**Specify Template**" area, select "**Upload Template**" and browse for the template we downloaded just        before. Click "**Next**" and give the stack a name, like "KMSWorkshop-Stack". Hit "**Next**". Leave the default values that appear in this new page and hit "**Next**" again. In this new page, make sure you click the checkbox "**The following  
   resource(s) require capabilities: [AWS::IAM::Role]**" at the botton, and click "**Create Stack**". 
   
   The stack is now being created. If you got lost in the process, please look into the [CloudFormation Stack Creation documentation](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/cfn-console-create-stack.html)
   
   
   

4. Once the CloudFormation Stack is Ready, launch an instance from Amazon Linux AMI on the VPC and subnet of your choice (but in the same region you started in). We will use this instance to work with the AWS CLI, so you can select a really small instance size, like "t2.micro". You can always create the VPC and Subnet when you launch the instance, at Step 3: "**Configure Instance Details**".
  It is important that you make sure the instance has internet access in the subnet it is launched. You need to use an    
  Internet Gateway and update the subnet Route table.
  If you need help with these steps, make sure you check [this section of the AWS Documentation](https://docs.aws.amazon.com/vpc/latest/userguide/vpc-getting-started.html).

   If you need overall help with CloudFormation stacks, see [the CloudFormation documenation](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/stacks.html).


5. Make sure the security groups associated with the instance allow it to be accessible via SSH from your IP. **NOTE:**  you should restrict the initially created security group rule to be accesible only to your IP or the range of IPs from your LAN. Check the following [documentation](https://docs.aws.amazon.com/vpc/latest/userguide/VPC_SecurityGroups.html#SG_Changing_Group_Membership) if you need guidance.


6. Once the EC2 instance is up and running, assign the "**KMSWorkshop-InstanceInitRole**" to the instance you have launched. We do it to ensure that the AWS CLI on the instance has enough permissions to run AWS KMS operations.
If you need help with the operation, navigate to the EC2 service in the AWS console and take a look into picture below to locate the role attachment option. Optionally, use the following [AWS Security Blog article](https://aws.amazon.com/blogs/security/easily-replace-or-attach-an-iam-role-to-an-existing-ec2-instance-by-using-the-ec2-console/).


![alt text](/res/S0F0.png)




7. Once the instance is launched and contains the Role, try to connect to it via terminal. If you need help, [check the options here](https://docs.aws.amazon.com/quickstarts/latest/vmlaunch/step-2-connect-to-instance.html).


If you can connect to your instance then **You should now be ready to start with the workshop**, let's [Go to first section of workshop](https://github.com/DanBerr/aws-kms-workshop/blob/master/Section-1-Operating-with-AWS-KMS.md)


