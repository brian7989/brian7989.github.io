---
layout: post
title:  "DialogFlow API V2"
date:   "2021-06-20 19:56:30"
categories: NodeJS
tags: featured
image: /assets/article_images/post_cover.jpeg
---

## NodeJS & DialogFlow API V2
<br>
> Creating a NodeJS backend that 
> interacts with the DialogFlow API (V2)


### Steps

1. Create an DialogFlow agent
2. Create a test intent
3. Create a Service Account
4. Create JSON key
5. Install necessary packages
6. Set environment variables
7. Create a web server (Express)
8. Initialize SessionsClient
9. Execute the query
10. Send back JSON response to client

### Step 1

Login to DialogFlow, and create an agent

### Step 2

Create a test intent.

<img src="/assets/article_images/DialogFlow/Step%202.png">

### Step 3

Goto agent settings, and under the General tab, click on ProjectID.<br>
In the API dashboard, click on Credentials

<img src="/assets/article_images/DialogFlow/Step%203.1.png">
<img src="/assets/article_images/DialogFlow/Step%203.2.png">

### Step 4

Under create credentials, choose Service Account.<br>
For step 1, pick a service account name of your choice.<br>
**For step 2**, grant the DialogFlow API Client permissions for the account.<br>
Don't modify anything for step 3, then click Complete.<br>

<img src="/assets/article_images/DialogFlow/Step%204.1.png">
<img src="/assets/article_images/DialogFlow/Step%204.2.png">
<img src="/assets/article_images/DialogFlow/Step%204.3.png">

Click on the service account that you've just created, and under Keys, 
generate a new key in JSON format.<br>
Remember where you downloaded this JSON key.

<img src="/assets/article_images/DialogFlow/Step%204.4.png">
<img src="/assets/article_images/DialogFlow/Step%204.5.png">

### Step 5

**Local Testing:**

First, create a new repository on GitHub, and clone it.

<img src="/assets/article_images/DialogFlow/Step%205.1.png">

Then, open up your terminal in the source directory and

```
npm init
```

Install the necessary Node packages for the application.

```
npm i dotenv express @google-cloud/dialogflow
```

### Step 6

Create a .env file to set up following env variables.

| Key          | Value                       | Example                                          |
|--------------|-----------------------------|--------------------------------------------------|
| PORT         | Any port you want to use    | 3000                                             |
| CLIENT_EMAIL | 'client_email' in JSON file | "test-446@testagent2-awpr.iam.gserviceaccount.com" |
| PRIVATE_KEY  | 'private_key' in JSON file  | "-----BEGIN PRIVATE KEY-----\n..."                 |
| PROJECTID    | 'project_id' in JSON file   | "testagent2-awpr"                                  |
{: .tablelines}

<br>

<img src="/assets/article_images/DialogFlow/Step%205.2.png">

Import modules and set up the environment in index.js

{% highlight javascript %}
// IMPORTS
require('dotenv').config()                                  // Only for local testing
const dialogflow = require('@google-cloud/dialogflow');     // Imports the Dialogflow library
const express = require('express')                          // Imports the NodeJS web applicaion framework 

// ENVIRONMENT SETUP
const projectId = process.env.PROJECTID;                    // ProjectID
const sessionId = Math.random().toString(36).substring(7);  // Random sessionId for client
const languageCode = 'en';                                  // Language code in English
const port = process.env.PORT                               // Port
const options = {                                           // Credentials
    credentials: {
        client_email: process.env.CLIENT_EMAIL,
        private_key: process.env.PRIVATE_KEY
    }
};
{% endhighlight %}

### Step 7

Initialize Express App, <br>
listen on specified port, <br>
 and send a response when request is received. 

{% highlight javascript %}
// Initialize Express App
const expressApp = express()

// Listen on specified port
expressApp.listen(port, () => {
    console.log(`Listening on http://localhost:${port}`)
})

// Send response on base URL
expressApp.get('/', (req, res) => {
    // Request parameter q  -->  .../?q
    const query = req.query.q

    /* 'query' is the question we ask to our chatbot
    The query will be received as a request parameter from our clients
    Alternatively, we can just set our query to something like
    const query = "What's your favorite song?", or 
    const query = "Hello" for testing */

    // We'll make sendAPIquery method in Step 9
    sendAPIquery(projectId, sessionId, query, languageCode).then(function(resp) {
        console.log("Successful response: ", resp);
        const response = {
            resp: resp,
        }
        // Send off the DialogFlow API response back to our client
        res.json(response)
    })
})
{% endhighlight %}

### Step 8

Initialize DialogFlow SessionsClient.<br>
(Note that each session will last upto 20 minutes)

{% highlight javascript %}
// Create session client
const sessionClient = new dialogflow.SessionsClient(options);

// Function that gets the intent of query
async function getIntent(
    projectId,
    sessionId,
    query,
    contexts,
    languageCode
) {
    // The path to identify the agent that owns the created intent
    const sessionPath = sessionClient.projectAgentSessionPath(
        projectId,
        sessionId
    );

    // The text query request
    const request = {
        session: sessionPath,
        queryInput: {
            text: {
                text: query,
                languageCode: languageCode,
            },
        },
    };

    if (contexts && contexts.length > 0) {
        request.queryParams = {
            contexts: contexts,
        };
    }

    const responses = await sessionClient.detectIntent(request);
    return responses[0];
}
{% endhighlight %}

### Step 9

Send off request to DialogFlow API.

{% highlight javascript %}

// Sends off query to the API
async function sendAPIquery(projectId, sessionId, query, languageCode) {
    // Context to continue conversation with chatbot
    let context;
    let intent;
    try {
        console.log(`Sending query: ${query}`);
        // First, get intent
        intent = await getIntent(
            projectId,
            sessionId,
            query,
            context,
            languageCode
        );
        console.log('Intent Detected');
        // Use the context from this response for next queries, if required
        context = intent.queryResult.outputContexts;

        // Fulfillment text from our DialogFlow API
        let response = `${intent.queryResult.fulfillmentText}`
        console.log("Fulfillment Text:", response);
        return response
    } catch (error) {
        console.log(error);
    }
}
{% endhighlight %}

### Step 10

Set up the "start" script in package.json.

```
    "start": "node index.js",
```

In the terminal, type in "npm run start"<br>

Then, open up http://localhost:{YOUR PORT} in browser.<br>
You'll have successfully received a JSON response.<br>

{% highlight javascript %}
{
  "resp": "Hello! How can I help you?"
}
{% endhighlight %}

### Next Steps

- Deploy your server (Heroku, Azure, etc...)
- Create a client-side application that sends query data to your server

### Sample

<br>
{% include dfTestComponent.html %} <br>

### Links

[Github Repo](https://github.com/brian7989/dfTest) <br>