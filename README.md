# Lambda-Xray-Proj

# Overview 
I will be creating a Lambda function that can be callved via Function URL, that will query a Dog photo API . It will then save the image to an S3 bucket and then show us the image in the browser. 
Dog API: https://dog.ceo/dog-api/

I will then set up Lambda X-Ray to view in the code if there are any bottle necks or errors. 
You need to download the function.zip file in the repo to upload it to Lambda. 
The Lambda function is in courtesey of Adam Cantrill's Course: Mini Project - Using Lambda and AWS X-Ray to debug ( serverless ) applications [^1]

You need to download the **function.zip** file in the repo. You can upload to Lambda as is. 
**lambda-function** is the main component of what the lambda function will be reading within the **function.zip** file.

I will be creating this environment in the us-east-1 region. If you are deploying elsewhere make sure it is consistent with a region that works for you.

[^1]: https://www.youtube.com/watch?v=V1Fj8uEyp-E&t=54s

# Instructions
