---
title: Authenticating to TaskCluster
layout:             default
class:              markdown
sequence_diagrams:  true
interactive:        true
followup:
  links:
    create-task-via-api: Create a task with createTask
---

TaskCluster uses its own kind of "credentials" to
[authenticate API requests](/manual/apis). These credentials can come from
a variety of sources, but in this section we will use
[temporary credentials](/manual/apis/temporary-credentials) issued by the TaskCluster
login service.

If you haven't already, try the "[Hello World](hello-world)" tutorial to make
sure that you have a username and password recognized by the login service.

In this tutorial we'll show a static web application can obtain temporary
TaskCluster credentials from the login service, just like the tools site does.
In the process of doing so, we'll also store credentials in `localStorage` for
use in the other tutorials.

The authentication flow involves redirecting to `login.taskcluster.net`, where
the user agrees to login and authenticates against some external
_source of authority_ (Persona, LDAP, GitHub). Once authenticated,
`login.taskcluster.net` will show a "Grant Access" button, which redirects the
user back to the web-application with temporary credentials in the URL's query string.
The flow may be illustrated as follows.

<div class="sequence-diagram-hand">
participant WebApp
participant login.taskcluster.net
participant Source of Authority

WebApp -> login.taskcluster.net : Click Login
login.taskcluster.net -> Source of Authority : Login
Note over Source of Authority : User Authenticates
Source of Authority --> login.taskcluster.net : Logged In
login.taskcluster.net -> WebApp : "Grant Access"
</div>

---

## Redirect User to TaskCluster Login

The first step implemented in the web-application is to redirect to
`login.taskcluster.net`, when doing so we **must** include `target` and
`description` in our query string. `login.taskcluster.net` will offer a
"Grant Access" button only when these query string parameters are defined.

<pre data-plugin="interactive-example">
let querystring = require('querystring');

// Redirect
window.location = "https://login.taskcluster.net?" + querystring.stringify({
  // Target to redirect back to, in this case we want to back to the tutorial
  target:       window.location.href,
  // Description to explain to the user why we want his credentials
  description:  "Tutorial needs credentials to do things, we're not evil :)"
});
</pre>

When you click "Run Code" on the example above, you'll be redirected to
`login.taskcluster.net`, there you must authenticate and click the
"Grant Access" button to be redirect back to this tutorial for the next step.

## Reading Temporary Credentials

If you arrived back on this page by clicking the "Grant Access" button on
`login.taskcluster.net`, you should now have a set of temporary credentials
in the query string for this page. We shall now parse them and store them
`localStorage`, so they can be loaded in later tutorials.

<pre data-plugin="interactive-example">
let querystring = require('querystring');

// Parse query string
let credentials = querystring.parse(window.location.search.substr(1));

// If we have credentials, we store them
if (credentials.clientId && credentials.accessToken) {
  // Store credentials in localStorage for later tutorials
  localStorage.credentials = JSON.stringify({
    clientId:     credentials.clientId,
    accessToken:  credentials.accessToken,
    certificate:  credentials.certificate
  });
  console.log("TaskCluster credentials loaded from query string:");
  console.log(credentials);
  console.log("You may now continue to the next step...");
} else {
  console.log("Missing credentials in query string!");
  console.log("Please, start from top of this page...");
}
</pre>

If you successfully managed to store temporary credentials in `localStorage`
using the example code above, you should now be ready to run all of the other
tutorials that require credentials. Please, be aware that eventually your
credentials will expire and you'll have to come back here and authenticate
again. Anyways, for now you should be able to execute an operation that
requires authentication, as in the example below.

<pre data-plugin="interactive-example">
let taskcluster = require('taskcluster-client');

// Create queue client object with temporary credentials
let queue = new taskcluster.Queue({
  credentials: JSON.parse(localStorage.credentials)
});

// Query for number of pending tutorial tasks
let result = await queue.pendingTasks('aws-provisioner-v1', 'tutorial');

// Print result
console.log(result);
</pre>

Don't worry if the number of pending tasks is zero; that is the case most of
the time, as we aim to process tasks as soon as they arrive.
