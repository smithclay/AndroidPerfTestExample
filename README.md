
Android NetworkConnect Sample + New Relic Monitoring + AWS Device Farm
===================================

This sample demonstrates how to connect to the network and fetch raw HTML using
HttpURLConnection. AsyncTask is used to perform the fetch on a background thread.

New Relic is also installed along with the AWS Device Farm for automated performance testing.

### Pre-requisites

- Android SDK v23
- Android Build Tools v24.0.0 rc3
- Android Support Repository
- [New Relic](https://www.newrelic.com) Account
- Amazon Web Services Account

### Screenshots

<img src="screenshots/main.png" height="400" alt="Screenshot"/> 

### Getting Started

This sample uses the Gradle build system. To build this project, use the
"gradlew build" command or use "Import Project" in Android Studio.

## AWS Device Farm Setup

The AWS Command-line interface (CLI) is required.

### Create a new AWS Device Farm Project

```sh
sh $ aws devicefarm create-project --name NetworkConnect
```

### Update the Android Gradle File

Set `projectName` to the name of the Device Farm project that was creating using the CLI.

```
// Merge this with existing build.gradle
buildscript {
    repositories {
        jcenter()
        mavenCentral()
    }

    dependencies {
        classpath 'com.amazonaws:aws-devicefarm-gradle-plugin:1.0'
    }
}

apply plugin: 'devicefarm'

devicefarm {
    // The project must already exist!
    projectName "NetworkConnect"

    authentication {
        accessKey System.env.AWS_ACCESS_KEY_ID
        secretKey System.env.AWS_SECRET_ACCESS_KEY
    }

    appexplorer {}
}

```

### Use CloudTrail for Device Farm Logging (optional)

```
aws cloudtrail create-subscription --name CloudTrailTest --s3-new-bucket cloud-trail-test.clay.fail --sns-new-topic CloudTrailTopic
```

### Connect the SNS Topic to an AWS Lambda Function (optional)

Create a Node.js lambda function attached to a role with execution access (to see, `aws iam list-roles`):

```
aws lambda create-function --function-name cloudtrail-lambda --runtime nodejs --role arn:aws:iam::156280089524:role/lambda_default --handler SNSLambdaProcessor.handler --zip-file fileb://$(pwd)/SNSLambdaProcessor.zip
```

Get the ARN of the SNS topic that will receive the CloudTrail events:

```
aws sns list-topics
```

Using the SNS topic name and the lambda function ARN, create a subscription to invoke the function:

```
aws sns subscribe --topic-arn arn:aws:sns:us-west-2:156280089524:CloudTrailTopic --protocol lambda --notification-endpoint arn:aws:lambda:us-west-2:156280089524:function:cloudtrail-lambda
```

### Run 

```
./gradlew devicefarmUpload
```
