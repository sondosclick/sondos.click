---
layout: post
title: "AWS in Two Clicks: An embarrassingly simple EC2 launch"
date: 2025-11-19 12:15:00 +0000
tags: [aws, ec2, tutorial]
excerpt: "A minimal example to launch an EC2 instance with the AWS CLI — the two-click spirit applied to infrastructure."
---

Sometimes you don't need Terraform, CloudFormation, or an orchestration framework. You just need a VM and a strong coffee.

Quick steps (AWS CLI):

1. Create a keypair:
```
aws ec2 create-key-pair --key-name my-key --query 'KeyMaterial' --output text > my-key.pem
chmod 600 my-key.pem
```

2. Find latest Amazon Linux 2 AMI:
```
AMI=$(aws ec2 describe-images --owners amazon \
  --filters "Name=name,Values=amzn2-ami-hvm-*-x86_64-gp2" \
  --query 'Images | sort_by(@, &CreationDate)[-1].ImageId' --output text)
```

3. Launch:
```
aws ec2 run-instances --image-id $AMI --count 1 --instance-type t3.micro \
  --key-name my-key --tag-specifications 'ResourceType=instance,Tags=[{Key=Name,Value=two-click-test}]'
```

Yes, it's primitive. Yes, sometimes primitive is production-viable. Add an AMI lookup function, or don't.
