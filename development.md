## Repositories

- https://github.com/fczanetti/django_aws_challenge
- https://github.com/fczanetti/lambdas_aws_challenge

## Creating and configuring API - Gateway
To create an API - Gateway I followed these steps:
- create an AWS account;
- create an API - Gateway;
- create a resource named `/uploads` (or any other name);
- create a POST method. The integration type here will be Lambda function, since it will direct requests to a Lambda. The next step is to select the lambda to which this API - Gateway will direct the requests to. Then click 'Create method';
- inside our new method, we have to setup the Integration request. In the Mapping templates section, we have to create a content type named `application/png`. This is necessary because we are sending images to the Lambda. Still in Mapping templates, on the Generate template field we have to choose `Method request passthrough`. After it click 'Save';
- finally, also to make our API - Gateway able to accept binary data (images), we have to go into 'API Settings' section and create a 'Binary media type' called `application/png`.

After creating the method we have to deploy the API - Gateway. When we click 'Deploy API' we can choose a stage for this deploy or create a new one. The stage is basically the version of the API. We can have, for example, the stages `dev` and `prod`. The `dev` stage can be used to test new endpoints and, if everything works fine, we can do another deploy, but this time to `prod` stage. 

If we navigate into 'Stages' section we'll see that each of our stages have a different 'Invoke URL' to be used.

To test it, we can send a POST request to the URL shown in the Stages section (`https://xxxxxxxxxx.execute-api.sa-east-1.amazonaws.com/dev/uploads`) after submiting our Django form. It can be done in the view that receives the data from the form. [Check it here](https://github.com/fczanetti/django_aws_challenge/blob/main/image_upload/base/views.py).

## Creating and configuring the S3 Bucket

To have S3 Bucket able to receive images from our Lambda we need to:

- create an AWS account;
- create an IAM User with the 'AmazonS3FullAccess' policy attached;
- create an S3 Bucket;
- on the bucket, set a policy that allows this IAM User to put objects on the bucket. Something like this:
```
{
    "Version": "2012-10-17",
    "Id": "Policy1740923855193",
    "Statement": [
        {
            "Sid": "Stmt1740923853930",
            "Effect": "Allow",
            "Principal": {
                "AWS": "<IAM_USER_ARN>"
            },
            "Action": "s3:PutObject",
            "Resource": "<BUCKET_ARN>/*"
        }
    ]
}
```

## Creating an SQS Queue

We also need an SQS Queue allowed to receive notifications from our bucket, and this should happen every time a image is uploaded to the bucket. The following steps are enough:

- create a standard SQS Queue;
- insert a policy on this queue. This policy should allow our S3 Bucket to send notifications to this queue. This is an example:
```
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "AllowS3SendMessage",
      "Effect": "Allow",
      "Principal": {
        "Service": "s3.amazonaws.com"
      },
      "Action": "sqs:SendMessage",
      "Resource": "<SQS_QUEUE_ARN>",
      "Condition": {
        "ArnLike": {
          "aws:SourceArn": "<S3_BUCKET_ARN>"
        }
      }
    }
  ]
}
```
The condition on this policy means that only that specific bucket can send messages to our queue.

## Creating a notification on the S3 Bucket
When creating the Event notification, it's important to set a prefix, meaning that only when images with keys starting with that prefix would trigger notifications.

For example, we can have a folder in the bucket called `images/`. For an image to be uploaded to this folder, it's key have to be `images/name_of_the_image.png`. And when these images are sent to the bucket, a notification/message will be sent to our SQS Queue.

If the prefix is not set, every upload would trigger a notification. This notification would trigger a lambda to get the image, process and send it to the bucket again. When sendind the image processed back to the bucket, another notification would be sent to SQS, and it could start a loop (we don't want it to happen).

The Event types can be just a Put action, and in the Destination we can choose SQS queue and select our queue created previously.

By now, every time our bucket folder `images/` receives an image via Put action, a message will be sent to the SQS.