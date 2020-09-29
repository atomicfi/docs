---
title: Atomic Developer Documentation
language_tabs:
    - shell: cURL
    - javascript--nodejs: Node.JS
    - ruby: Ruby
    - python: Python
    - go: Go
# toc_footers:
#     - >-
#         <a href="https://mermade.github.io/shins/asyncapi.html">See AsyncAPI
#         example</a>
includes: []
search: true
highlight_theme: darkula
headingLevel: 2
---

# Overview

## Introduction

> Production Base URL

```
https://api.atomicfi.com
```

> Sandbox Base URL

```
https://sandbox-api.atomicfi.com
```

### Welcome to Atomic's developer documentation!

You'll find resources to help you integrate with [Transact](#transact-sdk), our web app that handles the heavy-lifting of integration, as well as a number of [REST](en.wikipedia.org/wiki/Representational_State_Transfer)ful API endpoints.

We've tried our best to make this documentation comprehensive and helpful. If you have suggestions or find issues, [please let us know](mailto:engineering@atomicfi.com).

To get going quickly, we recommend using an API collaboration tool called [Postman](https://www.getpostman.com). You can use the button below to import our collection of endpoints. Be sure to set your `apiUrl`, `apiKey`, and `apiSecret` [environment variables](https://learning.getpostman.com/docs/postman/environments-and-globals/manage-environments/).

<div class="postman-run-button"
data-postman-action="collection/import"
data-postman-var-1="4d830a56bc79a690a118"></div>
<script type="text/javascript">
  (function (p,o,s,t,m,a,n) {
    !p[s] && (p[s] = function () { (p[t] || (p[t] = [])).push(arguments); });
    !o.getElementById(s+t) && o.getElementsByTagName("head")[0].appendChild((
      (n = o.createElement("script")),
      (n.id = s+t), (n.async = 1), (n.src = m), n
    ));
  }(window, document, "_pm", "PostmanRunObject", "https://run.pstmn.io/button.js"));
</script>

## Authentication

Atomic uses a combination of API keys and access tokens to authenticate requests.

You can retrieve and manage your API keys in the [Atomic Dashboard](https://dashboard.atomicfi.com/).

Your API keys carry many privileges, so be sure to keep them secure! Do not share your secret API keys in publicly accessible areas such as GitHub, client-side code, etc.

## Errors

Our API returns standard HTTP success or error status codes. For errors, we will also include extra information about what went wrong encoded in the response as JSON. The various HTTP status codes we might return are listed below.

| Code | Title                 | Description                     |
| ---- | --------------------- | ------------------------------- |
| 200  | OK                    | The request was successful.     |
| 400  | Bad Request           | The request was unacceptable.   |
| 401  | Unauthorized          | Missing or invalid credentials. |
| 404  | Not Found             | The resource does not exist.    |
| 422  | Validation error      | A validation error occurred.    |
| 50X  | Internal Server Error | An error occurred with our API. |

# Products

## Payroll Connect

### Atomic's Payroll Connect products empower users to access and update their payroll data.

When users are authenticating with their payroll account, Atomic requires that the process be facilitated through [Transact](#transact-sdk).

### `deposit`

Update the destination bank account of paychecks

### `verify`

Verify a user’s income and employment status.

### `identify`

Reduce fraud and onboarding friction with verified profile data from a user’s employer.

### Testing

To aid in testing various user experiences, you may use any of these pre-determined "test" credentials for employer authentication. Any password will work, as long as the username is found in this list. When answering MFA questions, any answer will be accepted. If the authentication requires an email, simply append `@test.com` to the end of the chosen username.

| Username            | Test Case                                                  |
| ------------------- | ---------------------------------------------------------- |
| `test-good`         | Successful authentication.                                 |
| `test-bad`          | Unsuccessful authentication.                               |
| `test-code-mfa`     | Authentication that includes a device code based MFA flow. |
| `test-question-mfa` | Authentication that includes a question-based MFA flow.    |

## Transfer

### Atomic's Transfer products facilitate the movement of assets and liabilities between financial institutions.

### `balance`

Move a user’s credit card or loan balance to a new account.

### Deployment options

#### Transact SDK

[Transact](#transact-sdk) allows the user to easily choose the source of their transfer, while the destination is pre-configured during [Access Token](#create-access-token) creation.

#### Atomic Dashboard

The [Atomic Dashboard](https://dashboard.atomicfi.com/) may be used to initiate a transfer, as well as monitor it's progress as it travels through various states of processing.

#### API

Atomic's API may be used to initiate a transfer by first creating an [Access Token](#create-access-token), and subsequently generating a [Task](#create-task). The status of the task may be monitored through [webhooks](#webhooks) and the [Atomic Dashboard](https://dashboard.atomicfi.com/).

### Testing

When testing `balance`, you may use `4111111111111111` as the origin account number. This will mimic a transfer.

# Transact SDK

![alt text](/source/images/transfer-devices.png "Transact Preview")

### Atomic Transact SDK is a UI designed to securely handle interactions with our products, while performing the heavy-lifting of integration.

### Integration

> Initalize Transact via Javascript

> Sandbox URL

```html
<script src="https://cdn-sandbox.atomicfi.com/transact.js"></script>
```

> Production URL

```html
<script src="https://cdn.atomicfi.com/transact.js"></script>
```

```javascript
const startTransact = () => {
    Atomic.transact({
        // Replace with the server-side generated `publicToken`
        publicToken: "PUBLIC_TOKEN",
        // Could be either 'balance', `verify`, `identify`, or 'deposit'
        product: "balance",
        // Optionally theme Transact with a *dark* color
        color: "#4B39EF",
        // Optionally change the language. Could be either 'en' for English or 'es' for Spanish. Default is 'en'.
        language: "en",
        onFinish: function (data) {
            // Called when the user finishes the transaction
            // We recommend saving the `data` object which could be useful for support purposes
        },
        onClose: function () {
            // Called when the user exits Transact prematurely
        },
    });
};

// `startTransact` would typically be user-initiated, such as a button click or tap
startTransact();
```

An [AccessToken](#create-access-token) is required to initialize Transact, and is generated server-side using your API key, and the `publicToken` from the response is returned to your client-side code.

### Mobile & Web

Transact can be initialized by including our JavaScript SDK into your page, and then calling the `Atomic.start` method.

Within the context of a mobile app, we recommend loading Transact into a web view (for example, `WKWebView` on iOS) by building a simple self-hosted wrapper page. The URL used in the webview will be `https://transact.atomicfi.com/initialize/BASE64_ENCODED_STRING`. The `BASE64 Encoded String` is made up of an object containing the parameters used within the SDK (excluding `onFinish` and `onClose`).

> Initialize Transact within a Webview

> Sandbox URL

```html
https://transact-sandbox.atomicfi.com/initialize/BASE64_ENCODED_STRING
```

> Production URL

```html
https://transact.atomicfi.com/initialize/BASE64_ENCODED_STRING
```

```javascript
//Object to be BASE64 Encoded
{
  publicToken: "PUBLIC_TOKEN",
  // Could be either 'balance', 'verify', 'identify', or 'deposit'
  product: "deposit",
  // Optionally theme Transact with a *dark* color
  color: '#1b1464',
  // Optionally start on a specific step. Accepts an object
  deeplink: {
    step: 'search-company'
  },
  // Optionally change the language. Could be either 'en' for English or 'es' for Spanish. Default is 'en'.
  language: 'en'
}

```

Here are examples for [Swift (iOS)](https://github.com/atomicfi/transact-ios), [Kotlin (Android)](https://github.com/atomicfi/transact-android), and [React Native](https://github.com/atomicfi/transact-react-native)
<br />
<a href="https://github.com/atomicfi/transact-android"><img src="/source/images/kotlin.png" class="transact-example-platform" /></a>
<a href="https://github.com/atomicfi/transact-ios"><img src="/source/images/swift.png" class="transact-example-platform" /></a>
<a href="https://github.com/atomicfi/transact-react-native"><img src="/source/images/react-native.png" class="transact-example-platform" /></a>

### SMS

To invite a user to use [Transact](#transact-sdk) over SMS, follow the instructions as described in the [AccessToken](#create-access-token) section.

### Javascript SDK parameters

| Attribute                       | Description                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                  |
| ------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `publicToken` <h6>required</h6> | The public token returned during [AccessToken](#create-access-token) creation.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                               |
| `product` <h6>required</h6>     | The [product](#products) to initiate. Valid values include `balance` `deposit`, `verify`, or `identify`                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                      |
| `color`                         | Optionally, provide a hex color code to customize Transact.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                  |
| `deeplink`                      | Optionally, start on a specific step. Accepts an object. <table><tr><th>Property</th><th>Value</th></tr><tr><td>`step`<h6>required</h6></td><td>Acceptable values: `search-company`, `login-company` or `login-connector`. (If `login-company`, then the `companyId` is required. If `login-connector`, then `connectorId` and `companyName` are required)</td></tr><tr><td>`companyId`</td><td>Required if the step is `login-company`. Accepts the [ID](#company-search) of the company.</td></tr><tr><td>`connectorId`</td><td>Required if the step is `login-connector`. Accepts the [ID](#connector-search) of the connector.</td></tr><tr><td>`companyName`</td><td>Required if the step is `login-connector`. Accepts a string of the company name.</td></tr></table> |
| `language`                      | Optionally pass in a language. Acceptable values: `en` for English and `es` for Spanish. Default value is `en`                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                               |
| `linkedAccount`                 | Optionally pass the `_id` of a [LinkedAccount](#linkedaccount-object). When used, Transact will immediately begin authenticating upon opening. This parameter is used when the [LinkedAccount](#linkedaccount-object)'s `transactRequired` flag is set to `true`.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                            |
| `onFinish`                      | A function that is called when the user finishes the transaction. The function will receive a `data` object.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                 |
| `onClose`                       | Called when the user exits Transact prematurely.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                             |

#### Event Listeners

When using the SDK, events will be emitted and passed to the native application. Such events allow native applications to react and perform functions as needed. Some events will be passed with a data object with additional information.
| Event | Description |
| ------------------------- | ----------------------------------------------------------------------------------------------------------- |
| `atomic-transact-close` | Triggered in several different instances. <br/><br/> 1. If a user does not find their employer and payroll provider, the data passed with the event will be `{reason: 'zero-search-results'}`. <br/><br/> 2. During the Transact process if a user is prompted to keep waiting or exit and they choose to exit, the data passed with the event will be `{reason: 'task-pending'}`.<br/><br/> 3. At any point if the user clicks on the `x` the data passed with the event will be `{reason: 'unknown'}`|
| `atomic-transact-finish` | Triggered when the user reaches the success screen and closes Transact|
| `atomic-transact-open-url` | Triggered when external links are accessed. The data passed with the event will be `{url: Full URL path}`|
| `atomic-transact-interaction` | Triggered on interactions within Transact. For example, when a user transitions to a news screen or presses the back button. The data passed with the event will be `{name: NAME OF THE EVENT, value: OBJECT CONTAINING EVENT VALUES}`. |

#### Event Interactions

```javascript
{
  name: "EVENT_NAME"
  value: {
    //Default property
    customer: "Atomic",
    //Default property. Possible values are 'en' or 'es'
    language: "en"
    //Default property. Possible values are 'deposit', 'verify', 'identify', or 'balance'
    product: "deposit"
    //Additional properties will be included based on the event that has occurred.
  }
}

```

Each event, by default, will have `customer`, `product`, and `language` in the `value object`. Most events have additional data. See the Additional Properties table below

| Name                                      | Description                                              |
| ----------------------------------------- | -------------------------------------------------------- |
| `back`                                    | Back button pressed                                      |
| `step-initialize-welcome`                 | Welcome screen is initialized                            |
| `step-initialize-deeplink-search-company` | User deeplinks to the company search                     |
| `step-welcome`                            | User visits the Welcome screen                           |
| `step-search-company`                     | User visits the Company Search screen                    |
| `step-search-payroll`                     | User visits the Payroll Provider Search screen           |
| `step-company-uses`                       | User visits Company X uses Payroll Provider X screen     |
| `step-phone-verification`                 | User prompted to verify their phone number               |
| `step-phone-update`                       | User visits the Phone Update screen                      |
| `step-unauthorized`                       | User is unauthorized                                     |
| `step-access-expired-token`               | User is using an expired token                           |
| `authentication-login`                    | User visits the Payroll Provider Login screen            |
| `authentication-login-help`               | User visits the Forgot Credentials screen                |
| `authentication-mfa-step`                 | User is presented with Multi Factor Authentication (MFA) |
| `authentication-success`                  | User completes the authentication process                |
| `authentication-error`                    | User receives an error during authentication             |
| `search`                                  | User is searching an Company or Payroll Provider         |
| `select-company`                          | User selects a company (aka employer)                    |
| `select-payroll`                          | User selects a payroll provider                          |

#### Additional Properties

| Property        | Description                                                    |
| --------------- | -------------------------------------------------------------- |
| `company`       | User's employer that they have selected                        |
| `payroll`       | Payroll provider the the company uses (ie: ADP, Gusto, etc...) |
| `searchCompany` | Search input for a company                                     |
| `searchPayroll` | Search input for a payroll provider                            |
| `exitScreen`    | Screen user was on when they exited the app                    |
| `fromScreen`    | Screen user was on when they pressed the back button           |
| `depositOption` | Option user chose during the deposit options                   |

# Webhooks

### Webhooks are a useful way to receive automated updates, and send timely notification to your user as a transaction progresses.

You can configure webhook endpoints via the [Atomic Dashboard](https://dashboard.atomicfi.com/) or during the [initialization of Transact](#transact-sdk). Atomic will issue `POST` requests to the designated endpoint(s). We recommend using [Request Bin](https://requestbin.com/) as a way to inspect the payload of events during development without needing to stand up a server.

## Securing webhooks

To validate a webhook request came from Atomic, we suggest verifying the payloads with the `X-HMAC-Signature-Sha-256` header (which we pass with every request).

| Header                     | Description                                                                          |
| -------------------------- | ------------------------------------------------------------------------------------ |
| `X-HMAC-Signature-Sha-256` | A SHA1 HMAC hexdigest computed with your API secret and the raw body of the request. |

## Event object

> Sample

```json
{
    "eventType": "task-status-updated",
    "eventTime": "2020-01-28T22:04:18.778Z",
    "product": "deposit",
    "user": {
        "_id": "5d8d3fecbf637ef3b11a877a",
        "identifier": "YOUR_INTERNAL_GUID"
    },
    "task": "5e30afde097146a8fc3d5cec",
    "data": {
        "previousStatus": "processing",
        "status": "completed",
        "transferType": "total"
    }
}
```

| Attribute   | Description                                                                     |
| ----------- | ------------------------------------------------------------------------------- |
| `eventType` | Status of a task was changed. Currently this value is `task-status-updated`     |
| `eventTime` | The date and time of the event creation.                                        |
| `user`      | Object containg `_id` and `identifier`. `Identifier` will be your internal GUID |
| `task`      | Contains the task ID.                                                           |
| `data`      | Payload object containing `previousStatus`, `status`, and `transferType`        |

## Event types

> Task Status Update

```json
{
    "_id": "5c1821dbc6b7baf3435e1d23",
    "created": "2019-11-20T16:51:12Z",
    "task": "5d97e4abc90a0a0007993e9c",
    "user": {
        "_id": "5c17c632e1d8ca3b08b2586f",
        "identifier": "YOUR_UNIQUE_GUID"
    },
    "type": "task-status-updated",
    "data": {
        "previousStatus": "processing",
        "status": "completed"
    }
}
```

### task-status-updated

The status of a [Task](#create-task) was changed. Possible statuses include:

| Status       | Description                      |
| ------------ | -------------------------------- |
| `queued`     | The task is awaiting processing. |
| `processing` | The task has started processing. |
| `failed`     | The task failed to process.      |
| `completed`  | The task completed successfully. |

## Outputs

> Sample response for `verify`

```json
{
    "eventType": "task-status-updated",
    "eventTime": "2020-01-28T22:04:18.778Z",
    "product": "verify",
    "user": {
        "_id": "5d8d3fecbf637ef3b11a877a",
        "identifier": "YOUR_INTERNAL_GUID"
    },
    "task": "5e30afde097146a8fc3d5cec",
    "data": {
        "previousStatus": "processing",
        "status": "completed",
        "outputs": {
            "income": "45000",
            "incomeType": "yearly",
            "employeeType": "fulltime",
            "employmentStatus": "active",
            "jobTitle": "Product Manager",
            "startDate": "4/19/2017",
            "weeklyHours": "40",
            "statements": [
                {
                    "date": "2020-01-01T00:00:00.000Z",
                    "grossAmount": "1266.11",
                    "netAmount": "1014.29",
                    "paymentMethod": "deposit",
                    "paystub": "https://example.url/statement1.pdf",
                    "hours": "32"
                },
                {
                    "date": "2019-12-15T00:00:00.000Z",
                    "grossAmount": "1266.11",
                    "netAmount": "1014.29",
                    "paymentMethod": "deposit",
                    "paystub": "https://example.url/statement2.pdf",
                    "hours": "32"
                }
            ],
            "accounts": [
                {
                    "accountNumber": "220000000",
                    "routingNumber": "110000000",
                    "type": "checking",
                    "distributionType": "total"
                }
            ]
        }
    }
}
```

### Outputs for the `verify` product

| Attribute          | Description                                                                                       |
| ------------------ | ------------------------------------------------------------------------------------------------- |
| `income`           | Employee's income, represented as a number.                                                       |
| `incomeType`       | Employee's income type. Possible values are `yearly`, `monthly`, `weekly`, `daily`, and `hourly`. |
| `employeeType`     | Employee type. Possible values are `parttime` and `fulltime`.                                     |
| `employmentStatus` | Employment status. Possible values are `active` and `terminated`.                                 |
| `jobTitle`         | Employee's job title.                                                                             |
| `startDate`        | Employee's hire date.                                                                             |
| `weeklyHours`      | Number of hours worked per week.                                                                  |
| `payCycle`         | Payment period. Possible values are `monthly`, `semimonthly`, `biweekly`, and `weekly`.           |
| `statements`       | An array of [statements](#statement).                                                             |
| `accounts`         | An array of bank [accounts](#deposit-account) on file for paycheck distributions.                 |

### Nested object properties

#### Statement

> Sample statement

```json
{
    "date": "2020-01-01T00:00:00.000Z",
    "grossAmount": "1266.11",
    "netAmount": "1014.29",
    "paymentMethod": "deposit",
    "paystub": "https://example.url/statement1.pdf",
    "hours": "32"
}
```

##### Properties

| Name                     | Type   | Description                                                               |
| ------------------------ | ------ | ------------------------------------------------------------------------- |
| `date` <h6>required</h6> | string | Date of the deposit.                                                      |
| `grossAmount`            | string | Gross dollar amount of the deposit.                                       |
| `netAmount`              | string | Net dollar amount of the deposit.                                         |
| `paymentMethod`          | string | Method used for the payment. Possible values include `deposit` or `check` |
| `paystub`                | string | A link to download a PDF of the paystub.                                  |
| `hours`                  | number | Hours worked within the pay period.                                       |

#### Deposit account

> Sample deposit account

```json
{
    "accountNumber": "220000000",
    "routingNumber": "110000000",
    "type": "checking",
    "distributionType": "percent",
    "distributionAmount": "75"
}
```

##### Properties

| Name                              | Type   | Description                                                                                                                                                                                                                                                                         |
| --------------------------------- | ------ | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `accountNumber` <h6>required</h6> | string | Account number.                                                                                                                                                                                                                                                                     |
| `routingNumber`                   | string | When account the account is bank account, this is the ABA routing number.                                                                                                                                                                                                           |
| `type`                            | string | Type of account. Possible values include `checking` or `savings`                                                                                                                                                                                                                    |
| `distributionType`                | string | The type of distribution for the account. Possible values include `total`, `percent`, or `fixed`..                                                                                                                                                                                  |
| `distributionAmount`              | number | The amount being distributed to the account. When `distributionType` is `percent`, the number represents a percentage of the total pay. When `distributionType` is `fixed`, this number represents a fixed dollar amount. This value is not set when `distributionType` is `total`. |

> Sample response for `identify`

```json
{
    "eventType": "task-status-updated",
    "eventTime": "2020-01-28T22:04:18.778Z",
    "product": "identify",
    "user": {
        "_id": "5d8d3fecbf637ef3b11a877a",
        "identifier": "YOUR_INTERNAL_GUID"
    },
    "task": "5e30afde097146a8fc3d5cec",
    "data": {
        "previousStatus": "processing",
        "status": "completed",
        "outputs": {
            "firstName": "Jane",
            "lastName": "Appleseed",
            "dateOfBirth": "6/4/98",
            "email": "jane@doe.com",
            "phone": "5558881111",
            "ssn": "111223333",
            "address": "123 Example St.",
            "city": "Salt Lake City",
            "state": "UT",
            "postalCode": "84111"
        }
    }
}
```

### Outputs for the `identify` product

| Attribute     | Description             |
| ------------- | ----------------------- |
| `firstName`   | First name.             |
| `lastName`    | Last name.              |
| `dateOfBirth` | Date of birth.          |
| `email`       | Email address.          |
| `phone`       | Phone number.           |
| `ssn`         | Social security number. |
| `address`     | Address street address. |
| `city`        | Address city.           |
| `state`       | Address state.          |
| `postalcode`  | Address postal code.    |

# API Reference

## Create Access Token

> Code samples

```shell
# You can also use wget
curl --location --request POST "https://api.atomicfi.com/access-token" \
  --header "Content-Type: application/json" \
  --header "x-api-key: f0d0a166-96de-4898-8879-da309801968b" \
  --header "x-api-secret: afcce08f-95bd-4317-9119-ecb8debae4f2" \
  --data "{
	\"identifier\": \"YOUR_INTERNAL_GUID\",
	\"phone\": \"8016554444\",
	\"email\": \"jdoe@example.org\",
	\"names\": [
		{
			\"firstName\": \"Jane\",
			\"lastName\": \"Doe\"
		}
	],
	\"addresses\": [
		{
			\"line1\": \"123 Atomic Ave.\",
			\"line2\": \"Apt. #1\",
			\"city\": \"Salt Lake City\",
			\"state\": \"UT\",
			\"zipcode\": \"84044\",
			\"country\": \"US\"
		}
	],
	\"accounts\": [
		{
			\"accountNumber\": \"220000000\",
			\"routingNumber\": \"110000000\",
			\"type\": \"checking\",
			\"title\": \"Premier Plus Checking\"
		}
	]
}"

```

```javascript--nodejs
var https = require('https');

var options = {
  'method': 'POST',
  'hostname': 'https://api.atomicfi.com',
  'path': '/access-token',
  'headers': {
    'Content-Type': 'application/json',
    'x-api-key': 'f0d0a166-96de-4898-8879-da309801968b',
    'x-api-secret': 'afcce08f-95bd-4317-9119-ecb8debae4f2'
  }
};

var req = https.request(options, function (res) {
  var chunks = [];

  res.on("data", function (chunk) {
    chunks.push(chunk);
  });

  res.on("end", function (chunk) {
    var body = Buffer.concat(chunks);
    console.log(body.toString());
  });

  res.on("error", function (error) {
    console.error(error);
  });
});

var postData =  "{\n\t\"identifier\": \"YOUR_INTERNAL_GUID\",\n\t\"phone\": \"8016554444\",\n\t\"email\": \"jdoe@example.org\",\n\t\"names\": [\n\t\t{\n\t\t\t\"firstName\": \"Jane\",\n\t\t\t\"lastName\": \"Doe\"\n\t\t}\n\t],\n\t\"addresses\": [\n\t\t{\n\t\t\t\"line1\": \"123 Atomic Ave.\",\n\t\t\t\"line2\": \"Apt. #1\",\n\t\t\t\"city\": \"Salt Lake City\",\n\t\t\t\"state\": \"UT\",\n\t\t\t\"zipcode\": \"84044\",\n\t\t\t\"country\": \"US\"\n\t\t}\n\t],\n\t\"accounts\": [\n\t\t{\n\t\t\t\"accountNumber\": \"220000000\",\n\t\t\t\"routingNumber\": \"110000000\",\n\t\t\t\"type\": \"checking\",\n\t\t\t\"title\": \"Premier Plus Checking\"\n\t\t}\n\t]\n}";

req.write(postData);

req.end();

```

```ruby
require "uri"
require "net/http"

url = URI("https://api.atomicfi.com/access-token")

http = Net::HTTP.new(url.host, url.port)

request = Net::HTTP::Post.new(url)
request["Content-Type"] = "application/json"
request["x-api-key"] = "f0d0a166-96de-4898-8879-da309801968b"
request["x-api-secret"] = "afcce08f-95bd-4317-9119-ecb8debae4f2"
request.body = "{\n\t\"identifier\": \"YOUR_INTERNAL_GUID\",\n\t\"phone\": \"8016554444\",\n\t\"email\": \"jdoe@example.org\",\n\t\"names\": [\n\t\t{\n\t\t\t\"firstName\": \"Jane\",\n\t\t\t\"lastName\": \"Doe\"\n\t\t}\n\t],\n\t\"addresses\": [\n\t\t{\n\t\t\t\"line1\": \"123 Atomic Ave.\",\n\t\t\t\"line2\": \"Apt. #1\",\n\t\t\t\"city\": \"Salt Lake City\",\n\t\t\t\"state\": \"UT\",\n\t\t\t\"zipcode\": \"84044\",\n\t\t\t\"country\": \"US\"\n\t\t}\n\t],\n\t\"accounts\": [\n\t\t{\n\t\t\t\"accountNumber\": \"220000000\",\n\t\t\t\"routingNumber\": \"110000000\",\n\t\t\t\"type\": \"checking\",\n\t\t\t\"title\": \"Premier Plus Checking\"\n\t\t}\n\t]\n}"

response = http.request(request)
puts response.read_body


```

```python
import requests
url = 'https://api.atomicfi.com/access-token'
payload = "{\n\t\"identifier\": \"YOUR_INTERNAL_GUID\",\n\t\"phone\": \"8016554444\",\n\t\"email\": \"jdoe@example.org\",\n\t\"names\": [\n\t\t{\n\t\t\t\"firstName\": \"Jane\",\n\t\t\t\"lastName\": \"Doe\"\n\t\t}\n\t],\n\t\"addresses\": [\n\t\t{\n\t\t\t\"line1\": \"123 Atomic Ave.\",\n\t\t\t\"line2\": \"Apt. #1\",\n\t\t\t\"city\": \"Salt Lake City\",\n\t\t\t\"state\": \"UT\",\n\t\t\t\"zipcode\": \"84044\",\n\t\t\t\"country\": \"US\"\n\t\t}\n\t],\n\t\"accounts\": [\n\t\t{\n\t\t\t\"accountNumber\": \"220000000\",\n\t\t\t\"routingNumber\": \"110000000\",\n\t\t\t\"type\": \"checking\",\n\t\t\t\"title\": \"Premier Plus Checking\"\n\t\t}\n\t]\n}"
headers = {
  'Content-Type': 'application/json',
  'x-api-key': 'f0d0a166-96de-4898-8879-da309801968b',
  'x-api-secret': 'afcce08f-95bd-4317-9119-ecb8debae4f2'
}
response = requests.request('POST', url, headers = headers, data = payload, allow_redirects=False, timeout=undefined, allow_redirects=false)
print(response.text)
```

```go
package main

import (
  "fmt"
  "strings"
  "os"
  "path/filepath"
  "net/http"
  "io/ioutil"
)

func main() {

  url := "https://api.atomicfi.com/access-token"
  method := "POST"

  payload := strings.NewReader("{\n	\"identifier\": \"YOUR_INTERNAL_GUID\",\n	\"phone\": \"8016554444\",\n	\"email\": \"jdoe@example.org\",\n	\"names\": [\n		{\n			\"firstName\": \"Jane\",\n			\"lastName\": \"Doe\"\n		}\n	],\n	\"addresses\": [\n		{\n			\"line1\": \"123 Atomic Ave.\",\n			\"line2\": \"Apt. #1\",\n			\"city\": \"Salt Lake City\",\n			\"state\": \"UT\",\n			\"zipcode\": \"84044\",\n			\"country\": \"US\"\n		}\n	],\n	\"accounts\": [\n		{\n			\"accountNumber\": \"220000000\",\n			\"routingNumber\": \"110000000\",\n			\"type\": \"checking\",\n			\"title\": \"Premier Plus Checking\"\n		}\n	]\n}")

  client := &http.Client {
    CheckRedirect: func(req *http.Request, via []*http.Request) error {
      return http.ErrUseLastResponse
    },
  }
  req, err := http.NewRequest(method, url, payload)

  if err != nil {
    fmt.Println(err)
  }
  req.Header.Add("Content-Type", "application/json")
  req.Header.Add("x-api-key", "f0d0a166-96de-4898-8879-da309801968b")
  req.Header.Add("x-api-secret", "afcce08f-95bd-4317-9119-ecb8debae4f2")

  res, err := client.Do(req)
  defer res.Body.Close()
  body, err := ioutil.ReadAll(res.Body)

  fmt.Println(string(body))
}

```

> Example request

```json
{
    "identifier": "YOUR_INTERNAL_GUID",
    "phone": "8016554444",
    "email": "jdoe@example.org",
    "expiry": "2019-12-06T12:22:54Z",
    "names": [
        {
            "firstName": "Jane",
            "lastName": "Doe"
        }
    ],
    "addresses": [
        {
            "line1": "123 Atomic Ave.",
            "line2": "Apt. #1",
            "city": "Salt Lake City",
            "state": "UT",
            "zipcode": "84044",
            "country": "US"
        }
    ],
    "accounts": [
        {
            "accountNumber": "220000000",
            "routingNumber": "110000000",
            "type": "checking",
            "title": "Premier Plus Checking"
        }
    ]
}
```

An `AccessToken` grants access to Atomic's API resources for a specific user.

### HTTP Request

`POST /access-token`

### Authentication headers

| Name           | Description                        |
| -------------- | ---------------------------------- |
| `x-api-key`    | API Key for your Atomic account    |
| `x-api-secret` | API Secret for your Atomic account |

### Request properties

| Name                           | Type                  | Description                                                                                                                                                                                                             |
| ------------------------------ | --------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `identifier` <h6>required</h6> | string                | A unique identifier (GUID) from your system that will be used to reference this user.                                                                                                                                   |
| `expiry`                       | string                | An optional [ISO 8601](https://en.wikipedia.org/wiki/ISO_8601) date and time by which the `AccessToken` will expire. By default, it will be set to 24 hours after access token creation.                                |
| `sendUserInvite`               | string                | An optional field, that if set to a product (e.g. `deposit`, `verify`, `identify`, or `balance`), will send a text message to the user with a unique link to access Transact.                                           |
| `phone`                        | string                | A mobile phone number for the user is required if you plan on inviting a user to use [Transact](#transact-sdk) via SMS.                                                                                                 |
| `email`                        | string                | An email address for the user is required if you plan on inviting a user to use [Transact](#transact-sdk) via email.                                                                                                    |
| `accounts` <h6>required</h6>   | [[Account](#account)] | An array of bank and/or credit card accounts. At least one bank account is required for an [Deposit](#payroll-connect) transaction, and at least one card account is required for an [Transfer](#transfer) transaction. |
| `names`                        | [[Name](#name)]       | Account holder names, for reference.                                                                                                                                                                                    |
| `addresses`                    | [[Address](#address)] | Account holder addresses, for reference.                                                                                                                                                                                |

### Nested object properties

#### Account

> Bank account example

```json
{
    "accountNumber": "220000000",
    "routingNumber": "110000000",
    "type": "checking",
    "title": "Premier Plus Checking"
}
```

> Card account example

```json
{
    "accountNumber": "4111111111111111",
    "title": "ABC Bank Platinum Card",
    "type": "card",
    "transferLimit": "50000"
}
```

##### Properties

| Name                              | Type   | Description                                                                                                                     |
| --------------------------------- | ------ | ------------------------------------------------------------------------------------------------------------------------------- |
| `accountNumber` <h6>required</h6> | string | Account number.                                                                                                                 |
| `routingNumber`                   | string | When account the account is bank account, this is the ABA routing number.                                                       |
| `type` <h6>required</h6>          | string | Type of account. Possible values include `card`, `checking`, or `savings`. This field is required for creating an access token. |
| `title`                           | string | A friendly name for the account that could be shown to the user.                                                                |
| `transferLimit`                   | string | A balance transfer limit (in dollars) that may be optionally imposed when executing an [Transfer](#transfer) transaction.       |

#### Name

> Name example

```json
{
    "firstName": "Jane",
    "lastName": "Doe"
}
```

##### Properties

| Name        | Type   | Description                   |
| ----------- | ------ | ----------------------------- |
| `firstName` | string | First name of account holder. |
| `lastName`  | string | Last name of account holder.  |

#### Address

> Address example

```json
{
    "line1": "123 Atomic Ave.",
    "line2": "Apt. #1",
    "city": "Salt Lake City",
    "state": "UT",
    "zipcode": "84044",
    "country": "US"
}
```

##### Properties

| Name      | Type   | Description              |
| --------- | ------ | ------------------------ |
| `line1`   | string | First line of address.   |
| `line2`   | string | Second line of address . |
| `city`    | string | State of address.        |
| `state`   | string | Second line of address   |
| `zipcode` | string | 5-digit zipcode.         |
| `country` | string | Country of address.      |

### Response

> Example response

```json
{
    "data": {
        "publicToken": "6e93549e-3571-4f57-b0f7-77b7cb0b5e48",
        "token": "c00f869e-0fce-4d32-9277-a68658578ba2",
        "publicTokenExpiration": "2019-12-06T12:22:54Z"
    }
}
```

Successfully creating an `Access Token` will return a payload with a `data` object containing both a public `publicToken` and a private `token`. The `publicToken` may be used to initialize the [Transact SDK](#transact-sdk). If you plan on making subsequent API requests for this user, we recommend saving these tokens to authenticate future requests.

### Response Properties

| Name                    | Type   | Description                                                                                                         |
| ----------------------- | ------ | ------------------------------------------------------------------------------------------------------------------- |
| `publicToken`           | string | Public `AccessToken` that can be used to initialize [Transact SDK](#transact-sdk) and make subsequent API requests. |
| `publicTokenExpiration` | string | [ISO 8601](https://en.wikipedia.org/wiki/ISO_8601) date by which the `AccessToken` will no longer be valid.         |
| `token`                 | string | Private token that should be kept secret.                                                                           |
|                         |

## List Linked Accounts

> Code samples

```shell
curl --location --request GET "https://api.atomicfi.com/user/accounts" \
  --header "x-public-token: e0d2f67e-dc98-45d8-8b22-db76cb52f732"
```

```javascript--nodejs
var https = require('https');

var options = {
  'method': 'GET',
  'hostname': 'https://api.atomicfi.com',
  'path': '/user/accounts',
  'headers': {
    'x-public-token': 'e0d2f67e-dc98-45d8-8b22-db76cb52f732'
  }
};

var req = https.request(options, function (res) {
  var chunks = [];

  res.on("data", function (chunk) {
    chunks.push(chunk);
  });

  res.on("end", function (chunk) {
    var body = Buffer.concat(chunks);
    console.log(body.toString());
  });

  res.on("error", function (error) {
    console.error(error);
  });
});

req.end();

```

```ruby
require "uri"
require "net/http"

url = URI("https://api.atomicfi.com/user/accounts")

http = Net::HTTP.new(url.host, url.port)

request = Net::HTTP::Get.new(url)
request["x-public-token"] = "e0d2f67e-dc98-45d8-8b22-db76cb52f732"

response = http.request(request)
puts response.read_body


```

```python
import requests
url = 'https://api.atomicfi.com/user/accounts'
payload = {}
headers = {
  'x-public-token': 'e0d2f67e-dc98-45d8-8b22-db76cb52f732'
}
response = requests.request('GET', url, headers = headers, data = payload, allow_redirects=False, timeout=undefined, allow_redirects=false)
print(response.text)


```

```go
package main

import (
  "fmt"
  "os"
  "path/filepath"
  "net/http"
  "io/ioutil"
)

func main() {

  url := "https://api.atomicfi.com/user/accounts"
  method := "GET"

  client := &http.Client {
    CheckRedirect: func(req *http.Request, via []*http.Request) error {
      return http.ErrUseLastResponse
    },
  }
  req, err := http.NewRequest(method, url, nil)

  if err != nil {
    fmt.Println(err)
  }
  req.Header.Add("x-public-token", "e0d2f67e-dc98-45d8-8b22-db76cb52f732")

  res, err := client.Do(req)
  defer res.Body.Close()
  body, err := ioutil.ReadAll(res.Body)

  fmt.Println(string(body))
}
```

If enabled on your account, can be used to list accounts linked to a particular user.

### HTTP Request

`GET /user/accounts`

### Authentication headers

| Name             | Description                                                                  |
| ---------------- | ---------------------------------------------------------------------------- |
| `x-public-token` | Public token generated during [access token creation](#create-access-token). |

                                            |

### Response

> Example response

```json
{
    "data": [
        {
            "_id": "5f7212103a40e91f95ba376d",
            "valid": true,
            "connector": {
                "_id": "5d77f95207085632a58945c3",
                "name": "ADP",
                "branding": {}
            },
            "company": {
                "_id": "5d77f9e1070856f3828945c6",
                "name": "DoTerra",
                "branding": {
                    "logo": {
                        "_id": "5eb62781b4b83f0008f638cc",
                        "url": "https://atomicfi-public-production.s3.amazonaws.com/979115f4-34a0-44f5-901e-753a33337444_atomic-logo-dark.png"
                    }
                }
            },
            "lastSuccess": "2020-09-28T16:40:48.009Z",
            "transactRequired": false
        }
    ]
}
```

### LinkedAccount object

| Name               | Type    | Description                                                                                        |
| ------------------ | ------- | -------------------------------------------------------------------------------------------------- |
| `_id`              | string  | Unique identifier.                                                                                 |
| `valid`            | boolean | Whether or not the account credentials were valid after the last attempted use.                    |
| `transactRequired` | boolean | Whether or not using the account requires the user to be present within [Transact](#transact-sdk). |
| `lastSuccess`      | date    | The datetime of the last successful usage of the account.                                          |
| `lastFailure`      | date    | The datetime of the last failed usage of the account.                                              |
| `company`          | object  | The [Company](#company-object) to which the account is linked.                                     |
| `connector`        | object  | The [Connector](#connector-object) to which the account is linked.                                 |

## Use a Linked Account

> Code samples

```shell
curl --location --request POST "https://api.atomicfi.com/task" \
  --header "Content-Type: application/json" \
  --header "x-public-token: e0d2f67e-dc98-45d8-8b22-db76cb52f732" \
  --data "{
    \"product\": \"verify\",
    \"linkedAccount\": \"5d77f9e1070856f3828945c6\"
}"

```

```javascript--nodejs
var https = require('https');

var options = {
  'method': 'POST',
  'hostname': 'https://api.atomicfi.com',
  'path': '/task',
  'headers': {
    'Content-Type': 'application/json',
    'x-public-token': 'e0d2f67e-dc98-45d8-8b22-db76cb52f732'
  }
};

var req = https.request(options, function (res) {
  var chunks = [];

  res.on("data", function (chunk) {
    chunks.push(chunk);
  });

  res.on("end", function (chunk) {
    var body = Buffer.concat(chunks);
    console.log(body.toString());
  });

  res.on("error", function (error) {
    console.error(error);
  });
});

var postData =  "{\n    \"product\": \"verify\",\n    \"linkedAccount\": \"5d77f9e1070856f3828945c6\",\n    }\n}";

req.write(postData);

req.end();

```

```ruby
require "uri"
require "net/http"

url = URI("https://api.atomicfi.com/task")

http = Net::HTTP.new(url.host, url.port)

request = Net::HTTP::Post.new(url)
request["Content-Type"] = "application/json"
request["x-public-token"] = "e0d2f67e-dc98-45d8-8b22-db76cb52f732"
request.body = "{\n    \"product\": \"verify\",\n    \"linkedAccount\": \"5d77f9e1070856f3828945c6\"\n    }\n}"

response = http.request(request)
puts response.read_body


```

```python
import requests
url = 'https://api.atomicfi.com/task'
payload = "{\n    \"product\": \"verify\",\n    \"linkedAccount\": \"5d77f9e1070856f3828945c6\"\n    }\n}"
headers = {
  'Content-Type': 'application/json',
  'x-public-token': 'e0d2f67e-dc98-45d8-8b22-db76cb52f732'
}
response = requests.request('POST', url, headers = headers, data = payload, allow_redirects=False, timeout=undefined, allow_redirects=false)
print(response.text)

```

```go
package main

import (
  "fmt"
  "strings"
  "os"
  "path/filepath"
  "net/http"
  "io/ioutil"
)

func main() {

  url := "https://api.atomicfi.com/task"
  method := "POST"

  payload := strings.NewReader("{\n    \"product\": \"verify\",\n    \"linkedAccount\": \"5d77f9e1070856f3828945c6\"\n    }\n}")

  client := &http.Client {
    CheckRedirect: func(req *http.Request, via []*http.Request) error {
      return http.ErrUseLastResponse
    },
  }
  req, err := http.NewRequest(method, url, payload)

  if err != nil {
    fmt.Println(err)
  }
  req.Header.Add("Content-Type", "application/json")
  req.Header.Add("x-public-token", "e0d2f67e-dc98-45d8-8b22-db76cb52f732")

  res, err := client.Do(req)
  defer res.Body.Close()
  body, err := ioutil.ReadAll(res.Body)

  fmt.Println(string(body))
}


```

> Request body example

```json
{
    "product": "verify",
    "linkedAccount": "5d77f9e1070856f3828945c6"
}
```

To generate a task using a Linked Account, a `Task` request is created that contains the `product` and the `_id` of the [LinkedAccount](#linkedaccount-object). A `Task` is then created, and associated with a user via the `publicToken` used to authenticate the request.

### HTTP Request

`POST /task`

### Authentication headers

| Name             | Description                                                                  |
| ---------------- | ---------------------------------------------------------------------------- |
| `x-public-token` | Public token generated during [access token creation](#create-access-token). |

### Request properties

| Name                              | Type   | Description                                                    |
| --------------------------------- | ------ | -------------------------------------------------------------- |
| `product` <h6>required</h6>       | string | One of `verify`, `identify`, or `deposit`.                     |
| `linkedAccount` <h6>required</h6> | string | The `_id` of a [LinkedAccount](#linked-account-object) object. |

### Response

> Example responses

```json
{
    "data": {
        "id": "5d3b23b155f500465c895f60",
        "status": "queued"
    }
}
```

Successfully creating a `Task` will return a payload with a `data` object containing both a public `_id_` and a private `status` of `queued`. The `status` will change as the task progresses towards completion. The [Atomic Dashboard](https://dashboard.atomicfi.com/) can be used to track task progress. If you plan on implementing [webhooks](#webhooks), we recommend saving the task `_id` for reference.

### Response Properties

| Name     | Type   | Description                                   |
| -------- | ------ | --------------------------------------------- |
| `_id`    | string | Unique identifier for the `Task`.             |
| `status` | string | [Status](#task-status-updated) of the `Task`. |

## Company Search

> Code samples

```shell
curl --location --request GET "https://api.atomicfi.com/company/search?query=microsoft&product=deposit" \
  --header "x-public-token: e0d2f67e-dc98-45d8-8b22-db76cb52f732"
```

```javascript--nodejs
var https = require('https');

var options = {
  'method': 'GET',
  'hostname': 'https://api.atomicfi.com',
  'path': '/company/search?query=microsoft&product=deposit',
  'headers': {
    'x-public-token': 'e0d2f67e-dc98-45d8-8b22-db76cb52f732'
  }
};

var req = https.request(options, function (res) {
  var chunks = [];

  res.on("data", function (chunk) {
    chunks.push(chunk);
  });

  res.on("end", function (chunk) {
    var body = Buffer.concat(chunks);
    console.log(body.toString());
  });

  res.on("error", function (error) {
    console.error(error);
  });
});

req.end();

```

```ruby
require "uri"
require "net/http"

url = URI("https://api.atomicfi.com/company/search?query=microsoft&product=deposit")

http = Net::HTTP.new(url.host, url.port)

request = Net::HTTP::Get.new(url)
request["x-public-token"] = "e0d2f67e-dc98-45d8-8b22-db76cb52f732"

response = http.request(request)
puts response.read_body


```

```python
import requests
url = 'https://api.atomicfi.com/company/search?query=microsoft&product=deposit'
payload = {}
headers = {
  'x-public-token': 'e0d2f67e-dc98-45d8-8b22-db76cb52f732'
}
response = requests.request('GET', url, headers = headers, data = payload, allow_redirects=False, timeout=undefined, allow_redirects=false)
print(response.text)


```

```go
package main

import (
  "fmt"
  "os"
  "path/filepath"
  "net/http"
  "io/ioutil"
)

func main() {

  url := "https://api.atomicfi.com/company/search?query=microsoft&product=deposit"
  method := "GET"

  client := &http.Client {
    CheckRedirect: func(req *http.Request, via []*http.Request) error {
      return http.ErrUseLastResponse
    },
  }
  req, err := http.NewRequest(method, url, nil)

  if err != nil {
    fmt.Println(err)
  }
  req.Header.Add("x-public-token", "e0d2f67e-dc98-45d8-8b22-db76cb52f732")

  res, err := client.Do(req)
  defer res.Body.Close()
  body, err := ioutil.ReadAll(res.Body)

  fmt.Println(string(body))
}
```

Searches for a `Company` using a text `query`. Searches can also be narrowed by passing in a specific `product`. The primary use case of this endpoint is for an autocomplete search component.

### HTTP Request

`GET /company/search`

### Authentication headers

| Name             | Description                                                                  |
| ---------------- | ---------------------------------------------------------------------------- |
| `x-public-token` | Public token generated during [access token creation](#create-access-token). |

                                            |

### Request properties

| Name                      | Type   | Description                                                                                                     |
| ------------------------- | ------ | --------------------------------------------------------------------------------------------------------------- |
| `query` <h6>required</h6> | string | Filters companies by name. Uses fuzzy matching to narrow results.                                               |
| `product`                 | string | Filters companies by a specific product. Possible values include `balance`, `verify`, `identify`, and `deposit` |

### Response

> Example response

```json
{
    "data": [
        {
            "_id": "5d38f1e8512bbf71fb776015",
            "name": "Wells Fargo",
            "connector": {
                "_id": "5d38f182512bbf0c06776013",
                "availableProducts": ["balance"]
            },
            "branding": {
                "logo": "https://atomicfi-public.s3.amazonaws.com/1b27b5a3-db2d-4dbd-83e9-5e256a91d573.svg",
                "color": "#FFFFFF"
            }
        }
    ]
}
```

Successfully querying the `Company` search endpoint will return a payload with a `data` array of `Company` objects.

### Company object

| Name                | Type   | Description                                            |
| ------------------- | ------ | ------------------------------------------------------ |
| `_id`               | string | Unique identifier.                                     |
| `name`              | string | Company name.                                          |
| `availableProducts` | array  | A list of compatible products.                         |
| `branding.logo`     | string | Logo for the company, typically an `svg` if available. |
| `branding.color`    | string | Branding color for the company.                        |

## Connector Search

Searches for a `Connector` using a text `query`. Searches can also be narrowed by passing in a specific `product`. The primary use case of this endpoint is for an autocomplete search component.

### HTTP Request

`GET /connector/search`

### Authentication headers

| Name             | Description                                                                  |
| ---------------- | ---------------------------------------------------------------------------- |
| `x-public-token` | Public token generated during [access token creation](#create-access-token). |

                                            |

### Request properties

| Name                      | Type   | Description                                                                                                      |
| ------------------------- | ------ | ---------------------------------------------------------------------------------------------------------------- |
| `query` <h6>required</h6> | string | Filters connectors by name. Uses fuzzy matching to narrow results.                                               |
| `product`                 | string | Filters connectors by a specific product. Possible values include `balance`, `verify`, `identify`, and `deposit` |

### Response

> Example response

```json
{
    "success": true,
    "data": [
        {
            "_id": "5da2a2372a5c5600081d0052",
            "options": {
                "inputs": [
                    {
                        "title": "Username",
                        "name": "username",
                        "type": "text",
                        "placeholder": "Username"
                    },
                    {
                        "title": "Password",
                        "name": "password",
                        "type": "password",
                        "placeholder": "Password"
                    }
                ],
                "loginRecoveryOptions": [
                    {
                        "_id": "5e97992de81d67000802dda7",
                        "action": "url",
                        "title": "Recover User ID/Password",
                        "url": "https://netsecure.adp.com/ilink/pub/smsess/v3/forgot/theme.jsp?returnUrl=https%3A%2F%2Fmy.adp.com%2Fstatic%2Fredbox%2Flogin.html&totalURL=https://my.adp.com/static/redbox/login.html",
                        "flow": ""
                    },
                    {
                        "_id": "5e97992de81d67000802dda8",
                        "title": "Create Account",
                        "url": "https://netsecure.adp.com/pages/sms/ess/v3/pub/ssr/theme.jsp?returnUrl=https%3A%2F%2Fmy.adp.com%2Fstatic%2Fredbox%2Flogin.html",
                        "action": "url"
                    }
                ]
            },
            "name": "ADP",
            "createdAt": "2019-10-13T04:04:07.598Z",
            "branding": {
                "color": "#E20000",
                "logo": {
                    "_id": "5ed93faa3e36220007d157f6",
                    "url": "https://atomicfi-public-production.s3.amazonaws.com/a8d7e778-b718-45e0-b639-2305e33e7f95_ADP.svg"
                }
            },
            "authenticators": [
                {
                    "_id": "5f0ded47a5892d50f7fb38bb",
                    "connector": "5f03b5d210b6e90007e536cf",
                    "buttonLabel": "Sign in with Mocky",
                    "optional": true
                }
            ],
            "isAuthenticator": false
        }
    ]
}
```

Successfully querying the `Connector` search endpoint will return a payload with a `data` array of `Connector` objects.

### Connector object

| Name             | Type   | Description                                              |
| ---------------- | ------ | -------------------------------------------------------- |
| `_id`            | string | Unique identifier.                                       |
| `options`        | string | Object of authentications options.                       |
| `name`           | array  | Name of the connector.                                   |
| `branding.logo`  | string | Logo for the connector, typically an `svg` if available. |
| `branding.color` | string | Branding color for the company.                          |
| `authenticators` | string | Array of third party authenticators.                     |
