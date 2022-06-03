---
published: false
title: 'AWS Lambda 101: Shave That Bundle Down'
cover_image:
description: ''
tags: serverless, lambda, quality, AWS
series:
canonical_url:
---

_This article is part of a series on [Guardian][guardian], an open-source, highly configurable, automated best-practice audit tool for AWS serverless architectures._

# The lighter, the better

A light bundle will positively impact your Serverless architecture for at least two reasons.

## 1. A light deployment package reduces the cold start of your Lambda function

Reducing the size of the bundle will reduce the time it takes to upload the bundle, and also significantly reduce the lambda's cold start.

![Cold start durations](./assets/bundle-size-impact-on-cold-start.png 'Cold start durations per deployment size (https://mikhail.io/serverless/coldstarts/aws/#does-package-size-matter)')

You should use appropriate tools to bundle your Lambda functions with a minimal size. For example, the Serverless Framework automatically bundles your code and uploads it to AWS. In Node.js functions, the underlying tools are [serverless-webpack][serverless-webpack] or [serverless-esbuild][serverless-esbuild], which should be preferred for its shorter configuration and better performance.

## 2. AWS enforces quotas on deployment resources

Smaller bundles will help you respect [AWS quotas][quotas].

| Resource                                          | Quota  |
| ------------------------------------------------- | ------ |
| Lambda function deployment package (zipped)       | 50 MB  |
| Lambda function deployment package (uncompressed) | 250 MB |
| Total deployment packages                         | 75 GB  |

# Guardian is your solution to easily check the size of your bundles

[Guardian][guardian] now offers a **new rule** to warn you when uploading bundles over 5 MB. This is a demanding rule because this threshold can easily be reached. Gather your team and decide what size the bundles should be limited to.

Guardian comes with **many rules** to help you make the best decisions for your Serverless project. It will help you identify where your code can be optimized to achieve better performance at a lower cost.

# Guardian how-to

Using Guardian is very easy.

Install `sls-dev-tools`

```
npm -g install sls-dev-tools
```

Check your deployment

```
sls-dev-tools --ci [-l {YOUR_PROJECT_LOCATION}] [-p {PROFILE}] [-n {YOUR_STACK_NAME}] [-r {YOUR_REGION}] [-t {START_TIME}] [-i {INTERVAL}]
```

# See also

There are plenty of tools out there to optimize the size of your bundles. For instance the [Serverless Analyze Bundle Plugin][serverless-analyze-bundle-plugin] integrates with the Serverless Framework and helps you diagnose which of your NodeJS dependency is not properly tree-shaked.

[guardian]: https://github.com/aleios-cloud/sls-dev-tools#guardian
[quotas]: https://docs.aws.amazon.com/lambda/latest/dg/gettingstarted-limits.html
[serverless-analyze-bundle-plugin]: https://github.com/adriencaccia/serverless-analyze-bundle-plugin
[serverless-webpack]: https://github.com/serverless-heaven/serverless-webpack
[serverless-esbuild]: https://github.com/floydspace/serverless-esbuild
