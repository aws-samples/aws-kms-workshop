# Operating with AWS KMS and CMKs

In this first section we are going to learn the core operations of AWS KMS, that would allow us to go deeper into the service and its best practices. The section has four main areas:
 * [Creating Customer Master Keys (CMK)](https://github.com/aws-samples/aws-kms-workshop/blob/master/Section-1-Operating-with-AWS-KMS.md#creating-customer-master-keys-cmk)
 * [Generate CMKs with your own key material](https://github.com/aws-samples/aws-kms-workshop/blob/master/Section-1-Operating-with-AWS-KMS.md#generate-cmks-with-your-own-key-material)
 * [Rotating AWS KMS CMKs](https://github.com/aws-samples/aws-kms-workshop/blob/master/Section-1-Operating-with-AWS-KMS.md#rotating-AWS-KMS-CMKs)
 * [Deleting AWS KMS CMKs](https://github.com/aws-samples/aws-kms-workshop/blob/master/Section-1-Operating-with-AWS-KMS.md#deleting-AWS-KMS-CMKs)
----


## Creating Customer Master Keys (CMK)

CMKs are the primary resources in AWS KMS. You can use a CMK to encrypt and decrypt up to 4 kilobytes (4096 bytes) of data. However, most commonly, you will use CMKs to generate, encrypt, and decrypt the [data keys](https://docs.aws.amazon.com/kms/latest/developerguide/concepts.html#data-keys) that you use outside of AWS KMS to encrypt your data.

There are different types of CMKs in KMS, [see documentation here](https://docs.aws.amazon.com/kms/latest/developerguide/concepts.html#master_keys). 
In this section we will create a CMK with key material coming from AWS KMS, and later we will generate a CMK with your own key material. It is important to remember that CMKs never leave AWS KMS unencrypted.

### Step 1 - create CMKs

Time to get hands-on with KMS. **Connect to the instance created via terminal** to start working with the CMKs.

In order to create our first CMK, we will use the [AWS CLI](https://aws.amazon.com/cli/) in the instance. We will use it, instead of the AWS console, beacuse it will provide you with deeper insights and understanding of the process. Once you understand it, creating CMKs from the console will be a breeze.

To use the AWS CLI you might need to configure your region in AWS CLI first. You can do so typing the command "**aws configure**" in the instance once you are connected via terminal. 
Leave all fields blank except for the default region. Choose the code of the region you are working in ([region codes](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/using-regions-availability-zones.html#concepts-available-regions)) and set it as default region in AWS CLI. 

See an example below setting up Ireland as the default region in AWS CLI:

```
[ec2-user@ip-10-0-X-X ~]$ aws configure
AWS Access Key ID [None]: 
AWS Secret Access Key [None]: 
Default region name [None]: eu-west-1
Default output format [None]: 
[ec2-user@ip-10-0-X-X ~]$ 
```

More information can be found [here](https://docs.aws.amazon.com/cli/latest/userguide/cli-chap-getting-started.html#cli-quick-configuration).

Once AWS CLI is ready, try typing the following command to create the key:

```
$ aws kms create-key
```


The response from above command should be an error message like the one below. 

```
An error occurred (AccessDeniedException) when calling the CreateKey operation: User: arn:aws:sts::account-id:assumed-role/KMSWorkshop-InstaceInitRole/instanceid is not authorized to perform: kms:CreateKey on resource:
```

This is because the initial role we have assigned to the instance does not include the capability to create keys. We need to add a policy to the role in order to enable us to perform certain actions with AWS KMS during the workshop. 

Following **least privilege best practices**, we will be attaching policies with permissions as needed for the different AWS KMS operations we are working with. In this way it is easy track and detach the policies from the role, once they are not needed, and keep the least privilege principles.



### Step 2 - Create Policy and Attach it to the role

We can add the needed permissions via the CLI or the AWS console. We will use the AWS console for this operation.

Logging into the AWS console by[click here](https://console.aws.amazon.com/?nc2=h_m_mc), and navigate to the IAM service. Then click on "**Roles**", left area of the screen.
![alt text](/res/S1F1%20IAM.png)
<**Figure-1**>


Search for the role that has been set up and attached to the instance by the CloudFormation template, its name is **KMSWorkshop-InstanceInitRole**. 

![alt text](/res/S1F2%20KMSinitRole.png)

<**Figure-2**>


Click on the role and then on "**Attach Policies**" button, we are going to provide permissions so the instance can create Keys. A new screen where you can now search for Policies will appear.
![alt text](/res/S1F3%20AttachPolicy.png)

<**Figure-3**>


Now, search "**AWSKeyManagement**", and select the policy "**AWSKeyManagementSystemPowerUser**".  That is the policy we are going to use for the instance role. **Please note**, the assigment of KMS Power User permissions is **just** for the initial walk-through in KMS, a typical user might not need the whole set of permissions. Later in the workshop we will work on how to implement more fine grained "Least Privilege" access, according to best practices,  in order to assign appropriate permissions to users and roles into KMS operations.
As this point we can review the policy, just by clicking on the small black arrow close to the policy name to expand it.

![Figure-4](/res/S1F4%20KMSPowerUserPolicy.png)

<**Figure-4**>



 See the operations the policy allowing as a Poweruser into AWS KMS.

```
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": [
                "kms:CreateAlias",
                "kms:CreateKey",
                "kms:DeleteAlias",
                "kms:Describe*",
                "kms:GenerateRandom",
                "kms:Get*",
                "kms:List*",
                "kms:TagResource",
                "kms:UntagResource",
                "iam:ListGroups",
                "iam:ListRoles",
                "iam:ListUsers"
            ],
            "Resource": "*"
        }
    ]
}
```


Select the policy and click the "Attach policy" button at botton right of the page. The role attached to the instance is modified with his new policy and we should be able to create a Customer Master Key (CMK) now from the instance. Let's try it.



### Step 3 - Create the key  - again - and set an alias

Run the aws kms create-key command again and this time you will get the result from the creation of the key as a JSON block with the metadata of the key. See example below:

```
$ aws kms create-key

{
    "KeyMetadata": {
        "Origin": "AWS_KMS", 
        "KeyId": "your-key-id", 
        "Description": "", 
        "KeyManager": "CUSTOMER", 
        "Enabled": true, 
        "KeyUsage": "ENCRYPT_DECRYPT", 
        "KeyState": "Enabled", 
        "CreationDate": 1538497627.686, 
        "Arn": "arn:aws:kms:eu-west-1::key/", 
        "AWSAccountId": "your-account-id"
    }
}

```
There are important fields in the metadata response. First, The **KeyId** is very relevant as it is the unique identifier of the CMK within AWS KMS. It is in the form of five blocks of digits. Take good note of the KeyId as you will be using it during the workshop extensively. Also, along the Workshop, I will reference its value as "**your-key-id**". 
Please change it for your corresponding KeyId value when applicable.

The ARN of the key and its status ("Enabled", "Disabled") are highly relevant too. Other type of information is displayed. For example: the Account Id where the key belongs to. Another relevant piece of information is the the target key usage, this is: encrypt and decrypt in this case. 

If you go back to the AWS console and navigate to the IAM service. Click in the left area to the bottom, "Encryption Keys", the key you have just created is already listed there. **Important: Remember to select the right Region in the KMS screen**. However, as we used the create-key command without parameters, it does not contain any alias to display and looks like its alias is empty. See image below the region selection and the blank alias name.

![Figure-15](/res/S1F15.png)
<**Figure-15**>


Key alias are very useful. They are easier to remenber when operating with keys. Most importantly, when rotation keys, as we will see later in this section, we will not have to update our code to update the new KeyIDs or ARN references. By using alias in our code to call the CMKs by them, and updating the alias CMKs to point to the newly generated key, the amount of change in our code gets minimized.

Let's create it an alias, "**FirstCMK**",  with the command aws kms create-alias. 
Remember to replace 'your-key-id' with the value obtained from previous command (aws kms create-key).


```
$ aws kms create-alias --alias-name alias/FirstCMK --target-key-id 'your-key-id'
```

If you look now in the console, the CMK you just created displays now the right alias. 

![Figure-5](/res/S1F5%20Alias.png)
<**Figure-5**>


When you create the CMK from the console, just by clicking the button "create key" there are other parameters you need to set like tags, key administrators and usage permissions. This steps will basically create a Key Policy and attach it to the key together with the tags you have set. 
For the workshop, we will see how creating CMKs, policies and tags can be done from the CLI to have greater insights on their scope and implications. 


----

## Generate CMKs with your own key material

With AWS KMS you can import your own key material to create a CMK. In order to do so, a special wrapping is needed to upload your key material to AWS KMS. See more details in [this part of the KMS documentation](https://docs.aws.amazon.com/kms/latest/developerguide/importing-keys.html).
In this section we are going to generate a CMK importing our own key material.

### What are going to do?

In order to experiment how to import your own key material, we are going to take a few steps.

* First, we will create an empty CMK in AWS KMS. The CMK later will be filled up with the the key material that you generate and import into AWS KMS.

* In order to import your own key material, it is needed to do a special wrapping on it: AWS KMS will provide you with a public key that we will use to wrap our key material. Also, AWS KMS will provide you with an import token that will  use to call the AWS KMS API for importing your wrapped key material. We will, therefore, download both public and import token from AWS KMS and use them accordingly.

* Of course, we need the key material itself. Normally, it will come from an enterprise HSM or any sort of key management system. As per the workshop, we will create one with the library **OpenSSL**. This is the key material that will be wrapped and imported, as described in previous point.

* Finally, we will call the import API, to import it under the empty CMK we created in the first step. Note, this is a very sensible operation and we might or might not have permission to do so. If we don´t, we will needed to create them, based on the **Least Privilege** best practice.

Let's go through to the steps in deep:



### Step 1 - Create and empty key with origin set to external


The first step to do so is to issue the same create-key command but indicating the origin is external - this is, the key material will not come from AWS KMS, but from an external source

$ aws kms create-key --origin EXTERNAL

```
{
    "KeyMetadata": {
        "Origin": "EXTERNAL", 
        "KeyId": "external-key-id", 
        "Description": "", 
        "KeyManager": "CUSTOMER", 
        "Enabled": false, 
        "KeyUsage": "ENCRYPT_DECRYPT", 
        "KeyState": "PendingImport", 
        "CreationDate": 1538511755.698, 
        "Arn": "arn:aws:kms:eu-west-1::key/ca", 
        "AWSAccountId": "your-account-id"
    }
```


The key metadata response we have to the command is similar to the previous key generation. Note, however, the "Enabled" field shows "false" this time and the "KeyState" is indicating "PendingImport" instead of "Enabled". Basically we need to import our key material to have this CMK ready to use. The KeyId format is again a block of five sequences of numbers. Remenber to replace **<external-key-id>** by the actual value you obtained in previous command when it corresponds.

### Step 2 - Download public key and import token from AWS KMS

Now, we need to download a public key and an import token from AWS KMS, so we can wrap our key material for the upload into AWS KMS with them. The public key and import token that we will download from AWS KMS will be different for each key we generate with origin as EXTERNAL.

To download the public key and import token needed for the wrapping, we need to execute the following command, replacing "**external-key-id**" with the KeyId obtained in the previous step:

```
$ aws kms get-parameters-for-import --key-id external-key-id  --wrapping-algorithm RSAES_OAEP_SHA_1 --wrapping-key-spec RSA_2048
```



In this command we are specifying a specific wrapping algorithm and the wrapping key spec. There are more options that you can check in the [KMS documentation](https://docs.aws.amazon.com/kms/latest/APIReference/API_GetParametersForImport.htm) 
  to suit other cryptographic needs.

As a response, you will obtain a JSON file with the public key  base64 encoded, key id and import token base64 encoded. Copy the public key, this is the value of "PublicKey" field in the JSON response, into a new file and name it, for example **pkey.b64**.
You can use vim or nano or any other Linux text editor of your choice to create the file and paste the public key value copied. Do the same for the import token, this is the value from the "ImportToken" field in the JSON response, naming the file for example **token.b64**.
You will finally have two files in your directory:

```
$ ls -l
total 8
-rw-rw-r-- 1 ec2-user ec2-user  393 Oct  2 20:43 pkey.b64
-rw-rw-r-- 1 ec2-user ec2-user 2345 Oct  2 20:43 token.b64
```

We are ready to decode the b64 format. We will use the [OpenSSL](https://openssl.org/library), issuing the following command that will produce a binary file with the same filename but extenion .bin:

```
$ openssl enc -d -base64 -A -in pkey.b64 -out pkey.bin
```


We need to do the same for the import token:

```
$ openssl enc -d -base64 -A -in token.b64 -out token.bin
```


Both public key and import token are decoded from b64 format, and stored in binary format in its corresponding files: pkey.bin and token.bin. 
We have all we need to wrap our key material in order to upload it into AWS KMS. 

### Step 3 - Create the import material and encrypt it for the import

Now, where is our key material? Usually key material will come from an enterprise HSM or other key management system in the company that would be in charge of generating the keys.
For the workshop, we will generate the keys with the OpenSSL library directly in our instance. Using the following command:

```
$ openssl rand -out genkey.bin 32
```

It will generate a 256 bit symmetric key and stores into file **genkey.bin**.
The key that the file contains is our key material for the CMK.

We now will wrap this key material with the public key obatined from AWS KMS before (pkey.bin):
```
$ openssl rsautl -encrypt -in genkey.bin -oaep -inkey pkey.bin -keyform DER -pubin -out WrappedKeyMaterial.bin
```

This command takes the generated key material and encrypt it with the public key we downloaded from AWS KMS. Then, saves the output in another file named **WrappedKeyMaterial.bin**.


#### Step 4 - Import your key material 

The final step is to do the import itself. For that operation,  we will use the aws kms **import-key-material** command. We will need the import token we have in the **token.bin** file and the wrapped encrypted key material we have just stored in file  **WrappedKeyMaterial.bin**.

Note in command below  the "fileb://" prefix. It is basically telling AWS CLI that the parameter data is binary data.  
Also note the expiration date we have provided for the key material. On that date, AWS KMS will delete the key material and the CMK is not longer there for being used. You can always set the expiration model to "KEY_MATERIAL_DOES_NOT_EXPIRE" instead of "KEY_MATERIAL_EXPIRES", if you don't want the CMK to expire. We will go deeper into key rotation and deletion later in the workshop.
As usual, replace "**your-key-id**", with the real **KeyId** of the key generated in Step 1 above for external origin.

```
$ aws kms import-key-material --key-id your-key-id --encrypted-key-material fileb://WrappedKeyMaterial.bin --import-token fileb://token.bin --expiration-model KEY_MATERIAL_EXPIRES --valid-to 2021-02-01T12:00:00-08:00
```

All going well, the above command must have failed with the following error message:

```
when calling the ImportKeyMaterial operation: User: arn:aws:sts:::assumed-role/is not authorized to perform: kms:ImportKeyMaterial on resource: arn:aws:kms:eu-west-1:account-id:key/key-id
```
As you can read in the error message, even though our instance has a "Power user" role, it is still missing some capabilities. We are following Least Privilege practices, therefore we are only providing the role the permissions it needs. In this case, we need to provide it with accesss to the "**ImportKeyMaterial**" operation.

We need to go back to the IAM service into the AWS console and add this permission to the role we are working with "**KMSWorkshop-InstanceInitRole**".  
Go back to the console, navigate to the IAM service. Look and click on the left column, the  "**Policies**" section. Then hit "**Create Policy**" button. Search and select for the service "KMS" among all displayed on the screen. You will land in  the policy creator/editor for KMS, as in image below: 


![Figure-6](/res/S1F6%20KMSPolicy.png)

<**Figure-6**>


Scroll down a little bit, and in the "**Actions**" section, go to "**Access Level**", within "**Write**" section,  and select "**ImportKeyMaterial**" , like it is seen in the image below:


![Figure-7](/res/S1F7%20KMSImport.png)

<**Figure-7**>


Finally, select resources "**Any**" and click "**Review Policy**". 

![Figure-8](/res/S1F8%20KMSResources.png)

<**Figure-8**>

In this step, give the policy a name, for example "**KMS-Workshop-ImportMaterialPermissions**" and hit "**Create Policy**".
 
With this, go back to the "**Roles**" section again (left side of the console within IAM service).
Search again for "**KMS**" to find the role **KMSWorkshop-InstaceInitRole**, as we did in the second step when creating a CMK with no import material. Click on it.

To attach the new policy we have just created to the role. Hit the button "**Attach policies**". 

![Figure-9](/res/S1F9%20KMSARole.png)
<**Figure-9**>
 
 
an new screen will appear. Search for the policy we created by "**KMS**", you should find its name "**KMS-Workshop-ImportMaterialPermissions**", As you can see in image below. Then select it and hit the "**Attach Policy**" button.
The Role now has pemissions to import the key material.

 
![Figure-10](/res/S1F10%20KMSApolicy.png)
<**Figure-10**>


Let's try now the command again, this time it will succeed. We have successfully imported our key into AWS KMS. 
```
$ aws kms import-key-material --key-id your-key-id --encrypted-key-material fileb://WrappedKeyMaterial.bin --import-token fileb://token.bin --expiration-model KEY_MATERIAL_EXPIRES --valid-to 2019-09-01T12:00:00-08:00
```

**NOTE:** Depending when you follow this workshop you might need to adjust the time in "**KEY_MATERIAL_EXPIRES --valid-to**" as it can not be more than 365 days from the current date.


We might want to set an alias for this new key as well. We will use the alias "**ImportedCMK**". The command is the same as we used in previous section. Remember to replace "**external-key-id**" with the actual KeyId you obtain in Step-1 when creating a CMK with external origin:

```
$ aws kms create-alias --alias-name alias/ImportedCMK --target-key-id 'external-key-id'
```

If you go back into the AWS console, browse to the IAM service and select "**Encryption Keys**" to open the KMS console. Remember to make sure you have selected the right region (picture below- upper red bounding box). The new imported key with its alias is shown and it is ready to use.

![Figure-11](/res/S1F11.png)

<**Figure-11**>

You also have the option to display into the AWS CLI the ids and aliases of your keys with the command "**aws kms list-aliases**". The JSON output will display the keys we have created and its alias, along with the aws service CMKs created by default for some of the services. 

```
$ aws kms list-aliases

{
    "Aliases": [
        {
            "AliasArn": "arn:aws:kms:your-region:your-account:alias/FirstCMK", 
            "AliasName": "alias/FirstCMK", 
            "TargetKeyId": "your-key-id"
        }, 
        {
            "AliasArn": "arn:aws:kms:your-region:your-account:alias/ImportedCMK", 
            "AliasName": "alias/ImportedCMK", 
            "TargetKeyId": "external-key-id"
        }, 
        ...}
        
```


 ----
 
 

## Rotating AWS KMS CMKs

Key rotation is a very important in key management and a security best practice.
In AWS KMS there are different ways to rotate keys according to the way they were created.

### Step 1 - CMKs generated with AWS key material 

For CMKs created with AWS key material, you can opt-in to automatically rotate the key every year
AWS KMS generates new cryptographic material for the CMK every year. In this case,  AWS KMS also saves the CMK's older cryptographic material so it can be used to decrypt data that it encrypted.

Automatic key rotation preserves the properties of the CMK: key ID, key ARN, region, policies, and permissions, do not change when the key is rotated, so you don´t have to manually update the alias of the CMK to point to a newly generated CMK.

Let's opt-in to automatically rotate the CMK key we created before with AWS key material, remenber its alias was "**FirstCMK**", the KeyID was "**your-key-id**". 

```
$ aws kms enable-key-rotation --key-id your-key-id

```


If the command executed successfully, you have enabled the automatic rotation of the CMK, that will happen in 365 days since the command executed, this is: 1 year from now.

Another way to rotate the CMKs built with AWS key material is to generate a new key.
Then, to use then new CMK after rotation you might need to replace the information on our applications to point to this new key one by one every place in the application's code. However, it is more efficient work with aliases, as we explained before, and to update the CMK alias to point to the new key just created. Let's do it:


Firt of all create a new key with AWS key material. We already used this command before, in fact, it was the first command in the workshop.

```
$ aws kms create-key
```

You will obtain a response with a new KeyID and a new Key ARN. We can update the alias we set i for the first key we generated, "**FirstCMK**", to point to this new key, so it replaces the old one. For that, you will use the KeyID or key ARN of the new key you have just created.

```
$ aws kms update-alias --alias alias/FirstCMK --target-key-id KeyId
```

It turns out that the role our instance uses does not have permissions to update aliases. You need to follow the same steps you followed when providing permissions to import keys to the role. This is: create a policy for AWS KMS service that allows you update aliases and then attach it to the role. Also, this policy should allow to disable, enable, schedule deletion and delete CMKs; we are needing this permissions for the next step, so it is wise adding them now.

At this point of the workshop you should be able to do it with no issues. One tip: Update aliases, enable and disable key and delete operation are part of the Write operations of AWS KMS. See image below.

![Figure-16](/res/S1F16.png)

<**Figure-16**>


For resources select both "**alias**" and "**key**". You can name the policy "**KMSWorshop-RotationDisableOps**". 

Try the command again after you attached the policy to the role. All the applications that were using "FirstCMK" key alias, are now using the new key. In this way, we did not have to manually change the "KeyId" or key ARN one by one in all occurrences of our code were the CMK is invoked. 

The old key remains in AWS KMS (until you delete it). When you use the CMK to decrypt, AWS KMS uses the backing key that was used to encrypt, this is, for example, if you needed to decrypt some encrypted data previous to the alias change. It will happen automatically.  More information about Key Rotation in [this section](https://docs.aws.amazon.com/kms/latest/developerguide/rotate-keys.html) of AWS KMS documentation.

### Step 2 - Rotating CMKs generated with your own key material 


With the CMKs generated with your own key material, automatic rotation is not possible. You have to manually create a new key, with your own material, and again:  Either update the alias of the CMK (recommendable) or change your code to point to the new key. It seems much easier just to change alias pointer. 

In order to do so, we would need to create a new key with imported key material, as we did with to cfreate the CMK "ImportedCMK" with external origin and then udpate the alias "**ImportedCMK**". **NOTE:** Timewise, you are **not required** to do it as part of the workshop, as the procedure is already covered.

In case you would like to rotate the CMK created with your own key material, follow the procedure in the section above "[Generate CMK with your own key material](https://github.com/DanGOTO100/Draft-AWS-KMS-Workshop/blob/master/Working-with-CMKs.md#generate-cmks-with-your-own-key-material)".
Once you have created a new CMK with you new imported key material, update the alias "**ImportedCMK**" to point to the new key you have provided. Replace **KeyID** in command below with the KeyID of your newly created CMK.

```
$ aws kms update-alias --alias ImportedCMK --target-key-id KeyID
```

For CMKs created by AWS and using AWS key material:  AWS Managed CMKs, the rotation is automatically happening every three years. See [this link](https://docs.aws.amazon.com/kms/latest/developerguide/rotate-keys.html) of AWS KMS documentation.to the documentation for more info on key rotation.

---


### Deleting AWS KMS CMKs

Deleting customer master keys is a very sensitive operation.  You should delete a CMK only when you are sure that you don't need to use it anymore. The implications of key deletion are explained in the following [section](https://docs.aws.amazon.com/kms/latest/developerguide/deleting-keys.html) of the AWS KMS documentation, please read carefully.

Providing the right permissions for key deletion are an important part of best practices working with AWS KMS, as we will see in next section.

If you are not sure that you need to delete the key, you might want to disable the key only. Execute the following command to change the state of our first key "**FirstCMK**" to disabled. You will have to replace "**your-key-id**" with your corresponding KeyId or ARN (**NOTE:**) Key Aliases are not supported for this operation. 

```
$ aws kms disable-key --key-id your-key-id
```

Let's re-enable it to keep using it. In order to do so, execute the following command (again, replace "**your-key-id**" with your corresponding KeyID or ARN) :
```
$ aws kms enable-key --key-id your-key-id
```
For the deletion operation, AWS KMS enforces a waiting period. To delete a CMK in AWS KMS you have to schedule a key deletion.
You can set the waiting period from a minimum of 7 days up to a maximum of 30 days. The default waiting period is 30 days. Let's schedule key deletion in seven days, use the following command. Please, replace "**your-key-id**" with the  corresponding KeyID or ARN for the first CMK you created with the firt AWS KMS command in this workshop, the one is not currently being point at by the alias.
```
$ aws kms schedule-key-deletion --key-id your-key-id --pending-window-in-days 7
{
    "KeyId": "arn:aws:kms:your-region:your-account-id:key/your-key-id", 
    "DeletionDate": 1544659200.0
}

```

Working with CMKs that have been generated with your own key material is a bit different because you can schedule a key deletion but you can also delete key material on demand. Therefore, for deletion of key material, you can schedule a date and wait for the key material to expire or you delete it manually.

If you may want to delete it **immediately**, you could issue a command like the one below to delete the key material you have imported, rendering the key unusable. You should replace "your-key-id" with your corresponding KeyID or ARN.

**The command below is for information purposes, don´t execute it as part of the workshop**. 

If for any reason you delete the key we generated with our own key material "**ImportedCMK**", later you would have to import again your key material into the CMK and into the same alias to get it back to an usable state.

## Just for information
```
$ delete-imported-key-material --key-id  your-key-id.   
```

Congratulations, you have now completed this section of the workshop. You can now go to the second section of the workshop: [Encryption with AWS KMS](https://github.com/aws-samples/aws-kms-workshop/blob/master/Section-2-Encryption-with-AWS-KMS.md)


