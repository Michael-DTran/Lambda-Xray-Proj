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

>
import aws_xray_sdk.core
import boto3
import requests
import os
import base64
import io
import mimetypes

#Initialize the AWS X-Ray SDK
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
    >




[^1]: https://www.youtube.com/watch?v=V1Fj8uEyp-E&t=54s


