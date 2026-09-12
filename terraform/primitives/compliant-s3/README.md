## This module enforces SC-28, AU-3, AU-6, CM-6, AC-3 on a single S3 bucket.

# Compliant S3 Primitive

## What you'll build, in plain language

You'll use Terraform to create an Amazon S3 bucket (cloud storage) that is locked down to a security baseline, and a second bucket that records who accessed the first one. Then you'll capture a file that proves the baseline is in place. That proof file is the thing an auditor would accept instead of a screenshot, because it comes straight from the system and can't be faked by cropping a browser window.

The five controls you'll satisfy, translated out of NIST-speak:

| **Control** | **What it means in plain terms** | **Where it lives in your code** |
|---|---|---|
| **SC-28** | Data is encrypted while it sits in storage, so a stolen disk is useless. | `aws_s3_bucket_server_side_encryption_configuration` |
| **AC-3** | Nobody on the public internet can reach the bucket. | `aws_s3_bucket_public_access_block` (four flags, all `true`) |
| **AU-3** | There's a record of who accessed the data. | `aws_s3_bucket_logging` writing to the log bucket |
| **AU-6** | Those records are kept somewhere you can actually review. | the separate log bucket |
| **CM-6** | The resource is set to an approved configuration, and that's labeled and enforced. | required tags + versioning |

You don't need to memorize the control IDs. You need to be able to point at a line of code and say which control it enforces, because that's the muscle every later lab builds on.

## Learning objectives

- Express NIST 800-53 controls as Terraform resources, and cite each control where it's enforced.
- Capture pre- and post-deploy compliance evidence as JSON instead of screenshots.
- Stand up the repository structure that every later lab (and your capstone) will reuse.
- Build a primitive that the rest of the course verifies automatically.
