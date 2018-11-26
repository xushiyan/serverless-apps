# aws-firehose-s3-transform-with-backup

## Prerequisites

- `aws cli`
- `aws sam cli`

## Create Stacks

### Create the stack for Kinesis Firehose transformation processor

#### Create S3 bucket for Lambda deployment

```bash
aws s3 mb s3://rxu-sls
```

_Note: change to a globally unique bucket name._

#### Package Lambda function

```bash
sam package --s3-bucket rxu-sls --output-template-file packaged.yaml --force-upload
```

#### Deploy Lambda function

```bash
export PROCESSOR_STACK_NAME=aws-firehose-s3-transform-with-backup-processor

sam deploy \
  --template-file packaged.yaml \
  --stack-name $PROCESSOR_STACK_NAME \
  --capabilities CAPABILITY_IAM \
  --parameter-overrides FunctionNameParameter=aws-firehose-s3-transform-with-backup-processor-lambda
```

#### Export the Lambda function ARN

```bash
export PROCESSOR_ARN=xxx
```

### Create the stack for Kinesis Firehose delivery stream

```bash
export DELIVERY_STREAM_STACK_NAME=aws-firehose-s3-transform-with-backup

aws cloudformation create-stack \
  --stack-name $DELIVERY_STREAM_STACK_NAME \
  --template-body file://cf_kinesis_s3_stack.yaml \
  --capabilities CAPABILITY_IAM \
  --parameters ParameterKey=ProcessorArn,ParameterValue=$PROCESSOR_ARN
```

## Produce Test Data

```bash
aws firehose put-record \
  --delivery-stream-name DELIVERY_STREAM_NAME \
  --record '{"Data":"83.149.9.216 - - [17/May/2015:10:05:50 +0000] \"GET /presentations/logstash-monitorama-2013/images/kibana-dashboard.png HTTP/1.1\" 200 321631 \"http://semicomplete.com/presentations/logstash-monitorama-2013/\" \"Mozilla/5.0 (Macintosh; Intel Mac OS X 10_9_1) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/32.0.1700.77 Safari/537.36\""}'
```

## Delete Stacks

```bash
aws cloudformation delete-stack  --stack-name $PROCESSOR_STACK_NAME
aws cloudformation delete-stack  --stack-name $DELIVERY_STREAM_STACK_NAME
```
