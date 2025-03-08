## Repositories

- https://github.com/fczanetti/lambda_aws_challenge
- https://github.com/fczanetti/django_aws_challenge

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