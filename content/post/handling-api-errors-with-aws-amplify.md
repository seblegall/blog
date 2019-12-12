---
title: "Handling validation errors from API-Gateway with AWS Amplify using ReactJs"
description: "How to properly manage errors coming from the API with aws-amplify"
date:   2018-04-11T18:00:00+02:00
author: Sebastien Le Gall
categories: [AWS, API-Gateway, Amplify]
---

Recently, I've been working with AWS in order to experience how it is to build a MVP really quickly. The goal I've been trying to achieve is to use as much AWS tools as possible to get a working product in production the fastest.
![aws-amplify](/img/aws-amplify-json-schema-gateway.png)

To do that, I choose to rely on :

* Lambda for back-end code
* API-Gateway for REST API
* DynamoDB as a database
* Cognito to manage Singnin / Singup and authenticated calls to the API.

<!--more-->

Once Cognito configured and a user created, I've been able to create a login form quite easily following the [server-stack](https://serverless-stack.com/) tutorial.

Then, I've started protecting my API-Gateway endpoints with an IAM role in order to make some of them only reachable for logged users.

```yml
functions:
  addSnippet:
      handler: bin/handlers/add_snippet
      package:
        include:
          - ./bin/handlers/add_snippet
      events:
        - http:
            path: snippets
            method: post
            cors: true
            authorizer: aws_iam
```

By doing this, I needed to start making my http calls authenticated... this is where things started to mess.

Make an authenticated call when working with Cognito and API-Gateway means using a [JWT](https://jwt.io/) retrieved from the authorizer (the IAM role here). But this token must be signed.

AWS used to recommend their Javascript SDK to do that. But the procedure is quite complicated. We need to use at least 3 different libs to do the job ([here is an example](https://serverless-stack.com/chapters/connect-to-api-gateway-with-iam-auth.html)). To fix that and make things easier for developers, Amazon shipped a React lib called `aws-amplify` : [https://github.com/aws/aws-amplify](https://github.com/aws/aws-amplify).

Amplify may be seen as a unification of those 3 libs (including the AWS JS SDK) thought to make developer life easier.

And indeed, here is how it works :

_Configuration_

```js
Amplify.configure({
    Auth: {
        mandatorySignIn: true,
        region: config.cognito.REGION,
        userPoolId: config.cognito.USER_POOL_ID,
        identityPoolId: config.cognito.IDENTITY_POOL_ID,
        userPoolWebClientId: config.cognito.APP_CLIENT_ID
    },
    API: {
        endpoints: [
            {
                name: "snippets",
                endpoint: config.apiGateway.URL,
                region: config.apiGateway.REGION
            },
        ]
    }
});
```

_Login_

```js
Auth.signIn(email, password);
```

Once logged in, you can easily call your API endpoints using :

```js
API.post("snippets","snippets/", {
            body: data,
        });
```

Simple, right? You can find more detailed examples in [the AWS Amplify API documentation](https://aws.github.io/aws-amplify/media/api_guide).

## JSON schema validation with API-Gateway

About getting a working product as soon as possible, **there is a thing you probably don't want to do by yourself : payload validation**.

Good news, API-Gateway provide a JSON schema validation feature !

Documentation can be found here : [https://docs.aws.amazon.com/apigateway/latest/developerguide/api-gateway-method-request-validation.html](https://docs.aws.amazon.com/apigateway/latest/developerguide/api-gateway-method-request-validation.html)

The big picture is :

* An http call is made to a API-gateway endpoint;
* Before the integration request, a JSON schema validation is done;
* If the payload / parameters don't pass the validation, the lambda isn't called at all;
* API-gateway returns a 400 status code and an error message in the returned payload such as :

```json
{
    "message": "Missing required request parameters: [q1]"
}
```

Well, let's put it together. What if I want to use those validation error messages in my front-end and display them to the user?

### Amplify under the hood

Under the hood, `ws-amplify` use [Axios](https://github.com/axios/axios) to make http calls.

It means than, contrary to `Fetch`, not only network errors are thrown as an exception. With `Axios`, all > 299 response status code are thrown as an exception.

Here are 2 examples of use.

_With Fetch :_

```js
async function postSnippet(data) {
    try {
        const response = await fetch(`http://example.com/snippets/`, {
            method: 'POST',
            body: JSON.stringify(data),
        });

        if (!response.ok) {
            throw response;
        }
        return await response.json();
    } catch (err) {
        console.log(await err.json())
    }

}
```

_With Amplify :_

```js
async function postSnippet(data) {
    try {
        return await API.post("snippets","snippets/", {
            body: data,
        });
    } catch (err) {
        console.log(err.response)
    }
}
```

*See the difference?*

Because Amplify use Axios, when API-Gateway returns a 400 status code, the error is thrown and if you want to use the returned payload you need to access the `err.response.data` object (see [this issue](https://github.com/axios/axios/issues/960)). But this is not so obvious and more over, there is no example of error handling in the `aws-amplify` documentation.

I have opened a Pull Request ([https://github.com/aws/aws-amplify/pull/633](https://github.com/aws/aws-amplify/pull/633)) to document this behaviour.

I also answer my own question on stackoverflow to make things clearer : [https://stackoverflow.com/questions/49633463/how-to-handle-api-errors-using-aws-amplify](https://stackoverflow.com/questions/49633463/how-to-handle-api-errors-using-aws-amplify)