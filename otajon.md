# Lab: Hosting a Static Website on Amazon S3

## Overview
In this lab, you will create a static website using Amazon S3's static website hosting feature. You'll learn how to:
- Create and configure an S3 bucket for web hosting
- Generate HTML content using bash
- Upload content using the AWS CLI
- Configure public access to your website

## Prerequisites
- AWS CLI v2 installed and configured with appropriate credentials
- Basic understanding of bash commands
- Text editor of your choice

## Time to Complete
Approximately 15-20 minutes

## Steps

### 1. Create the HTML Content

First, let's create a simple HTML file using bash's echo command:

```bash
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
```

### 2. Create an S3 Bucket

Choose a globally unique bucket name and create it:

```bash
BUCKET_NAME="your-unique-bucket-name"
aws s3api create-bucket \
    --bucket "$BUCKET_NAME" \
    --region us-east-1
```

### 3. Enable Static Website Hosting

Configure the bucket for static website hosting:

```bash
aws s3api put-bucket-website \
    --bucket "$BUCKET_NAME" \
    --website-configuration '{
        "IndexDocument": {
            "Suffix": "index.html"
        },
        "ErrorDocument": {
            "Key": "index.html"
        }
    }'
```

### 4. Configure Public Access

First, disable the S3 Block Public Access settings for your bucket:

```bash
aws s3api put-public-access-block \
    --bucket "$BUCKET_NAME" \
    --public-access-block-configuration "BlockPublicAcls=false,IgnorePublicAcls=false,BlockPublicPolicy=false,RestrictPublicBuckets=false"
```

Then create and apply a bucket policy to allow public read access:

```bash
cat << EOF > bucket-policy.json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "PublicReadGetObject",
            "Effect": "Allow",
            "Principal": "*",
            "Action": "s3:GetObject",
            "Resource": "arn:aws:s3:::${BUCKET_NAME}/*"
        }
    ]
}
EOF

aws s3api put-bucket-policy \
    --bucket "$BUCKET_NAME" \
    --policy file://bucket-policy.json
```

### 5. Upload the Website Content

Upload the HTML file to your bucket:

```bash
aws s3 cp index.html "s3://${BUCKET_NAME}/index.html" \
    --content-type "text/html"
```

### 6. Access Your Website

Your website will be available at:
```
echo http://${BUCKET_NAME}.s3-website-us-east-1.amazonaws.com
```

Replace `${BUCKET_NAME}` with your actual bucket name.

## Cleanup

To delete your lab resources:

```bash
# Remove all objects from the bucket
aws s3 rm s3://${BUCKET_NAME} --recursive

# Delete the bucket
aws s3api delete-bucket --bucket ${BUCKET_NAME}

# Remove temporary files
rm index.html bucket-policy.json
```

## Troubleshooting

### Common Issues:

1. **Bucket Creation Fails**
   - Ensure the bucket name is globally unique
   - Verify you have permissions to create buckets
   - Check if the bucket name follows S3 naming rules

2. **Website Not Accessible**
   - Verify the bucket policy is correctly applied
   - Ensure the index.html file was uploaded successfully
   - Check that static website hosting is enabled

3. **Permission Errors**
   - Verify your AWS CLI credentials are configured correctly
   - Ensure you have the necessary IAM permissions
   - Check if bucket policies are properly configured
   - Verify S3 Block Public Access settings are disabled for the bucket
   - If you get "BlockPublicPolicy" errors, ensure you've run the put-public-access-block command

## Extended Learning

Try these additional challenges:
1. Add CSS styling to create a more sophisticated design
2. Include images in your static website
3. Create a multi-page website with navigation
4. Set up a custom error page
5. Enable CloudFront distribution for your website

## References
- [AWS S3 Documentation](https://docs.aws.amazon.com/AmazonS3/latest/userguide/WebsiteHosting.html)
- [AWS CLI S3 Commands](https://docs.aws.amazon.com/cli/latest/reference/s3/index.html)
- [S3 Bucket Naming Rules](https://docs.aws.amazon.com/AmazonS3/latest/userguide/bucketnamingrules.html)
