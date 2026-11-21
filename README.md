# AWS Serverless File Processing

**An event-driven, serverless file processing solution built using Amazon S3, Amazon SNS, Amazon SQS and AWS Lambda.**

## Project Overview

This project demonstrates how to build an event-driven file processing workflow using AWS serverless services.

When a file is uploaded to an Amazon S3 input bucket, an event notification triggers an automated workflow. Messages are routed through Amazon SNS and Amazon SQS to an AWS Lambda function, which processes the file and stores the output in a separate S3 bucket.

The project also includes audit logging and file validation components.

## Architecture

```text
                 User uploads file
                         |
                         v
                  S3 Ingest Bucket
                         |
                         v
                    Amazon SNS
                         |
                         v
                    Amazon SQS
                         |
                         v
                  AWS Lambda
                  File Processor
                         |
                         v
                S3 Processed Bucket
                         |
                         v
                 Processed Output
```

Additional components include an audit queue, audit logging Lambda and file validation Lambda. Their complete integration is subject to final verification.

## AWS Services Used

| AWS Service | Purpose |
|---|---|
| Amazon S3 | Stores uploaded and processed files |
| Amazon SNS | Distributes event notifications |
| Amazon SQS | Queues messages for processing |
| AWS Lambda | Executes serverless processing logic |
| Amazon CloudWatch | Stores execution logs and monitoring data |
| AWS IAM | Controls permissions and access |
| S3 Lifecycle | Manages object retention and transitions |

## File Processing Workflow

1. A user uploads a file to the S3 ingest bucket.
2. Amazon S3 generates an object-created event.
3. Amazon SNS distributes the notification.
4. Amazon SQS receives the message.
5. AWS Lambda processes the queued event.
6. The Lambda function reads and transforms the file content.
7. The processed file is saved in the S3 processed bucket.
8. CloudWatch captures execution logs.

## Processing Example

**Input:**

```text
hello aws serverless
```

**Expected processed output:**

```text
HELLO AWS SERVERLESS
```

## Project Evidence

Screenshots of AWS resource configuration, implementation and testing are available here:

[View AWS Project Screenshots](screenshots/)

## Detailed Documentation

- [Implementation Documentation](documentation/implementation.md)
- [PDF Project Documentation](documentation/aws-serverless-file-processing-documentation.pdf)
- [Project Screenshots](screenshots/)

## Security Considerations

- IAM roles are used to grant permissions to AWS services.
- S3 buckets are used for separate input and processed storage.
- Lambda functions use execution roles.
- CloudWatch logs support troubleshooting and monitoring.
- AWS credentials and secrets should never be committed to this repository.

## Project Status

The main file-processing workflow has been implemented and tested. Final verification of audit integration, validator integration, lifecycle rule scope and error-handling scenarios remains part of the project checklist.

## Skills Demonstrated

- AWS serverless architecture
- Event-driven application design
- Amazon S3 event notifications
- SNS and SQS messaging
- AWS Lambda development
- IAM permissions and roles
- CloudWatch logging
- Troubleshooting and testing
- Technical documentation using GitHub

## Author

**Jaspinder Singh**

GitHub: [jaspindersinghcheema](https://github.com/jaspindersinghcheema)
