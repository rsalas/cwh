# AWS CloudWatch Logs Handler for Monolog

Handler for PHP logging library [Monolog](https://github.com/Seldaek/monolog) for sending log entries to 
[AWS CloudWatch Logs](http://docs.aws.amazon.com/AmazonCloudWatch/latest/logs/WhatIsCloudWatchLogs.html) service.

Before using this library, it's recommended to get acquainted with the [pricing](https://aws.amazon.com/en/cloudwatch/pricing/) for AWS CloudWatch services.

## Disclaimer
This library uses AWS API through AWS PHP SDK, which has limits on concurrent requests. It means that on high concurrent or high load applications it may not work on it's best way. Please consider using another solution such as logging to the stdout and redirecting logs with fluentd.

## Requirements
* PHP ^8.0
* AWS account with proper permissions (see list of permissions below)

## Features
* Up to 10000 batch logs sending in order to avoid _Rate exceeded_ errors 
* Log Groups creating with tags
* AWS CloudWatch Logs staff lazy loading
* Suitable for web applications and for long-living CLI daemons and workers

## Installation
Install the latest version with [Composer](https://getcomposer.org/) by running

```bash
$ composer require rsalas/cwh:^2.0
```

## Basic Usage
```php
<?php

use Aws\CloudWatchLogs\CloudWatchLogsClient;
use Rsalas\Cwh\Handler\CloudWatch;
use Monolog\Logger;
use Monolog\Formatter\JsonFormatter;

$sdkParams = [
    'region' => 'eu-west-1',
    'version' => 'latest',
    'credentials' => [
        'key' => 'your AWS key',
        'secret' => 'your AWS secret',
        'token' => 'your AWS session token', // token is optional
    ]
];

// Instantiate AWS SDK CloudWatch Logs Client
$client = new CloudWatchLogsClient($sdkParams);

// Log group name, will be created if none
$groupName = 'php-logtest';

// Log stream name, will be created if none
$streamName = 'ec2-instance-1';

// Days to keep logs, 14 by default. Set to `null` to allow indefinite retention.
$retentionDays = 30;

// Instantiate handler (tags are optional)
$handler = new CloudWatch($client, $groupName, $streamName, $retentionDays, 10000, ['my-awesome-tag' => 'tag-value']);

// Optionally set the JsonFormatter to be able to access your log messages in a structured way
$handler->setFormatter(new JsonFormatter());

// Create a log channel
$log = new Logger('name');

// Set handler
$log->pushHandler($handler);

// Add records to the log
$log->debug('Foo');
$log->warning('Bar');
$log->error('Baz');
```

## Frameworks integration
 - [Silex](http://silex.sensiolabs.org/doc/master/providers/monolog.html#customization)
 - [Symfony](http://symfony.com/doc/current/logging.html)
 - [Lumen](https://lumen.laravel.com/docs/5.2/errors)
 - [Laravel](https://laravel.com/docs/5.4/errors)
  
 [And many others](https://github.com/Seldaek/monolog#framework-integrations)
 
# AWS IAM needed permissions
if you prefer to use a separate programmatic IAM user (recommended) or want to define a policy, make sure following permissions are included:
1. `CreateLogGroup` [aws docs](https://docs.aws.amazon.com/AmazonCloudWatchLogs/latest/APIReference/API_CreateLogGroup.html)
1. `CreateLogStream` [aws docs](https://docs.aws.amazon.com/AmazonCloudWatchLogs/latest/APIReference/API_CreateLogStream.html)
1. `PutLogEvents` [aws docs](https://docs.aws.amazon.com/AmazonCloudWatchLogs/latest/APIReference/API_PutLogEvents.html)
1. `PutRetentionPolicy` [aws docs](https://docs.aws.amazon.com/AmazonCloudWatchLogs/latest/APIReference/API_PutRetentionPolicy.html)
1. `DescribeLogStreams` [aws docs](https://docs.aws.amazon.com/AmazonCloudWatchLogs/latest/APIReference/API_DescribeLogStreams.html)
1. `DescribeLogGroups` [aws docs](https://docs.aws.amazon.com/AmazonCloudWatchLogs/latest/APIReference/API_DescribeLogGroups.html)

When setting the `$createGroup` argument to `false`, permissions `DescribeLogGroups` and `CreateLogGroup` can be omitted

## AWS IAM Policy full json example
```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": [
                "logs:CreateLogGroup",
                "logs:DescribeLogGroups"
            ],
            "Resource": "*"
        },
        {
            "Effect": "Allow",
            "Action": [
                "logs:CreateLogStream",
                "logs:DescribeLogStreams",
                "logs:PutRetentionPolicy"
            ],
            "Resource": "{LOG_GROUP_ARN}"
        },
        {
            "Effect": "Allow",
            "Action": [
                "logs:PutLogEvents"
            ],
            "Resource": [
                "{LOG_STREAM_1_ARN}",
                "{LOG_STREAM_2_ARN}"
            ]
        }
    ]
}
```
