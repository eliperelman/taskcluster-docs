---
title: Generic Worker
order: 2
docson:       true
---

# Generic Worker

The generic worker was our first worker that was intended for non-linux
environments. In theory it can run against any platform which supports the go
programming language, however in practice we use it currently only for
Windows™, and the generic worker will be soon superceded by the TaskCluster
Worker.

If you are looking at standing up new jobs, we recommend you take a look at the
TaskCluster Worker, which is intended to eventually replace the Generic Worker.

## Generic Worker payload definition

When submitting a task graph to the Task Cluster scheduler (see
[createTaskGraph](/reference/platform/scheduler/api-docs#createTaskGraph)) you must provide a
payload for defining the tasks to be executed by the worker. In the case of the
generic worker, the payload must conform to the following schema.

<div data-render-schema="http://schemas.taskcluster.net/generic-worker/v1/payload.json"></div>

The payload comprises of a command to run, environment variables to be set
(optionally encrypted) and a timeout for the task (`maxTimeRun`).

The worker will run the task, upload log files, and report back status to the
Queue.

## Features

Features are capabilities that can be enabled in the generic worker for use by
a task.

These features are enabled by declaring them within the task payload in the
`features` object.

Note: Some features require additional information within the task definition.
Features may also require scopes.  Consult the documentation for each feature
to understand the requirements.

Example:

```js
{
  "payload": {
    "features": {
      "generateCertificate": true
    }
  }
}
```

#### Feature: `generateCertificate`

Enabling this feature will mean that the generic worker will publish an additional task artifact `public/logs/certificate.json.gpg`. This will be a clear text openpgp-signed json object, storing the SHA 256 hashes of the task artifacts, plus some information about the worker. This is signed by a openpgp private key, both generated and stored on the worker. This private key is never transmitted across the network, and therefore if you are able to verify the signature of this artifact, you can be confident that it really was created by the worker.

Please note that the mechanism to locate the public key of the worker is specific to the worker type, and the workflow of the party that created that worker type (typically the Mozilla Release Engineering Team).

No scopes are presently required for enabling this feature.

References:

* [Bugzilla bug](https://bugzilla.mozilla.org/show_bug.cgi?id=1287112)
* [Source code](https://github.com/taskcluster/generic-worker/blob/master/chain_of_trust.go)
