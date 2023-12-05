Introduction to AWS Lambda
 

Overview
The lab provides a basic explanation of AWS Lambda. It will demonstrate the steps required to get started to create a Lambda function in an event-driven environment.

AWS Lambda is a compute service that runs your code in response to events and automatically manages the compute resources for you, making it easy to build applications that respond quickly to new information. AWS Lambda starts running your code within milliseconds of an event such as an image upload, in-app activity, website click, or output from a connected device. You can also use AWS Lambda to create new back-end services where compute resources are automatically triggered based on custom requests.

Topics covered
By the end of this lab you will be able to:

Create an AWS Lambda function

Configure an Amazon S3 bucket as a Lambda Event Source

Trigger a Lambda function by uploading an object to Amazon S3

Monitor AWS Lambda S3 functions through Amazon CloudWatch Log

Prerequisites
Familiarity with Amazon S3 would be beneficial.


Scenario
This lab demonstrates AWS Lambda by creating a serverless image thumbnail application.

The following diagram illustrates the application flow:

Application-Flow

1 - A user uploads an object to the source bucket in Amazon S3 (object-created event).

2 - Amazon S3 detects the object-created event.

3 - Amazon S3 publishes the object-created event to AWS Lambda by invoking the Lambda function and passing event data as a function parameter.

4 - AWS Lambda runs the Lambda function.

5 - From the event data it receives, the Lambda function knows the source bucket name and object key name. The Lambda function reads the object and creates a thumbnail using graphics libraries, then saves the thumbnail to the target bucket.

Upon completing this tutorial, you will have the following resources in your account:

Account-Resources

The steps in this lab will show you how to create the Amazon S3 buckets and the Lambda function. You will then test the service by uploading images for resizing.

 

Task 1: Create the Amazon S3 Buckets
In this task, you will create two Amazon S3 buckets -- one for input and one for output.

In the AWS Management Console, select the  Services menu, and then select S3 under Storage.

Click  Create bucket then configure:

Bucket name: images-NUMBER

Replace NUMBER with a random number

Copy the name of your bucket to a text editor

Region: choose US West (Oregon)

Click Create

Every bucket in Amazon S3 requires a unique name such as images-34523452345.

 If you receive an error stating The requested bucket name is not available, then click the first Edit link, change the bucket name and try again until it works.

You will now create another bucket for output.

Click  Create bucket then configure:

Bucket name: Paste the name of your images bucket

At the end of the bucket name, append -resized

Region: choose US West (Oregon)

Click Create

 Be sure both buckets were created in the US West (Oregon) Region.

You should now have buckets named similar to:

images-123

images-123-resized



You will now upload a picture for testing purposes.

download the picture to your computer:

HappyFace

Name the file HappyFace.jpg.

 Firefox users: Make sure the saved filename is HappyFace.jpg (not .jpeg).



Open the image on your computer.

It is a large picture, with dimensions of 1280 x 853.

In the S3 Management Console, click the images- bucket. (Not the -resized bucket)

Click  Upload

In the Upload window, click Add files

Browse to and select the HappyFace.jpg picture you downloaded.

Click Upload

Later in this lab you will invoke the Lambda function manually by passing sample event data to the function. The sample data will refer to this HappyFace.jpg image.

 

Task 2: Create an AWS Lambda Function
In this task, you will create an AWS Lambda function that reads an image from Amazon S3, resizes the image and then stores the new image in Amazon S3.

In the AWS Management Console, select the  Services menu, and then select Lambda under Compute. From the Region menu at the top of the console, choose US West (Oregon). 

 You must use US West (Oregon) for this lab.

Click Create function

 Blueprints are code templates for writing Lambda functions. Blueprints are provided for standard Lambda triggers such as creating Alexa skills and processing Amazon Kinesis Firehose streams. This lab provides you with a pre-written Lambda function, so you will Author from scratch.

In the Create function window, configure:

Function name: Create-Thumbnail

Runtime: Python 3.10

Expand  Change default execution role

Execution role: Use an existing role

Existing role: lambda-lab2-role

This role grants permission to the Lambda function to access Amazon S3 to read and write the images.

Click Create function

A page will be displayed with your function configuration.

AWS Lambda functions can be triggered automatically by activities such as data being received by Amazon Kinesis or data being updated in an Amazon DynamoDB database. For this lab, you will trigger the Lambda function whenever a new object is created in your Amazon S3 bucket.

Click Add trigger, then in the Trigger configuration screen, select S3 from the drop down menu.

Make these additional configurations:

Bucket: Select your images- bucket (e.g. images-123)

Event type: All object create events

Scroll to the bottom of the screen, then click Add

Click Create-Thumbnail at the top of the diagram:
Create-thumbnail-button

 

You will now configure the Lambda function.
    

