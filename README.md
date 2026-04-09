# Bedrock Knowledge Base CloudFormation

[![Deploy to AWS CloudFormation](https://img.shields.io/badge/Deploy%20to-AWS%20CloudFormation-orange?logo=amazon-aws)](https://console.aws.amazon.com/cloudformation/home?region=eu-west-2#/stacks/create/review?templateURL=https%3A%2F%2Fsecuredev-llm-maps-data.s3.eu-west-2.amazonaws.com%2Fcloudformation.yaml)

This repository contains a CloudFormation template (`cloudformation.yaml`) that deploys an Amazon Bedrock Knowledge Base backed by Amazon OpenSearch Serverless, plus S3 and web crawler data sources.

## One-click deploy

Use the button above, or open this direct link:

`https://console.aws.amazon.com/cloudformation/home?region=eu-west-2#/stacks/create/review?templateURL=https%3A%2F%2Fsecuredev-llm-maps-data.s3.eu-west-2.amazonaws.com%2Fcloudformation.yaml`

## What this stack deploys

- `AWS::S3::Bucket` for optional document uploads
- `AWS::OpenSearchServerless::SecurityPolicy` (encryption)
- `AWS::OpenSearchServerless::SecurityPolicy` (network)
- `AWS::OpenSearchServerless::Collection` (vector search collection)
- `AWS::OpenSearchServerless::AccessPolicy` (collection/index permissions)
- `AWS::OpenSearchServerless::Index` with Bedrock-compatible vector/text/metadata mappings
- `AWS::IAM::Role` for Bedrock Knowledge Base access
- `AWS::Bedrock::KnowledgeBase` (VECTOR type)
- `AWS::Bedrock::DataSource` (S3)
- `AWS::Bedrock::DataSource` (WEB) x4 crawler groups

## Key configuration in this template

- Embedding model: `amazon.titan-embed-text-v2:0`
- Vector index name: `${Name}-kb-index`
- Vector dimension: `1024`
- Chunking strategy for all data sources: `SEMANTIC`
- Data deletion policy for data sources: `RETAIN`
- Web crawler limits (each web data source):
  - `MaxPages: 25000`
  - `RateLimit: 300` URLs/host/minute
  - exclusion filters for query-string and login/cart/checkout paths

## Parameters

- `Name` (required): base name used in created resources
- `Region` (default `eu-west-2`): must match stack deployment region
- `WebSeedUrlsGroup1` (10 seed URLs)
- `WebSeedUrlsGroup2` (10 seed URLs)
- `WebSeedUrlsGroup3` (10 seed URLs)
- `WebSeedUrlsGroup4` (currently 7 seed URLs in template defaults)

## Outputs

- `BucketName`
- `OpenSearchCollectionArn`
- `KnowledgeBaseId`
- `DataSourceId` (S3)
- `WebDataSourceGroup1Id`
- `WebDataSourceGroup2Id`
- `WebDataSourceGroup3Id`
- `WebDataSourceGroup4Id`

## Deploy with AWS CLI

[knowledge base schema (1).xlsx](https://github.com/user-attachments/files/26604994/knowledge.base.schema.1.xlsx)


```bash
aws cloudformation deploy \
  --template-file cloudformation.yaml \
  --stack-name kb-cloudformation \
  --parameter-overrides Name=<your-name> Region=eu-west-2 \
  --capabilities CAPABILITY_NAMED_IAM \
  --region eu-west-2
```

## After deployment

CloudFormation creates data sources but does not run ingestion jobs automatically. Start ingestion for each data source in Bedrock (console/API) after the stack is complete.
