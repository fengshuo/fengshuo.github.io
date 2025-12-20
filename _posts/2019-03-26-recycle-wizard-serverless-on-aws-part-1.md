---
title: Recycle Wizard + Serverless on AWS (Part 1)
categories:
- Frontend
- Web
tags:
- aws
- serveless
- API Gateway
- Lambda
- DynamoDB
- S3
- ios
- react native
- CloudSearch
- Rekognition
description: Add API Gatway, Lambda, DynamoDB, S3, Rekognition to the new version
  of Recycle Wizard
date: 2019-03-26
author_profile: true
classes: wide
---

I recently updated the iOS app [Recycle Wizard](https://itunes.apple.com/us/app/recycle-wizard/id1300177023?mt=8) I developed a while back. This time I updated the backend structure and did a little bit UI change, this article documents what I did, what I think can be improved in the future.

# Intro

The first version of the app is rather simple:

![recycleWizard-first-version-structure](/assets/img/posts/2019-03-26-recycle-wizard-serverless-on-aws-part-1/recycleWizard-first-version-structure.png)

When I worked on the first version, I knew I wanted to improve the search functionality and add some level of data tracking so that I can adjust the functionality. Now I am more familiar with AWS, I decided to apply some of the serverless tools from AWS, below is the new structure:

![recycleWizard-current-version-structure](/assets/img/posts/2019-03-26-recycle-wizard-serverless-on-aws-part-1/recycleWizard-current-version-structure.png)

There are two ways of getting a result, search by taking a photo or typing a keyword. If searching by a photo, I use AWS rekognition to detect the labels of the image, and use the labels as the keyword to search. The shared part of those two ways is the search part, which I will add more details later.

# API Gateway + Lambda + Rekognition + S3 + DynamoDB

For the image route, what I want to have is that the app code calls an API, this API call should 1) get the image label 2) upload the image to a S3 bucket 3) save the image label to a DynamoDB table. For 2 and 3, I just wanted to get a sense of if the labels are relevant enough and then decide if I should change the image detection configuration to better search result, they are lower priority and shouldn't affect getting the image labels.

Therefore I have two lambda functions, lambda function A is connected to API Gateway, and is responsible for triggering Rekognition as well as calling lambda function B, which takes care of S3 and DynamoDB.

```javascript
    const AWS = require("aws-sdk");
    const rekognition = new AWS.Rekognition();
    const lambda = new AWS.Lambda();

    exports.handler = (event, context, callback) => {
        let eventContent = JSON.parse(event.body);
        let encodedImage = eventContent.user_image;
        let decodedImage = Buffer.from(encodedImage, 'base64');
        var filePath = eventContent.file_name + ".jpg"
        let rekognitionParams = {
            Image: {
              "Bytes": decodedImage,
            },
            MaxLabels: 3,
            MinConfidence: 80
        };

        rekognition.detectLabels(rekognitionParams, function(err, data) {
            if (err) {
              console.log(err, err.stack)      
            } else {
              let labelData = data.Labels;

              let response = {
                  "statusCode": 200,
                  "body": JSON.stringify(labelData),
                  "isBase64Encoded": false
              }

              callback(null, response);

              let lambdaPayload = {
                    s3: {
                        "encodedImage": encodedImage,
                        "Bucket": "recycle-wizard-v2-images",
                        "Key": filePath  
                    },
                    dynamoDB: {
                         TableName: 'searched-images-label',
                         Item: {
                             id: new Date().getTime(),
                             filename: new Date().getTime() + '.jpg',
                             labels: labelData
                         }
                    }
                };

              let invokeLambdaParams = {
                  FunctionName: 'uploadS3andSaveDynamoTable', // the lambda function we are going to invoke
                  InvocationType: 'Event',
                  LogType: 'Tail',
                  Payload: JSON.stringify(lambdaPayload)
              };


              lambda.invoke(invokeLambdaParams, function(err, data) {
                    if (err) {
                      context.fail(err);
                    } else {
                      console.log("invokeLambda: " + data)
                    }
                })


            }
        })
    };
```

A few things I found out while implementing this route are:

- When invoking lambda function in a lambda function, the invocation type can be `RequestResponse`(synchronously) or `Event`(asynchronously), and if the invocation type is asynchronously, the API response only includes a status code, no data. In my case, uploading to S3 and DynamoDB don't block getting the image label hence I used `Event`
- When invoking lambda function from another lambda function, you have to check if the role you attached to the lambda function has the proper policy attached, in this case, on top of S3, DynamoDB and Rekognition full access, I also added `lambdaFullAccess`, `lambdaExecute`, `lambdaBasicExcutionrole`, `lamdbaRole` to make it work
- Initially I thought I need to upload the image to S3, then Rekognition can use the image in the bucket to analyze the photo, it turns out [Rekognition takes](https://docs.aws.amazon.com/rekognition/latest/dg/API_Image.html) the input image either as bytes(blob of image bytes up to 5 MB) or an S3 object, I did some quick testing and it seems that once I resize and compress the picture it is usually less than 50KB, so using image bytes should be ok. Although it would be interesting to see if there is a way to monitor the S3 upload **finish** event, maybe with SNS?
- In API Gateway, I used *User Lambda Proxy Integration* since it's the easiest to get the API running, but ideally the API gateway setup should be more comprehensive. And since I am using proxy integration in API Gateway, enabling CORS from API Gateway doesn't work, I have to [set the header](https://stackoverflow.com/questions/38987256/aws-api-gateway-cors-post-not-working) `Access-Control-Allow-Origin` in the response.
- After you set up the new api in API Gateway, don't forget to deploy it to a stage and use the correct the URL in the stage tab, otherwise you might see error message like 'missing authentication token'.
- Lambda and API Gateway both have "Test" functionality, you can test the api and check the logs in CloudWatch

Next part is about the search functionality.
