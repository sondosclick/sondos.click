---
layout: post
title: "AI Prompts for Engineers — tiny practical prompt recipes"
date: 2025-11-18 10:00:00 +0000
tags: [ai, prompts, productivity]
excerpt: "Short, reusable prompts to help generate infra-as-code, explainers, and debugging tips."
---

AI is a helpful intern: brilliant at ideas, terrible at context. Here are compact prompts to get useful outputs quickly.

- Generate an AWS CloudFormation snippet to create an S3 bucket with versioning and encryption:
```
"Produce a CloudFormation YAML snippet that creates an S3 bucket with versioning enabled and SSE-S3 encryption. Use logical name MyBucket, and include a BucketName parameter."
```

- Explain an error:
```
"I have this AWS CLI error message: '<PASTE ERROR>'. Explain the most probable causes and three steps to diagnose it."
```

- Create a small README for a repo:
```
"Write a 5-sentence README for a repository that stores AWS Lambda functions for image processing. Include how to run locally and how to deploy with SAM."
```

Use these as templates, tweak for your org context, and remember to verify everything the AI hands back.
