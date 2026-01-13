# REST API with AWS CDK

A serverless REST API built with AWS CDK, featuring API Gateway, Lambda functions, and DynamoDB with API key authentication.

## Architecture

This project implements a REST API that allows users to create, retrieve, and delete blog posts from a DynamoDB table. The architecture includes:

- **API Gateway**: REST API with API key authentication
- **AWS Lambda**: Serverless functions for handling requests
- **DynamoDB**: NoSQL database for storing post data
- **AWS CDK**: Infrastructure as Code for deployment

## API Endpoints

| Endpoint | Method | Description |
|----------|--------|-------------|
| `/posts` | GET | Retrieve all posts |
| `/posts` | POST | Create a new post |
| `/posts/{id}` | GET | Retrieve a specific post |
| `/posts/{id}` | DELETE | Delete a specific post |

## Prerequisites

Before getting started, ensure you have the following installed:

- **Node.js** (v14.x or later)
- **npm** or **yarn**
- **AWS CLI** (configured with credentials)
- **AWS CDK CLI**
- **An AWS Account**

## Installation

### 1. Install AWS CLI

**macOS:**
```bash
brew install awscli
```

**Linux:**
```bash
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
unzip awscliv2.zip
sudo ./aws/install
```

**Windows:**
Download and run the AWS CLI MSI installer from [AWS CLI website](https://aws.amazon.com/cli/)

### 2. Configure AWS CLI

```bash
aws configure
```

Enter your AWS credentials:
- AWS Access Key ID
- AWS Secret Access Key
- Default region name (e.g., `us-east-1`)
- Default output format (e.g., `json`)

### 3. Install AWS CDK CLI

```bash
npm install -g aws-cdk
```

Verify installation:
```bash
cdk --version
```

### 4. Clone and Setup Project

```bash
# Clone the repository
git clone https://github.com/vipulkarke12/RestAPI.git
cd RestAPI

# Install project dependencies
npm install
```

### 5. Install Required Dependencies

```bash
# Install AWS CDK libraries
npm install @aws-cdk/aws-apigateway @aws-cdk/aws-lambda @aws-cdk/aws-dynamodb

# Install AWS SDK for Lambda functions
npm install @aws-sdk/client-dynamodb @aws-sdk/lib-dynamodb

# Install TypeScript types
npm install --save-dev @types/aws-lambda aws-lambda

# Install esbuild for Lambda bundling
npm install --save-dev esbuild
```

### 6. Bootstrap CDK (First Time Only)

If this is your first time using CDK in your AWS account and region, run:

```bash
cdk bootstrap aws://ACCOUNT-NUMBER/REGION
```

Example:
```bash
cdk bootstrap aws://123456789012/us-east-1
```

## Project Structure

```
.
├── bin/
│   └── *.ts                    # CDK app entry point
├── lib/
│   └── *-stack.ts              # CDK stack definition
├── resources/
│   ├── endpoints/
│   │   ├── post.ts             # Lambda for /posts/{id}
│   │   └── posts.ts            # Lambda for /posts
│   └── handlers/
│       └── posts/
│           ├── create.ts       # Create post handler
│           ├── delete.ts       # Delete post handler
│           ├── get-all.ts      # Get all posts handler
│           └── get-one.ts      # Get single post handler
├── types.ts                    # TypeScript type definitions
├── cdk.json                    # CDK configuration
├── package.json
└── tsconfig.json
```

## Deployment

### 1. Synthesize CloudFormation Template

```bash
cdk synth
```

### 2. Deploy to AWS

```bash
cdk deploy
```

Accept the prompts to deploy the stack. This will create:
- DynamoDB table
- Lambda functions
- API Gateway REST API
- IAM roles and permissions
- API Key and Usage Plan

### 3. Get API Key Value

After deployment, note the **API Key ID** from the output. Retrieve the API key value:

```bash
aws apigateway get-api-key --api-key <API_KEY_ID> --include-value
```

Replace `<API_KEY_ID>` with the ID from the deployment output.

## Testing the API

### Using cURL

**Get All Posts:**
```bash
curl -X GET https://<API_URL>/posts \
  -H "x-api-key: <YOUR_API_KEY>"
```

**Create a Post:**
```bash
curl -X POST https://<API_URL>/posts \
  -H "x-api-key: <YOUR_API_KEY>" \
  -H "Content-Type: application/json" \
  -d '{
    "title": "Example Post",
    "description": "This is an example post",
    "author": "John Doe",
    "publicationDate": "2025-01-12"
  }'
```

**Get a Specific Post:**
```bash
curl -X GET https://<API_URL>/posts/<POST_ID> \
  -H "x-api-key: <YOUR_API_KEY>"
```

**Delete a Post:**
```bash
curl -X DELETE https://<API_URL>/posts/<POST_ID> \
  -H "x-api-key: <YOUR_API_KEY>"
```

### Using Postman

1. Create a new request in Postman
2. Set the request URL to your API endpoint
3. Add header: `x-api-key` with your API key value
4. For POST requests, add JSON body under the "Body" tab (select "raw" and "JSON")
5. Send the request

### Expected Responses

| Test Case | Expected Status | Expected Response |
|-----------|----------------|-------------------|
| Request without API key | 403 | Forbidden |
| GET /posts | 200 | Array of posts |
| POST /posts (valid body) | 200 | "Post created" |
| POST /posts (missing body) | 400 | "Missing body" |
| GET /posts/{id} | 200 | Post data |
| GET /posts/{invalid-id} | 404 | "Post not found" |
| DELETE /posts/{id} | 200 | "Post deleted" |

## Post Data Schema

```typescript
{
  "title": string,
  "description": string,
  "author": string,
  "publicationDate": string
}
```

## Useful CDK Commands

```bash
# Show differences between deployed stack and local code
cdk diff

# List all stacks in the app
cdk ls

# Output the CloudFormation template
cdk synth

# Deploy the stack
cdk deploy

# Destroy the stack (cleanup)
cdk destroy
```

## Cleanup

To avoid incurring AWS charges, destroy the stack when you're done:

```bash
cdk destroy
```

This will remove all resources created by the stack, including:
- API Gateway
- Lambda functions
- DynamoDB table
- IAM roles
- API keys

## Environment Variables

The Lambda functions use the following environment variables (automatically set by CDK):

- `TABLE_NAME`: DynamoDB table name

## Security Features

- **API Key Authentication**: All endpoints require a valid API key
- **IAM Permissions**: Lambda functions have least-privilege access to DynamoDB
- **CORS Configuration**: CORS is enabled for all origins (customize as needed)

## Troubleshooting

### Common Issues

**Issue: "Cannot find module 'esbuild'"**
```bash
npm install --save-dev esbuild
```

**Issue: CDK deployment fails with permissions error**
- Ensure your AWS credentials have sufficient permissions
- Check that CDK has been bootstrapped in your account/region

**Issue: API returns 403 Forbidden**
- Verify you're including the `x-api-key` header in your requests
- Confirm the API key value is correct

**Issue: Lambda function timeout**
- Check CloudWatch logs for errors
- Verify DynamoDB table name is correct

## Learn More

- [AWS CDK Documentation](https://docs.aws.amazon.com/cdk/)
- [API Gateway Documentation](https://docs.aws.amazon.com/apigateway/)
- [DynamoDB Documentation](https://docs.aws.amazon.com/dynamodb/)
- [AWS Lambda Documentation](https://docs.aws.amazon.com/lambda/)

## Contributing

Feel free to submit issues and pull requests for improvements.

## License

This project is licensed under the MIT License.

## Acknowledgments

Based on the tutorial: [How to Build a REST API With the AWS CDK](https://conermurphy.com/blog/build-rest-api-aws-cdk-api-gateway-lambda-dynamodb-api-key-authentication)
