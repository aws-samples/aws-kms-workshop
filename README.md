## AWS Kms Workshop

This AWS KMS workshop pretends to provide a better understanding on AWS Key Management Service (KMS) through a set of practical exercises. Even though previous experience with AWS KMS is not needed, it would be helpful to read the documentation listed in the Pre-Requisites section below, before starting the Workshop.

## License

This library is licensed under the Apache 2.0 License. 

---

The workshop is aligned with the AWS KMS best practices "must-read" Whitepaper "**[AWS Key Management Service Best Practices](https://d0.awsstatic.com/whitepapers/aws-kms-best-practices.pdf)**" and the practices follow its guidelines.

The entire Workshop can be covered in around two hours, depending on your previous experience with AWS.

---

# Workshop content:
The workshop contains four different sections (**NOTE:** designed to be followed in order) covering areas like AWS CMKs operations, Types of encryption in AWS KMS with focus on envelope encryption, key policies and best practices working with a demo Web App and AWS KMS monitoring.

* [Workshop Environment Set-up](https://github.com/aws-samples/aws-kms-workshop/blob/master/Section-0-Workshop-Environment-Set-up.md)
* [Section I - Operating with AWS KMS and the CMKs](https://github.com/aws-samples/aws-kms-workshop/blob/master/Section-1-Operating-with-AWS-KMS.md)
* [Section II - Encryption with AWS KMS](https://github.com/aws-samples/aws-kms-workshop/blob/master/Section-2-Encryption-with-AWS-KMS.md)
* [Section III - Key Policies and best practices - Working with a Web App](https://github.com/aws-samples/aws-kms-workshop/blob/master/Section-3-Working-with-Web-App.md)
* [Section IV - Monitoring AWS KMS](https://github.com/aws-samples/aws-kms-workshop/blob/master/Section-4-Monitoring-AWS-KMS.md).

The workshop is mostly practical and will operate in AWS KMS using the AWS CLI (through an EC2 instance), AWS console and AWS KMS API calls, to get a better understanding of the different options. 

---

# Pre - Requisites:

In order to set up the working environment for the workshop, you need the following:

* An AWS account.
* An user with enough permissions to generate policies and create/modify roles in IAM.
* An user with permissions to run CloudFormation templates and launch EC2 instances.
* A VPC, public subnet and security groups (or being able to create them), to launch the EC2 instances. 
  If you need help with creating those, please use the following [quickstart from AWS](https://aws.amazon.com/quickstart/architecture/vpc/).

AWS KMS prior knowledge is not really needed, but if would be great if you take a look into this brief introduction:

* [What is Key Management Service?](https://docs.aws.amazon.com/kms/latest/developerguide/overview.html)
* [Getting Started with AWS KMS](https://docs.aws.amazon.com/kms/latest/developerguide/getting-started.html)
---

# Ready to go?

**Once you are ready**, go to the set up section of the workshop and launch the CloudFormation template that will provide with the needed resources to start the workshop: [Section 0 - Workshop Environment Set-up](https://github.com/aws-samples/aws-kms-workshop/blob/master/Section-0-Workshop-Environment-Set-up.md).


