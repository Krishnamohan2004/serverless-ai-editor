# AWS AI-Powered Image Editing Web Application

A serverless web application that leverages AWS Bedrock's Titan Image Generator V2 for AI-powered image inpainting and outpainting capabilities. Built with AWS Amplify, Lambda, API Gateway, and Cognito for a fully managed, scalable solution.

![Architecture](architecture.jpg)

## Features

- **AI-Powered Image Editing**: Inpainting and outpainting using Amazon Titan Image Generator V2
- **Multiple Editing Modes**:
  - Inpainting: Fill in masked areas of images
  - Outpainting: Extend images beyond their original boundaries
  - Precise Outpainting: High-precision image extension
- **Serverless Architecture**: Fully managed AWS services for scalability and reliability
- **User Authentication**: Secure access with AWS Cognito
- **Usage Tracking**: DynamoDB logging for monitoring and analytics
- **CORS-Enabled API**: Ready for cross-origin requests

## Architecture

The application uses a serverless architecture built on AWS:

- **Frontend**: Static web app hosted on AWS Amplify
- **Authentication**: AWS Cognito for user management
- **API**: AWS API Gateway for RESTful endpoints
- **Compute**: AWS Lambda for serverless image processing
- **AI/ML**: AWS Bedrock with Titan Image Generator V2
- **Database**: DynamoDB for usage logging and analytics

## Prerequisites

- AWS Account with access to:
  - AWS Bedrock (Titan Image Generator V2 model enabled)
  - AWS Lambda
  - AWS API Gateway
  - AWS Cognito
  - AWS DynamoDB
  - AWS Amplify
- Node.js and npm (for local development)
- AWS CLI configured with appropriate credentials

## Setup Instructions

### 1. Clone the Repository

```bash
git clone <repository-url>
cd <repository-name>
```

### 2. Deploy Backend Resources

#### Create DynamoDB Table

```bash
aws dynamodb create-table \
  --table-name ImageGenerationTable \
  --attribute-definitions AttributeName=id,AttributeType=S \
  --key-schema AttributeName=id,KeyType=HASH \
  --billing-mode PAY_PER_REQUEST \
  --region us-east-1
```

#### Deploy Lambda Function

1. Package the Lambda function:
```bash
zip lambda-function.zip lambda.py
```

2. Create the Lambda function:
```bash
aws lambda create-function \
  --function-name ImageEditingFunction \
  --runtime python3.11 \
  --role <your-lambda-execution-role-arn> \
  --handler lambda.lambda_handler \
  --zip-file fileb://lambda-function.zip \
  --timeout 60 \
  --memory-size 512 \
  --environment Variables={DYNAMODB_TABLE_NAME=ImageGenerationTable} \
  --region us-east-1
```

3. Grant Bedrock permissions to the Lambda execution role:
```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "bedrock:InvokeModel"
      ],
      "Resource": "arn:aws:bedrock:us-east-1::foundation-model/amazon.titan-image-generator-v2:0"
    },
    {
      "Effect": "Allow",
      "Action": [
        "dynamodb:PutItem"
      ],
      "Resource": "arn:aws:dynamodb:us-east-1:*:table/ImageGenerationTable"
    }
  ]
}
```

#### Create API Gateway

1. Create a REST API in API Gateway
2. Create a POST method pointing to your Lambda function
3. Enable CORS with the following headers:
   - Access-Control-Allow-Headers: `access-control-allow-headers, access-control-allow-origin, Content-Type, Authorization`
   - Access-Control-Allow-Origin: `*`
   - Access-Control-Allow-Methods: `OPTIONS,POST,GET`
4. Deploy the API to a stage (e.g., `prod`)

#### Set Up Cognito

1. Create a Cognito User Pool
2. Create an App Client (disable client secret for web apps)
3. Configure the app client settings for your domain

### 3. Configure Frontend

1. Navigate to the config directory:
```bash
cd AWS-Amplify-Code/config
```

2. Copy the example config:
```bash
cp config.example.js config.js
```

