# cloudportfolio Serverless Image Processing Pipeline
Codelab: Process images uploaded to Cloud Storage using Cloud Functions

🔧 Tech Stack:
Cloud Storage

Cloud Functions

Vision API

Pub/Sub

Firebase Hosting (optional)

💡 What You’ll Learn:
Trigger functions on file upload

Analyze image metadata (e.g., labels, safe search)

Store processed metadata


Cloud Function Code (Example: Python):
Python

from google.cloud import storage
import io
from PIL import Image

def process_image(event, context):
    """
    This function is triggered when an object is uploaded to Cloud Storage.
    It downloads the image, resizes it, and uploads the resized image back.
    """

    # Get the object details from the event
    bucket_name = event['bucket']
    object_name = event['name']

    # Create a Cloud Storage client
    storage_client = storage.Client()

    # Get the bucket and blob
    bucket = storage_client.bucket(bucket_name)
    blob = bucket.blob(object_name)

    # Download the image
    image_data = blob.download_as_string()
    image = Image.open(io.BytesIO(image_data))

    # Resize the image (example)
    image.thumbnail((200, 200))

    # Create a new blob for the resized image
    resized_blob = bucket.blob(f"resized/{object_name}")

    # Upload the resized image
    with io.BytesIO() as output:
        image.save(output, format="JPEG")
        output.seek(0)
        resized_blob.upload(output, content_type="image/jpeg")

    print(f"Image {object_name} processed and resized. Resized image uploaded to gs://{bucket_name}/resized/{object_name}")
3. Key Concepts and Considerations:
Event-Driven Architecture:
Cloud Functions are event-driven, meaning they execute in response to specific events. 
Pub/Sub:
Cloud Storage uses Pub/Sub to publish notifications when events occur, which Cloud Functions can subscribe to. 
Dependencies:
If your Cloud Function needs external libraries (e.g., PIL for image processing), you'll need to install them in your function's environment. 
Permissions:
Ensure your Cloud Function has the necessary permissions to access Cloud Storage and other services it interacts with. 
Error Handling:
Implement robust error handling in your Cloud Function to gracefully handle failures. 
Asynchronous Operations:
For tasks that take a long time, consider using asynchronous operations (e.g., using return statements that return promises) to avoid blocking the function's execution. 
Temporary Files:
If your function needs to download and process files, you can use temporary files on the function's instance. 
Cost:
Be mindful of the cost of Cloud Functions, especially if you're processing a large number of images. 
Image Processing Libraries:
Explore different image processing libraries (e.g., ImageMagick, OpenCV) based on your needs. 
Vision API:
You can use the Cloud Vision API to analyze images and extract information. 
Eventarc:
Eventarc can be used to trigger Cloud Functions from Cloud Storage events. 
https://cloud.google.com/run/docs/tutorials/codelabs
https://cloud.google.com/run/docs/tutorials/image-processing
https://cloud.google.com/run/docs/tutorials/pubsub
