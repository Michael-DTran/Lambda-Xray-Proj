# Lambda-Xray-Proj

# Overview 
Below are applications I will use for the project:
![lambda-xray-arc](https://github.com/Michael-DTran/Lambda-Xray-Proj/assets/112426094/220257c1-db38-4b6a-a81e-bb441d63a7df)

I will be creating a Lambda function that can be callved via Function URL, that will query a Dog photo API . It will then save the image to an S3 bucket and then show us the image in the browser. 
Dog API: https://dog.ceo/dog-api/

I will then set up Lambda X-Ray to view in the code if there are any bottle necks or errors.
IAM will be used to gain permissions to authorize the lambda function.
You need to download the function.zip file in the repo to upload it to Lambda. 
The Lambda function is courtesy of Adam Cantrill's Course: Mini Project - Using Lambda and AWS X-Ray to debug ( serverless ) applications [^1]

You need to download the **function.zip** file in the repo. You can upload to Lambda as is. 
**lambda-function** is the main component of what the lambda function will be reading within the **function.zip** file.

I will be creating this environment in the US East(N.Virginia) us-east-1 region. If you are deploying elsewhere make sure it is consistent with a region that works for you.
Make sure you have an IAM account with proper account privileges. 



# Instructions
# Stage 1  - Create dog photo bucket
Head to the S3 console: https://s3.console.aws.amazon.com/s3/buckets
![Stage1-S3-Bucket](https://github.com/Michael-DTran/Lambda-Xray-Proj/assets/112426094/a8f31cc1-930e-4e52-b22a-7cfeb05e1b51)

I will call the bucket **cute-doge** (bucket names must be globally unique so use a name that is not taken) and create the bucket)
The region will be the US East(N.Virginia) us-east-1 region. All the other settings can be left as is. 

![Stage1 1](https://github.com/Michael-DTran/Lambda-Xray-Proj/assets/112426094/7cb520ab-892f-4c27-b2a7-0116a613c7ff)

# Stage 2 - Creating the Lambda IAM role

Go to the IAM console: https://s3.console.aws.amazon.com/s3/buckets

Click on **Roles**, then click **Create Role**

Set trusted entity type to "AWS Service" and select "Lambda"

Click **Next**

On the 'Add Permissions' page search for and select **Amazons3FullAccess**
Press on **Clear filters** and then search for **CloudWatchFullAccess**

This can be further locked down to a specific bucket with specific actions
Click **Next** 
Set the Role name to **dog-photo-function-role**
Click **Create role**

# Stage 3 - Create Lambda function
Head to the Lambda console https://us-east-1.console.aws.amazon.com/lambda
and click **Create Function**
![Stage3 1](https://github.com/Michael-DTran/Lambda-Xray-Proj/assets/112426094/6fc52dfa-e846-4917-9263-91ba2723ac94)


[^1]: https://www.youtube.com/watch?v=V1Fj8uEyp-E&t=54s


