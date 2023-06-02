# Lambda-Xray-Proj

# Overview 
I will be creating a Lambda function that can be callved via Function URL, that will query a Dog photo API . It will then save the image to an S3 bucket and then show us the image in the browser. 
Dog API: https://dog.ceo/dog-api/

I will then set up Lambda X-Ray to view in the code if there are any bottle necks or errors. 
You need to download the function.zip file in the repo to upload it to Lambda. 
The Lambda function is in courtesey of Adam Cantrill's Course: Mini Project - Using Lambda and AWS X-Ray to debug ( serverless ) applications [^1]

You need to download the **function.zip** file in the repo. You can upload to Lambda as is. 
**lambda-function** is the main component of what the lambda function will be reading within the **function.zip** file.

I will be creating this environment in the us-east-1 or the US East(N.Virginia) region. If you are deploying elsewhere make sure it is consistent with a region that works for you.
Make sure you have an IAM account with proper privileges. It is recommended you login with an account that has admin privileges granted from the root account that you signed up with.

# Instructions
# Stage 1  - Create dog photo bucket
Head to the S3 console: https://s3.console.aws.amazon.com/s3/buckets
![Screenshot (647)](https://github.com/Michael-DTran/Lambda-Xray-Proj/assets/112426094/a8f31cc1-930e-4e52-b22a-7cfeb05e1b51)

[^1]: https://www.youtube.com/watch?v=V1Fj8uEyp-E&t=54s


