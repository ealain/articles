---
published: true
title: 'Deliver perfect HTTP security headers with AWS CloudFront'
cover_image: 'https://dev-to-uploads.s3.amazonaws.com/uploads/articles/q5f8qmpiczu9jzvx6mb2.png'
description: 'HTTP security headers protects your users from common attacks. AWS CloudFront makes it easy to add HTTP security headers to your application.'
tags: cloudfront, security, AWS, audit
canonical_url:
---

– Did you just say security headers?

– Yes.

– Are you talking about CORS headers?

– No.

## TL;DR

AWS CloudFront enables you to easily make your users browse safer with managed security headers. Quickly check your compliance with [sls-mentor](https://www.sls-mentor.dev/).

## HTTP security headers: a very simple addition to your website to efficiently prevent attacks

In the last 5 years, I have worked in a dozen teams to develop and run web applications. However, I have learnt about **HTTP security headers** very late in my full stack developer journey. This is only very recently, when I joined [Theodo](https://www.theodo.fr/), that I was explained what HTTP security headers actually are. 😮

HTTP security headers are a mechanism to **protect your application from common attacks**. They are, as their name suggests, a set of HTTP headers with security purposes. They are set by the server and read by the browser, which then applies the **security policy** described by the header.

HTTP security headers are good *deployment* practices. Thus, they fall under the responsibility of ops teams, rather than dev teams. But more and more, dev teams are also responsible for the deployment of their applications.

The table below shows a subset of HTTP security headers. The description mentions the common attacks that the header protects against. The recommended value column contains optimal header values to protect your users against these threats.

| Name | Recommended value | Description |
| --- | --- | --- |
| [Strict-Transport-Security](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Strict-Transport-Security) | `max-age=31536000` | Informs browsers that the site should only be accessed using HTTPS, and that any future attempts to access it using HTTP should automatically be converted to HTTPS. The user's inputs will never be readable by a man on the middle (except potentially the first call ever, which does not contain any sensitive information). |
| [X-Content-Type-Options](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/X-Content-Type-Options) | `nosniff` | Prevents older browsers from guessing the content of files sent by the server rather than trusting the Content-Type header. This prevents the detection and execution of malicious Javascript in an image uploaded by an attacker and then retrieved by a victim. |
| [X-Frame-Options](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/X-Frame-Options) | `SAMEORIGIN` | Prevents the display of the site in an iframe, and thus protects against click-jacking. |
| [X-XSS-Protection](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/X-XSS-Protection) | `1; mode=block` | Prevents rendering of the page if an XSS attack is detected. |
| [Referrer-Policy](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Referrer-Policy) | `strict-origin-when-cross-origin` | Prevents sending sensitive information (origin, path and query string) to another website or when the protocol is downgraded (HTTPS → HTTP) |

An [extensive list](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers#security) can be found on the Mozilla Developer Network (MDN).

These headers do not impact the application itself. Thus adding these can be done at a very low cost.

### Side note on CORS headers

Let’s digress a little bit to mention the Cross Origin Resource Sharing (CORS) headers. As a web application developer, you certainly already have come across them. Otherwise, believe me, you will encounter them sooner or later. 😉

HTTP security headers differ from CORS headers because:
- they strengthen security features of the browser (instead of weakening the same origin policy)
- they integrate very well in your development journey 😂

## Demystifying HTTP security headers on AWS

AWS mentions [several methods](https://aws.amazon.com/websites/) to deploy your website. Let's focus here on S3 and CloudFront and see how to configure security headers.

### S3

Amazon Simple Storage Service (S3) offers the option to host static websites. However, with this method, configuring HTTPS and all the more so HTTP security headers is *not possible*. 😮

### CloudFront

On the contrary, **HTTP security headers can be very easily enabled on CloudFront**. If this is your first time dealing with CloudFront:

- don’t worry, this is actually pretty easy and cheap
- brace yourself, CloudFront’s performance is mind blowing

Let’s dive in!

The CloudFront service works by creating _distributions_. These describe how your content will be delivered to your users.

An option that CloudFront offers is managing headers policies for you. The service defines subsets of headers with relevant values (like the one above) that you can easily add to request and responses in the flow.

Head over to the CloudFront service in the console.

Create or modify the distribution of your choice. Here, let’s assume the origin is S3 (alternatively this could be your own HTTP server).

![edit-behavior](./assets/edit-behavior.png 'Edit the behavior of your distribution')

Now, you can look for the *Response headers policy* section and choose `SecurityHeadersPolicy`.

![enable-security-headers](./assets/enable-security-headers.png 'Enable security headers on your distribution')

That’s it! See, this was quite straightforward.

## Further thoughts

As I dived in this topic, I added a **security check** to the **audit tool** that I use in my team: [sls-mentor](https://www.sls-mentor.dev/). This is an **open source** audit tool that helps us enforce **best practices** on our projects. Don’t hesitate to try it and send feedback to the team via a [GitHub issue](https://github.com/sls-mentor/sls-mentor/pulls), the project is actively maintained. 😉

In the future, I will improve this security check to support projects that use their own managed policies on CloudFront. sls-mentor will check for the presence of the headers, and make finer recommendations on each header.

Stay tuned!
