---
published: true
title: 'Guardian 1.0.0 available now! Your Free Open Source audit tool for AWS architectures!'
cover_image: 'https://dev-to-uploads.s3.amazonaws.com/uploads/articles/slzm4vusv17fgkvinoyo.png'
description: 'Announcing the first stable version of Guardian, a free open source audit tool for AWS architectures.'
tags: lambda, quality, AWS, audit
series: guardian
canonical_url:
---

### Hi there! **Guardian 1.0.0** is now available! 🚀

# TL;DR

[Guardian](https://github.com/Kumo-by-Theodo/guardian) is a command line tool that produces a detailed **report** of the **improvements** you can make to your **AWS architecture**.

### As a developer, I use Guardian to ensure that my project meets the highest quality standards

Guardian is the result of a two-year journey learning and improving AWS architectures.

- Guardian covers a **wide range of concerns**: green IT, costs, security and more [below](#some-issues-are-more-important-to-you-than-others)),
- Guardian has a **thorough documentation** of its checks to help your team understand the underlying issues,
- Guardian **suggests detailed actions** to solve issues,
- Guardian provides a **clear API** for the community to maintain and add new checks,
- Guardian is **framework agnostic**.

### The one liner of your dreams

```bash
npx @kumo-by-theodo/guardian
```

![npx @kumo-by-theodo/guardian](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/l3i0inddqzv6z3z04j5o.gif 'Simple Guardian run')

### How does Guardian work?

Under the hood, Guardian uses the AWS SDK to check your provisioned resources against a set of rules. As a result, Guardian is **compatible with all frameworks** (Serverless, CDK, Terraform, CloudFormation, ...).

# Guardian is designed for you

### I know... You know the installation process

Of course you can install Guardian globally to use it across all your projects.

```bash
npm install -g @kumo-by-theodo/guardian
```

Otherwise, you can add Guardian to your project's development dependencies.

```bash
npm install --save-dev @kumo-by-theodo/guardian
```

To help you quickly understand the improvements you can bring your infrastructure, a bit at a time, Guardian comes with tons of **options** to reduce the scope of the analysis in a run.

### Focus the analysis on a specific CloudFormation stack

Use the option `-c` to focus the analysis on specific CloudFormation stacks.

```bash
npx @kumo-by-theodo/guardian -c my-stack
```

### Focus the analysis on specific tags

Use the option `-t` to focus the analysis on specific tags.

```bash
npx @kumo-by-theodo/guardian -t Key=my-tag,Value=my-value
```

> Because AWS has quite [restrictive rate limiting policy](https://docs.aws.amazon.com/lambda/latest/dg/gettingstarted-limits.html#api-requests), it may be required to use at least one option if you have really big stacks.

Guardian is **optimized** to reduce its number of requests. For projects with a lot of resources, it will **throttle** requests to respect AWS quotas. This might cause a longer analysis time.

## Guardian is best run in your CI

I recommend adding Guardian checks to a recurring job in your CI platform to get a periodic report of the improvements you can bring to your infrastructure. Depending on how often your infrastructure changes, you can run Guardian once a day, once a week or once a month.

By default, Guardian will exit with a non-zero code if it finds any issue. I recommend keeping this default setting and optionally use your CI provider option to allow this failing job. This way, you will be notified of any issue, and you will not block your deployment flow. Otherwise, you can use the option `--noFail` to always exit with a success code.

### CI snippets

#### GitLab CI

This snippet will run Guardian after merge.

```yaml
guardian:
  image: node:16.17.0
  stage:
    - post-deploy-test # the stage after your deployment
  extends:
    - .setup-cli # setup AWS CLI (export environment variables)
  needs:
    - install # the job that installs your dependencies
  script:
    - yarn guardian -r eu-west-1 -c <stacks> # fill in the stacks you want to check
  allow_failure: true # allow the job to fail, when you want to run subsequent tests
```

#### Circle CI

This snippet will run Guardian at 01:00 on Sundays.

```yaml
jobs:
  guardian-checks:
    docker:
      - image: cimg/node:16.17.0
    steps:
      - checkout # checkout your code
      - setup-aws-cli # setup AWS CLI (export environment variables)
      - install # install your dependencies
      - run: yarn guardian -p my-aws-profile -c my-stack-1 my-stack-2
workflows:
  weekly-guardian-checks:
    triggers:
      - schedule:
          cron: '0 1 * * 0' # run at 01:00 on Sundays
          filters:
            branches:
              only:
                - main
    jobs:
      - guardian-checks
```

> ⚠ You should be running at least Node v16 to get the appropriate return code.

## Some issues are more important to you than others

That's why Guardian organizes the checks in 5 categories.

🌱 **Green IT**: this one is my favorite, it's about reducing your carbon footprint

💲 **Costs**: this one helps you reduce your AWS bill,

🔒 **Security**: you know, the thing that is important to you,

⚡ **Speed**: you know, the thing that is important to your users,

🧘 **Stability**: that's for your team.

To illustrate, let's consider the following two checks.

#### 1. ARM – 🌱 + 💲 + ⚡

The default architecture for **Lambdas functions** is x86_64 but **should be configure to arm64**. Lambda functions that use arm64 architecture (AWS **Graviton2** CPU) can achieve [34% better price performance](https://aws.amazon.com/blogs/compute/migrating-aws-lambda-functions-to-arm-based-aws-graviton2-processors/) than the equivalent function running on x86_64 architecture.

[Full article on Lambda function architectures](https://dev.to/kumo/that-one-aws-lambda-hidden-configuration-that-will-make-you-a-hero-guardian-is-watching-over-you-5gi7)

#### 2. S3 Intelligent Tiering – 💲

Guardian not only has checks for Lambda, but other AWS services as well. For example, the S3 Intelligent Tiering storage class is designed to optimize storage costs by automatically moving data to the most cost-effective access tier. For a small monthly charge, S3 Intelligent Tiering monitors access patterns and automatically moves objects that have not been accessed for a long time to lower-cost access tiers.

### As I mentioned in the introduction, new checks are frequently added

We are currently working on a check to avoid infinite log retention in CloudWatch.

Starting from version 1.0.0, whenever a new rule is added, at least the minor version will be bumped.

> ⚠ If Guardian is configured to stop your deployment whenever a rule fails, do not to allow any minor version update (see [semver](https://docs.npmjs.com/cli/v6/using-npm/semver)), as the introduction of a new rule could suddenly stop your deployment.

# Guardian: AWS Trusted Advisor's little brother?

On your path to a perfect AWS Cloud architecture, Guardian is a good first step before the advanced [AWS Trusted Advisor](https://aws.amazon.com/premiumsupport/technology/trusted-advisor/).

Guardian's background is a different from AWS Trusted Advisor because rules are sourced from the community. Because Guardian uses the AWS SDK, power users can also add their own rules with maximum flexibility.

| Solution | Cost | Source |
| --- | --- | --- |
| Guardian | Free | Community |
| Trusted Advisor | Start from $29 / month ([depends on your plan and monthly AWS charges](https://aws.amazon.com/premiumsupport/pricing)) | AWS |

In the end, both tools may be used together. ✨

# About Kumo

This project was started at [Kumo](https://www.theodo.com/experts/serverless), a group of passionate developers highly experienced in Serverless architectures for the Web, to bring the highest quality solutions to our clients. Guardian has many valuable learnings of our team to help _you_ learn faster.

Feel free to reach out to us on [Twitter](https://twitter.com/kumoserverless) if you have any questions or feedback.

# See also

- [AWS Lambda 101: Shave That Bundle Down](https://dev.to/kumo/aws-lambda-101-shave-that-bundle-down-48c7)
- [AWS Lambda Versions: Time to clean up! - Guardian is watching over you](https://dev.to/kumo/aws-lambda-versions-time-to-clean-up-guardian-is-watching-over-you-jkd)
- [Kumo's blog](https://dev.to/kumo)
- [Theodo's website](https://www.theodo.com/)