Scroll down to the Code section and configure the following settings (and ignore any settings that aren't listed):

In the Code section, from the Upload from drop-down menu, select Amazon S3 location.

Amazon S3 link URL: Copy and paste this URL into the field:

https://aws-tc-largeobjects.s3.us-west-2.amazonaws.com/CUR-TF-100-PRYODA-1-37918/8-lab-introduction-lambda/s3/CreateThumbnail.zip
Choose the Code tab. In the Runtime settings section , click Edit and then for Handler: paste in CreateThumbnail.handler. Choose Save.

The CreateThumbnail.zip file contains the following Lambda function:

 Do not copy this code -- it is just showing you what is in the Zip file.

import boto3
import os
import sys
import urllib
from PIL import Image
import PIL.Image

s3_client = boto3.client('s3')

def resize_image(image_path, resized_path):
    with Image.open(image_path) as image:
        image.thumbnail((128,128))
        image.save(resized_path)

def handler(event, context):
    for record in event['Records']:
        bucket = record['s3']['bucket']['name']
        key = record['s3']['object']['key']
        raw_key = urllib.parse.unquote_plus(key)
        download_path = '/tmp/{}'.format(key)
        upload_path = '/tmp/resized-{}'.format(key)

        s3_client.download_file(bucket, raw_key, download_path)
        resize_image(download_path, upload_path)
        s3_client.upload_file(upload_path,
             '{}-resized'.format(bucket),
             'thumbnail-{}'.format(raw_key),
             ExtraArgs={'ContentType': 'image/jpeg'})

Examine the above code. It is performing the following steps:

Receives an Event, which contains the name of the incoming object (Bucket, Key)

Downloads the image to local storage

Resizes the image using the Pillow library

Uploads the resized image to the -resized bucket

Choose the Configuration tab. Choose General configuration and choose Edit. For Description enter: Create a thumbnail-sized image

You will leave the other settings as default, but here is a brief explanation of these settings:

Memory defines the resources that will be allocated to your function. Increasing memory also increases CPU allocated to the function.

Timeout sets the maximum duration for the function to run.

VPC (under Network) provides the Lambda function access to resources within a Virtual Private Cloud (VPC) network.

Dead Letter Queue (DLQ) Resource (under Debugging and error handling) defines how to handle failed function runs.

Enable active tracing allows tracing and monitoring of distributed code via AWS X-Ray.

Click Save at the bottom of the window.

Your Lambda function has now been configured.

 

Task 3: Test Your Function
In this task, you will test your Lambda function. This is done by simulating an event with the same information normally sent from Amazon S3 when a new object is uploaded.

Choose the Test tab, then configure:

Event name: Upload

Event template: s3-put

A sample template will be displayed that shows the event data sent to a Lambda function when it is triggered by an upload into Amazon S3. You will need to edit the bucket name so that it uses the bucket you created earlier.

Replace example-bucket with the name of your images bucket (e.g. images-123) that you copied to your text editor.

Be sure to replace example-bucket in both locations.

Bucket

Replace test/key with the name of the picture that you uploaded. This should be HappyFace.jpg

Key

Click Save

Click Test

AWS Lambda will now trigger your function, using HappyFace.jpg as the input image.

Towards the top of the page you should see the message: Execution result: succeeded

 If your test did not succeed, the error message will explain the cause of failure. For example, a Forbidden message means that the image was not found possibly due to an incorrect bucket name. Review the previous steps to confirm that you have configured the function correctly.

Click  Details to expand it (towards the top of the screen).

You will be shown information including:

Execution duration

Resources consumed

Maximum memory used

Log output

You can now view the resized image that was stored in Amazon S3.

In the AWS Management Console, select the  Services menu, and then select S3 under Storage.

Click the name of your -resized bucket (which is the second bucket you created), then:

Click HappyFace.jpg

Click Open (If the image does not open, disable your pop-up blocker.)

The image should now be a smaller thumbnail of the original image.

You are welcome to upload your own images to the images- bucket and then check for thumbnails in the -resized bucket.

 

Task 4: Monitoring and Logging
You can monitor AWS Lambda functions to identify problems and view log files to assist in debugging.

In the AWS Management Console, select the  Services menu, and then select Lambda under Compute.

Click your Create-Thumbnail function.

Click the Monitor tab.

The console displays graphs showing:

Invocations: The number of times the function has been invoked.

Duration: How long the function took to run (in milliseconds).

Errors: How many times the function failed.

Throttles: When too many functions are invoked simultaneously, they will be throttled. The default is 1000 concurrent runs.

Iterator Age: Measures the age of the last record processed from streaming triggers (Amazon Kinesis and Amazon DynamoDB Streams).

Dead Letter Errors: Failures when sending messages to the Dead Letter Queue.

Log messages from Lambda functions are retained in Amazon CloudWatch Logs.

Click View CloudWatch logs

Click the log stream that appears.

Expand  each message to view the log message details.

The Event Data includes the Request Id, the duration (in milliseconds), the billed duration (rounded up to the nearest 100 ms, the Memory Size of the function and the Maximum Memory that the function used. In addition, any logging messages or print statements from the functions are displayed in the logs. This assists in debugging Lambda functions.

 

Conclusion
 Congratulations! You have successfully:

Created an AWS Lambda function

Configured an Amazon S3 bucket as a Lambda Event Source

Triggered a Lambda function by uploading an object to Amazon S3

Monitored AWS Lambda S3 functions through Amazon CloudWatch Log