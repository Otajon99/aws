Overview
In this lab, you will create a static website using Amazon S3's static website hosting feature. You'll learn how to:

Create and configure an S3 bucket for web hosting
Generate HTML content using bash
Upload content using the AWS CLI
Configure public access to your website
Prerequisites
AWS CLI v2 installed and configured with appropriate credentials
Basic understanding of bash commands
Text editor of your choice
Time to Complete
Approximately 15-20 minutes

Steps
1. Create the HTML Content
First, let's create a simple HTML file using bash's echo command:

cat << EOF > index.html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>My Cloud Lab</title>
    <style>
        body {
            font-family: Arial, sans-serif;
            margin: 40px auto;
            max-width: 650px;
            line-height: 1.6;
            padding: 0 10px;
            background-color: #f0f0f0;
        }
        .container {
            background-color: white;
            padding: 20px;
            border-radius: 8px;
            box-shadow: 0 2px 4px rgba(0,0,0,0.1);
        }
    </style>
</head>
<body>
    <div class="container">
        <h1>Welcome to My Cloud Lab!</h1>
        <p>This is a static website hosted on Amazon S3.</p>
        <p>Created on: $(date)</p>
    </div>
</body>
</html>
EOF
2. Create an S3 Bucket
Choose a globally unique bucket name and create it:

BUCKET_NAME="your-unique-bucket-name"
aws s3api create-bucket \
    --bucket "$BUCKET_NAME" \
    --region us-east-1
