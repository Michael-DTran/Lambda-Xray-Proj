# Lambda-Xray-Proj

# Overview 
Below are applications we will use for the project:
![lambda-xray-arc](https://github.com/Michael-DTran/Lambda-Xray-Proj/assets/112426094/220257c1-db38-4b6a-a81e-bb441d63a7df)

We will be creating a Lambda function that can be callved via Function URL, that will query a Dog photo API . It will then save the image to an S3 bucket and then show us the image in the browser. 
Dog API: https://dog.ceo/dog-api/

We will then set up Lambda X-Ray to view in the code if there are any bottle necks or errors.
IAM will be used to gain permissions to authorize the lambda function.
You need to download the function.zip file in the repo to upload it to Lambda. 
The Lambda function and project idea is courtesy of *Adam Cantrill's Course: Mini Project - Using Lambda and AWS X-Ray to debug ( serverless ) applications* [^1]

You need to download the **function.zip** file in the repo. You can upload to Lambda as is. 
**lambda-function** is the main component of what the lambda function will be reading within the **function.zip** file.

We will be creating this environment in the US East(N.Virginia) us-east-1 region. If you are deploying elsewhere make sure it is consistent with a region that works for you.
Make sure you have an IAM account with proper account privileges. 

# Instructions
# Stage 1  - Create dog photo bucket
Head to the console: https://s3.console.aws.amazon.com/s3/buckets
![Stage1-S3-Bucket](https://github.com/Michael-DTran/Lambda-Xray-Proj/assets/112426094/a8f31cc1-930e-4e52-b22a-7cfeb05e1b51)

We will call the bucket **cute-doge** (bucket names must be globally unique so use a name that is not taken) and create the bucket)
The region will be the US East(N.Virginia) us-east-1 region. All the other settings can be left as is. 

![Stage1 1](https://github.com/Michael-DTran/Lambda-Xray-Proj/assets/112426094/7cb520ab-892f-4c27-b2a7-0116a613c7ff)

# Stage 2 - Creating the Lambda IAM role

Go to the IAM console: https://us-east-1.console.aws.amazon.com/iamv2/
Click on **Roles**, then click **Create Role**.

![Stage2 1](https://github.com/Michael-DTran/Lambda-Xray-Proj/assets/112426094/bbdd07eb-c491-4206-bddd-6a6a60e88c4f)

Set trusted entity type to "AWS Service" and select **Lambda**. Click **Next**.
![Stage2 2](https://github.com/Michael-DTran/Lambda-Xray-Proj/assets/112426094/87c1eb66-d787-47d5-89fe-c9d9898d6b33)


On the 'Add Permissions' page search for and select **Amazons3FullAccess**. Checkmark the policy.

Press on **Clear filters**.

![Stage2 3](https://github.com/Michael-DTran/Lambda-Xray-Proj/assets/112426094/368bbcd6-6b70-4b32-a849-f6d43cbe229e)

and then search for **CloudWatchFullAccess**. Checkmark that policy and make sure there are 2 permissions polices selected
Click **Next**. 
![Stage2 4](https://github.com/Michael-DTran/Lambda-Xray-Proj/assets/112426094/6f1b96aa-dec5-4b25-a01b-89cb4aa5dfc8)

Set the Role name to **dog-photo-function-role**. Make sure that the two policy names that you added are included for this role.
Click **Create role**.

![Stage2 5](https://github.com/Michael-DTran/Lambda-Xray-Proj/assets/112426094/7ec7dbb2-44f1-4105-817d-bc740ba87f10)

# Stage 3 - Create Lambda function
Head to the Lambda console https://us-east-1.console.aws.amazon.com/lambda

Click **Create Function**.
![Stage3 1](https://github.com/Michael-DTran/Lambda-Xray-Proj/assets/112426094/6fc52dfa-e846-4917-9263-91ba2723ac94)

Name the function as **dog-image-scraper**. Set the runtime to **Python 3.9**. Architecture can stay on **x86_64**.

![Stage3 2](https://github.com/Michael-DTran/Lambda-Xray-Proj/assets/112426094/7764794b-df5e-4d05-8bb5-59e43b88a51f)

Under Execution role change it to **Use an existing role** and make sure it is set to **dog-photo-function-role** from the previous permissions policy we made. Click on **Create Function**.

![Stage3 3](https://github.com/Michael-DTran/Lambda-Xray-Proj/assets/112426094/0be2faf3-8ea2-4e2a-8d0c-a9466c504d6a)

Back on the function page. Click on the **Code** tab and upload the zip file.

![Stage3 4](https://github.com/Michael-DTran/Lambda-Xray-Proj/assets/112426094/5b777f04-f2ff-4033-8544-89452001a83f)

Upload the whole zip file and save it.

![Stage3 5](https://github.com/Michael-DTran/Lambda-Xray-Proj/assets/112426094/7effc7b2-acb0-409f-a56e-25bfa406de8a)

```
import aws_xray_sdk.core
import boto3
import requests
import os
import base64
import io
import mimetypes

# Initialize the AWS X-Ray SDK
aws_xray_sdk.core.patch_all()

def lambda_handler(event, context):
    # Start a new X-Ray segment
    with aws_xray_sdk.core.xray_recorder.capture('get_dog_images'):
        # Create an S3 client
        session = boto3.Session()
        s3 = session.resource('s3')
        bucket_name = os.getenv('BUCKET_NAME')

        # Call the Dog API
        with aws_xray_sdk.core.xray_recorder.capture('call_dog_api'):
            # Define the endpoint for the Dog API
            endpoint = 'https://dog.ceo/api/breeds/image/random'
            
            # Make a GET request to the Dog API
            response = requests.get(endpoint)
            
            # Get the image URL from the response
            image_url = response.json()['message']

            # Get the name of the image
            image_name = str(response.json()['message']).split('/')[-1]
            
            # Download the image from the URL
            image = requests.get(image_url, stream=True).content
            
        # Save the weather data to S3
        with aws_xray_sdk.core.xray_recorder.capture('save_dog_to_s3'):
            contenttype = mimetypes.types_map['.' + image_name.split('.')[-1]]
            bucket = s3.Bucket(bucket_name)
            bucket.upload_fileobj(io.BytesIO(image), image_name, ExtraArgs={'ContentType': contenttype})
        
    # Generate a response with the image in the body
    response = {
        'statusCode': 200,
        'headers': {
            'Content-Type': 'image/jpeg'
        },
        'body': base64.b64encode(image),
        'isBase64Encoded': True
    }
    return response

```

*Note*: The lambda function is attached to the source file. A zip file is used to package the x-ray sdk due to the file being too large for modifications in Lambda. Files must then be packaged in a zip file to be uploaded. If you need to further modify the code it must be then unzipped, changes made, and then zipped back up to upload to Lambda again. Modifications to the function at whole do not need to be modified. https://docs.aws.amazon.com/lambda/latest/dg/python-package.html

The lambda function is calling the API through the endpoint to generate a random dog image. X-ray is calling the lambda function through the API and through the S3 bucket where they will both be analyzed and debugged through X-ray.

![Stage3 6](https://github.com/Michael-DTran/Lambda-Xray-Proj/assets/112426094/d882ae56-8cc7-4099-b19f-da613ffa4c90)

Go to the *Configuration* tab, then *Function URL* then click **Create function URL**.

![Stage3 7](https://github.com/Michael-DTran/Lambda-Xray-Proj/assets/112426094/896accec-be04-40b1-b09d-2f00692df109)

On the Function URL page select *Auth type* to **NONE**. Select **Save** after.
This leaves our Lambda function open to the public to be called without a limit. It is recommended to set up an IAM role to only allow certain user(s) to use this function URL. 

For the purpose and scope of this demo we don't need to cover this aspect but it is something to keep in mind when creating more security in your cloud environment.

![Stage3 8](https://github.com/Michael-DTran/Lambda-Xray-Proj/assets/112426094/f7429dab-5ca0-4b3f-866c-4d4fb31e98e4)

Copy the Function URL onto your clipboard. We will need that soon.

![Stage3 9](https://github.com/Michael-DTran/Lambda-Xray-Proj/assets/112426094/fc385fe0-e82b-4bb9-9b41-bc83f9423868)

Under the tab *Environmental Variables* click on *edit*.
Enter in *'BUCKET_NAME'* for **Key** and the bucket we created under **Value**.

![Stage3 11](https://github.com/Michael-DTran/Lambda-Xray-Proj/assets/112426094/ef830cca-01bd-4b4e-8044-cef8b82143f6)

Under *Monitoring and operations tools* click on **Edit**.

![Stage3 12](https://github.com/Michael-DTran/Lambda-Xray-Proj/assets/112426094/61f73d22-d807-4065-9dba-eda24df1e47a)

Toggle on the *active tracing* under **AWS X-Ray**.

![Stage3 13](https://github.com/Michael-DTran/Lambda-Xray-Proj/assets/112426094/63e23a0a-18be-42ba-90cd-74f00dc3580a)

Under General Configuration click on *edit* change the timeout timer to *0 min and 15 sec* to prevent the Lambda  function from timing out.

Click **Save**.

![Stage3 15](https://github.com/Michael-DTran/Lambda-Xray-Proj/assets/112426094/288737b9-b656-4604-a104-fbef05ff5812)

 
[^1]: https://www.youtube.com/watch?v=V1Fj8uEyp-E&t=54s

# Stage 4 - Testing the Function

If you saved the function URL from the previous stage, copy and paste the function in a different tab to open it.
You can alternatively head to the Lambda console: https://us-east-1.console.aws.amazon.com/lambda

Under **functions** click on the function name *dog-image-scraper* and click on the *Function URL* from there. You should see a photo of a dog via this function URL

![Stage4 1](https://github.com/Michael-DTran/Lambda-Xray-Proj/assets/112426094/ef6926fe-c6a2-4e42-a979-c8eb080cf64a)

Head to S3 and click on the S3 bucket you made. The jpg file should be uploaded to the S3 bucket. If there is more than one image try to find the corresponding image with the Function URL you just opened.

![Stage4 2](https://github.com/Michael-DTran/Lambda-Xray-Proj/assets/112426094/98c1f76d-3131-466e-a78f-1f821b5859a8)

Click on the jpg file and open it.

![Stage4 3](https://github.com/Michael-DTran/Lambda-Xray-Proj/assets/112426094/f0470693-7286-4838-ad47-78b5cfd4113a)

You should notice that the file attached to the S3 bucket is the same image but the image is under a security token that authorizes you to view the file.
As you refresh the Function URL you will call the API to generate a random photo of another dog into the S3 bucket.

![Stage4 4](https://github.com/Michael-DTran/Lambda-Xray-Proj/assets/112426094/74bbd9ed-9fa0-4842-a4a4-40e7ebb76ec1)

# Stage 5 - View X-Ray Metrics
Go to the CloudWatch console: https://us-east-1.console.aws.amazon.com/cloudwatch

Head to service map to see the map of our application stack.

![Stage5 1](https://github.com/Michael-DTran/Lambda-Xray-Proj/assets/112426094/c25ada01-39f0-4da6-a382-c09d504d4bad)

You can find in-depth metrics of each application and its function calls. 

![Stage5 2](https://github.com/Michael-DTran/Lambda-Xray-Proj/assets/112426094/9a5eebdd-ea7f-4d78-99ad-1707c9540523)

You can specify deeper into the calls that are made.

![Stage5 3](https://github.com/Michael-DTran/Lambda-Xray-Proj/assets/112426094/e8e83d69-0e8f-4fa0-bb2e-9486b107fb33)

![Stage5 4](https://github.com/Michael-DTran/Lambda-Xray-Proj/assets/112426094/15ba20e3-0a11-4c10-810f-9b340ed7315c)

The next segment is to break the function and to monitor how X-ray picks up on it.
Go to Lambda: https://us-east-1.console.aws.amazon.com/lambda

Head to *environmental variables* and click *edit*.

![Stage5 5](https://github.com/Michael-DTran/Lambda-Xray-Proj/assets/112426094/59d731c1-11ce-4fd3-8c67-0ab5c03349d8)

Change the *value* to something other than the configured S3 bucket. Click **Save**.

![Stage5 6](https://github.com/Michael-DTran/Lambda-Xray-Proj/assets/112426094/74c547d9-0544-44bb-b8d3-ef2fedcd6a9c)

If you refresh the Function URL under the Lambda function you should receive an error.

![Stage5 8](https://github.com/Michael-DTran/Lambda-Xray-Proj/assets/112426094/549bee78-c920-4f00-94e0-2240a7449ebe)

If you head back to X-ray under the *traces* tab you will see the calls having errors. It will also point out the S3 bucket name change taking place. The icons in the traces map will be brown in color if they had errors in their path.

![Stage5 9](https://github.com/Michael-DTran/Lambda-Xray-Proj/assets/112426094/0fd48b2b-6560-433d-97de-0d850ebc06d2)

![Stage5 10](https://github.com/Michael-DTran/Lambda-Xray-Proj/assets/112426094/d7a78022-8eba-465b-9562-9cdb15abc917)

![Stage5 11](https://github.com/Michael-DTran/Lambda-Xray-Proj/assets/112426094/b212e825-ec11-4698-967e-0c5f7b76807c)

Investigating further on the *Exceptions* tab you will see how the bucket does not exist anymore with the one it was configured to. 

# Stage 6 - Cleaning up 

To tidy up the account to its orignal state head to S3: https://s3.console.aws.amazon.com/s3
Head to the bucket and empty it. Follow the prompts and click *empty*.

![Stage6 1](https://github.com/Michael-DTran/Lambda-Xray-Proj/assets/112426094/be4d3f06-8703-464d-ab92-3e30fcb6fce2)

![Stage6 2](https://github.com/Michael-DTran/Lambda-Xray-Proj/assets/112426094/1bfdf3a7-6884-48fc-8e35-0871111f6f37)

After it is emptied, delete the bucket. Follow the prompts to *delete bucket*.

![Stage6 3](https://github.com/Michael-DTran/Lambda-Xray-Proj/assets/112426094/fdf42114-e2a1-4057-b1cc-5749c4d3299b)

![Stage6 4](https://github.com/Michael-DTran/Lambda-Xray-Proj/assets/112426094/5f9d271f-31ca-4042-90c9-36f9eff3b5aa)

Next, head to Lambda: https://us-east-1.console.aws.amazon.com/lambda
Head to the actions tab to delete the function. Follow the prompts and then *delete*.

![Stage6 5](https://github.com/Michael-DTran/Lambda-Xray-Proj/assets/112426094/da9fa9a1-e73c-408b-8d03-f3264978420e)

![Stage6 6](https://github.com/Michael-DTran/Lambda-Xray-Proj/assets/112426094/98519e77-9a8c-46b2-862e-73ca5ceb68be)

Next, head to IAM: https://us-east-1.console.aws.amazon.com/iamv2
Under the search bar enter in **dog-photo-function-role** Click on the role and press the *delete* button. Follow the prompts to delete.

![Stage6 8](https://github.com/Michael-DTran/Lambda-Xray-Proj/assets/112426094/7d40807a-2dea-4735-8d59-b0a7112fe061)

![Stage6 9](https://github.com/Michael-DTran/Lambda-Xray-Proj/assets/112426094/877bcbec-4fd2-4495-8b18-06185e2d2548)

![Stage6 10](https://github.com/Michael-DTran/Lambda-Xray-Proj/assets/112426094/4c272574-65bf-4f25-8946-2235f5075aa1)

Finally, head to Cloudwatch: https://us-east-1.console.aws.amazon.com/cloudwatch
Go to *Log groups* and under the *Actions* tab go to *Delete log group(s)*.
Press **Delete**.

![Stage6 11](https://github.com/Michael-DTran/Lambda-Xray-Proj/assets/112426094/177b4fee-325b-4c8b-abc5-1d09754cfb64)

![Stage6 12](https://github.com/Michael-DTran/Lambda-Xray-Proj/assets/112426094/8ab06a77-3601-448d-9114-3a8e1b2fd5e3)



