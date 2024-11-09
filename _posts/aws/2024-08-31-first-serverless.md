---
layout: post
title: "はじめてのサーバーレス"
date: 2023-09-21
categories: aws lambda
---

- TOC
{:toc}

# はじめてのサーバーレス

こちらのハンズオンを実施した記録

https://github.com/harunobukameda/AWS-Amplify-AWS-Lambda-Amazon-DynamoDB-AWS-API-Gateway-Amazon-Cognito/blob/master/Scenario.pdf

Cloud9が使えなくなっているので、頑張って実施してく。。。

# DynamoDB作成

パーティションキーとソートキーを設定してデフォルト設定で作成

![]({{site.baseurl}}/images/aws/first-serverless/dynamo_init.png)

# Lambda用のpolicy作成

![]({{site.baseurl}}/images/aws/first-serverless/lambda_policy.png)

# Lambda用のRole作成

![]({{site.baseurl}}/images/aws/first-serverless/role.png)


# Lambda作成

作成したロールを付与してlambda関数作成

ソースが古くて動かなかったので修正

write

```js
console.log('Loading function');

import { DynamoDBClient } from "@aws-sdk/client-dynamodb";
import { PutCommand, DynamoDBDocumentClient } from "@aws-sdk/lib-dynamodb";

const client = new  DynamoDBClient({});
const docClient = DynamoDBDocumentClient.from(client);

export const handler = async(event, context, callback) => {
    console.log('Received event:', JSON.stringify(event, null, 2));

    const command = new PutCommand({
        // TODO: 作成したDynamoDBテーブルの名前に書き換えてください
        TableName: '20240831serverless',
        Item: {
            Artist: event.artist,
            Title: event.title
        }
    });

    
    const response = await docClient.send(command)
                                    .then((data) => {
                                        console.log('Success', data);
                                        callback(null, data);
                                    })
                                    .catch(err => {
                                        console.log('Error', err);
                                        callback(Error(err));
                                    });
};

```

read

```js
console.log('Loading function');

import { DynamoDBClient, QueryCommand } from "@aws-sdk/client-dynamodb";

const client = new DynamoDBClient({});

export const handler = async(event, context, callback) => {
    console.log('Received event:', JSON.stringify(event, null, 2));

    const command = new QueryCommand({
        // TODO: 作成したDynamoDBテーブルの名前に書き換えてください
        TableName: '20240831serverless',
        KeyConditionExpression: 'Artist = :artist',
        ExpressionAttributeValues: {
            ':artist': {S: event.artist}
        }
    });

    const response = await client.send(command)
                                .then((data) => {
                                  console.log('Success', data);
                                  callback(null, data);
                                })
                                .catch(err => {
                                  console.log('Error', err);
                                  callback(Error(err));
                                });
};
```


 DynamoDBClientとDynamoDBDocumentClientの違い
- DynamoDBDocumentClientの方がシンプルにかける
- DynamoDBDocumentClientのGetItemだとパーティションキーだけのitem取得はできないみたい