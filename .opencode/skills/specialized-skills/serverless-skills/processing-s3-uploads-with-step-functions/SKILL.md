---
name: processing-s3-uploads-with-step-functions
description: >
  Deploy an event-driven workflow that routes S3 uploads to either Lambda or Fargate
  via Step Functions based on file size. Uses EventBridge to trigger a Step Functions
  state machine when objects are uploaded to S3. Small files are processed by Lambda,
  large files by a Fargate task. Includes VPC, ECR repository, ECS cluster, and scoped
  IAM roles. Trigger keywords: Step Functions, Fargate, Lambda, S3 event, EventBridge,
  ECS, ECR, file processing, workflow orchestration, serverless.
---