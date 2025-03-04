# AWS Challenge [WIP]

I recently asked ChatGPT for a challenge to create a simple application using Python, Django and some AWS tools, specifically API-Gateway, Lambda and S3. The goal was to develop some AWS skills, and here's what it returned. The links to the repositories and some important notes that I took while developing will be [on this other file](https://github.com/fczanetti/aws_challenge/blob/main/development.md).

### ChatGPT answer:

Cool! I'll propose an interesting challenge that will combine several AWS services you mentioned, as well as encourage you to use Docker. The goal will be to create an application that allows users to upload files to S3, process those files asynchronously with a Lambda function, and manage the workflow with SQS and the API Gateway.**

## Challenge: Asynchronous File Processing Application

### Scenario:

You will need to create an application that allows users to upload files (such as images or documents) via an API exposed by the API Gateway. After the upload, the file will be stored in S3, and then an event will be triggered to process the file via Lambda. The file processing will be done asynchronously using an SQS queue to ensure that the files are processed in order without overloading the system.

### Objectives:

- **API Gateway:** Create an API that allows file uploads.
- **S3:** Store the uploaded files.
- **SQS:** Send a message to the SQS queue every time a file is uploaded, indicating that it needs to be processed.
- **Lambda:** Process the files in the SQS queue. The processing can be any simple action, like resizing an image or extracting text from a document.
- **Docker:** Use Docker to package the Python/Django application and deploy it on AWS.
- **Deploy:** Use **AWS Elastic Beanstalk** to deploy the Django application. This is a tool you can use to simplify managing and deploying your application without needing to manually configure EC2 servers.

## Step-by-Step Instructions:

### Configure API Gateway:

- Create a RESTful API in API Gateway.
- Define an endpoint for file uploads (e.g., POST /upload).
- This endpoint should trigger a Lambda function responsible for uploading the file to S3.
- The Lambda function will be the first to be triggered upon upload.

### Configure S3:

- Create an S3 bucket to store the uploaded files.
- In the Lambda code, upload the received file to this bucket.
- After uploading, place a message in the SQS queue containing the file name and the bucket where it was stored.

### Configure SQS:

- Create an SQS queue to manage the files to be processed.
- The SQS queue will receive messages indicating which files need to be processed.
- Configure Lambda to be triggered by events from the SQS queue.

### Configure Lambda:

- The Lambda function will be triggered whenever a new message arrives in the SQS queue.
- In the Lambda code, access the file in S3 based on the information sent by the queue (bucket and file name).
- Process the file. For example, if it's an image, you can resize it, or if it's a PDF, you can extract text.
- After processing, you can store the processed file in a new bucket or in a folder inside the original bucket.

### Package with Docker:

- Package the Django application in a Docker container.
- Add all necessary dependencies, such as boto3 (to interact with AWS services) and other libraries you may need.
- Ensure the Dockerfile is correctly configured to run the Django application with all required resources.

### Deploy Using Elastic Beanstalk:

- Use AWS Elastic Beanstalk to deploy your Django application.
- Elastic Beanstalk is a simplified way to deploy applications and configure the underlying infrastructure, such as EC2, load balancers, auto-scaling, etc.
- Configure the environment to use Docker (Elastic Beanstalk supports Docker directly).
- You can create the Elastic Beanstalk environment either through the AWS Management Console or using the AWS CLI.

### Monitoring and Logs:

- Use CloudWatch to monitor the logs for your Lambda and Django application.
- Enable CloudWatch Logs for both Lambda and Elastic Beanstalk so that you can easily track the processing status and detect any failures.

### AWS Services to Use:

- **API Gateway** – To create and manage the API for file uploads.
- **S3** – To store the uploaded files.
- **SQS** – To control and manage the flow of messages for asynchronous processing.
- **Lambda** – To process the files asynchronously.
- **Elastic Beanstalk** – To deploy your Django application using Docker.
- **CloudWatch** – For monitoring and logs.

### Extra Challenge (Optional): 

Add authentication and authorization to the API, using **Cognito** or **IAM**, to ensure that only authenticated users can upload files.

### Deployment Steps:

- Create and configure resources on AWS (API Gateway, S3, SQS, Lambda).
- Prepare the Dockerfile to package your Django application.
- Deploy the Django application with Docker using Elastic Beanstalk.
- Connect the API Gateway API to the Lambda function that will upload files to S3 and place messages in SQS.
- Implement the processing code in Lambda.