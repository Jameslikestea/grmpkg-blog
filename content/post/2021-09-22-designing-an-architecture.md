---
title: "Designing an Architecture"
date: 2021-09-22T21:37:15+01:00
---

In the last post I talked about the need for a package manager, I still believe that this is the case.
In this post I'm going to explore a potential architecture and how using existing services this could be achieved
more quickly and reliably than running servers and databases.

## A Cloud Native Solution, For A Cloud Native Language

As I alluded to in the preface of this post, my propsal for architecture will rely heavily on using
existing cloud services. In particular I plan to use AWS Services to implement the vast majority of the service
and will open this up to be self hosted on your own AWS account too if you so desire.

For the core product of what I'm planning, I know that I will need to use the following services for access
to the service:

- AWS Cognito (for Identity Management)
- AWS Lambda (for business logic)
- AWS S3 (for storing the hosted libraries, and generated pages)
- AWS SQS (for messaging and notification)

Aside from these products for the core product, it's likely that I'll probably also employ:

- AWS CloudFront (for distributing the libraries globally)
- AWS API Gateway (for a REST API backed by lambda, I don't want to leave S3 buckets open on the internet)
- AWS SES/SNS (for notifying when errors occur, such as conflicts)

### From 30,000ft

Often times, it is advantageous to have a visual representation of what an architecture might look like
in practice. It aids with thinking not only about the services, but also how they hang together. The below
diagram shows a rough outline of how I think that the sevrices will work together to provide this package repository

![Diagram showing connections between AWS Services](/2021-09-22-architecture-diagram.png)

At the moment the architecture looks very simple however over time I expect this will grow out
this is just the very basic requirements to serve pacakges to users from the service.

## What about the client?!?

I've talked, as I often do, primarily about the backend code and architecture, however this project
not only has a backend but also a client application that will need to be written. As part of this project
I will need to define a couple of things:

- The protocols
  - Authentication
  - Upload
  - User Information
- Standards
- Timeframes
  - Upload timeout
  - When to "archive" to S3 Glacier

All of this will feed into the client development, but initially it will have one **very** important job

**Packaging**

Without this function the rest of the service might as well not exist. This element is also the one that
is not a decision for me to make about standards. There are already standards that exist for this, fortunately
the Go maintainers have kindly produced some files for interacting with modules, including turning a directory
into a zip, so I imagine that I'll use that code for the packaging element.

Keep an eye out for the next post, I'm going to start working on the architecture and writing some of the wiring
lambdas soon.

J