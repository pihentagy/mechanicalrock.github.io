---
layout: post
title: Putting the Serverless in GraphQL with AppSync
date: 2020-05-04
tags: AWS AppSync Cloudformation
author: Shermayne Lee
---

<center><img src="/img/aws_appsync.png" /></center>
<br/>

# Introduction

AWS AppSync and GraphQL have gained popularity in recent years. However it can be confusing and difficult to start for developers new to GraphQL concepts.

GraphQL is an open source query language that was developed to allow communication between your application and server. For more information visit [GraphQL](https://GraphQL.org/).

AWS AppSync is a managed, serverless, GraphQL implementation on AWS. It can translate GraphQL requests from client to backend, get the data from AWS Resources and pass the data to the client.

It is a service that simplifies your application development process by providing the ability to easily create a flexible, GraphQL based API. It can be the central hub that connects all your data sources securely.

One of the main benefits of AppSync is letting your application get the exact data that you need. Additionally for mobile and web applications, it provides the ability to access data locally when devices go offline and synchronize application data in real-time when devices are back online.

# Quick Start

The nice thing about using AWS services is we can create resources with a CloudFormation template. It allows you to create and provision AWS resources predictably and repeatedly.

If you are new to CloudFormation, visit [Cloudformation Getting Started](https://aws.amazon.com/cloudformation/getting-started/) to get more information. Alternatively, you can create resources via AWS Management Console, it is a good starting point if you are not familar with AWS services.

### Step 1: Create AWS AppSync API

I will build an API for a coffee ordering application that store all the orders in the database.

The following example is written using CloudFormation YAML.

```yml
Resources:
  CoffeeShopAppSyncApi:
    Type: AWS::AppSync::GraphQLApi
    Properties:
      Name: CoffeeShopAppSyncApi
      AuthenticationType: API_Key

  CoffeeShopAppSyncApiKey:
    Type: AWS::AppSync::ApiKey
    Properties:
      ApiId: !GetAtt CoffeeShopAppSyncApi.ApiId
      Description: API key for GraphQL Api
```

There are two resources in the template.

Firstly, it is creating the resource for AppSyncAPI. There are few methods available for authentication type such as

- API_KEY : For using API keys
- AWS_IAM : For using AWS Identity and Access Management (IAM) permissions
- AMAZON_COGNITO_USER_POOLS: For using an Amazon Cognito User Pool
- OPENID_CONNECT: For using OpenID Connect Provider

In this case, I am using an API key to protect the API, so I will need to create one.

### Step 2: Create the Schema

Now I have the API ready, I will need to create AppSync schema for the API.

```yml
CoffeeShopAppSyncSchema:
  Type: AWS::AppSync::GraphQLSchema
  Properties:
    ApiId: !GetAtt CoffeeShopAppSyncApi.ApiId
    Definition: |
      input orderInput {
        id: String
        coffee: String
        size: String
      }

      type Order {
        id: String
        coffee: String
        size: String
      }

      type Mutation {
      createOrder(input: orderInput): Order
      }

      type Query{
      getOrder(id: String): Order
      }

      schema {
      query: Query
      mutation: Mutation
      }
```

The schema I have created forms a contract between our client and the AWS data sources ensuring the data sent back and forth adheres to the specified types.

APIId is the AWS AppSync GraphQL API identifiers that you want to apply to this schema. I have created one in step 1.

A GraphQL schema must have a root query and mutation type. In the schema, I have added Query with getOrder field that return an order that matches the id. Additionally, I have also added Mutation with createOrder field that create an order. This expects the input argument to match the type defined in orderInput. As a result of the successful query or mutation, the result return will match the type defined in Order.

### Step 3: Create the Datasource

The type of data sources supported by AppSync are

- DynamoDB
- Lambda, HTTP
- Elasticsearch
- Relational Database

In this case, I will use DynamoDB as our data source.

```yml
CoffeeShopDynamoDB:
  Type: AWS::DynamoDB::Table
  Properties:
    TableName: "CoffeeShopTable"
    AttributeDefinitions:
      - AttributeName: "id"
        AttributeType: "S"
    KeySchema:
      - AttributeName: "id"
        KeyType: "HASH"
    ProvisionedThroughput:
      ReadCapacityUnits: 3
      WriteCapacityUnits: 1
    Projection:
      ProjectionType: "ALL"

CoffeeShopDynamoDBDataSource:
  Type: AWS::AppSync::DataSource
  Properties:
    ApiId: !GetAtt CoffeeShopAppSyncApi.ApiId
    Name: CoffeeShopDynamoDBDataSource
    Description: Contain order data from coffee shop
    Type: “AMAZON_DYNAMODB”
    ServiceRoleArn: arn:aws:iam::xxxxxxxxxxxx:role/xxxxxxxxx
    DynamoDBConfig:
      TableName: CoffeeShopTable
      AwsRegion: ap-southeast-2
```

From the template, I am provisioning new DynamoDB table for the API. It has the attribute id as the hash key and resourceId as the range key for the table.

I am connecting the AppSync data source with the newly created DynamoDB table by specifying the table name. You can connect to an existing data source if you already have one.

[Import a DynamoDB Table](https://docs.aws.amazon.com/appsync/latest/devguide/import-dynamodb.html#import-a-ddb-table)

### Step 4: Create the Resolver

Adding a resolver for a mutation, query or subscription allows us to map data from our client to our data source, in this case DynamoDB, and vice versa.

```yml
CreateOrderResolver:
  Type: AWS:: AppSync::Resolver
  Properties:
    ApiId: !GetAtt CoffeeShopAppSyncApi.ApiId
    TypeName: Mutation
    FieldName: createOrder
    DataSourceName: !GetAtt CoffeeShopDynamoDBDataSource.Name
    RequestMappingTemplate: |
      {
        "version" : "2017-02-28",
        "operation" : "PutItem",
        "key" : {
          "id": { "S" : "${ctx.args.orderInput.id}" },
          "resourceId": { "S": "COFFEE" }
        },
      }
    ResponseMappingTemplate: |
        $util.toJson($context.result)

GetOrderResolver:
  Type: AWS:: AppSync::Resolver
  Properties:
    ApiId: !GetAtt CoffeeShopAppSyncApi.ApiId
    TypeName: Query
    FieldName: getOrder
    DataSourceName: !GetAtt CoffeeShopDynamoDBDataSource.Name
    RequestMappingTemplate: |
      {
          "version": "2017-02-28",
          "operation": "GetItem",
          "key": {
            "id": { "S": "${ctx.args.id}" },
            "resourceId": { "S": "COFFEE" }
          }
        }
    ResponseMappingTemplate: |
        $util.toJson($context.result)
```

# Conclusion

We had a brief introduction on how you can provision AWS AppSync with DynamoDb as the data source by using CloudFormation. AWS has pretty good documentation, it is worth reading if you want to get more details.

[AWS APPSYNC](https://aws.amazon.com/appsync/)

Give it a try and let us know your thoughts! Tweet at us
[@mechanicalrock\_](https://twitter.com/mechanicalrock_) with your thoughts!

If you are struggling to start, we can help you! Please contact us at
[Contact Mechanical Rock to Get Started!](https://www.mechanicalrock.io/lets-get-started)
