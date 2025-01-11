# Serverless Automation for Image Processing!

serverless-image-processing/
|
├── lambda_function.py       # Codul AWS Lambda
├── README.md                # Documentația proiectului
├── requirements.txt         # Dependențe (ex: boto3, Pillow)
└── assets/                  # Imagini demo, example

# Codul pentru lambda_function.py

import boto3
from PIL import Image
import os
import io

s3 = boto3.client('s3')

def lambda_handler(event, context):
    bucket_name = event['Records'][0]['s3']['bucket']['name']
    key = event['Records'][0]['s3']['object']['key']
    
    try:
        # Descarcă imaginea din S3
        tmp_file = f"/tmp/{key.split('/')[-1]}"
        s3.download_file(bucket_name, key, tmp_file)
        
        # Procesează imaginea
        with Image.open(tmp_file) as img:
            img.thumbnail((200, 200))
            buffer = io.BytesIO()
            img.save(buffer, "JPEG")
            buffer.seek(0)
        
        # Încarcă imaginea procesată înapoi în S3
        processed_key = f"processed/{key.split('/')[-1]}"
        s3.upload_fileobj(buffer, bucket_name, processed_key, ExtraArgs={"ContentType": "image/jpeg"})
        
        return {
            'statusCode': 200,
            'body': f"Imaginea procesată a fost salvată ca {processed_key}"
        }
    
    except Exception as e:
        return {
            'statusCode': 500,
            'body': f"Ceva n-a mers bine: {str(e)}"
        }

# Fișierul README.md

"""
# Serverless Image Processing Tool

This project demonstrates a serverless solution to automatically process images uploaded to an AWS S3 bucket. When an image is uploaded, it is resized to a thumbnail and saved in a separate folder within the same bucket.

## Features
- Automatically resizes uploaded images.
- Serverless architecture using AWS Lambda.
- Easy deployment and integration.

## Requirements
- AWS Account
- Python 3.8 or higher
- Libraries: `boto3`, `Pillow`

## Setup

1. Clone the repository:
   ```bash
   git clone https://github.com/username/serverless-image-processing.git
   cd serverless-image-processing
   ```

2. Install dependencies:
   ```bash
   pip install -r requirements.txt
   ```

3. Deploy the Lambda function:
   - Upload `lambda_function.py` to an AWS Lambda function.
   - Attach an S3 bucket trigger for image uploads.

4. Test the application:
   - Upload an image to the S3 bucket.
   - Check the `processed/` folder in the same bucket for the resized image.

## Demo
### Input
![Input Image](assets/input.jpg)

### Output
![Processed Image](assets/output.jpg)

## Contributing
Feel free to fork this repository and add more features!
"""

# requirements.txt

boto3
Pillow

