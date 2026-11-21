# AWS Serverless File Processing System

**Project type:** Hands-on AWS cloud engineering lab  
**AWS Region:** `us-east-2` (US East – Ohio)  
**GitHub repository:** `aws-serverless-file-processing`  
**Status:** Implementation and verification in progress

## 1. Project overview

This project builds an event-driven file-processing workflow using AWS managed and serverless services. A text file is uploaded to an Amazon S3 ingestion bucket. The upload event is published to Amazon SNS, which fans the notification out to Amazon SQS queues. An AWS Lambda processor consumes processing messages, transforms the text content to uppercase, and writes the result to a separate S3 processed bucket. A separate audit/validation path is part of the project design.

The project is intended to demonstrate practical skills in object storage, event notifications, message queuing, serverless compute, IAM permissions, monitoring, troubleshooting, and lifecycle management.

## 2. Architecture

### Main processing path

```text
User uploads .txt file
        |
        v
S3 Ingest Bucket
        |
        | ObjectCreated notification
        v
SNS Topic: jsp-file-upload-topic
        |
        +------------------------------+
        |                              |
        v                              v
SQS: jsp-file-processing-queue   SQS: jsp-file-audit-queue
        |                              |
        v                              v
Lambda: jsp-file-processor        Audit Lambda (implementation/
        |                          verification status to be recorded)
        |
        v
S3 Processed Bucket
        |
        v
Validator Lambda (separate validation stage;
implementation/permissions and successful test to be verified)
```

**Important:** The diagram shows the intended overall design. Do not describe every branch as successfully tested until the corresponding Lambda trigger, permissions, and end-to-end execution have been verified.

## 3. AWS resources

| Resource | Name / configuration | Current documentation status |
|---|---|---|
| AWS Region | `us-east-2` | Used for this lab |
| S3 ingestion bucket | `jsp-serverless-ingest-2026-unique` | Created |
| S3 processed bucket | `jsp-serverless-processed-2026-unique` | Created |
| SNS topic | `jsp-file-upload-topic` | Created |
| SQS processing queue | `jsp-file-processing-queue` | Created; subscription reported confirmed |
| SQS audit queue | `jsp-file-audit-queue` | Created; verify its subscription and consumer |
| Processor Lambda | `jsp-file-processor` | Created; processing logic configured |
| Validator Lambda | `jsp-file-validator` | Work/verification remains; record final permissions and test result |
| IAM roles and policies | Processor role and related execution/S3 permissions | Processor permissions configured; verify least-privilege scope |
| CloudWatch Logs | Lambda log groups | Used to inspect execution and troubleshoot |
| S3 lifecycle rule | 30-day expiration objective | Rule configured; retain screenshot and verify rule scope/status |

Resource names are included to make the lab reproducible. Do not publish AWS account IDs, access keys, secret keys, session tokens, passwords, or other sensitive account information.

## 4. Implementation record

### 4.1 Amazon S3

- Created an ingestion bucket for original uploaded files: `jsp-serverless-ingest-2026-unique`.
- Created a separate destination bucket for processed files: `jsp-serverless-processed-2026-unique`.
- Configured an S3 `ObjectCreated` event notification as part of the event workflow.
- Configured a lifecycle rule with a 30-day expiration objective. Confirm the rule applies to the intended bucket and object scope before describing expiration as fully verified.

### 4.2 Amazon SNS

- Created the SNS topic `jsp-file-upload-topic`.
- Used the topic as the notification fan-out point for S3 upload events.
- Added SQS subscriptions for the processing and audit queues.
- The processing queue subscription was reported as confirmed.

### 4.3 Amazon SQS

- Created `jsp-file-processing-queue` for file-processing messages.
- Created `jsp-file-audit-queue` for the audit path.
- Configured queue access so the SNS topic can send messages, with a source-ARN restriction recorded for the processing queue.
- Verify the audit queue policy, subscription confirmation, and consumer trigger before marking that branch complete.

### 4.4 AWS Lambda

**Processor function:** `jsp-file-processor`

- Implemented the file-processing stage in Python.
- Intended behavior: read the uploaded text file, convert its text to uppercase, and write the result to the processed S3 bucket.
- Configured an SQS event source mapping for `jsp-file-processing-queue` (batch size 1 was recorded during setup).
- The function's SQS event structure must be handled correctly. An SQS-triggered Lambda event contains a `Records` array whose records contain an SQS message body; it is not the same event shape as a direct S3 notification.
- During testing, a `KeyError: 'Records'` was observed. Investigate the actual event received and ensure the handler reads the correct event structure before claiming reliable end-to-end processing.

**Validator function:** `jsp-file-validator`

- A separate validation stage is included in the project plan.
- Record its final trigger, IAM permissions, validation rules, and test evidence after these have been verified.
- Do not claim validation is complete based only on the function's existence.

**Audit function / path**

