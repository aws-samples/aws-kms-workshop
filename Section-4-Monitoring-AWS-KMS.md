# Monitoring and Logging in AWS KMS

Monitoring is an important part of understanding the availability, state, and usage of your customer master keys (CMKs) in AWS KMS and maintaining the reliability and performance of your AWS solutions. 
As as baseline, in AWS KMS you may want to monitor:

* Activity related to cryptographic operations, such as Encrypt or Decrypt.
* Activity related to management operations on the CMKs: EnableKey, ImportKeyMarterial,etc…
* Activity on other events and metrics, such as key expiration, key rotation or time remaining until imported key material expiration.

To monitor that activity we will the AWS service [AWS CloudTrail](https://aws.amazon.com/cloudtrail/) and [Amazon CloudWatch](https://aws.amazon.com/cloudwatch/), escecially its logs, events and alarms.

---
### AWS KMS and AWS CloudTrail

AWS KMS is integrated with [AWS CloudTrail](https://aws.amazon.com/cloudtrail/). AWS CloudTrail is a service that will provide us with a record of actions performed by a user, role, or an AWS service in AWS KMS.
CloudTrail captures all API calls for AWS KMS as events, including calls from the AWS KMS console and from code calls to the AWS KMS APIs.

In order to see how CloudTrail logs this information, let's first create a data key with the corresponding AWS KMS command we have used before a few times:

```
$ aws kms generate-data-key --key-id alias/ImportedCMK --key-spec AES_256 --encryption-context project=workshop
```


This action has been logged in AWS CloudTrail, and we can obtain its details. Let's use the AWS console for that.

Go back to the AWS console in your browser, navigate to "**CloudTrail**" service and select "**Event history**" on the right panel. You have the full history of events and they can be filtered for a fine grained view.

To establish a filter, go to filter area, select "**Event name**", and set the name as "**GenerateDataKey**". Press "**Enter**" while still on the "**Enter Event Name**", leaving "**Select time range**" as it is. Alterntively, you could select a time range if you wish.

You will have a display of the GenerateDataKey operations that you have performed during the workshop. You can see image below as a reference:

![alt text](/res/S4F1.png)
<**Figure-1**>

If you open any of the request in the list,  you will have further details of the operation that took place and. For example take a look at the "**User name**" value responsible for the requests and write it down, we will use it later. These parameters provides us with a full view of who, what, how and when an operation took place.

In CloudTrail you can not only filter by event names on AWS KMS operations. There are other filter parameters that you can use. For example, you can use filtering by "**Event source**", that would allow you understand which AWS service has made request.

The filter parameter "**User name**" allows you to filter by the identity of the user referenced in the event.
Another useful parameter is the "**AWS Access Key**". With it, you can filter by the AWS access key ID that was used to sign the request. If the request was made with temporary credentials, the access key ID of the temporary credential is what will show up as the access key.

Now let's try to set up a new filter by "**User Name**" attribute. For the attribute value, use the same "**User name**" that you have obtained in one of the request listed when you filtered by "**GenerateDataKey**" Event name. 
You should obtain a full list of AWS KMS operations performed by the user. Also other logged operations in other services, if the user has made any. An example in figure below:


![alt text](/res/S4F15.png)
<**Figure-2**>


You can filter by many other parameters to collect all needed information to audit AWS KMS usage. A list of AWS KMS events that can be displayed in CloudTrail can be checked in this part of the [AWS KMS Documentation](https://docs.aws.amazon.com/kms/latest/developerguide/logging-using-cloudtrail.html).

It is a powerful service by itself, and can become even more powerful if we combine it with other services.
Let's create, for example, custom notifications based on AWS Cloudtrail events coming from AWS KMS.

---

### AWS KMS Real time notifications with AWS CloudTrail, Amazon CLoudWatch and Amazon SNS.

In this part of the monitoring section, we are going to create a notification system that will notify us when certain type of events happen and are logged in CloudTrail. We will use the following services: [Amazon CloudWatch](https://aws.amazon.com/cloudwatch/), [AWS CloudTrail](https://aws.amazon.com/cloudtrail/) and [Amazon SNS](https://aws.amazon.com/sns/). Let's open the AWS console. 


First, let's create a notification topic and subscribe to it. The Topic will allow us to receive notifications when an event we define in CloudWatch coming from CloudTrail is triggered.
In the AWS console, navigate to Simple Notification Service (SNS) service. Click on "**Create Topic**" and provide a topic name and a display name. Any name may work for topic name, try with "**snsworkshop**". Choose now a  display name, it must not exceed 10 characters anyway. 
Take note of the "**Topic ARN**" listed as a result of your topic creation. Something like: "**arn:aws:sns:your-region:yout-account-id:snsworkshop**"

Now on the right pane, select "**Subscriptions**" and click on "**Create subscriptions**".
Provide the topic ARN that you took note in the previous step and in "**Protocol**" select "**Email**". Provide SNS with your mail.
In a moment, you will receive an email to confirm your subscription.  

With this, navigate to now to Amazon CloudWatch service in the AWS console.
Inside Amazon CloudWatch, look in the right pane and click on "**Events**".
Leave "**Event Pattern**" clicked and select "**Events by Service**" in the "**Build event pattern to match...**" area.

![alt text](/res/S4F2.png)
<**Figure-3**>


Select "**Service Name**" as "**CloudTrail**" and "**Event Type**"  as "**AWS API Call via CloudTrail**".
Then select "**Specific operation(s)**" and in the blank space, type the event name we are pursuing: "**GenerateDataKey**".

![alt text](/res/S4F3.png)
<**Figure-4**>


Now press the "**+**" symbol to aggregate it to then event source filter. We have the input part ready, now let's do the target part. 


On the right side of the screen, find "**+ Add Target**" and click it.
On the first row, it would say "**Lambda Function**", change it to "**SNS Topic**", Then select the topic created before, this is: "**snsworkshop**". 

You are ready to hit the "**Configure details**" button on the botton of the page.
Now just provide a name to the rule and hit "**Create Rule**".

You have just created a rule that will help you audit AWS KMS usage. Everytime a Data Key is generated, you wil be notified in the email address you provided. 

If everything went well you should  receive an email notifying you of the operation that took place. 
**Note:** Don´t forget to hae confirmed your subcripution to SNS topic (you should have recevied an email).
We have established a notification for a specific operation. For a comprehensive list of the log entries that AWS KMS generates in AWS CloudTrail, please check the following [section of the AWS KMS documentation](https://docs.aws.amazon.com/kms/latest/developerguide/logging-using-cloudtrail.html).


### AWS KMS and CloudWatch Metrics

In Amazon Cloudwatch you also have **metrics** avaiable about the AWS KMS service and CMKs. For example,  when you import key material into a CMK and set it to expire, AWS KMS sends metrics and dimensions to CloudWatch.
For details on the dimensions of these metrics, you can check the [following section](https://docs.aws.amazon.com/kms/latest/developerguide/monitoring-cloudwatch.html) of the AWS KMS documentation.
With Amazon Cloudwatch you can create alarms and dashboards that will give you certain insights on AWS KMS. 

To check these metrics into Amazon CloudWatch navigate again into Amazon CloudWatch and select "**Metrics**" on the right pane.
Inside metrics, find the ones that belong to AWS KMS:

![alt text](/res/S4F4.png)
<**Figure-4**>

If you click through it you will find the metric "**SecondsUntilKeyMaterialExpiration**" for your CMK built with imported  key material. 
With this metric you can now build an alarm into CloudWatch to warn you about the expiration of the key material for example. 

Now, as a final assignment, let's create an Amazon Cloudwatch alarm over an AWS KMS metric. Follow the Amazon Cloudwatch step by step guide in [this Amazon CloudWatch section](https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/ConsoleAlarms.html). 

In this case, we only have a metric coming from AWS KMS so it should be easy for you to identify it and creating the alarm for it. 
You can use the SNS topic that we created before: "**snsworkshop**". 
The process is well described in previous link, so it is not reproduced here in the workshop's instructions. 

In case you need more details about building the alarm, please look into how to build an alarm from AWS KMS metrics in the followin [section of the AWS KMS documentation](https://docs.aws.amazon.com/kms/latest/developerguide/monitoring-cloudwatch.html#key-material-expiration-alarm). 


---
### Finishing and Cleaning up 


Congratulations for making it to the end of the workshop. You should now have a better understanding AWS KMS and its operations with CMKs. 

In order to clean up the resources we have used along the workshop, we need to perform a few steps, in the order they are listed:

1. Schedule for deletion the CMKs we have created. At this point it is very easy for you to do it. You can either use the AWS console, browsing to the IAM service, selecting "**Encryption keys**" in the left pane.
You may also use the AWS CLI commands. In first section of the document, you learned how to delete CMKs
	
2. Detach the Role from the EC2 instance. You need to follow the same procedure you used to attach it. If you need to refresh it, go back to the first section of the workshop, in the "Getting things ready to kick-off" section.

3. Still on the AWS Console, navigate to the IAM service again and select "**Roles**" on the left pane. Locate the Role we have been working on all the workshop: "**KMSWorkshop-InstanceInitRole**" and detach from it all the customer policies that we have created. To do it, one you are in the "**KMSWorkshop-InstanceInitRole**" Role screen, click on the black "x" on the right side of the policy listing.
	
4. Now you can terminate the EC2 instance. The process is very simple, just select the instance in the EC2 service within the AWS Console and in "**Actions**" select "**Terminate Instance**".
	
5. The final step is to delete the CloudFormation Stack that we had launched at the beginning of the workshop. To do it, just go back to the CloudFormation service again within the AWS Console. In the "**Stacks**" menu, select the stack used to launch the workshop and in the "**Actions**" button, click on "**Delete Stack**"


All the resources should now have been deleted from your account, with the exception of the CMKs, they will not be deleted until the scheduled delete date is met.

---
Workshop has finished - Thank you for completing this Workshop.

 