3. Edit `config.js` with your AWS resource identifiers:
```javascript
window._workshopConfig = {
  cognito: {
    userPoolId: 'YOUR_USER_POOL_ID',
    userPoolClientId: 'YOUR_APP_CLIENT_ID',
    region: 'YOUR_AWS_REGION'
  },
  api: {
    invokeUrl: 'YOUR_API_GATEWAY_URL'
  }
};
```

### 4. Deploy Frontend

#### Option A: AWS Amplify Hosting

1. Connect your repository to AWS Amplify
2. Configure build settings (Amplify will auto-detect the build configuration)
3. Deploy

#### Option B: Local Development

```bash
cd AWS-Amplify-Code
# Serve the static files with any web server
python -m http.server 8000
# or
npx serve .
```

## Usage

1. **Sign Up/Sign In**: Create an account or log in using AWS Cognito authentication
2. **Upload Image**: Select a base image to edit
3. **Create Mask**: Draw or upload a mask indicating the area to edit
4. **Enter Prompt**: Describe what you want to generate or modify
5. **Select Mode**: Choose between inpainting, outpainting, or precise outpainting
6. **Generate**: Submit the request and wait for AI-generated results
7. **Download**: Save your edited images

## API Reference

### POST /generate

Generate or edit images using AI.

**Request Body:**
```json
{
  "prompt": {
    "text": "Description of desired output",
    "mode": "INPAINTING | OUTPAINTING | precise-outpaint"
  },
  "base_image": "data:image/png;base64,...",
  "mask": "data:image/png;base64,...",
  "model": "titan"
}
```

**Response:**
```json
{
  "images": ["base64_encoded_image_1", "base64_encoded_image_2"],
  "model_used": "titan",
  "request_id": "uuid",
  "generation_time_ms": 1234
}
```

## Configuration

### Lambda Environment Variables

- `DYNAMODB_TABLE_NAME`: Name of the DynamoDB table for logging (default: `ImageGenerationTable`)

### Image Generation Parameters

Configured in `lambda.py`:
- Number of images: 2
- Quality: Premium
- Resolution: 1024x1024
- CFG Scale: 8.0

## Monitoring and Logging

The application logs all requests to DynamoDB with the following metrics:
- Request ID and timestamp
- Model used and prompt
- Editing mode
- Image sizes (input and output)
- Generation time
- Success/failure status
- Error messages (if applicable)

Query logs using AWS Console or CLI:
```bash
aws dynamodb scan --table-name ImageGenerationTable
```

## Cost Considerations

- **AWS Bedrock**: Charged per image generated
- **Lambda**: Charged per invocation and execution time
- **API Gateway**: Charged per API call
- **DynamoDB**: On-demand pricing for read/write operations
- **Amplify Hosting**: Charged for build minutes and data transfer
- **Cognito**: Free tier available, then charged per MAU

## Security Best Practices

- Keep `config.js` out of version control (use `.gitignore`)
- Use environment-specific configurations
- Implement proper IAM roles with least privilege
- Enable CloudWatch logging for monitoring
- Use AWS WAF for API Gateway protection
- Implement rate limiting to prevent abuse
- Regularly rotate credentials

## Troubleshooting

### CORS Issues
- Verify API Gateway CORS configuration matches `cors.txt`
- Check Lambda function returns proper CORS headers

### Authentication Errors
- Verify Cognito User Pool and App Client IDs
- Check region configuration matches your resources

### Image Generation Failures
- Ensure Bedrock model access is enabled in your AWS account
- Check Lambda execution role has `bedrock:InvokeModel` permission
- Verify image sizes are within Bedrock limits

### Lambda Timeout
- Increase Lambda timeout if processing large images
- Current timeout: 60 seconds

## Credits

**Instructor**: Avinash Reddy Thipparthi  
**Academy**: Aviz Academy  
**YouTube**: [youtube.com/@avizway](https://youtube.com/@avizway)  
**LinkedIn**: [linkedin.com/in/avizway](https://linkedin.com/in/avizway)  
**Website**: [avizacademy.com](https://avizacademy.com)

## License

This project is created for educational purposes as part of AWS workshops by Aviz Academy.

## Support

For questions and support:
- Visit [Aviz Academy](https://avizacademy.com)
- Watch tutorials on [YouTube](https://youtube.com/@avizway)
- Connect on [LinkedIn](https://linkedin.com/in/avizway)