- An audit queue was created as part of the fan-out design.
- Record the audit Lambda name, trigger, permissions, and observed output once verified.

### 4.5 IAM and monitoring

- Configured a processor execution role with Lambda logging and SQS access permissions, plus S3 access required by the processing workflow.
- Keep permissions limited to the resources and actions required by each function.
- Used Amazon CloudWatch Logs to inspect Lambda initialization and execution messages and investigate errors.

## 5. Testing and evidence

Use the table below as a living test record. Change a status to **Passed** only after the test has been repeated and the evidence screenshot has been saved in this repository.

| Test | Expected result | Status |
|---|---|---|
| Upload a `.txt` file to the ingestion bucket | Object appears in the ingest bucket | Verify / record screenshot |
| S3 event notification | SNS receives the object-created notification | Verify |
| SNS-to-processing-SQS delivery | Message appears in processing queue | Subscription confirmed; delivery test to record |
| SQS-to-processor trigger | Lambda receives an SQS event | Verify event source mapping and invocation |
| Text transformation | Output text is uppercase | Record final repeatable test |
| Processed output | Output object is saved in processed bucket | Evidence to retain |
| CloudWatch review | Logs show successful invocation without unhandled errors | Troubleshooting in progress |
| Audit branch | Audit event is consumed and recorded | Pending verification |
| Validator branch | Validation function runs and produces expected result | Pending verification |
| Lifecycle rule | Intended objects have a 30-day expiration rule | Rule configured; scope/evidence to verify |

### Suggested screenshots

Save genuine AWS Console screenshots in a `screenshots/` folder using clear names, for example:

- `01-ingest-bucket.png`
- `02-processed-bucket.png`
- `03-sns-topic.png`
- `04-sqs-queues.png`
- `05-sns-sqs-subscriptions.png`
- `06-processor-lambda.png`
- `07-lambda-event-source-mapping.png`
- `08-cloudwatch-execution-logs.png`
- `09-processed-output-object.png`
- `10-lifecycle-rule.png`
- `11-validator-lambda.png` (after configured)
- `12-end-to-end-test.png` (after passed)

Only include screenshots that you actually captured. Redact account identifiers or other sensitive details where appropriate.

## 6. Troubleshooting notes

### `KeyError: 'Records'`

A `KeyError: 'Records'` was observed during Lambda testing. Possible causes include the function being invoked with a test payload that does not match the configured trigger, or the handler expecting the wrong event format.

Troubleshooting approach:

1. Open the Lambda function's CloudWatch log stream and inspect the full exception and stack trace.
2. Identify the event source that invoked the function.
3. For an SQS event source mapping, inspect the event's `Records` array and each record's `body`.
4. If the SQS message body contains an SNS notification envelope, parse that JSON and extract the original S3 event from the SNS `Message` field.
5. Handle malformed or unexpected messages safely and log useful error details without exposing sensitive data.
6. Re-test using a real file upload through the configured S3 → SNS → SQS path.

Do not mark the issue resolved until a real upload results in the expected processed object and the Lambda invocation completes successfully.

## 7. Security and cost considerations

- Never commit AWS access keys, secret keys, session tokens, passwords, or `.env` files.
- Use IAM roles for Lambda access instead of embedding credentials in code.
- Restrict S3 and SQS permissions to the required resources and actions.
- Review bucket public-access settings and encryption settings.
- Monitor usage and delete lab resources that are no longer needed.
- Check for billable resources such as NAT Gateways, public IPv4 addresses, and other services before leaving the lab running.
- Confirm lifecycle rules carefully; lifecycle expiration is not necessarily immediate deletion at the exact 30-day mark.

## 8. Lessons learned

This lab provides practical experience with:

- S3 object storage and event notifications.
- SNS topic subscriptions and message fan-out.
- SQS queues and Lambda event source mappings.
- Python Lambda event parsing and file transformation.
- IAM execution roles and resource-based queue policies.
- CloudWatch logging and troubleshooting.
- S3 lifecycle management.
- Designing loosely coupled, event-driven architectures.

## 9. Remaining work

- [ ] Verify the complete S3 → SNS → SQS → processor Lambda workflow.
- [ ] Resolve and retest the `KeyError: 'Records'` issue.
- [ ] Capture successful processing evidence and the processed file contents.
- [ ] Verify the audit queue subscription, permissions, and consumer.
- [ ] Complete and test the Validator Lambda permissions and trigger.
- [ ] Verify the 30-day lifecycle rule scope and save its screenshot.
- [ ] Add the final architecture diagram and screenshots to GitHub.
- [ ] Review IAM permissions and remove unused lab resources.

## 10. Project status statement

This is a personal, hands-on AWS learning project. Core infrastructure components have been created and configured, while some event delivery, Lambda integration, validation, and end-to-end tests remain to be verified. The repository should be updated as each test passes, with screenshots and concise notes as evidence.
