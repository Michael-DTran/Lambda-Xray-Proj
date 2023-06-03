# Lambda-Xray-Proj

# Overview 
Below are applications I will use for the project:
![lambda-xray-arc](https://github.com/Michael-DTran/Lambda-Xray-Proj/assets/112426094/220257c1-db38-4b6a-a81e-bb441d63a7df)

We will be creating a Lambda function that can be callved via Function URL, that will query a Dog photo API . It will then save the image to an S3 bucket and then show us the image in the browser. 
Dog API: https://dog.ceo/dog-api/

We will then set up Lambda X-Ray to view in the code if there are any bottle necks or errors.
IAM will be used to gain permissions to authorize the lambda function.
You need to download the function.zip file in the repo to upload it to Lambda. 
The Lambda function is courtesy of Adam Cantrill's Course: Mini Project - Using Lambda and AWS X-Ray to debug ( serverless ) applications [^1]

You need to download the **function.zip** file in the repo. You can upload to Lambda as is. 
**lambda-function** is the main component of what the lambda function will be reading within the **function.zip** file.

We will be creating this environment in the US East(N.Virginia) us-east-1 region. If you are deploying elsewhere make sure it is consistent with a region that works for you.
Make sure you have an IAM account with proper account privileges. 



# Instructions
# Stage 1  - Create dog photo bucket
Head to the S3 console: https://s3.console.aws.amazon.com/s3/buckets
![Stage1-S3-Bucket](https://github.com/Michael-DTran/Lambda-Xray-Proj/assets/112426094/a8f31cc1-930e-4e52-b22a-7cfeb05e1b51)

We will call the bucket **cute-doge** (bucket names must be globally unique so use a name that is not taken) and create the bucket)
The region will be the US East(N.Virginia) us-east-1 region. All the other settings can be left as is. 

![Stage1 1](https://github.com/Michael-DTran/Lambda-Xray-Proj/assets/112426094/7cb520ab-892f-4c27-b2a7-0116a613c7ff)

# Stage 2 - Creating the Lambda IAM role

Go to the IAM console: https://us-east-1.console.aws.amazon.com/iamv2/
Click on **Roles**, then click **Create Role**

![Stage2 1](https://github.com/Michael-DTran/Lambda-Xray-Proj/assets/112426094/bbdd07eb-c491-4206-bddd-6a6a60e88c4f)

Set trusted entity type to "AWS Service" and select **Lambda**. Click **Next**
![Stage2 2](https://github.com/Michael-DTran/Lambda-Xray-Proj/assets/112426094/87c1eb66-d787-47d5-89fe-c9d9898d6b33)


On the 'Add Permissions' page search for and select **Amazons3FullAccess**. Checkmark the policy.
Press on **Clear filters** 

![Stage2 3](https://github.com/Michael-DTran/Lambda-Xray-Proj/assets/112426094/368bbcd6-6b70-4b32-a849-f6d43cbe229e)

and then search for **CloudWatchFullAccess**. Checkmark that policy and make sure there are 2 permissions polices selected
Click **Next**. 
![Stage2 4](https://github.com/Michael-DTran/Lambda-Xray-Proj/assets/112426094/6f1b96aa-dec5-4b25-a01b-89cb4aa5dfc8)

Set the Role name to **dog-photo-function-role**. Make sure that the two policy names that you added are included for this role.
Click **Create role**.

![Stage2 5](https://github.com/Michael-DTran/Lambda-Xray-Proj/assets/112426094/7ec7dbb2-44f1-4105-817d-bc740ba87f10)

# Stage 3 - Create Lambda function
Head to the Lambda console https://us-east-1.console.aws.amazon.com/lambda
and click **Create Function**
![Stage3 1](https://github.com/Michael-DTran/Lambda-Xray-Proj/assets/112426094/6fc52dfa-e846-4917-9263-91ba2723ac94)

Name the function as **dog-image-scraper**. Set the runtime to **Python 3.9**. Architecture can stay on **x86_64**.

![Stage3 2](https://github.com/Michael-DTran/Lambda-Xray-Proj/assets/112426094/7764794b-df5e-4d05-8bb5-59e43b88a51f)

Under Execution role change it to **Use an existing role** and make sure it is set to **dog-photo-function-role** from the previous permissions policy we made. Click on **Create Function**.

![Stage3 3](https://github.com/Michael-DTran/Lambda-Xray-Proj/assets/112426094/0be2faf3-8ea2-4e2a-8d0c-a9466c504d6a)

Back on the function page. Click on the **Code** tab and upload the zip file

![Stage3 4](https://github.com/Michael-DTran/Lambda-Xray-Proj/assets/112426094/5b777f04-f2ff-4033-8544-89452001a83f)

Upload the whole zip file and save it.

![Stage3 5](https://github.com/Michael-DTran/Lambda-Xray-Proj/assets/112426094/7effc7b2-acb0-409f-a56e-25bfa406de8a)

*Note*: The lambda function is attached to the source file. A zip file is used to package the x-ray sdk due to the file being too large for modifications in Lambda. Files must then be packaged in a zip file to be uploaded. If you need to further modify the code it must be then unzipped, making the changes, and then zipping it back up to upload to Lambda again. Modifications to the function at whole do not need to be modified. https://docs.aws.amazon.com/lambda/latest/dg/python-package.html
![Stage3 6](https://github.com/Michael-DTran/Lambda-Xray-Proj/assets/112426094/d882ae56-8cc7-4099-b19f-da613ffa4c90)


Go to the *Configuration* tab, then *Function URL* then click **Create function URL**

![Stage3 7](https://github.com/Michael-DTran/Lambda-Xray-Proj/assets/112426094/896accec-be04-40b1-b09d-2f00692df109)

On the Function URL page select *Auth type* to **NONE**. Select **Save** after.
This leaves our Lambda function open to the public to be called without a limit. It is recommended to set up an IAM role to only allow certain user(s) to use this function URL. 
For the purpose and scope of this demo we don't need to cover this aspect but it is something to keep in mind when creating more security in the cloud.
![Stage3 8](https://github.com/Michael-DTran/Lambda-Xray-Proj/assets/112426094/f7429dab-5ca0-4b3f-866c-4d4fb31e98e4)

Copy the Function URL onto your clipboard. We will need that soon.
![Stage3 9](https://github.com/Michael-DTran/Lambda-Xray-Proj/assets/112426094/fc385fe0-e82b-4bb9-9b41-bc83f9423868)

Under the tab *Environmental Variables* click on *edit*
Enter in *'BUCKET_NAME'* for **Key** and the bucket we created under **Value**

![Stage3 11](https://github.com/Michael-DTran/Lambda-Xray-Proj/assets/112426094/ef830cca-01bd-4b4e-8044-cef8b82143f6)

Under *Monitoring and operations tools* click on **Edit**
![Stage3 12](https://github.com/Michael-DTran/Lambda-Xray-Proj/assets/112426094/61f73d22-d807-4065-9dba-eda24df1e47a)

Toggle on the *active tracing* under **AWS X-Ray**
Under General Configuration click on *edit* change the timeout timer to *0 min and 15 sec* to prevent the Lambda  function from timing out.
Click **Save**
![Stage3 15](https://github.com/Michael-DTran/Lambda-Xray-Proj/assets/112426094/288737b9-b656-4604-a104-fbef05ff5812)

 
[^1]: https://www.youtube.com/watch?v=V1Fj8uEyp-E&t=54s


