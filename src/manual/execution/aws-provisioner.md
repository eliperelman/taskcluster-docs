---
title: AWS Provisioner
order: 1
---

The AWS provisioner provisions EC2 spot instances in response to queue length.
It is identified with `aws-provisioner-v1` as the `provisionerId` in a task
definition.  Its administrative interface is available at
[tools.taskcluster.net/aws-provisioner/](https://tools.taskcluster.net/aws-provisioner/).
This interface allows monitoring of current load, as well as management of
worker types.

If you are setting up a new worker type with a new AMI, it is your
responsibility to ensure that a worker starts on the AMI when the AMI boots
that this worker understand the task payload and shutdown when no more tasks
are available from the queue,
