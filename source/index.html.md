---
title: Budgea API Documentation
language_tabs:
  - http: HTTP
  - python: Python
toc_footers: []
includes: []
search: false
highlight_theme: darkula
headingLevel: 2

---

<h1 id="budgea-api-documentation">Budgea API Documentation v2.0</h1>

> Scroll down for code samples, example requests and responses. Select a language for code samples from the tabs above or the mobile navigation menu.

# Budgea Development Guides

Welcome to **Budgea**'s documentation.

This documentation is intended to get you up-and-running with our APIs and advise on the implementation of some regulatory aspects of your application, following the DSP2's guidelines.

## Getting Started
**IMPORTANT**
Depending on your status with regard of the DSP2 regulation, **agent** or **partner**, you may call our APIs or simply use our Webview and callbacks to get the financial data of your users.
As an **agent**, you are allowed to call directly our APIs and implement your own form to get the user's credentials.
As a **partner**, you cannot manipulate the credentials, and have to delegate this step to us through our webview.

The sections below will document how to use our APIs, make sure you have the **agent** status to do so.
For the **partner**, please refer to the section *Webview* and *Callbacks* of this documentation.

### Overview
Your API is a REST API which requires a communication through https to send and receive JSON documents.
During your tests, we recommend to make calls to the API with curl or any other HTTP client of your choice.
You can watch a video demonstration on this [URL](https://asciinema.org/a/FsaFyt3WAPyDm7sfaZPkwal3V).
For the examples we'll use the demo API with address `https://demo.biapi.pro`, you should change that name to your API's name.

### Hello World
Let's start by calling the service `/banks` which lists all available banks.
```
curl -X GET \
  https://demo.biapi.pro/2.0/banks/
```
To log in to a bank webpage, you'll need to know for a given bank, the fields your user should fill in the form.
Let's call a  specific bank and ask for an additional resource *fields*.
```
curl -X GET \
  https://demo.biapi.pro/2.0/banks/59?expand=fields
```
The response here concerns only 1 bank (since we specified an id) and the resource _Fields_ is added to the response thanks to the query parameter `expand`.

To get more interesting things done, you'll need to send authenticated requests.

### Authentication
The way to authenticate is by passing the `Authorization: Bearer <token>` header in your request.
At the setup a _manage token_ have been generated, you can use this token for now, when creating your user we'll see how to generate a user's token.
```
curl -X GET \
  https://demo.biapi.pro/2.0/config \
  -H 'Authorization: Bearer <token>'
```
This endpoint will list all the parameters you can change to adapt Budgea to your needs.

We've covered the very first calls. Before diving deeper, let's see some general information about the APIs.

## Abstract

### API URL
`https://demo.biapi.pro/2.0`

### Requests format
Data format: **application/x-www-form-urlencoded** or **application/json** (suggested)

Additional headers: Authorization: User's token (private)

### Responses format
Data format: **application/json** ([http://www.json.org](http://www.json.org/))
Charset: **UTF-8**

### Resources
Each call on an endpoint will return resources. The main resources are:
| Resource            | Description                                                                                                           |
| ---------------------|:------------------------------------------------------------------------------------------------------------------   |
|Users                 |Represent a user                                                                                                      |
|Connection            |A set of data used to authenticate on a website (usually a login and password). There is 1 connection for each website|
|Account               |A bank account contained in a connection                                                                              |
|Transaction           |An entry in a bank account                                                                                            |
|Investment            |An asset in a bank account                                                                                            |

The chain of resources is as follow: **Users ∈ Connections ∈ Accounts ∈ Transactions or Investments**

### RESTful API
This API is RESTful, which means it is stateless and each resource is accessed with an uniq URI.

Several HTTP methods are available:

| Method                  | Description                    |
| ------------------------|:-------------------------------|
| GET /resources          | List resources                 |
| GET /resources/{ID}     | Get a resource from its ID     |
| POST /resources         | Create a new resource          |
| POST /resources/{ID}    | Update a resource              |
| PUT /resources  /{ID}   | Update a resource              |
| DELETE /resources       | Remove every resources         |
| DELETE /resources/{ID}  | Delete a resource              |

Each resource can contain sub-resources, for example:
`/users/me/connections/2/accounts/23/transactions/48`

### HTTP response codes

| Code        | Message               | Description                                                                                   |
| ----------- |:---------------------:|-----------------------------------------------------------------------------------------------|
| 200         | OK                    |Default response when a GET or POST request has succeed                                        |
| 202         | Accepted              |For a new connection this code means it is necessary to provide complementary information (2FA)|
| 204         | No content            |Default response when a POST request succeed without content                                   |
| 400         | Bad request           |Supplied parameters are incorrect                                                              |
| 403         | Forbidden             |Invalid token                                                                                  |
| 500         | Internal Servor Error |Server error                                                                                   |
| 503         | Service Unavailable   |Service is temporarily unavailable                                                             |

### Errors management
In case an error occurs (code 4xx or 5xx), the response can contain a JSON object describing this error:
```json
{
   "code": "authFailure",
   "message": "Wrong password"  // Optional
}
```
If an error is displayed on the website, Its content is returned in error_message field.
The list of all possible errors is listed further down this page.

### Authentication
A user is authenticated by an access_token which is sent by the API during a call on one of the authentication services, and can be supplied with this header:
`Authorization: Bearer YYYYYYYYYYYYYYYYYYYYYYYYYYY`

 There are two user levels:

    - Normal user, which can only access to his own accounts
    - Administrator, with extended rights

### Default filters
During a call to an URI which lists resources, some filters can be passed as query parameters:

| Parameter   | Type      | Description                                               |
| ----------- |:---------:|-----------------------------------------------------------|
| offset      | Integer   |Offset of the first returned resource                      |
| limit       | Integer   |Limit number of results                                    |
| min_date    | Date      |Minimal date (if supported by service), format: YYYY-MM-DD |
| max_date    | Date      |Maximal date (if supported by service), format: YYYY-MM-DD |

### Extend requests
During a GET on a set of resources or on a unique resource, it is possible to add a parameter expand to the request to extend relations with other resources:

`GET /2.0/users/me/accounts/123?expand=transactions[category],connection`

```json
{
   "id" : 123
   "name" : "Compte chèque"
   "balance" : 1561.15
   "transactions" : [
      {
         "id" : 9849,
         "simplified_wording" : "HALL'S BEER",
         "value" : -513.20,
         ...
         "category" : {
            "id" : 561,
            "name" : "Sorties / Bar",
            ...
         }
       },
       ...
   ],
   "id_user" : 1,
   "connection" : {
      "id" : 1518,
      "id_bank" : 41,
      "id_user" : 1,
      "error" : null,
      ...
   }
}
```

### Request example
```http
GET /2.0/banks?offset=0&limit=10&expand=fields
Host: demo.biapi.pro
Accept: application/json
Authorization: Bearer <token>
```
```http
HTTP/1.1 200 OK
Content-Type: application/json
Content-Length: 3026
Server: Apache
Date: Fri, 14 Mar 2014 08:24:02 GMT

{
   "banks" : [
      {
         "id_weboob" : "bnporc",
         "name" : "BNP Paribas",
         "id" : 3,
         "hidden" : false,
         "fields" : [
            {
               "id" : 1,
               "id_bank" : 3,
               "regex" : "^[0-9]{5,10}$",
               "name" : "login",
               "type" : "text",
               "label" : "Numéro client"
            },
            {
               "id" : 2,
               "id_bank" : 3,
               "regex" : "^[0-9]{6}$",
               "name" : "password",
               "type" : "password",
               "label" : "Code secret"
            }
         ]
      },
      ...
   ]
   "total" : 41
}
```

### Constants
#### List of bank account types
| Type          |Description                        |
| -----------   |-----------------------------------|
| checking      |Checking account                   |
| savings       |Savings account                    |
| deposit       |Deposit accounts                   |
| loan          |Loan                               |
| market        | Market accounts                   |
| joint         |Joint account                      |
| card          |Card                               |
| lifeinsurance |Life insurance accounts            |
| pee           |Plan Épargne Entreprise            |
| perco         |Plan Épargne Retraite              |
| article83     |Article 83                         |
| rsp           |Réserve spéciale de participation  |
| pea           |Plan d'épargne en actions          |
| capitalisation|Contrat de capitalisation          |
| perp          |Plan d'épargne retraite populaire  |
| madelin       |Contrat retraite Madelin           |
| unknown       |Inconnu                            |

#### List of transaction types

| Type         |Description                        |
| -----------  |-----------------------------------|
|transfer      |Transfers                          |
|order         |Orders                             |
|check         |Checks                             |
|deposit       |Cash deposit                       |
|payback       |Payback                            |
|withdrawal    |Withdrawal                         |
|loan_payment  |Loan payment                       |
|bank          |Bank fees                          |
|card          |Card operation                     |
|deferred_card |Deferred card operation            |
|card_summary  |Mensual debit of a deferred card   |

#### List of synchronization errors
##### Error on Connection object
The error field may take one of the below values in case of error when accessing the user space.

| Error                      |Description                                                                                       |
| -----------------------    |--------------------------------------------------------------------------------------------------|
|wrongpass                   |The authentication on website has failed                                                          |
|additionalInformationNeeded |Additional information is needed such as an OTP                                                  |
|websiteUnavailable          |The website is unavailable, for instance we get a HTTP 503 response when requesting the website   |
|actionNeeded                |An action is needed on the website by the user, scraping is blocked                               |
|bug                         |A bug has occurred during the synchronization. An alert has been sent to Budget Insight           |

#### Error on Account object
Errors can be filled at the account level in case we access the user's dashboard but some account related data cannot be retrieved.
For instance, we may not access the transactions or investments for a specific account.
Getting an error during an account synchronization does not impact the scraping of other acccounts.

| Error                      |Description                                                                                       |
| -----------------------    |--------------------------------------------------------------------------------------------------|
|websiteUnavailable          |The website or a page is unavailable                                                              |
|actionNeeded                |An action is needed on the website by the user, scraping is blocked                               |
|bug                         |A bug has occurred during the synchronization. An alert has been sent to Budget Insight           |

Now you know the basics of Budgea API
- Basic call to retrieve resources
- Add query parameters to aplly filters
- Expand resources
- Authenticated calls

We're good for the basics! Now let's see how to integrate Budgea in your app and create your first user.

## Integrate Budgea *(protocol or Webview)*
### The workflow
Users of your application exist in the Budgea API.
Every User is identified by an access_token which is the shared secret between your application and our API.

The workflow is as below:
1. The user is on your application and wants to share his bank accounts or invoices.
2. A call is made **client side** (browser's javascript or desktop application) to create a temporarily token which will be used to make API calls.
3. A form is built, allowing the user to select the connector to use (bank or provider, depending on context). Every connector requires different kind of credentials.
4. A call on the API is made with the temporarily token to add a **Connection** with the credentials supplied by user.
5. In case of success, the user chooses what bank accounts (**Account**) or subscriptions (**Subscription**) he wants to share with your application.
6. When he validates the share, the temporarily token is transmitted to your server. This one will call the Budgea API with this token to get a permanent token.

**Note**
In case your application works without a server (for example a desktop application), the permanent token can be obtained on the 1st step, by supplying a client_secret to /auth/init and the step 6 is omitted. To get more information, read the protocol.

There are 3 steps to integrate Budgea in your application:
1. Provide a way for your users to share their credentials with you
2. Get the data scraped from Budgea
3. Be sure to follow the good practices before going into production

### Get credentials from users
You have 2 options here:
- Integrate the Budget Insight's Webview, a turnkey solution to get user's credentials
- Create your own form following the protocol (must have the *agent* status)

#### Webview
The webview allows your users to give their bank or provider credentials to the safety third-party Budgea API.
It is responsive, which means it displays correctly on a desktop or a mobile device.

The API meets the standard [OAuth 2](http://tools.ietf.org/html/rfc6749) and any client library that implements it will be able to facilitate the integration to the Budgea API.
We provide PHP, Python and Ruby modules to facilitate the integration.

|Language         | Library                                                                                  |
|-----------------|------------------------------------------------------------------------------------------|
|PHP              |[BudgeaAPI.php](https://github.com/budgetinsight/budgea-clients/blob/master/BudgeaAPI.php)|
|PYTHON           |[budgea_api.py](https://github.com/budgetinsight/budgea-clients/blob/master/budgea_api.py)|
|RUBY             |[budgea.rb](https://github.com/budgetinsight/budgea-clients/blob/master/budgea.rb)        |

To delegate to the Budgea API the management of the bank and provider credentials of your users, you have to provide a button which redirects to the webview hosted at:
`https://demo.biapi.pro/2.0/auth/webview/connect/select?client_id=<your_client_id>&redirect_uri=<your_uri>`

To fill the parameters you must first configure your client application through the [Admin console](https://console.budget-insight.com).
A client, in the OAuth 2 of the term, represents an application accessing to the Budgea API.
From the admin console, you can edit the *redirect_uri*, it can be a regular expression and must match with the url's parameter of the same name when calling the webview.
You can access your client_id and may also customize the logo, color and text.

Find below a quick summary of all available parameters:

|Parameter                 |Description                                                        |
|--------------------------|-------------------------------------------------------------------|
|client_id (required)      |The client id of your application                                  |
|redirect_uri (required)   |The address where the user shoud be redirected after sharing his credentials. The address must match the redirection URI set for the client |
|state (optional)          |An arbitrary string sent back to you for control                   |
|types (optional)          |The type of connectors (**banks** or **providers**)                |
|connectors (optional)     |Given a connector id, it will prefill the form and skip step 1     |
|#{user_token} (optional)  |If provided identify an existing user. This parameter has to be the last and preceded with a '#'|

You can use our libraries to generate this URL.
It is also possible to get the HTML code of the button with the libraries, the render is as below:
![Share your accounts button](https://demo.biapi.pro/2.0/auth/share/button_icon.png)

When the user confirms the share of his accounts with you, he is redirected to the callback URL you have defined. This one receives the following parameters:

|Parameter            |Description                                                                                      |
|---------------------|-------------------------------------------------------------------------------------------------|
|code                 |A temporary token allowing you to get the **access_token**                                       |
|state                |The same string passed when redirecting to the webview                                           |
|error                |In case of error, the value will be **access_denied**, meaning the user has canceled the process |

Eventually, to get the **access_token** from the temporarily code which has been transmitted to you, you must call the API:
```python
  try:
      if client.handleCallback(received_params):
          # Keep the token associated with the user
          # you can't get it twice
          user.session['access_token'] = client.access_token
  except client.StateInvalid:
      # error if 'state' param provided to handleCallback doesn't match
  except client.AuthFailed:
      # There may be an error in the query (look for the message),
      # Or return code is 'access_denied', the user has stopped the process
 ```
 You can now associate this **access_token** to your user, and use it in your next calls to the API with the **Authorization** header.

 **IMPORTANT**
It is important that your users are able to go back on the webview authenticated so that they can add or remove accounts, and mostly update their credentials when needed.

To do this, provide the user a link to access the webview, this time with the following URL:
`https://demo.biapi.pro/2.0/auth/webview/accounts?client_id=<your_client_id>&redirect_uri=<your_uri>#<user_token>`

Also, please note we recommend not to expose the user's access_token.
You can request the API to get a new temporary token (lifetime of 30mn) for the user and use this one instead.
```http
GET /2.0/auth/token/code
Authorization: Bearer <token>
```
The token can alternatively be made with the library, by generating the URL with the following manner:
`  print '<a href="%s">Edit your accounts</a>' % client.get_settings_url()`
After the modification, the user can leave the webview and go back on the callback URL. It will not be necessary to do anything (the access_token won't be changed).

Below is a quick reminder of the Webview urls:
|Path                                                            |Description of the page                                         |
|----------------------------------------------------------------|----------------------------------------------------------------|
|/auth/webview/connect/select?<query parameters>#<user token>    |Form to add a new connection                                    |
|/auth/webview/accounts?<query parameters>#<user token>          |List user's accounts                                            |

#### Protocol
This section describes the protocol used to set bank and provider accounts of a user, in case you don't want to use the webview.

The idea is to call the following services client-side (with AJAX in case of a web application), to ensure the bank and providers credentials will not be sent to your servers.

1. /auth/init
```http
POST /auth/init
```
```json
{
   "auth_token" : "fBqjMZbYddebUGlkR445JKPA6pCoRaGb",
   "type" : "temporary",
   "expires_in" : 1800
}
```
This service creates a temporarily token, to use in the "Authorization" header in next calls to the API

The returned token has a life-time of 30 minutes, and should be transfered to the API then (cf Permanent Token), so that your server can get a permanent access_token.

It is possible to generate a permanent token immediately, by calling the service with the manage_token, or by supply parameters client_id and client_secret.

2. /banks or /providers
```http
GET /banks?expand=fields
Authorization: Bearer <token>
```
```json
{
   "banks" : [
      {
         "hidden" : false,
         "charged" : true,
         "name" : "American Express",
         "id" : 30,
         "fields" : [
            {
               "values" : [
                  {
                     "label" : "Particuliers/Professionnels",
                     "value" : "pp"
                  },
                  {
                     "value" : "ent",
                     "label" : "Entreprises"
                  }
               ],
               "label" : "Type de compte",
               "regex" : null,
               "name" : "website",
               "type" : "list"
            },
            {
               "type" : "password",
               "label" : "Code secret",
               "name" : "password",
               "regex" : "^[0-9]{6}$"
            }
         ],
      },
      ...
   ],
   "total" : 44,
}
```
You get a list of connectors, and all associated fields needed to build the form at step 3.
You can also use that list to show to your user, all available banks.

3. /users/me/connections
Make a POST request and supply the id_bank (ID of the chosen bank) or id_provider (ID of provider), and all requested fields as key/value parameters.
For example:
```http
POST /users/me/connections
Authorization: Bearer <token>
-F login=12345678
-F password=123456
-F id_bank=59
```
You can get the following return codes:

|Code           |Description                                                  |
|---------------|------------------------------------------------------------ |
|200            |The connection has succeed and has been created              |
|202            |It is necessary to provide complementary information. This occurs on the first connection on some kind of Boursorama accounts for example, where a SMS is sent to the customer. It is necessary to ask the user to fill fields requested in the fields, and do a POST again on /users/me/connections/ID, with the connection ID in id_connection. |
|400            |Unable to connect to the website, the field error in the JSON can be **websiteUnavailable** or **wrongpass**  |
|403            |Invalid token                                                |

4. Permanent token
If the user validates the share of his accounts, it is necessary to transform the temporary code to a permanent access_token (so that the user won't expire).

To do that, make a POST request on /auth/token/access with the following parameters:
|Parameter            |Description                                                     |
|---------------------|----------------------------------------------------------------|
|code                 |The temporarily token which will let you get the access_token   |
|client_id            |The ID of your client application                               |
|client_secret        |The secret of your client application                           |

```json
POST /auth/token/access

{
   "client_id" : 17473055,
   "client_secret" : "54tHJHjvodbANVzaRtcLzlHGXQiOgw80",
   "code" : "fBqjMZbYddebUGlkR445JKPA6pCoRaGb"
}
```
```http
HTTP/1.1 200 OK

{
   "access_token" : "7wBPuFfb1Hod82f1+KNa0AmrkIuQ3h1G",
   "token_type":"Bearer"
}
```

### Update accounts
Another important call is when a user wants to add/remove connections to banks or providers, or to change the password on one of them, it is advised to give him a temporarily code from the permanent access_token, with the following call (using the access_token as bearer):
```http
POST /auth/token/code
Authorization: Bearer <token>
```
```json
{
   "code" : "/JiDppWgbmc+5ztHIUJtHl0ynYfw682Z",
   "type" : "temporary",
   "expires_in" : 1800,
}
```
Its life-time is 30 minutes, and let the browser to list connections and accounts, via `GET /users/me/connections?expand=accounts` for example.

 To update the password of a connection, you can do a POST on the *connection* resource, with the field *password* in the data. The new credentials are checked to make sure they are valid, and the return codes are the same as when adding a connection.

## Getting the data (Webhooks)
You have created your users and their connections, now it's time to get the data.
There are 2 ways to retrieve it, the 2 can be complementary:
- make regular calls to the API
- use the webhooks (recommended)

### Manual Synchronization
It is possible to do a manual synchronization of a user. We recommend to use this method in case the user wants fresh data after logging in.

To trigger the synchronization, call the API as below:
`PUT /users/ID/connections`
The following call is blocking until the synchronization is terminated.

Even if it is not recommended, it's possible to fetch synchronously new data. To do that, you can use the *expand* parameter:
` /users/ID/connections?expand=accounts[transactions,investments[type]],subscriptions`
```json
{
   "connections" : [
      {
         "accounts" : [
            {
               "balance" : 7481.01,
               "currency" : {
                  "symbol" : "€",
                  "id" : "EUR",
                  "prefix" : false
               },
               "deleted" : null,
               "display" : true,
               "formatted_balance" : "7 481,01 €",
               "iban" : "FR76131048379405300290000016",
               "id" : 17,
               "id_connection" : 7,
               "investments" : [
                  {
                     "code" : "FR0010330902",
                     "description" : "",
                     "diff" : -67.86,
                     "id" : 55,
                     "id_account" : 19,
                     "id_type" : 1,
                     "label" : "Agressor PEA",
                     "portfolio_share" : 0.48,
                     "prev_diff" : 2019.57,
                     "quantity" : 7.338,
                     "type" : {
                        "color" : "AABBCC",
                        "id" : 1,
                        "name" : "Fonds action"
                     },
                     "unitprice" : 488.98,
                     "unitvalue" : 479.73,
                     "valuation" : 3520.28
                  }
               ],
               "last_update" : "2015-07-04 15:17:30",
               "name" : "Compte chèque",
               "number" : "3002900000",
               "transactions" : [
                  {
                     "active" : true,
                     "application_date" : "2015-06-17",
                     "coming" : false,
                     "comment" : null,
                     "commission" : null,
                     "country" : null,
                     "date" : "2015-06-18",
                     "date_scraped" : "2015-07-04 15:17:30",
                     "deleted" : null,
                     "documents_count" : 0,
                     "formatted_value" : "-16,22 €",
                     "id" : 309,
                     "id_account" : 17,
                     "id_category" : 9998,
                     "id_cluster" : null,
                     "last_update" : "2015-07-04 15:17:30",
                     "new" : true,
                     "original_currency" : null,
                     "original_value" : null,
                     "original_wording" : "FACTURE CB HALL'S BEER",
                     "rdate" : "2015-06-17",
                     "simplified_wording" : "HALL'S BEER",
                     "state" : "parsed",
                     "stemmed_wording" : "HALL'S BEER",
                     "type" : "card",
                     "value" : -16.22,
                     "wording" : "HALL'S BEER"
                  }
               ],
               "type" : "checking"
            }
         ],
         "error" : null,
         "expire" : null,
         "id" : 7,
         "id_user" : 7,
         "id_bank" : 41,
         "last_update" : "2015-07-04 15:17:31"
      }
   ],
   "total" : 1,
}
```

### Background synchronizations & Webhooks
Webhooks are callbacks sent to your server, when an event is triggered during a synchronization.
Synchronizations are automatic, the frequency can be set using the configuration key `autosync.frequency`.
Using webhooks allows you to get the most up-to-date data of your users, after each synchronization.

The automatic synchronization makes it possible to recover new bank entries, or new invoices, at a given frequency.
You have the possibility to add webhooks on several events, and choose to receive each one on a distinct URL.
To see the list of available webhooks you can call the endpoint hereunder:
```
curl -X GET \
  https://demo.biapi.pro/2.0/webhooks_events \
  -H 'Authorization: Bearer <token>'
```

The background synchronizations for each user are independent, and their plannings are spread over the day so that they do not overload any website.

Once the synchronization of a user is over, a POST request is sent on the callback URL you have defined, including all webhook data.
A typical json sent to your server is as below:
```http
POST /callback HTTP/1.1
Host: example.org
Content-Length: 959
Accept-Encoding: gzip, deflate, compress
Accept: */*
User-Agent: Budgea API/2.0
Content-Type: application/json; charset=utf-8
Authorization: Bearer sl/wuqgD2eOo+4Zf9FjvAz3YJgU+JKsJ

{
   "connections" : [
      {
         "accounts" : [
            {
               "balance" : 7481.01,
               "currency" : {
                  "symbol" : "€",
                  "id" : "EUR",
                  "prefix" : false
               },
               "deleted" : null,
               "display" : true,
               "formatted_balance" : "7 481,01 €",
               "iban" : "FR76131048379405300290000016",
               "id" : 17,
               "id_connection" : 7,
               "investments" : [
                  {
                     "code" : "FR0010330902",
                     "description" : "",
                     "diff" : -67.86,
                     "id" : 55,
                     "id_account" : 19,
                     "id_type" : 1,
                     "label" : "Agressor PEA",
                     "portfolio_share" : 0.48,
                     "prev_diff" : 2019.57,
                     "quantity" : 7.338,
                     "type" : {
                        "color" : "AABBCC",
                        "id" : 1,
                        "name" : "Fonds action"
                     },
                     "unitprice" : 488.98,
                     "unitvalue" : 479.73,
                     "valuation" : 3520.28
                  }
               ],
               "last_update" : "2015-07-04 15:17:30",
               "name" : "Compte chèque",
               "number" : "3002900000",
               "transactions" : [
                  {
                     "active" : true,
                     "application_date" : "2015-06-17",
                     "coming" : false,
                     "comment" : null,
                     "commission" : null,
                     "country" : null,
                     "date" : "2015-06-18",
                     "date_scraped" : "2015-07-04 15:17:30",
                     "deleted" : null,
                     "documents_count" : 0,
                     "formatted_value" : "-16,22 €",
                     "id" : 309,
                     "id_account" : 17,
                     "id_category" : 9998,
                     "id_cluster" : null,
                     "last_update" : "2015-07-04 15:17:30",
                     "new" : true,
                     "original_currency" : null,
                     "original_value" : null,
                     "original_wording" : "FACTURE CB HALL'S BEER",
                     "rdate" : "2015-06-17",
                     "simplified_wording" : "HALL'S BEER",
                     "state" : "parsed",
                     "stemmed_wording" : "HALL'S BEER",
                     "type" : "card",
                     "value" : -16.22,
                     "wording" : "HALL'S BEER"
                  }
               ],
               "type" : "checking"
            }
         ],
         "bank" : {
            "id_weboob" : "ing",
            "charged" : true,
            "name" : "ING Direct",
            "id" : 7,
            "hidden" : false
         },
         "error" : null,
         "expire" : null,
         "id" : 7,
         "id_user" : 7,
         "id_bank" : 41,
         "last_update" : "2015-07-04 15:17:31"
      }
   ],
   "total" : 1,
   "user" : {
      "signin" : "2015-07-04 15:17:29",
      "id" : 7,
      "platform" : "sharedAccess"
   }
}
```
The authentication on the callback is made with the access_token of the user (which is a shared secret between your server and the Budgea API).

When you are in production, it is needed to define a HTTPS URL using a valid certificate, delivered by a recognized authority. If this is not the case, you can contact us to add your CA (Certificate Authority) to our PKI (Public Key Infrastructure).

Important: it is necessary to send back a HTTP 200 code, without what we consider that data is not correctly taken into account on your system, and it will be sent again at the next user synchronization.

## Guidelines for production
Now you should have integrated the API inside your application. Make sure your Webhooks URLs are in HTTPS, if so you can enable the production state of the API.

To make things great, here are some good practices, please check you have respected them:
- You have provided to your users a way to configure their accounts
- You have provided to your users a way to change their account passwords
- You consider the **error** field of Connections, to alert the user in case the state is **wrongpass**
- You map IDs of Accounts, Subscriptions, Transactions and Documents in your application, to be sure to correctly match them
- When the deleted field is set on a bank transaction, you delete it in your database
- You don't loop on all users to launch synchronizations, this might saturate the service

If you have questions about above points, please contact us. Otherwise, you can put into production!

### Going further
If you want to raise the bar for your app and add features such as the ability to do transfers, get invoices, aggregate patrimony and more, please refer to the sections below.
We'll discuss complementary APIs building upon the aggregation, allowing for the best of financial apps.

## Budgea API Pay
This API allows for the emition of transfers between the aggregated accounts.
Just like the simple aggregation, BI provides a webview or a protocol to follow, to implement this feature.

### API pay protocol
This section describes how the transfer and recipient protocol work, in case you don't want to integrate the webview.
The idea is to do following calls client side (with AJAX in case of a web application), so that the interaction with the Budgea API is transparent.

#### Executing a transfer
1. /auth/token/code
If you do calls client side, get a new temporary code for the user, from the access_token. This will prevent security issues.
```
curl -X POST \
  https://demo.biapi.pro/2.0/auth/token/code \
  -H 'Authorization: Bearer <token>'
```
```json
{
   "code": "/JiDppWgbmc+5ztHIUJtHl0ynYfw682Z",
   "type": "temporary",
   "expires_in": 1800
}
```
The returned token has a life-time of 30 minutes.

2. /users/me/accounts?able_to_transfer=1
List all the accounts that can do transfers. Authenticate the call with the code you got at step 1.
```
curl -X GET \
  https://demo.biapi.pro/2.0/users/me/accounts?able_to_transfer=1 \
  -H 'Authorization: Bearer /JiDppWgbmc+5ztHIUJtHl0ynYfw682Z'
```
```json
{
  "accounts" : [
      {
         "display" : true,
         "balance" : 2893.36,
         "id_type" : 2,
         "number" : "****1572",
         "type" : "checking",
         "deleted" : null,
         "bic" : "BNPAFRPPXXX",
         "bookmarked" : false,
         "coming" : -2702.74,
         "id_user" : 1,
         "original_name" : "Compte de chèques",
         "currency" : {
            "symbol" : "€",
            "id" : "EUR",
            "prefix" : false
         },
         "name" : "lol",
         "iban" : "FR7630004012550000041157244",
         "last_update" : "2016-12-28 12:31:04",
         "id" : 723,
         "formatted_balance" : "2893,36 €",
         "able_to_transfer" : true,
         "id_connection" : 202
      }
   ],
   "total" : 1
}
```

3. /users/me/accounts/ID/recipients
List all available recipients for a given account.
```
curl -X GET \
  https://demo.biapi.pro/2.0/users/me/accounts/723/recipients?limit=1 \
  -H 'Authorization: Bearer /JiDppWgbmc+5ztHIUJtHl0ynYfw682Z'
```
```json
{
  "total" : 27,
   "recipients" : [
      {
         "bank_name" : "BNP PARIBAS",
         "bic" : "BNPAFRPPXXX",
         "category" : "Interne",
         "deleted" : null,
         "enabled_at" : "2016-10-31 18:52:53",
         "expire" : null,
         "iban" : "FR7630004012550003027641744",
         "id" : 1,
         "id_account" : 1,
         "id_target_account" : 2,
         "label" : "Livret A",
         "last_update" : "2016-12-05 12:07:24",
         "time_scraped" : "2016-10-31 18:52:54",
         "webid" : "2741588268268091098819849694548441184167285851255682796371"
      }
   ]
}
```

4. /users/me/accounts/ID/recipients/ID/transfers
Create the transfer
```
curl -X POST \
  https://demo.biapi.pro/2.0/users/me/accounts/1/recipients/1/transfers \
  -H 'Authorization: Bearer /JiDppWgbmc+5ztHIUJtHl0ynYfw682Z' \
  -F amount=10, \
  -F label="Test virement", \
  -F exec_date="2018-09-12" // optional
```
```json
{
   "account_iban" : "FR7630004012550000041157244",
   "amount" : 10,
   "currency" : {
      "id" : "EUR",
      "prefix" : false,
      "symbol" : "€"
   },
   "exec_date" : "2018-09-12",
   "fees" : null
   "formatted_amount" : "10,00 €",
   "id" : 22,
   "id_account" : 1,,
   "id_recipient" : 1,
   "label" : "Test virement",
   "recipient_iban" : "FR7630004012550003027641744",
   "register_date" : "2018-09-12 10:34:59",
   "state" : "created",
   "webid" : null
}
```

5. /users/me/transfers/ID
Execute the transfer
```
curl -X POST \
  https://demo.biapi.pro/2.0/users/me/transfers/22 \
  -H 'Authorization: Bearer /JiDppWgbmc+5ztHIUJtHl0ynYfw682Z' \
  -F validated=true
```
```json
{
   "account_iban" : "FR7630004012550000041157244",
   "amount" : 10,
   "currency" : {
      "id" : "EUR",
      "prefix" : false,
      "symbol" : "€"
   },
   "exec_date" : "2016-12-19",
   "fees" : null,
   "fields" : [
      {
         "label" : "Code secret BNP Paribas",
         "type" : "password",
         "regex" : "^[0-9]{6}$",
         "name" : "password"
      }
   ],
   "formatted_amount" : "10,00 €",
   "id" : 22,
   "id_account" : 1,
   "id_recipient" : 1,
   "label" : "Test virement",
   "recipient_iban" : "FR7630004012550003027641744",
   "register_date" : "2016-12-19 10:34:59",
   "state" : "created",
   "webid" : null
}
```
Here, an authentication step asks user to enter his bank password. The transfer can be validated with:

```
curl -X POST \
  https://demo.biapi.pro/2.0/users/me/transfers/22 \
  -H 'Authorization: Bearer /JiDppWgbmc+5ztHIUJtHl0ynYfw682Z' \
  -F validated=true \
  -F password="123456"
```
```json
{
   "account_iban" : "FR7630004012550000041157244",
   "currency" : {
      "id" : "EUR",
      "prefix" : false,
      "symbol" : "€"
   },
   "amount" : 10,
   "exec_date" : "2016-12-19",
   "fees" : 0,
   "formatted_amount" : "10,00 €",
   "id" : 22,
   "id_account" : 1,
   "id_recipient" : 1,
   "label" : "Test virement",
   "recipient_iban" : "FR7630004012550003027641744",
   "register_date" : "2016-12-19 10:34:59",
   "state" : "pending",
   "webid" : "ZZ10C4FKSNP05TK95"
}
```
The field state is changed to *pending*, telling that the transfer has been correctly executed on the bank. A connection synchronization is then launched, to find the bank transaction in the movements history. In this case, the transfer state will be changed to *done*.

#### Adding a recipient
1. /auth/token/code
Get a temporary token for the user. Same procedure than step 1 for a transfer.

2. /users/me/accounts?able_to_transfer=1
List accounts allowing transfers. Same procedure than step 2 for a transfer.

3. /users/me/accounts/ID/recipients/
Add a new recipient.
```
curl -X POST \
  https://demo.biapi.pro/2.0/users/me/accounts/1/recipients \
  -H 'Authorization: Bearer /JiDppWgbmc+5ztHIUJtHl0ynYfw682Z' \
  -F iban=FR7613048379405300290000355 \
  -F label="Papa", \
  -F category="Famille" // optional
```
```json
{
   "bank_name" : "BNP PARIBAS",
   "bic" : "BNPAFRPPXXX",
   "category" : "Famille",
   "deleted" : null,
   "enabled_at" : null,
   "expire" : "2017-04-29 16:56:20",
   "fields" : [
      {
         "label" : "Veuillez entrer le code reçu par SMS",
         "type" : "password",
         "regex" : "^[0-9]{6}$",
         "name" : "sms"
      }
   ],
   "iban" : "FR7613048379405300290000355",
   "id" : 2,
   "id_account" : 1,
   "id_target_account" : null,
   "label" : "Papa",
   "last_update" : "2017-04-29 16:26:20",
   "time_scraped" : null,
   "webid" : null
}
```
It is necessary to post on the object Recipient with the requested fields (here sms), until the add is validated:
```
curl -X POST \
  https://demo.biapi.pro/2.0/users/me/accounts/1/recipients/2 \
  -H 'Authorization: Bearer /JiDppWgbmc+5ztHIUJtHl0ynYfw682Z' \
  -F sms="123456"
```
```json
{
   "bank_name" : "BNP PARIBAS",
   "bic" : "BNPAFRPPXXX",
   "category" : "Famille",
   "deleted" : null,
   "enabled_at" : "2017-05-01 00:00:00",
   "expire" : null,
   "iban" : "FR7613048379405300290000355",
   "id" : 2,
   "id_account" : 1,
   "id_target_account" : null,
   "label" : "Papa",
   "last_update" : "2017-04-29 16:26:20",
   "time_scraped" : null,
   "webid" : "2741588268268091098819849694548441184167285851255682796371"
}
```
If the field enabled_at is in the future, it means that it isn't possible yet to execute a transfer, as the bank requires to wait a validation period.

### API Pay Webview
This section describes how to integrate the webview of the Budgea Pay API inside your application, to let your users do transfers to their recipients.

#### User redirection
To redirect the user to the webview, it is necessary to build a URI authenticated with a temporary token.
This can be done from our library, or by calling the endpoint `/auth/token/code` (see the protocol section for an example).
If the parameter **redirect_uri** is supplied, the user will be redirected to that page once the transfer is done.

#### List of pages
Here are a list a pages you may call to redirect your user directly on a page of the process:
|Path                                 |Description of the page                                                           |
|-------------------------------------|----------------------------------------------------------------------------------|
|/transfers                           |List Transfers                                                                    |
|/transfers/accounts                  |List emitter accounts                                                             |
|/transfers/accounts/id/recipients    |List recipients                                                                   |
|/transfers/accounts/id/recipients/id |Initialization of a transfer between the account and the recipient                |
|/transfers/id                        |Detail of a given transfer                                                        |

Base URLs:

* <a href="//demo.biapi.pro/2.0/">//demo.biapi.pro/2.0/</a>

<h1 id="budgea-api-documentation-administration">Administration</h1>

## List clients

> Code samples

```http
GET /demo.biapi.pro/2.0/merchants HTTP/1.1

Accept: application/json

```

```python
import requests
headers = {
  'Accept': 'application/json'
}

r = requests.get('/demo.biapi.pro/2.0/merchants', params={

}, headers = headers)

print r.json()

```

`GET /merchants`

<h3 id="list-clients-parameters">Parameters</h3>

|Name|In|Type|Required|Description|
|---|---|---|---|---|
|expand|query|string|false|none|

> Example responses

> 200 Response

```json
{
  "clients": [
    {
      "id": 0,
      "name": "",
      "secret": "",
      "public_key": "",
      "private_key": "",
      "redirect_uri": "",
      "primary_color": "",
      "secondary_color": "",
      "pro": false,
      "description": "",
      "description_banks": "",
      "description_providers": "",
      "id_logo": 0
    }
  ]
}
```

<h3 id="list-clients-responses">Responses</h3>

|Status|Meaning|Description|Schema|
|---|---|---|---|
|200|[OK](https://tools.ietf.org/html/rfc7231#section-6.3.1)|clients|Inline|

<h3 id="list-clients-responseschema">Response Schema</h3>

Status Code **200**

|Name|Type|Required|Restrictions|Description|
|---|---|---|---|---|
|» clients|[[Client](#schemaclient)]|true|none|none|
|»» id|integer|true|none|none|
|»» name|string|true|none|none|
|»» secret|string|true|none|none|
|»» public_key|string|false|none|none|
|»» private_key|string|false|none|none|
|»» redirect_uri|string|true|none|none|
|»» primary_color|string|false|none|Primary color of client|
|»» secondary_color|string|false|none|Secondary color of client|
|»» pro|boolean|true|none|Should the client display the company manager page.|
|»» description|string|false|none|Text to display as a default description.|
|»» description_banks|string|false|none|Text to display as a description for banks.|
|»» description_providers|string|false|none|Text to display as a description for providers.|
|»» id_logo|integer|false|none|none|
|»» information|string|false|none|customizable information|

<aside class="success">
This operation does not require authentication
</aside>

## Create a client

> Code samples

```http
POST /demo.biapi.pro/2.0/clients HTTP/1.1

Content-Type: multipart/form-data
Accept: application/json

```

```python
import requests
headers = {
  'Content-Type': 'multipart/form-data',
  'Accept': 'application/json'
}

r = requests.post('/demo.biapi.pro/2.0/clients', params={

}, headers = headers)

print r.json()

```

`POST /clients`

> Body parameter

```yaml
generate_keys: true
name: string
redirect_uri: string
information: string

```

<h3 id="create-a-client-parameters">Parameters</h3>

|Name|In|Type|Required|Description|
|---|---|---|---|---|
|expand|query|string|false|none|
|body|body|object|false|none|
|» generate_keys|body|boolean|false|if True, generate a rsa pair of keys so the client can be used to generate jwt user tokens (default: False)|
|» name|body|string|false|name of client|
|» redirect_uri|body|string|false|redirect_uri|
|» information|body|string|false|custom information about the client|

> Example responses

> 200 Response

```json
{
  "id": 0,
  "name": "",
  "secret": "",
  "public_key": "",
  "private_key": "",
  "redirect_uri": "",
  "primary_color": "",
  "secondary_color": "",
  "pro": false,
  "description": "",
  "description_banks": "",
  "description_providers": "",
  "id_logo": 0
}
```

<h3 id="create-a-client-responses">Responses</h3>

|Status|Meaning|Description|Schema|
|---|---|---|---|
|200|[OK](https://tools.ietf.org/html/rfc7231#section-6.3.1)|Successful POST on Client resource|[Client](#schemaclient)|

<aside class="success">
This operation does not require authentication
</aside>

## Get information about a client

> Code samples

```http
GET /demo.biapi.pro/2.0/clients/{id_client} HTTP/1.1

Accept: application/json

```

```python
import requests
headers = {
  'Accept': 'application/json'
}

r = requests.get('/demo.biapi.pro/2.0/clients/{id_client}', params={

}, headers = headers)

print r.json()

```

`GET /clients/{id_client}`

If you use the manage_token or a configuration token, you will get also the client_secret<br><br>

<h3 id="get-information-about-a-client-parameters">Parameters</h3>

|Name|In|Type|Required|Description|
|---|---|---|---|---|
|id_client|path|integer|true|none|
|expand|query|string|false|none|

> Example responses

> 200 Response

```json
{
  "id": 0,
  "name": "",
  "secret": "",
  "public_key": "",
  "private_key": "",
  "redirect_uri": "",
  "primary_color": "",
  "secondary_color": "",
  "pro": false,
  "description": "",
  "description_banks": "",
  "description_providers": "",
  "id_logo": 0
}
```

<h3 id="get-information-about-a-client-responses">Responses</h3>

|Status|Meaning|Description|Schema|
|---|---|---|---|
|200|[OK](https://tools.ietf.org/html/rfc7231#section-6.3.1)|Successful GET on Client resource|[Client](#schemaclient)|

<aside class="success">
This operation does not require authentication
</aside>

## Update a client

> Code samples

```http
PUT /demo.biapi.pro/2.0/clients/{id_client} HTTP/1.1

Content-Type: multipart/form-data
Accept: application/json

```

```python
import requests
headers = {
  'Content-Type': 'multipart/form-data',
  'Accept': 'application/json'
}

r = requests.put('/demo.biapi.pro/2.0/clients/{id_client}', params={

}, headers = headers)

print r.json()

```

`PUT /clients/{id_client}`

> Body parameter

```yaml
generate_keys: true
name: string
secret: true
redirect_uri: string
primary_color: string
secondary_color: string
description: string
description_banks: string
description_providers: string
pro: true
information: string
update_information: true

```

<h3 id="update-a-client-parameters">Parameters</h3>

|Name|In|Type|Required|Description|
|---|---|---|---|---|
|id_client|path|integer|true|none|
|expand|query|string|false|none|
|body|body|object|false|none|
|» generate_keys|body|boolean|false|set a rsa key pair for the client, which make it possible to generate a jwt user token using this client. No effect if the client already has a set of keys(default: False)|
|» name|body|string|false|name of client|
|» secret|body|boolean|false|reset the secret|
|» redirect_uri|body|string|false|redirect_uri|
|» primary_color|body|string|false|hexadecimal code of the client primary color (e.g F45B9A)|
|» secondary_color|body|string|false|hexadecimal code of the client secondary color (e.g F45B9A)|
|» description|body|string|false|text to display as a default description|
|» description_banks|body|string|false|text to display as a description for banks|
|» description_providers|body|string|false|text to display as a description for providers|
|» pro|body|boolean|false|Wether the client should display the company manager page|
|» information|body|string|false|custom information about the client|
|» update_information|body|boolean|false|update the custom information about the client instead of replacing the existing one (default: True)|

> Example responses

> 200 Response

```json
{
  "id": 0,
  "name": "",
  "secret": "",
  "public_key": "",
  "private_key": "",
  "redirect_uri": "",
  "primary_color": "",
  "secondary_color": "",
  "pro": false,
  "description": "",
  "description_banks": "",
  "description_providers": "",
  "id_logo": 0
}
```

<h3 id="update-a-client-responses">Responses</h3>

|Status|Meaning|Description|Schema|
|---|---|---|---|
|200|[OK](https://tools.ietf.org/html/rfc7231#section-6.3.1)|Successful PUT on Client resource|[Client](#schemaclient)|

<aside class="success">
This operation does not require authentication
</aside>

## Delete a client

> Code samples

```http
DELETE /demo.biapi.pro/2.0/clients/{id_client} HTTP/1.1

Accept: application/json

```

```python
import requests
headers = {
  'Accept': 'application/json'
}

r = requests.delete('/demo.biapi.pro/2.0/clients/{id_client}', params={

}, headers = headers)

print r.json()

```

`DELETE /clients/{id_client}`

<h3 id="delete-a-client-parameters">Parameters</h3>

|Name|In|Type|Required|Description|
|---|---|---|---|---|
|id_client|path|integer|true|none|
|expand|query|string|false|none|

> Example responses

> 200 Response

```json
{
  "id": 0,
  "name": "",
  "secret": "",
  "public_key": "",
  "private_key": "",
  "redirect_uri": "",
  "primary_color": "",
  "secondary_color": "",
  "pro": false,
  "description": "",
  "description_banks": "",
  "description_providers": "",
  "id_logo": 0
}
```

<h3 id="delete-a-client-responses">Responses</h3>

|Status|Meaning|Description|Schema|
|---|---|---|---|
|200|[OK](https://tools.ietf.org/html/rfc7231#section-6.3.1)|Successful DELETE on Client resource|[Client](#schemaclient)|

<aside class="success">
This operation does not require authentication
</aside>

## Update the client logo

> Code samples

```http
POST /demo.biapi.pro/2.0/merchants/{id_client}/logo HTTP/1.1

Accept: application/json

```

```python
import requests
headers = {
  'Accept': 'application/json'
}

r = requests.post('/demo.biapi.pro/2.0/merchants/{id_client}/logo', params={

}, headers = headers)

print r.json()

```

`POST /merchants/{id_client}/logo`

<h3 id="update-the-client-logo-parameters">Parameters</h3>

|Name|In|Type|Required|Description|
|---|---|---|---|---|
|id_client|path|integer|true|none|
|expand|query|string|false|none|

> Example responses

> 200 Response

```json
{
  "id": 0,
  "content_type": "",
  "filename": "",
  "file_size": 0
}
```

<h3 id="update-the-client-logo-responses">Responses</h3>

|Status|Meaning|Description|Schema|
|---|---|---|---|
|200|[OK](https://tools.ietf.org/html/rfc7231#section-6.3.1)|Successful POST on File resource|[File](#schemafile)|

<aside class="success">
This operation does not require authentication
</aside>

## Delete the client logo

> Code samples

```http
DELETE /demo.biapi.pro/2.0/merchants/{id_client}/logo HTTP/1.1

Accept: application/json

```

```python
import requests
headers = {
  'Accept': 'application/json'
}

r = requests.delete('/demo.biapi.pro/2.0/merchants/{id_client}/logo', params={

}, headers = headers)

print r.json()

```

`DELETE /merchants/{id_client}/logo`

<h3 id="delete-the-client-logo-parameters">Parameters</h3>

|Name|In|Type|Required|Description|
|---|---|---|---|---|
|id_client|path|integer|true|none|
|expand|query|string|false|none|

> Example responses

> 200 Response

```json
{
  "id": 0,
  "content_type": "",
  "filename": "",
  "file_size": 0
}
```

<h3 id="delete-the-client-logo-responses">Responses</h3>

|Status|Meaning|Description|Schema|
|---|---|---|---|
|200|[OK](https://tools.ietf.org/html/rfc7231#section-6.3.1)|Successful DELETE on File resource|[File](#schemafile)|

<aside class="success">
This operation does not require authentication
</aside>

## Get configuration of the API.

> Code samples

```http
GET /demo.biapi.pro/2.0/config HTTP/1.1

```

```python
import requests

r = requests.get('/demo.biapi.pro/2.0/config', params={

)

print r.json()

```

`GET /config`

<br><br>

<h3 id="get-configuration-of-the-api.-parameters">Parameters</h3>

|Name|In|Type|Required|Description|
|---|---|---|---|---|
|search|query|string|false|limit the results to keys matching the given value|

<h3 id="get-configuration-of-the-api.-responses">Responses</h3>

|Status|Meaning|Description|Schema|
|---|---|---|---|
|200|[OK](https://tools.ietf.org/html/rfc7231#section-6.3.1)|standard HTTP response|None|

<aside class="success">
This operation does not require authentication
</aside>

## Set a configuration value on the API.

> Code samples

```http
POST /demo.biapi.pro/2.0/config HTTP/1.1

Accept: application/json

```

```python
import requests
headers = {
  'Accept': 'application/json'
}

r = requests.post('/demo.biapi.pro/2.0/config', params={

}, headers = headers)

print r.json()

```

`POST /config`

Request: { "connection.disable_new": "0", "search": "connection.disable_new" }<br><br>

<h3 id="set-a-configuration-value-on-the-api.-parameters">Parameters</h3>

|Name|In|Type|Required|Description|
|---|---|---|---|---|
|search|query|string|false|limit the results to keys matching the given value|

> Example responses

> 200 Response

```json
{}
```

> OK

<h3 id="set-a-configuration-value-on-the-api.-responses">Responses</h3>

|Status|Meaning|Description|Schema|
|---|---|---|---|
|200|[OK](https://tools.ietf.org/html/rfc7231#section-6.3.1)|OK|Inline|

<h3 id="set-a-configuration-value-on-the-api.-responseschema">Response Schema</h3>

<aside class="success">
This operation does not require authentication
</aside>

## Create a merchant. Needs a user identified in bearer to be used

> Code samples

```http
POST /demo.biapi.pro/2.0/merchants HTTP/1.1

Content-Type: multipart/form-data
Accept: application/json

```

```python
import requests
headers = {
  'Content-Type': 'multipart/form-data',
  'Accept': 'application/json'
}

r = requests.post('/demo.biapi.pro/2.0/merchants', params={

}, headers = headers)

print r.json()

```

`POST /merchants`

> Body parameter

```yaml
name: string
redirect_uri: string
iban: string

```

<h3 id="create-a-merchant.-needs-a-user-identified-in-bearer-to-be-used-parameters">Parameters</h3>

|Name|In|Type|Required|Description|
|---|---|---|---|---|
|expand|query|string|false|none|
|body|body|object|false|none|
|» name|body|string|true|name of merchant|
|» redirect_uri|body|string|true|regexp to check if given redirect_uri are authorized|
|» iban|body|string|true|payments initiated by this merchant will be done to this IBAN|

> Example responses

> 200 Response

```json
{
  "id": 0,
  "name": "",
  "secret": "",
  "public_key": "",
  "private_key": "",
  "redirect_uri": "",
  "primary_color": "",
  "secondary_color": "",
  "pro": false,
  "description": "",
  "description_banks": "",
  "description_providers": "",
  "id_logo": 0
}
```

<h3 id="create-a-merchant.-needs-a-user-identified-in-bearer-to-be-used-responses">Responses</h3>

|Status|Meaning|Description|Schema|
|---|---|---|---|
|200|[OK](https://tools.ietf.org/html/rfc7231#section-6.3.1)|Successful POST on Client resource|[Client](#schemaclient)|

<aside class="success">
This operation does not require authentication
</aside>

## get performances stats on this instance

> Code samples

```http
GET /demo.biapi.pro/2.0/monitoring HTTP/1.1

```

```python
import requests

r = requests.get('/demo.biapi.pro/2.0/monitoring', params={

)

print r.json()

```

`GET /monitoring`

<h3 id="get-performances-stats-on-this-instance-parameters">Parameters</h3>

|Name|In|Type|Required|Description|
|---|---|---|---|---|
|period|query|integer|false|number on days on which stats on synchronization have to be done per worker (Default: 1)|

<h3 id="get-performances-stats-on-this-instance-responses">Responses</h3>

|Status|Meaning|Description|Schema|
|---|---|---|---|
|200|[OK](https://tools.ietf.org/html/rfc7231#section-6.3.1)|standard HTTP response|None|

<aside class="success">
This operation does not require authentication
</aside>

## Test synchronization on a random connection.

> Code samples

```http
POST /demo.biapi.pro/2.0/test/webhooks HTTP/1.1

```

```python
import requests

r = requests.post('/demo.biapi.pro/2.0/test/webhooks', params={

)

print r.json()

```

`POST /test/webhooks`

It can be used to test receiving data on your webhooks.<br><br>

<h3 id="test-synchronization-on-a-random-connection.-responses">Responses</h3>

|Status|Meaning|Description|Schema|
|---|---|---|---|
|200|[OK](https://tools.ietf.org/html/rfc7231#section-6.3.1)|standard HTTP response|None|

<aside class="success">
This operation does not require authentication
</aside>

## Get webhooks

> Code samples

```http
GET /demo.biapi.pro/2.0/webhooks HTTP/1.1

Accept: application/json

```

```python
import requests
headers = {
  'Accept': 'application/json'
}

r = requests.get('/demo.biapi.pro/2.0/webhooks', params={

}, headers = headers)

print r.json()

```

`GET /webhooks`

<h3 id="get-webhooks-parameters">Parameters</h3>

|Name|In|Type|Required|Description|
|---|---|---|---|---|
|expand|query|string|false|none|

> Example responses

> 200 Response

```json
{
  "webhooks": [
    {
      "id": 0,
      "created": "2019-03-20 11:42:25.574358",
      "updated": "2019-03-20 11:42:25.574467",
      "deleted": "2019-03-20 11:42:25.574548",
      "id_service": 0,
      "id_user": 0,
      "id_event": 0,
      "url": ""
    }
  ]
}
```

<h3 id="get-webhooks-responses">Responses</h3>

|Status|Meaning|Description|Schema|
|---|---|---|---|
|200|[OK](https://tools.ietf.org/html/rfc7231#section-6.3.1)|webhooks|Inline|

<h3 id="get-webhooks-responseschema">Response Schema</h3>

Status Code **200**

|Name|Type|Required|Restrictions|Description|
|---|---|---|---|---|
|» webhooks|[[Webhook](#schemawebhook)]|true|none|none|
|»» id|integer|true|none|ID of the webhook|
|»» created|string(date-time)|true|none|Date of the webhook creation|
|»» updated|string(date-time)|true|none|Date of the webhook last update|
|»» deleted|string(date-time)|false|none|Date of the webhook deletion|
|»» id_service|integer|false|none|ID of the service|
|»» id_user|integer|false|none|ID of the emitter user|
|»» id_event|integer|false|none|ID of the webhook event|
|»» url|string|false|none|URL of the webhook|

<aside class="success">
This operation does not require authentication
</aside>

## Adds a new webhook

> Code samples

```http
POST /demo.biapi.pro/2.0/webhooks HTTP/1.1

Content-Type: multipart/form-data
Accept: application/json

```

```python
import requests
headers = {
  'Content-Type': 'multipart/form-data',
  'Accept': 'application/json'
}

r = requests.post('/demo.biapi.pro/2.0/webhooks', params={

}, headers = headers)

print r.json()

```

`POST /webhooks`

> Body parameter

```yaml
id_user: 0
id_service: 0
url: 0
event: string
params: string

```

<h3 id="adds-a-new-webhook-parameters">Parameters</h3>

|Name|In|Type|Required|Description|
|---|---|---|---|---|
|expand|query|string|false|none|
|body|body|object|false|none|
|» id_user|body|integer|false|The user ID to associate with the webhook|
|» id_service|body|integer|false|The service ID to associate with the webhook|
|» url|body|number(float)|false|The webhook callback url|
|» event|body|string|false|The webhook event|
|» params|body|string|false|The webhook parameters as an object with three keys: type, key and value|

> Example responses

> 200 Response

```json
{
  "id": 0,
  "created": "2019-03-20 11:42:25.574358",
  "updated": "2019-03-20 11:42:25.574467",
  "deleted": "2019-03-20 11:42:25.574548",
  "id_service": 0,
  "id_user": 0,
  "id_event": 0,
  "url": ""
}
```

<h3 id="adds-a-new-webhook-responses">Responses</h3>

|Status|Meaning|Description|Schema|
|---|---|---|---|
|200|[OK](https://tools.ietf.org/html/rfc7231#section-6.3.1)|Successful POST on Webhook resource|[Webhook](#schemawebhook)|

<aside class="success">
This operation does not require authentication
</aside>

## Deletes all webhooks

> Code samples

```http
DELETE /demo.biapi.pro/2.0/webhooks HTTP/1.1

Accept: application/json

```

```python
import requests
headers = {
  'Accept': 'application/json'
}

r = requests.delete('/demo.biapi.pro/2.0/webhooks', params={

}, headers = headers)

print r.json()

```

`DELETE /webhooks`

Updates the deleted field with the date of the deletion<br><br>

<h3 id="deletes-all-webhooks-parameters">Parameters</h3>

|Name|In|Type|Required|Description|
|---|---|---|---|---|
|expand|query|string|false|none|

> Example responses

> 200 Response

```json
{
  "id": 0,
  "created": "2019-03-20 11:42:25.574358",
  "updated": "2019-03-20 11:42:25.574467",
  "deleted": "2019-03-20 11:42:25.574548",
  "id_service": 0,
  "id_user": 0,
  "id_event": 0,
  "url": ""
}
```

<h3 id="deletes-all-webhooks-responses">Responses</h3>

|Status|Meaning|Description|Schema|
|---|---|---|---|
|200|[OK](https://tools.ietf.org/html/rfc7231#section-6.3.1)|Successful DELETE on Webhook resource|[Webhook](#schemawebhook)|

<aside class="success">
This operation does not require authentication
</aside>

## Updates a webhook

> Code samples

```http
PUT /demo.biapi.pro/2.0/webhooks/{id_webhook} HTTP/1.1

Content-Type: multipart/form-data
Accept: application/json

```

```python
import requests
headers = {
  'Content-Type': 'multipart/form-data',
  'Accept': 'application/json'
}

r = requests.put('/demo.biapi.pro/2.0/webhooks/{id_webhook}', params={

}, headers = headers)

print r.json()

```

`PUT /webhooks/{id_webhook}`

> Body parameter

```yaml
deleted: string
id_user: 0
id_service: 0
url: 0
event: string

```

<h3 id="updates-a-webhook-parameters">Parameters</h3>

|Name|In|Type|Required|Description|
|---|---|---|---|---|
|id_webhook|path|integer|true|none|
|expand|query|string|false|none|
|body|body|[postWebhooks_idWebhook_](#schemapostwebhooks_idwebhook_)|false|none|
|» deleted|body|string|false|a date to delete the webhook or 'null' to enable it|
|» id_user|body|integer|false|The user ID to associate with the webhook|
|» id_service|body|integer|false|The service ID to associate with the webhook|
|» url|body|number(float)|false|The webhook callback url|
|» event|body|string|false|The webhook event|

> Example responses

> 200 Response

```json
{
  "id": 0,
  "created": "2019-03-20 11:42:25.574358",
  "updated": "2019-03-20 11:42:25.574467",
  "deleted": "2019-03-20 11:42:25.574548",
  "id_service": 0,
  "id_user": 0,
  "id_event": 0,
  "url": ""
}
```

<h3 id="updates-a-webhook-responses">Responses</h3>

|Status|Meaning|Description|Schema|
|---|---|---|---|
|200|[OK](https://tools.ietf.org/html/rfc7231#section-6.3.1)|Successful PUT on Webhook resource|[Webhook](#schemawebhook)|

<aside class="success">
This operation does not require authentication
</aside>

## Deletes a webhook

> Code samples

```http
DELETE /demo.biapi.pro/2.0/webhooks/{id_webhook} HTTP/1.1

Accept: application/json

```

```python
import requests
headers = {
  'Accept': 'application/json'
}

r = requests.delete('/demo.biapi.pro/2.0/webhooks/{id_webhook}', params={

}, headers = headers)

print r.json()

```

`DELETE /webhooks/{id_webhook}`

Updates the deleted field with the date of the deletion<br><br>

<h3 id="deletes-a-webhook-parameters">Parameters</h3>

|Name|In|Type|Required|Description|
|---|---|---|---|---|
|id_webhook|path|integer|true|none|
|expand|query|string|false|none|

> Example responses

> 200 Response

```json
{
  "id": 0,
  "created": "2019-03-20 11:42:25.574358",
  "updated": "2019-03-20 11:42:25.574467",
  "deleted": "2019-03-20 11:42:25.574548",
  "id_service": 0,
  "id_user": 0,
  "id_event": 0,
  "url": ""
}
```

<h3 id="deletes-a-webhook-responses">Responses</h3>

|Status|Meaning|Description|Schema|
|---|---|---|---|
|200|[OK](https://tools.ietf.org/html/rfc7231#section-6.3.1)|Successful DELETE on Webhook resource|[Webhook](#schemawebhook)|

<aside class="success">
This operation does not require authentication
</aside>

## Get webhooks logs.

> Code samples

```http
GET /demo.biapi.pro/2.0/webhooks/{id_webhook}/logs HTTP/1.1

Accept: application/json

```

```python
import requests
headers = {
  'Accept': 'application/json'
}

r = requests.get('/demo.biapi.pro/2.0/webhooks/{id_webhook}/logs', params={

}, headers = headers)

print r.json()

```

`GET /webhooks/{id_webhook}/logs`

Get logs of the webhooks.<br><br>By default, it selects logs for the last month. You can use "min_date" and "max_date" to change boundary dates.<br><br>

<h3 id="get-webhooks-logs.-parameters">Parameters</h3>

|Name|In|Type|Required|Description|
|---|---|---|---|---|
|id_webhook|path|integer|true|none|
|limit|query|integer|false|limit number of results|
|offset|query|integer|false|offset of first result|
|min_date|query|string(date)|false|minimal (inclusive) date|
|max_date|query|string(date)|false|maximum (inclusive) date|
|expand|query|string|false|none|

> Example responses

> 200 Response

```json
{
  "webhooklogs": [
    {
      "id": 0,
      "id_webhook_data": 0,
      "id_service": 0,
      "id_user": 0,
      "timestamp": "2019-03-20 11:42:25.576657",
      "response_date": "2019-03-20 11:42:25.576761",
      "response_code": 0,
      "next_try": "2019-03-20 11:42:25.576909"
    }
  ]
}
```

<h3 id="get-webhooks-logs.-responses">Responses</h3>

|Status|Meaning|Description|Schema|
|---|---|---|---|
|200|[OK](https://tools.ietf.org/html/rfc7231#section-6.3.1)|webhooklogs|Inline|

<h3 id="get-webhooks-logs.-responseschema">Response Schema</h3>

Status Code **200**

|Name|Type|Required|Restrictions|Description|
|---|---|---|---|---|
|» webhooklogs|[[WebhookLog](#schemawebhooklog)]|true|none|none|
|»» id|integer|true|none|ID of the log|
|»» id_webhook_data|integer|false|none|ID of the webhook data|
|»» id_service|integer|false|none|ID of the service|
|»» id_user|integer|false|none|ID of the user|
|»» timestamp|string(date-time)|true|none|Timestamp when the hook was sent|
|»» response_date|string(date-time)|false|none|Timestamp of the reply to the hook|
|»» response_code|integer|false|none|Return code of the reply to the hook|
|»» next_try|string(date-time)|false|none|If the log is an error, do not retry to push before this timestamp|

<aside class="success">
This operation does not require authentication
</aside>

<h1 id="budgea-api-documentation-authentication">Authentication</h1>

## Generate a jwt manage token

> Code samples

```http
POST /demo.biapi.pro/2.0/admin/jwt HTTP/1.1

Content-Type: multipart/form-data
Accept: application/json

```

```python
import requests
headers = {
  'Content-Type': 'multipart/form-data',
  'Accept': 'application/json'
}

r = requests.post('/demo.biapi.pro/2.0/admin/jwt', params={

}, headers = headers)

print r.json()

```

`POST /admin/jwt`

This endpoint generates a new jwt manage token. It requires an admin manage token to be used<br><br>

> Body parameter

```yaml
scope: string
duration: 0

```

<h3 id="generate-a-jwt-manage-token-parameters">Parameters</h3>

|Name|In|Type|Required|Description|
|---|---|---|---|---|
|body|body|object|false|none|
|» scope|body|string|false|scope requested for the token (default: config)|
|» duration|body|integer|false|number of minute before the token expiration (0 for token that won't expire unless the client application is deleted) (default: 1)|

> Example responses

> 200 Response

```json
{}
```

> OK

<h3 id="generate-a-jwt-manage-token-responses">Responses</h3>

|Status|Meaning|Description|Schema|
|---|---|---|---|
|200|[OK](https://tools.ietf.org/html/rfc7231#section-6.3.1)|OK|Inline|

<h3 id="generate-a-jwt-manage-token-responseschema">Response Schema</h3>

<aside class="success">
This operation does not require authentication
</aside>

## Create a new anonymous user

> Code samples

```http
POST /demo.biapi.pro/2.0/auth/init HTTP/1.1

Content-Type: multipart/form-data
Accept: application/json

```

```python
import requests
headers = {
  'Content-Type': 'multipart/form-data',
  'Accept': 'application/json'
}

r = requests.post('/demo.biapi.pro/2.0/auth/init', params={

}, headers = headers)

print r.json()

```

`POST /auth/init`

This endpoint creates a new temporary token related to a new anonymous user.<br><br>It will expire 30 minutes after.<br><br>Note: if you supply client_id and client_secret, or if you call this endpoint with the manage_token, the token will be permanent.<br><br>

> Body parameter

```yaml
client_id: string
client_secret: string

```

<h3 id="create-a-new-anonymous-user-parameters">Parameters</h3>

|Name|In|Type|Required|Description|
|---|---|---|---|---|
|body|body|object|false|none|
|» client_id|body|string|false|ID of the client|
|» client_secret|body|string|false|secret of the client|

> Example responses

> 200 Response

```json
{
  "expires_in": 0,
  "auth_token": "string",
  "type": "string"
}
```

> OK

<h3 id="create-a-new-anonymous-user-responses">Responses</h3>

|Status|Meaning|Description|Schema|
|---|---|---|---|
|200|[OK](https://tools.ietf.org/html/rfc7231#section-6.3.1)|OK|Inline|

<h3 id="create-a-new-anonymous-user-responseschema">Response Schema</h3>

Status Code **200**

|Name|Type|Required|Restrictions|Description|
|---|---|---|---|---|
|» expires_in|integer|false|none|duration in seconds of the token validity|
|» auth_token|string|true|none|new token created for the new anonymous user|
|» type|string|true|none|type of the token|

<aside class="success">
This operation does not require authentication
</aside>

## Generate a user jwt token

> Code samples

```http
POST /demo.biapi.pro/2.0/auth/jwt HTTP/1.1

Content-Type: multipart/form-data
Accept: application/json

```

```python
import requests
headers = {
  'Content-Type': 'multipart/form-data',
  'Accept': 'application/json'
}

r = requests.post('/demo.biapi.pro/2.0/auth/jwt', params={

}, headers = headers)

print r.json()

```

`POST /auth/jwt`

This endpoint generates a new jwt token for the user. This token will last the time in minutes given by the config key auth.default_token_expire (permanent if this the parameter expire=False is given)<br><br>

> Body parameter

```yaml
client_id: string
client_secret: string
scope: string
id_user: 0
expire: true

```

<h3 id="generate-a-user-jwt-token-parameters">Parameters</h3>

|Name|In|Type|Required|Description|
|---|---|---|---|---|
|body|body|object|false|none|
|» client_id|body|string|false|id of the client|
|» client_secret|body|string|false|secret for the client|
|» scope|body|string|false|scope requested for the token|
|» id_user|body|integer|false|user for whom the token has to be generated. If not supplied, a user will be created|
|» expire|body|boolean|false|if set to True, the token will expire n minutes after its creation, n being the value of configuration key auth.default_token_expire (default: True)|

> Example responses

> 200 Response

```json
{
  "jwt_token": "string",
  "payload": {}
}
```

> OK

<h3 id="generate-a-user-jwt-token-responses">Responses</h3>

|Status|Meaning|Description|Schema|
|---|---|---|---|
|200|[OK](https://tools.ietf.org/html/rfc7231#section-6.3.1)|OK|Inline|

<h3 id="generate-a-user-jwt-token-responseschema">Response Schema</h3>

Status Code **200**

|Name|Type|Required|Restrictions|Description|
|---|---|---|---|---|
|» jwt_token|string|true|none|the jwt token|
|» payload|object|true|none|the payload contained in the jwt token|

<aside class="success">
This operation does not require authentication
</aside>

## Remove user access

> Code samples

```http
DELETE /demo.biapi.pro/2.0/auth/token HTTP/1.1

```

```python
import requests

r = requests.delete('/demo.biapi.pro/2.0/auth/token', params={

)

print r.json()

```

`DELETE /auth/token`

This endpoint removes the token in use.<br><br>

<h3 id="remove-user-access-responses">Responses</h3>

|Status|Meaning|Description|Schema|
|---|---|---|---|
|200|[OK](https://tools.ietf.org/html/rfc7231#section-6.3.1)|standard HTTP response|None|

<aside class="success">
This operation does not require authentication
</aside>

## Transform a temporary code to a access_token

> Code samples

```http
POST /demo.biapi.pro/2.0/auth/token/access HTTP/1.1

Content-Type: multipart/form-data
Accept: application/json

```

```python
import requests
headers = {
  'Content-Type': 'multipart/form-data',
  'Accept': 'application/json'
}

r = requests.post('/demo.biapi.pro/2.0/auth/token/access', params={

}, headers = headers)

print r.json()

```

`POST /auth/token/access`

In order to register a new user with the OAuth 2 process, the client has to call this endpoint to request a granted access_token with the received temporary code.<br><br>

> Body parameter

```yaml
grant_type: string
client_id: string
client_secret: string
code: string
redirect_uri: string

```

<h3 id="transform-a-temporary-code-to-a-access_token-parameters">Parameters</h3>

|Name|In|Type|Required|Description|
|---|---|---|---|---|
|body|body|object|false|none|
|» grant_type|body|string|false|default is "authorization_code"|
|» client_id|body|string|true|ID of the client|
|» client_secret|body|string|true|secret of the client|
|» code|body|string|true|user's temporary code|
|» redirect_uri|body|string|false|redirect uri used by user|

> Example responses

> 200 Response

```json
{
  "access_token": "string",
  "token_type": "string"
}
```

> OK

<h3 id="transform-a-temporary-code-to-a-access_token-responses">Responses</h3>

|Status|Meaning|Description|Schema|
|---|---|---|---|
|200|[OK](https://tools.ietf.org/html/rfc7231#section-6.3.1)|OK|Inline|

<h3 id="transform-a-temporary-code-to-a-access_token-responseschema">Response Schema</h3>

Status Code **200**

|Name|Type|Required|Restrictions|Description|
|---|---|---|---|---|
|» access_token|string|true|none|the access token transformed from the temporary one|
|» token_type|string|true|none|the access token type|

<aside class="success">
This operation does not require authentication
</aside>

## Generate a user temporary token

> Code samples

```http
GET /demo.biapi.pro/2.0/auth/token/code HTTP/1.1

Accept: application/json

```

```python
import requests
headers = {
  'Accept': 'application/json'
}

r = requests.get('/demo.biapi.pro/2.0/auth/token/code', params={

}, headers = headers)

print r.json()

```

`GET /auth/token/code`

This endpoint generates a new temporary token for the user.<br><br>In case the access_token is used by a trusted device, and you want to let another one (for example a web browser) access to user resources, use this service to create a token which will expire in 30 minutes.<br><br>

> Example responses

> 200 Response

```json
{
  "expires_in": 0,
  "code": "string",
  "type": {}
}
```

> OK

<h3 id="generate-a-user-temporary-token-responses">Responses</h3>

|Status|Meaning|Description|Schema|
|---|---|---|---|
|200|[OK](https://tools.ietf.org/html/rfc7231#section-6.3.1)|OK|Inline|

<h3 id="generate-a-user-temporary-token-responseschema">Response Schema</h3>

Status Code **200**

|Name|Type|Required|Restrictions|Description|
|---|---|---|---|---|
|» expires_in|integer|true|none|duration in seconds of the token validity|
|» code|string|true|none|the temporary token|
|» type|object|true|none|the token type|

<aside class="success">
This operation does not require authentication
</aside>

## Get the latest certificate of a type

> Code samples

```http
GET /demo.biapi.pro/2.0/certificate/{type} HTTP/1.1

Accept: application/json

```

```python
import requests
headers = {
  'Accept': 'application/json'
}

r = requests.get('/demo.biapi.pro/2.0/certificate/{type}', params={

}, headers = headers)

print r.json()

```

`GET /certificate/{type}`

<h3 id="get-the-latest-certificate-of-a-type-parameters">Parameters</h3>

|Name|In|Type|Required|Description|
|---|---|---|---|---|
|type|path|string|true|none|
|expand|query|string|false|none|

> Example responses

> 200 Response

```json
{
  "id": 0,
  "id_public_key_file": 0,
  "id_private_key_file": 0,
  "type": "",
  "created": "2019-03-20 11:42:25.604192"
}
```

<h3 id="get-the-latest-certificate-of-a-type-responses">Responses</h3>

|Status|Meaning|Description|Schema|
|---|---|---|---|
|200|[OK](https://tools.ietf.org/html/rfc7231#section-6.3.1)|Successful GET on Certificate resource|[Certificate](#schemacertificate)|

<aside class="success">
This operation does not require authentication
</aside>

## Delete the user

> Code samples

```http
DELETE /demo.biapi.pro/2.0/users/{id_user} HTTP/1.1

Accept: application/json

```

```python
import requests
headers = {
  'Accept': 'application/json'
}

r = requests.delete('/demo.biapi.pro/2.0/users/{id_user}', params={

}, headers = headers)

print r.json()

```

`DELETE /users/{id_user}`

This endpoint deletes the user.<br><br>

<h3 id="delete-the-user-parameters">Parameters</h3>

|Name|In|Type|Required|Description|
|---|---|---|---|---|
|id_user|path|string|true|Hint: you can use 'me' or 'all'|
|expand|query|string|false|none|

> Example responses

> 200 Response

```json
{
  "id": 0,
  "signin": "CURRENT_TIMESTAMP",
  "platform": ""
}
```

<h3 id="delete-the-user-responses">Responses</h3>

|Status|Meaning|Description|Schema|
|---|---|---|---|
|200|[OK](https://tools.ietf.org/html/rfc7231#section-6.3.1)|Successful DELETE on User resource|[User](#schemauser)|

<aside class="success">
This operation does not require authentication
</aside>

## Initialize a new OAuth2 proxy session.

> Code samples

```http
GET /demo.biapi.pro/2.0/webauth HTTP/1.1

Content-Type: multipart/form-data

```

```python
import requests
headers = {
  'Content-Type': 'multipart/form-data'
}

r = requests.get('/demo.biapi.pro/2.0/webauth', params={

}, headers = headers)

print r.json()

```

`GET /webauth`

> Body parameter

```yaml
id_connector: 0
client_id: 0
redirect_uri: string
state: string
fields: string
id_connection: 0

```

<h3 id="initialize-a-new-oauth2-proxy-session.-parameters">Parameters</h3>

|Name|In|Type|Required|Description|
|---|---|---|---|---|
|body|body|object|false|none|
|» id_connector|body|integer|false|ID of the connector|
|» client_id|body|integer|false|Client Application ID|
|» redirect_uri|body|string|false|Redirect URI|
|» state|body|string|false|Optional state|
|» fields|body|string|false|Optional fields|
|» id_connection|body|integer|false|Optional already existing connection to update|

<h3 id="initialize-a-new-oauth2-proxy-session.-responses">Responses</h3>

|Status|Meaning|Description|Schema|
|---|---|---|---|
|200|[OK](https://tools.ietf.org/html/rfc7231#section-6.3.1)|standard HTTP response|None|

<aside class="success">
This operation does not require authentication
</aside>

<h1 id="budgea-api-documentation-banks">Banks</h1>

## Get account types

> Code samples

```http
GET /demo.biapi.pro/2.0/users/{id_user}/account_types HTTP/1.1

Accept: application/json

```

```python
import requests
headers = {
  'Accept': 'application/json'
}

r = requests.get('/demo.biapi.pro/2.0/users/{id_user}/account_types', params={

}, headers = headers)

print r.json()

```

`GET /users/{id_user}/account_types`

<h3 id="get-account-types-parameters">Parameters</h3>

|Name|In|Type|Required|Description|
|---|---|---|---|---|
|id_user|path|string|true|Hint: you can use 'me' or 'all'|
|expand|query|string|false|none|

> Example responses

> 200 Response

```json
{
  "accounttypes": [
    {
      "id": 0,
      "name": "",
      "is_invest": false,
      "weboob_type_id": 0,
      "display_name_p": "",
      "display_name": "",
      "color": "",
      "id_parent": 0
    }
  ]
}
```

<h3 id="get-account-types-responses">Responses</h3>

|Status|Meaning|Description|Schema|
|---|---|---|---|
|200|[OK](https://tools.ietf.org/html/rfc7231#section-6.3.1)|accounttypes|Inline|

<h3 id="get-account-types-responseschema">Response Schema</h3>

Status Code **200**

|Name|Type|Required|Restrictions|Description|
|---|---|---|---|---|
|» accounttypes|[[AccountType](#schemaaccounttype)]|true|none|none|
|»» id|integer|true|none|ID of the account type|
|»» name|string|true|none|Name of the account type|
|»» is_invest|boolean|true|none|Is it an investment account|
|»» weboob_type_id|integer|true|none|Map to the weboob_type_id|
|»» display_name_p|string|true|none|Name to display in plurial|
|»» display_name|string|true|none|Name to display in singular|
|»» color|string|false|none|Color of the account type (hexdecimal)|
|»» id_parent|integer|false|none|Id of the parent type|

<aside class="success">
This operation does not require authentication
</aside>

## Get an account type

> Code samples

```http
GET /demo.biapi.pro/2.0/users/{id_user}/account_types/{id_account_type} HTTP/1.1

Accept: application/json

```

```python
import requests
headers = {
  'Accept': 'application/json'
}

r = requests.get('/demo.biapi.pro/2.0/users/{id_user}/account_types/{id_account_type}', params={

}, headers = headers)

print r.json()

```

`GET /users/{id_user}/account_types/{id_account_type}`

<h3 id="get-an-account-type-parameters">Parameters</h3>

|Name|In|Type|Required|Description|
|---|---|---|---|---|
|id_user|path|string|true|Hint: you can use 'me' or 'all'|
|id_account_type|path|integer|true|none|
|expand|query|string|false|none|

> Example responses

> 200 Response

```json
{
  "id": 0,
  "name": "",
  "is_invest": false,
  "weboob_type_id": 0,
  "display_name_p": "",
  "display_name": "",
  "color": "",
  "id_parent": 0
}
```

<h3 id="get-an-account-type-responses">Responses</h3>

|Status|Meaning|Description|Schema|
|---|---|---|---|
|200|[OK](https://tools.ietf.org/html/rfc7231#section-6.3.1)|Successful GET on AccountType resource|[AccountType](#schemaaccounttype)|

<aside class="success">
This operation does not require authentication
</aside>

## Create bank categories

> Code samples

```http
POST /demo.biapi.pro/2.0/banks/categories HTTP/1.1

Content-Type: multipart/form-data
Accept: application/json

```

```python
import requests
headers = {
  'Content-Type': 'multipart/form-data',
  'Accept': 'application/json'
}

r = requests.post('/demo.biapi.pro/2.0/banks/categories', params={

}, headers = headers)

print r.json()

```

`POST /banks/categories`

It requires the name of the category to be created<br><br>

> Body parameter

```yaml
name: string

```

<h3 id="create-bank-categories-parameters">Parameters</h3>

|Name|In|Type|Required|Description|
|---|---|---|---|---|
|expand|query|string|false|none|
|body|body|object|false|none|
|» name|body|string|true|name of the category to be created|

> Example responses

> 200 Response

```json
{
  "id": 0,
  "name": false
}
```

<h3 id="create-bank-categories-responses">Responses</h3>

|Status|Meaning|Description|Schema|
|---|---|---|---|
|200|[OK](https://tools.ietf.org/html/rfc7231#section-6.3.1)|Successful POST on ConnectorCategory resource|[ConnectorCategory](#schemaconnectorcategory)|

<aside class="success">
This operation does not require authentication
</aside>

## Edit a bank categories

> Code samples

```http
POST /demo.biapi.pro/2.0/banks/categories/{id_category} HTTP/1.1

Content-Type: multipart/form-data
Accept: application/json

```

```python
import requests
headers = {
  'Content-Type': 'multipart/form-data',
  'Accept': 'application/json'
}

r = requests.post('/demo.biapi.pro/2.0/banks/categories/{id_category}', params={

}, headers = headers)

print r.json()

```

`POST /banks/categories/{id_category}`

Edit the name for the supplied category.<br><br>

> Body parameter

```yaml
name: string

```

<h3 id="edit-a-bank-categories-parameters">Parameters</h3>

|Name|In|Type|Required|Description|
|---|---|---|---|---|
|id_category|path|integer|true|none|
|expand|query|string|false|none|
|body|body|object|false|none|
|» name|body|string|true|new name for the supplied category|

> Example responses

> 200 Response

```json
{
  "id": 0,
  "name": false
}
```

<h3 id="edit-a-bank-categories-responses">Responses</h3>

|Status|Meaning|Description|Schema|
|---|---|---|---|
|200|[OK](https://tools.ietf.org/html/rfc7231#section-6.3.1)|Successful POST on ConnectorCategory resource|[ConnectorCategory](#schemaconnectorcategory)|

<aside class="success">
This operation does not require authentication
</aside>

## Delete the supplied category

> Code samples

```http
DELETE /demo.biapi.pro/2.0/banks/categories/{id_category} HTTP/1.1

Accept: application/json

```

```python
import requests
headers = {
  'Accept': 'application/json'
}

r = requests.delete('/demo.biapi.pro/2.0/banks/categories/{id_category}', params={

}, headers = headers)

print r.json()

```

`DELETE /banks/categories/{id_category}`

<h3 id="delete-the-supplied-category-parameters">Parameters</h3>

|Name|In|Type|Required|Description|
|---|---|---|---|---|
|id_category|path|integer|true|none|
|expand|query|string|false|none|

> Example responses

> 200 Response

```json
{
  "id": 0,
  "name": false
}
```

<h3 id="delete-the-supplied-category-responses">Responses</h3>

|Status|Meaning|Description|Schema|
|---|---|---|---|
|200|[OK](https://tools.ietf.org/html/rfc7231#section-6.3.1)|Successful DELETE on ConnectorCategory resource|[ConnectorCategory](#schemaconnectorcategory)|

<aside class="success">
This operation does not require authentication
</aside>

## Get a subset of id_connection with the largest diversity of account

> Code samples

```http
GET /demo.biapi.pro/2.0/banks/{id_connector}/connections HTTP/1.1

Accept: application/json

```

```python
import requests
headers = {
  'Accept': 'application/json'
}

r = requests.get('/demo.biapi.pro/2.0/banks/{id_connector}/connections', params={

}, headers = headers)

print r.json()

```

`GET /banks/{id_connector}/connections`

By default, it selects a set of 3 connections.<br><br>

<h3 id="get-a-subset-of-id_connection-with-the-largest-diversity-of-account-parameters">Parameters</h3>

|Name|In|Type|Required|Description|
|---|---|---|---|---|
|id_connector|path|integer|true|none|
|range|query|integer|false|the length of the connection subset|
|type|query|integer|false|to target a specific account type which will be|
|expand|query|string|false|none|

> Example responses

> 200 Response

```json
{
  "connections": [
    {
      "id": 0,
      "id_user": 0,
      "id_connector": 0,
      "last_update": "2019-03-20 11:42:25.591002",
      "error": "",
      "expire": "2019-03-20 11:42:25.591259",
      "active": true,
      "last_push": "2019-03-20 11:42:25.591429",
      "next_try": "2019-03-20 11:42:25.591684"
    }
  ]
}
```

<h3 id="get-a-subset-of-id_connection-with-the-largest-diversity-of-account-responses">Responses</h3>

|Status|Meaning|Description|Schema|
|---|---|---|---|
|200|[OK](https://tools.ietf.org/html/rfc7231#section-6.3.1)|connections|Inline|

<h3 id="get-a-subset-of-id_connection-with-the-largest-diversity-of-account-responseschema">Response Schema</h3>

Status Code **200**

|Name|Type|Required|Restrictions|Description|
|---|---|---|---|---|
|» connections|[[Connection](#schemaconnection)]|true|none|none|
|»» id|integer|true|none|ID of connection|
|»» id_user|integer|false|none|ID of the related user|
|»» id_connector|integer|true|none|ID of the related connector|
|»» last_update|string(date-time)|false|none|Last successful update|
|»» error|string|false|none|If the last update has failed, the error code|
|»» expire|string(date-time)|false|none|Expiration of the connection. Used during add of a two-factor authentication, to purge the connection if the user abort|
|»» active|boolean|true|none|This connection is active and will be automatically synced|
|»» last_push|string(date-time)|false|none|Last successful push|
|»» next_try|string(date-time)|false|none|Date of next synchronization|

<aside class="success">
This operation does not require authentication
</aside>

## Get all links to the files associated with this connector.

> Code samples

```http
GET /demo.biapi.pro/2.0/providers/{id_connector}/logos/thumbnail HTTP/1.1

Accept: application/json

```

```python
import requests
headers = {
  'Accept': 'application/json'
}

r = requests.get('/demo.biapi.pro/2.0/providers/{id_connector}/logos/thumbnail', params={

}, headers = headers)

print r.json()

```

`GET /providers/{id_connector}/logos/thumbnail`

This endpoint returns all links to files associated with this connector.<br><br>

<h3 id="get-all-links-to-the-files-associated-with-this-connector.-parameters">Parameters</h3>

|Name|In|Type|Required|Description|
|---|---|---|---|---|
|id_connector|path|integer|true|none|
|expand|query|string|false|none|

> Example responses

> 200 Response

```json
{
  "connectorlogos": [
    {
      "id": 0,
      "id_connector": 0,
      "id_file": 0,
      "type": ""
    }
  ]
}
```

<h3 id="get-all-links-to-the-files-associated-with-this-connector.-responses">Responses</h3>

|Status|Meaning|Description|Schema|
|---|---|---|---|
|200|[OK](https://tools.ietf.org/html/rfc7231#section-6.3.1)|connectorlogos|Inline|

<h3 id="get-all-links-to-the-files-associated-with-this-connector.-responseschema">Response Schema</h3>

Status Code **200**

|Name|Type|Required|Restrictions|Description|
|---|---|---|---|---|
|» connectorlogos|[[ConnectorLogo](#schemaconnectorlogo)]|true|none|none|
|»» id|integer|true|none|none|
|»» id_connector|integer|true|none|ID of the connector|
|»» id_file|integer|true|none|Id of the Bank/Provider Logo|
|»» type|string|false|none|Logo's type|

<aside class="success">
This operation does not require authentication
</aside>

## Get all categories

> Code samples

```http
GET /demo.biapi.pro/2.0/categories HTTP/1.1

Accept: application/json

```

```python
import requests
headers = {
  'Accept': 'application/json'
}

r = requests.get('/demo.biapi.pro/2.0/categories', params={

}, headers = headers)

print r.json()

```

`GET /categories`

Ressource to get all existing categories<br><br>

<h3 id="get-all-categories-parameters">Parameters</h3>

|Name|In|Type|Required|Description|
|---|---|---|---|---|
|expand|query|string|false|none|

> Example responses

> 200 Response

```json
{
  "categories": [
    {
      "id": 0,
      "id_parent_category": 0,
      "name": "",
      "income": false,
      "color": "",
      "id_parent_category_in_menu": 0,
      "name_displayed": "",
      "refundable": false,
      "id_user": 0,
      "id_logo": 0
    }
  ]
}
```

<h3 id="get-all-categories-responses">Responses</h3>

|Status|Meaning|Description|Schema|
|---|---|---|---|
|200|[OK](https://tools.ietf.org/html/rfc7231#section-6.3.1)|categories|Inline|

<h3 id="get-all-categories-responseschema">Response Schema</h3>

Status Code **200**

|Name|Type|Required|Restrictions|Description|
|---|---|---|---|---|
|» categories|[[Category](#schemacategory)]|true|none|none|
|»» id|integer|true|none|ID of the category|
|»» id_parent_category|integer|true|none|ID of the parent category. If this is a parent category, it will be equal to its own ID|
|»» name|string|true|none|Name of the category|
|»» income|boolean|false|none|Is an income category. If null, this is both an income and an expense category|
|»» color|string|true|none|Color of the category|
|»» id_parent_category_in_menu|integer|true|none|ID of the parent category to be displayed|
|»» name_displayed|string|false|none|Displayed name, with HTML tags|
|»» refundable|boolean|true|none|This category accepts opposite sign of transactions|
|»» id_user|integer|false|none|If not null, this category is specific to a user|
|»» id_logo|integer|false|none|ID of the logo|

<aside class="success">
This operation does not require authentication
</aside>

## categorize transactions without storing them

> Code samples

```http
POST /demo.biapi.pro/2.0/categorize HTTP/1.1

Content-Type: multipart/form-data
Accept: application/json

```

```python
import requests
headers = {
  'Content-Type': 'multipart/form-data',
  'Accept': 'application/json'
}

r = requests.post('/demo.biapi.pro/2.0/categorize', params={

}, headers = headers)

print r.json()

```

`POST /categorize`

It requires an array of transaction dictionaries. Any fields of transactions that are not required will be kept in the response. The response contains the list of transactions with two more fields: id_category and state (it indicates how the transaction has been categorized)<br><br>

> Body parameter

```yaml
wording: string
value: 0
type: string

```

<h3 id="categorize-transactions-without-storing-them-parameters">Parameters</h3>

|Name|In|Type|Required|Description|
|---|---|---|---|---|
|body|body|object|false|none|
|» wording|body|string|true|label of the transaction|
|» value|body|integer|true|value of the transaction|
|» type|body|string|true|type of the transaction (default: unknown)|

> Example responses

> 200 Response

```json
{}
```

> OK

<h3 id="categorize-transactions-without-storing-them-responses">Responses</h3>

|Status|Meaning|Description|Schema|
|---|---|---|---|
|200|[OK](https://tools.ietf.org/html/rfc7231#section-6.3.1)|OK|Inline|

<h3 id="categorize-transactions-without-storing-them-responseschema">Response Schema</h3>

<aside class="success">
This operation does not require authentication
</aside>

## Edit the provided connector

> Code samples

```http
PUT /demo.biapi.pro/2.0/connectors/{id_connector} HTTP/1.1

Content-Type: multipart/form-data
Accept: application/json

```

```python
import requests
headers = {
  'Content-Type': 'multipart/form-data',
  'Accept': 'application/json'
}

r = requests.put('/demo.biapi.pro/2.0/connectors/{id_connector}', params={

}, headers = headers)

print r.json()

```

`PUT /connectors/{id_connector}`

<br><br>

> Body parameter

```yaml
id_categories: string
hidden: true
sync_frequency: 0

```

<h3 id="edit-the-provided-connector-parameters">Parameters</h3>

|Name|In|Type|Required|Description|
|---|---|---|---|---|
|id_connector|path|integer|true|none|
|expand|query|string|false|none|
|body|body|object|false|none|
|» id_categories|body|string|false|one or several comma separated categories to map to the given connector (or null to map no category)|
|» hidden|body|boolean|false|to enable  or disable connector (bank or provider)|
|» sync_frequency|body|integer|false|Allows you to overload global sync_frequency param|

> Example responses

> 200 Response

```json
{
  "id": 0,
  "name": "",
  "id_weboob": "",
  "hidden": false,
  "charged": true,
  "code": "",
  "beta": false,
  "color": "",
  "slug": "",
  "sync_frequency": 0,
  "months_to_fetch": 0,
  "auth_mechanism": ""
}
```

> Successful PUT on Connector resource

<h3 id="edit-the-provided-connector-responses">Responses</h3>

|Status|Meaning|Description|Schema|
|---|---|---|---|
|200|[OK](https://tools.ietf.org/html/rfc7231#section-6.3.1)|Successful PUT on Connector resource|[Connector](#schemaconnector)|

<aside class="success">
This operation does not require authentication
</aside>

## Create a connector Logo

> Code samples

```http
POST /demo.biapi.pro/2.0/connectors/{id_connector}/logos HTTP/1.1

Accept: application/json

```

```python
import requests
headers = {
  'Accept': 'application/json'
}

r = requests.post('/demo.biapi.pro/2.0/connectors/{id_connector}/logos', params={

}, headers = headers)

print r.json()

```

`POST /connectors/{id_connector}/logos`

This endpoint creates a connector logo. You can either pass a file to as a parameter to insert and link it with the connector or pass an id_file to link a connector with an existing file. Will fail if the file is already linked with that connector.<br><br>Form params: - id_file (integer): The id of the file to link with that connector. - img (string): Path to the image to link with that connector.<br><br>

<h3 id="create-a-connector-logo-parameters">Parameters</h3>

|Name|In|Type|Required|Description|
|---|---|---|---|---|
|id_connector|path|integer|true|none|
|expand|query|string|false|none|

> Example responses

> 200 Response

```json
{
  "id": 0,
  "id_connector": 0,
  "id_file": 0,
  "type": ""
}
```

<h3 id="create-a-connector-logo-responses">Responses</h3>

|Status|Meaning|Description|Schema|
|---|---|---|---|
|200|[OK](https://tools.ietf.org/html/rfc7231#section-6.3.1)|Successful POST on ConnectorLogo resource|[ConnectorLogo](#schemaconnectorlogo)|

<aside class="success">
This operation does not require authentication
</aside>

## Create or Update a connector Logo

> Code samples

```http
PUT /demo.biapi.pro/2.0/connectors/{id_connector}/logos HTTP/1.1

Accept: application/json

```

```python
import requests
headers = {
  'Accept': 'application/json'
}

r = requests.put('/demo.biapi.pro/2.0/connectors/{id_connector}/logos', params={

}, headers = headers)

print r.json()

```

`PUT /connectors/{id_connector}/logos`

This endpoint creates or update a connector logo. This logo is a mapping between a file (/file route) and a connector (/connectors route) or a provider (/providers route).<br><br>Form params: - id_file (integer): The id of the file to link with that connector.<br><br>

<h3 id="create-or-update-a-connector-logo-parameters">Parameters</h3>

|Name|In|Type|Required|Description|
|---|---|---|---|---|
|id_connector|path|integer|true|none|
|expand|query|string|false|none|

> Example responses

> 200 Response

```json
{
  "id": 0,
  "id_connector": 0,
  "id_file": 0,
  "type": ""
}
```

<h3 id="create-or-update-a-connector-logo-responses">Responses</h3>

|Status|Meaning|Description|Schema|
|---|---|---|---|
|200|[OK](https://tools.ietf.org/html/rfc7231#section-6.3.1)|Successful PUT on ConnectorLogo resource|[ConnectorLogo](#schemaconnectorlogo)|

<aside class="success">
This operation does not require authentication
</aside>

## Create or Update a connector Logo.

> Code samples

```http
PUT /demo.biapi.pro/2.0/connectors/{id_connector}/logos/{id_logo} HTTP/1.1

Accept: application/json

```

```python
import requests
headers = {
  'Accept': 'application/json'
}

r = requests.put('/demo.biapi.pro/2.0/connectors/{id_connector}/logos/{id_logo}', params={

}, headers = headers)

print r.json()

```

`PUT /connectors/{id_connector}/logos/{id_logo}`

This endpoint creates or update a connector logo. This logo is a mapping between a file (/file route) and a connector (/connectors route) or a provider (/providers route).<br><br>Form params: - id_file (integer): The id of the file to link with that connector.<br><br>

<h3 id="create-or-update-a-connector-logo.-parameters">Parameters</h3>

|Name|In|Type|Required|Description|
|---|---|---|---|---|
|id_connector|path|integer|true|none|
|id_logo|path|integer|true|none|
|expand|query|string|false|none|

> Example responses

> 200 Response

```json
{
  "id": 0,
  "id_connector": 0,
  "id_file": 0,
  "type": ""
}
```

<h3 id="create-or-update-a-connector-logo.-responses">Responses</h3>

|Status|Meaning|Description|Schema|
|---|---|---|---|
|200|[OK](https://tools.ietf.org/html/rfc7231#section-6.3.1)|Successful PUT on ConnectorLogo resource|[ConnectorLogo](#schemaconnectorlogo)|

<aside class="success">
This operation does not require authentication
</aside>

## Delete a single Logo object.

> Code samples

```http
DELETE /demo.biapi.pro/2.0/connectors/{id_connector}/logos/{id_logo} HTTP/1.1

Accept: application/json

```

```python
import requests
headers = {
  'Accept': 'application/json'
}

r = requests.delete('/demo.biapi.pro/2.0/connectors/{id_connector}/logos/{id_logo}', params={

}, headers = headers)

print r.json()

```

`DELETE /connectors/{id_connector}/logos/{id_logo}`

<h3 id="delete-a-single-logo-object.-parameters">Parameters</h3>

|Name|In|Type|Required|Description|
|---|---|---|---|---|
|id_connector|path|integer|true|none|
|id_logo|path|integer|true|none|
|expand|query|string|false|none|

> Example responses

> 200 Response

```json
{
  "id": 0,
  "id_connector": 0,
  "id_file": 0,
  "type": ""
}
```

<h3 id="delete-a-single-logo-object.-responses">Responses</h3>

|Status|Meaning|Description|Schema|
|---|---|---|---|
|200|[OK](https://tools.ietf.org/html/rfc7231#section-6.3.1)|Successful DELETE on ConnectorLogo resource|[ConnectorLogo](#schemaconnectorlogo)|

<aside class="success">
This operation does not require authentication
</aside>

## Get number of accounts, connections and users synced.

> Code samples

```http
GET /demo.biapi.pro/2.0/invoicing HTTP/1.1

```

```python
import requests

r = requests.get('/demo.biapi.pro/2.0/invoicing', params={

)

print r.json()

```

`GET /invoicing`

Get number of accounts, connections and users synced between two dates for the given period.<br><br>

<h3 id="get-number-of-accounts,-connections-and-users-synced.-parameters">Parameters</h3>

|Name|In|Type|Required|Description|
|---|---|---|---|---|
|min_date|query|string(date)|false|minimal date|
|max_date|query|string(date)|false|maximum date|
|period|query|string|false|period to group logs|
|all|query|string|false|get full ids list instead of numbers|

<h3 id="get-number-of-accounts,-connections-and-users-synced.-responses">Responses</h3>

|Status|Meaning|Description|Schema|
|---|---|---|---|
|200|[OK](https://tools.ietf.org/html/rfc7231#section-6.3.1)|standard HTTP response|None|

<aside class="success">
This operation does not require authentication
</aside>

## Get a random subset of provider's id_connection

> Code samples

```http
GET /demo.biapi.pro/2.0/providers/{id_connector}/connections HTTP/1.1

Accept: application/json

```

```python
import requests
headers = {
  'Accept': 'application/json'
}

r = requests.get('/demo.biapi.pro/2.0/providers/{id_connector}/connections', params={

}, headers = headers)

print r.json()

```

`GET /providers/{id_connector}/connections`

By default, it selects a set of 3 connections.<br><br>

<h3 id="get-a-random-subset-of-provider's-id_connection-parameters">Parameters</h3>

|Name|In|Type|Required|Description|
|---|---|---|---|---|
|id_connector|path|integer|true|none|
|range|query|integer|false|the length of the connection subset|
|expand|query|string|false|none|

> Example responses

> 200 Response

```json
{
  "connections": [
    {
      "id": 0,
      "id_user": 0,
      "id_connector": 0,
      "last_update": "2019-03-20 11:42:25.591002",
      "error": "",
      "expire": "2019-03-20 11:42:25.591259",
      "active": true,
      "last_push": "2019-03-20 11:42:25.591429",
      "next_try": "2019-03-20 11:42:25.591684"
    }
  ]
}
```

<h3 id="get-a-random-subset-of-provider's-id_connection-responses">Responses</h3>

|Status|Meaning|Description|Schema|
|---|---|---|---|
|200|[OK](https://tools.ietf.org/html/rfc7231#section-6.3.1)|connections|Inline|

<h3 id="get-a-random-subset-of-provider's-id_connection-responseschema">Response Schema</h3>

Status Code **200**

|Name|Type|Required|Restrictions|Description|
|---|---|---|---|---|
|» connections|[[Connection](#schemaconnection)]|true|none|none|
|»» id|integer|true|none|ID of connection|
|»» id_user|integer|false|none|ID of the related user|
|»» id_connector|integer|true|none|ID of the related connector|
|»» last_update|string(date-time)|false|none|Last successful update|
|»» error|string|false|none|If the last update has failed, the error code|
|»» expire|string(date-time)|false|none|Expiration of the connection. Used during add of a two-factor authentication, to purge the connection if the user abort|
|»» active|boolean|true|none|This connection is active and will be automatically synced|
|»» last_push|string(date-time)|false|none|Last successful push|
|»» next_try|string(date-time)|false|none|Date of next synchronization|

<aside class="success">
This operation does not require authentication
</aside>

## Get accounts list.

> Code samples

```http
GET /demo.biapi.pro/2.0/users/{id_user}/connections/{id_connection}/accounts HTTP/1.1

Accept: application/json

```

```python
import requests
headers = {
  'Accept': 'application/json'
}

r = requests.get('/demo.biapi.pro/2.0/users/{id_user}/connections/{id_connection}/accounts', params={

}, headers = headers)

print r.json()

```

`GET /users/{id_user}/connections/{id_connection}/accounts`

<h3 id="get-accounts-list.-parameters">Parameters</h3>

|Name|In|Type|Required|Description|
|---|---|---|---|---|
|id_user|path|string|true|Hint: you can use 'me' or 'all'|
|id_connection|path|integer|true|none|
|expand|query|string|false|none|

> Example responses

> 200 Response

```json
{
  "accounts": [
    {
      "id": 0,
      "id_connection": 0,
      "id_user": 0,
      "id_parent": 0,
      "number": "",
      "original_name": "",
      "balance": 0,
      "coming": 0,
      "display": true,
      "last_update": "2019-03-20 11:42:25.565293",
      "deleted": "2019-03-20 11:42:25.565379",
      "disabled": "2019-03-20 11:42:25.565461",
      "iban": "",
      "currency": "",
      "id_type": 0,
      "bookmarked": false,
      "name": "",
      "error": "",
      "usage": ""
    }
  ]
}
```

<h3 id="get-accounts-list.-responses">Responses</h3>

|Status|Meaning|Description|Schema|
|---|---|---|---|
|200|[OK](https://tools.ietf.org/html/rfc7231#section-6.3.1)|accounts|Inline|

<h3 id="get-accounts-list.-responseschema">Response Schema</h3>

Status Code **200**

|Name|Type|Required|Restrictions|Description|
|---|---|---|---|---|
|» accounts|[[Account](#schemaaccount)]|true|none|none|
|»» id|integer|true|none|ID of the account|
|»» id_connection|integer|false|none|ID of the related connection|
|»» id_user|integer|false|none|ID of the related user|
|»» id_parent|integer|false|none|Id of the parent account|
|»» number|string|false|none|Account number|
|»» original_name|string|true|none|Original name of the account on the bank|
|»» balance|number(float)|true|none|Balance of the account|
|»» coming|number(float)|false|none|Amount of coming operations not yet debited|
|»» display|boolean|true|none|Display this account in accounts list|
|»» last_update|string(date-time)|false|none|Last successful update of the account|
|»» deleted|string(date-time)|false|none|This account is not found on the website anymore|
|»» disabled|string(date-time)|false|none|This account has been deleted by user and will not be synchronized anymore|
|»» iban|string|false|none|Account IBAN|
|»» currency|object|false|none|Account currency|
|»» id_type|integer|false|none|ID of the account type|
|»» bookmarked|integer|true|none|This account has been bookmarked by user|
|»» name|string|false|none|Name of the account|
|»» error|string|false|none|If the last update has failed, the error code|
|»» usage|string|false|none|Account usage|

<aside class="success">
This operation does not require authentication
</aside>

## Create an account

> Code samples

```http
POST /demo.biapi.pro/2.0/users/{id_user}/connections/{id_connection}/accounts HTTP/1.1

Content-Type: multipart/form-data
Accept: application/json

```

```python
import requests
headers = {
  'Content-Type': 'multipart/form-data',
  'Accept': 'application/json'
}

r = requests.post('/demo.biapi.pro/2.0/users/{id_user}/connections/{id_connection}/accounts', params={

}, headers = headers)

print r.json()

```

`POST /users/{id_user}/connections/{id_connection}/accounts`

This endpoint creates an account related to a connection or not.<br><br>

> Body parameter

```yaml
name: string
balance: 0
number: string
iban: string
id_currency: string
id_connection: 0

```

<h3 id="create-an-account-parameters">Parameters</h3>

|Name|In|Type|Required|Description|
|---|---|---|---|---|
|id_user|path|string|true|Hint: you can use 'me' or 'all'|
|id_connection|path|integer|true|none|
|expand|query|string|false|none|
|body|body|[postUsers_idUser_accounts](#schemapostusers_iduser_accounts)|false|none|
|» name|body|string|true|name of account|
|» balance|body|number(float)|true|balance of account|
|» number|body|string|false|number of account|
|» iban|body|string|false|IBAN of account|
|» id_currency|body|string|false|the currency of the account (default: 'EUR')|
|» id_connection|body|integer|false|the connection to attach to the account|

> Example responses

> 200 Response

```json
{
  "id": 0,
  "id_connection": 0,
  "id_user": 0,
  "id_parent": 0,
  "number": "",
  "original_name": "",
  "balance": 0,
  "coming": 0,
  "display": true,
  "last_update": "2019-03-20 11:42:25.565293",
  "deleted": "2019-03-20 11:42:25.565379",
  "disabled": "2019-03-20 11:42:25.565461",
  "iban": "",
  "currency": "",
  "id_type": 0,
  "bookmarked": false,
  "name": "",
  "error": "",
  "usage": ""
}
```

<h3 id="create-an-account-responses">Responses</h3>

|Status|Meaning|Description|Schema|
|---|---|---|---|
|200|[OK](https://tools.ietf.org/html/rfc7231#section-6.3.1)|Successful POST on Account resource|[Account](#schemaaccount)|

<aside class="success">
This operation does not require authentication
</aside>

## Update many accounts at once

> Code samples

```http
PUT /demo.biapi.pro/2.0/users/{id_user}/connections/{id_connection}/accounts HTTP/1.1

Accept: application/json

```

```python
import requests
headers = {
  'Accept': 'application/json'
}

r = requests.put('/demo.biapi.pro/2.0/users/{id_user}/connections/{id_connection}/accounts', params={

}, headers = headers)

print r.json()

```

`PUT /users/{id_user}/connections/{id_connection}/accounts`

<h3 id="update-many-accounts-at-once-parameters">Parameters</h3>

|Name|In|Type|Required|Description|
|---|---|---|---|---|
|id_user|path|string|true|Hint: you can use 'me' or 'all'|
|id_connection|path|integer|true|none|
|expand|query|string|false|none|

> Example responses

> 200 Response

```json
{
  "id": 0,
  "id_connection": 0,
  "id_user": 0,
  "id_parent": 0,
  "number": "",
  "original_name": "",
  "balance": 0,
  "coming": 0,
  "display": true,
  "last_update": "2019-03-20 11:42:25.565293",
  "deleted": "2019-03-20 11:42:25.565379",
  "disabled": "2019-03-20 11:42:25.565461",
  "iban": "",
  "currency": "",
  "id_type": 0,
  "bookmarked": false,
  "name": "",
  "error": "",
  "usage": ""
}
```

<h3 id="update-many-accounts-at-once-responses">Responses</h3>

|Status|Meaning|Description|Schema|
|---|---|---|---|
|200|[OK](https://tools.ietf.org/html/rfc7231#section-6.3.1)|Successful PUT on Account resource|[Account](#schemaaccount)|

<aside class="success">
This operation does not require authentication
</aside>

## Delete all accounts

> Code samples

```http
DELETE /demo.biapi.pro/2.0/users/{id_user}/connections/{id_connection}/accounts HTTP/1.1

Accept: application/json

```

```python
import requests
headers = {
  'Accept': 'application/json'
}

r = requests.delete('/demo.biapi.pro/2.0/users/{id_user}/connections/{id_connection}/accounts', params={

}, headers = headers)

print r.json()

```

`DELETE /users/{id_user}/connections/{id_connection}/accounts`

<h3 id="delete-all-accounts-parameters">Parameters</h3>

|Name|In|Type|Required|Description|
|---|---|---|---|---|
|id_user|path|string|true|Hint: you can use 'me' or 'all'|
|id_connection|path|integer|true|none|
|expand|query|string|false|none|

> Example responses

> 200 Response

```json
{
  "id": 0,
  "id_connection": 0,
  "id_user": 0,
  "id_parent": 0,
  "number": "",
  "original_name": "",
  "balance": 0,
  "coming": 0,
  "display": true,
  "last_update": "2019-03-20 11:42:25.565293",
  "deleted": "2019-03-20 11:42:25.565379",
  "disabled": "2019-03-20 11:42:25.565461",
  "iban": "",
  "currency": "",
  "id_type": 0,
  "bookmarked": false,
  "name": "",
  "error": "",
  "usage": ""
}
```

<h3 id="delete-all-accounts-responses">Responses</h3>

|Status|Meaning|Description|Schema|
|---|---|---|---|
|200|[OK](https://tools.ietf.org/html/rfc7231#section-6.3.1)|Successful DELETE on Account resource|[Account](#schemaaccount)|

<aside class="success">
This operation does not require authentication
</aside>

## Update an account

> Code samples

```http
PUT /demo.biapi.pro/2.0/users/{id_user}/connections/{id_connection}/accounts/{id_account} HTTP/1.1

Content-Type: multipart/form-data
Accept: application/json

```

```python
import requests
headers = {
  'Content-Type': 'multipart/form-data',
  'Accept': 'application/json'
}

r = requests.put('/demo.biapi.pro/2.0/users/{id_user}/connections/{id_connection}/accounts/{id_account}', params={

}, headers = headers)

print r.json()

```

`PUT /users/{id_user}/connections/{id_connection}/accounts/{id_account}`

It updates a specific account<br><br>

> Body parameter

```yaml
display: true
name: string
balance: 0
disabled: true
iban: string
bookmarked: true
usage: string

```

<h3 id="update-an-account-parameters">Parameters</h3>

|Name|In|Type|Required|Description|
|---|---|---|---|---|
|id_user|path|string|true|Hint: you can use 'me' or 'all'|
|id_connection|path|integer|true|none|
|id_account|path|integer|true|none|
|expand|query|string|false|none|
|body|body|[putUsers_idUser_accounts_idAccount_](#schemaputusers_iduser_accounts_idaccount_)|false|none|
|» display|body|boolean|false|If the account is displayed|
|» name|body|string|false|Label of the account|
|» balance|body|number(float)|false|Balance of the account|
|» disabled|body|boolean|false|If the account is disabled (not synchronized)|
|» iban|body|string|false|IBAN of the account|
|» bookmarked|body|boolean|false|If the account is bookmarked|
|» usage|body|string|false|Usage of the account : PRIV, ORGA or ASSO|

> Example responses

> 200 Response

```json
{
  "id": 0,
  "id_connection": 0,
  "id_user": 0,
  "id_parent": 0,
  "number": "",
  "original_name": "",
  "balance": 0,
  "coming": 0,
  "display": true,
  "last_update": "2019-03-20 11:42:25.565293",
  "deleted": "2019-03-20 11:42:25.565379",
  "disabled": "2019-03-20 11:42:25.565461",
  "iban": "",
  "currency": "",
  "id_type": 0,
  "bookmarked": false,
  "name": "",
  "error": "",
  "usage": ""
}
```

<h3 id="update-an-account-responses">Responses</h3>

|Status|Meaning|Description|Schema|
|---|---|---|---|
|200|[OK](https://tools.ietf.org/html/rfc7231#section-6.3.1)|Successful PUT on Account resource|[Account](#schemaaccount)|

<aside class="success">
This operation does not require authentication
</aside>

## Delete an account.

> Code samples

```http
DELETE /demo.biapi.pro/2.0/users/{id_user}/connections/{id_connection}/accounts/{id_account} HTTP/1.1

Accept: application/json

```

```python
import requests
headers = {
  'Accept': 'application/json'
}

r = requests.delete('/demo.biapi.pro/2.0/users/{id_user}/connections/{id_connection}/accounts/{id_account}', params={

}, headers = headers)

print r.json()

```

`DELETE /users/{id_user}/connections/{id_connection}/accounts/{id_account}`

It deletes a specific account.<br><br>

<h3 id="delete-an-account.-parameters">Parameters</h3>

|Name|In|Type|Required|Description|
|---|---|---|---|---|
|id_user|path|string|true|Hint: you can use 'me' or 'all'|
|id_connection|path|integer|true|none|
|id_account|path|integer|true|none|
|expand|query|string|false|none|

> Example responses

> 200 Response

```json
{
  "id": 0,
  "id_connection": 0,
  "id_user": 0,
  "id_parent": 0,
  "number": "",
  "original_name": "",
  "balance": 0,
  "coming": 0,
  "display": true,
  "last_update": "2019-03-20 11:42:25.565293",
  "deleted": "2019-03-20 11:42:25.565379",
  "disabled": "2019-03-20 11:42:25.565461",
  "iban": "",
  "currency": "",
  "id_type": 0,
  "bookmarked": false,
  "name": "",
  "error": "",
  "usage": ""
}
```

<h3 id="delete-an-account.-responses">Responses</h3>

|Status|Meaning|Description|Schema|
|---|---|---|---|
|200|[OK](https://tools.ietf.org/html/rfc7231#section-6.3.1)|Successful DELETE on Account resource|[Account](#schemaaccount)|

<aside class="success">
This operation does not require authentication
</aside>

## Get the category

> Code samples

```http
GET /demo.biapi.pro/2.0/users/{id_user}/connections/{id_connection}/accounts/{id_account}/categories HTTP/1.1

```

```python
import requests

r = requests.get('/demo.biapi.pro/2.0/users/{id_user}/connections/{id_connection}/accounts/{id_account}/categories', params={

)

print r.json()

```

`GET /users/{id_user}/connections/{id_connection}/accounts/{id_account}/categories`

Ressource to get categories for the user's transactions<br><br>

<h3 id="get-the-category-parameters">Parameters</h3>

|Name|In|Type|Required|Description|
|---|---|---|---|---|
|id_user|path|string|true|Hint: you can use 'me' or 'all'|
|id_connection|path|integer|true|none|
|id_account|path|integer|true|none|

<h3 id="get-the-category-responses">Responses</h3>

|Status|Meaning|Description|Schema|
|---|---|---|---|
|200|[OK](https://tools.ietf.org/html/rfc7231#section-6.3.1)|standard HTTP response|None|

<aside class="success">
This operation does not require authentication
</aside>

## Get deltas of accounts

> Code samples

```http
GET /demo.biapi.pro/2.0/users/{id_user}/connections/{id_connection}/accounts/{id_account}/delta HTTP/1.1

```

```python
import requests

r = requests.get('/demo.biapi.pro/2.0/users/{id_user}/connections/{id_connection}/accounts/{id_account}/delta', params={

)

print r.json()

```

`GET /users/{id_user}/connections/{id_connection}/accounts/{id_account}/delta`

Get account delta between sums of transactions and difference of account balance for the given period.<br><br>

<h3 id="get-deltas-of-accounts-parameters">Parameters</h3>

|Name|In|Type|Required|Description|
|---|---|---|---|---|
|id_user|path|string|true|Hint: you can use 'me' or 'all'|
|id_connection|path|integer|true|none|
|id_account|path|integer|true|none|
|min_date|query|string(date)|false|minimal date|
|max_date|query|string(date)|false|maximum date|
|period|query|string|false|period to group logs|

<h3 id="get-deltas-of-accounts-responses">Responses</h3>

|Status|Meaning|Description|Schema|
|---|---|---|---|
|200|[OK](https://tools.ietf.org/html/rfc7231#section-6.3.1)|standard HTTP response|None|

<aside class="success">
This operation does not require authentication
</aside>

## Get accounts logs.

> Code samples

```http
GET /demo.biapi.pro/2.0/users/{id_user}/connections/{id_connection}/accounts/{id_account}/logs HTTP/1.1

Accept: application/json

```

```python
import requests
headers = {
  'Accept': 'application/json'
}

r = requests.get('/demo.biapi.pro/2.0/users/{id_user}/connections/{id_connection}/accounts/{id_account}/logs', params={

}, headers = headers)

print r.json()

```

`GET /users/{id_user}/connections/{id_connection}/accounts/{id_account}/logs`

Get logs of account.<br><br>By default, it selects logs for the last month. You can use "min_date" and "max_date" to change boundary dates.<br><br>

<h3 id="get-accounts-logs.-parameters">Parameters</h3>

|Name|In|Type|Required|Description|
|---|---|---|---|---|
|id_user|path|string|true|Hint: you can use 'me' or 'all'|
|id_connection|path|integer|true|none|
|id_account|path|integer|true|none|
|limit|query|integer|false|limit number of results|
|offset|query|integer|false|offset of first result|
|min_date|query|string(date)|false|minimal (inclusive) date|
|max_date|query|string(date)|false|maximum (inclusive) date|
|expand|query|string|false|none|

> Example responses

> 200 Response

```json
{
  "accountlogs": [
    {
      "id_account": 0,
      "id_connector": 0,
      "balance": 0,
      "coming": 0,
      "timestamp": "CURRENT_TIMESTAMP",
      "error": "",
      "error_message": "",
      "id_connection_log": 0
    }
  ]
}
```

<h3 id="get-accounts-logs.-responses">Responses</h3>

|Status|Meaning|Description|Schema|
|---|---|---|---|
|200|[OK](https://tools.ietf.org/html/rfc7231#section-6.3.1)|accountlogs|Inline|

<h3 id="get-accounts-logs.-responseschema">Response Schema</h3>

Status Code **200**

|Name|Type|Required|Restrictions|Description|
|---|---|---|---|---|
|» accountlogs|[[AccountLog](#schemaaccountlog)]|true|none|none|
|»» id_account|integer|true|none|ID of the related account|
|»» id_connector|integer|false|none|provider id|
|»» balance|number(float)|true|none|Balanced recorded|
|»» coming|number(float)|false|none|Coming debit recorded|
|»» timestamp|string(date-time)|true|none|Timestamp of log|
|»» error|string|false|none|If fail, contains the error code|
|»» error_message|string|false|none|If fail, error message received from bank or provider|
|»» id_connection_log|integer|false|none|ID of the related connection log|

<aside class="success">
This operation does not require authentication
</aside>

## Get transactions

> Code samples

```http
GET /demo.biapi.pro/2.0/users/{id_user}/transactions HTTP/1.1

Accept: application/json

```

```python
import requests
headers = {
  'Accept': 'application/json'
}

r = requests.get('/demo.biapi.pro/2.0/users/{id_user}/transactions', params={

}, headers = headers)

print r.json()

```

`GET /users/{id_user}/transactions`

Get list of transactions.<br><br>By default, it selects transactions for the last month. You can use "min_date" and "max_date" to change boundary dates.<br><br>

<h3 id="get-transactions-parameters">Parameters</h3>

|Name|In|Type|Required|Description|
|---|---|---|---|---|
|id_user|path|string|true|Hint: you can use 'me' or 'all'|
|limit|query|integer|false|limit number of results|
|offset|query|integer|false|offset of first result|
|min_date|query|string(date)|false|minimal (inclusive) date|
|max_date|query|string(date)|false|maximum (inclusive) date|
|income|query|boolean|false|filter on income or expenditures|
|deleted|query|boolean|false|display only deleted transactions|
|all|query|boolean|false|display all transactions, including deleted ones|
|last_update|query|string(date-time)|false|get only transactions updated after the specified datetime|
|wording|query|string|false|filter transactions containing the given string|
|min_value|query|number(float)|false|minimal (inclusive) value|
|max_value|query|number(float)|false|maximum (inclusive) value|
|search|query|string|false|search in labels, dates, values and categories|
|value|query|string|false|"XX|-XX" or "±XX"|
|expand|query|string|false|none|

> Example responses

> 200 Response

```json
{
  "transactions": [
    {
      "id": 0,
      "id_account": 0,
      "webid": "",
      "application_date": "2019-03-20",
      "date": "2019-03-20",
      "value": 0,
      "gross_value": 0,
      "nature": "inconnu",
      "wording": "",
      "id_category": 0,
      "state": "new",
      "date_scraped": "2019-03-20 11:42:25.569614",
      "rdate": "2019-03-20",
      "vdate": "2019-03-20",
      "coming": false,
      "active": true,
      "id_cluster": 0,
      "comment": "",
      "last_update": "2019-03-20 11:42:25.570182",
      "deleted": "2019-03-20 11:42:25.570269",
      "nopurge": false,
      "original_value": 0,
      "original_currency": "",
      "commission": 0,
      "commission_currency": "",
      "country": "",
      "counterparty": "",
      "card": ""
    }
  ]
}
```

<h3 id="get-transactions-responses">Responses</h3>

|Status|Meaning|Description|Schema|
|---|---|---|---|
|200|[OK](https://tools.ietf.org/html/rfc7231#section-6.3.1)|transactions|Inline|

<h3 id="get-transactions-responseschema">Response Schema</h3>

Status Code **200**

|Name|Type|Required|Restrictions|Description|
|---|---|---|---|---|
|» transactions|[[Transaction](#schematransaction)]|true|none|none|
|»» id|integer|true|none|ID of the transaction|
|»» id_account|integer|true|none|ID of the related account|
|»» webid|string|false|none|Webid of the transaction|
|»» application_date|string(date)|false|none|Date considered by PFM services. It is used to change the month of a transaction, for example.|
|»» date|string(date)|true|none|Debit date|
|»» value|number(float)|false|none|Value of the transaction|
|»» gross_value|number(float)|false|none|Gross value of the transaction|
|»» nature|string|true|none|Type of transaction|
|»» original_wording|string|true|none|Full label of the transaction|
|»» simplified_wording|string|true|none|Simplified label of the transaction|
|»» stemmed_wording|string|true|none|Do not use it|
|»» wording|string|false|none|Label set by the user|
|»» id_category|integer|false|none|ID of the related category|
|»» state|string|true|none|Internal state of the transaction|
|»» date_scraped|string(date-time)|true|none|Date when the transaction has been seen|
|»» rdate|string(date)|true|none|Realization of the transaction|
|»» vdate|string(date)|false|none|Value date of the transaction|
|»» coming|boolean|true|none|If true, this transaction hasn't been yet debited|
|»» active|boolean|true|none|If false, PFM services will ignore this transaction|
|»» id_cluster|integer|false|none|If the transaction is part of a cluster|
|»» comment|string|false|none|User comment|
|»» last_update|string(date-time)|false|none|Last update of the transaction|
|»» deleted|string(date-time)|false|none|If set, this transaction has been removed from the bank|
|»» nopurge|boolean|true|none|If set to true, this transaction will never be considered as deleted|
|»» original_value|number(float)|false|none|Value in the original currency|
|»» original_currency|object|false|none|Original currency|
|»» commission|number(float)|false|none|Commission taken on the transaction|
|»» commission_currency|object|false|none|Commission currency|
|»» country|string|false|none|Original country|
|»» counterparty|string|false|none|Counterparty|
|»» card|string|false|none|Card number associated to the transaction|

<aside class="success">
This operation does not require authentication
</aside>

## Create transactions

> Code samples

```http
POST /demo.biapi.pro/2.0/users/{id_user}/transactions HTTP/1.1

Content-Type: multipart/form-data
Accept: application/json

```

```python
import requests
headers = {
  'Content-Type': 'multipart/form-data',
  'Accept': 'application/json'
}

r = requests.post('/demo.biapi.pro/2.0/users/{id_user}/transactions', params={

}, headers = headers)

print r.json()

```

`POST /users/{id_user}/transactions`

Create transactions for the supplied account or the account whose id is given with form parameters. It requires an array of transaction dictionaries.<br><br><br><br>

> Body parameter

```yaml
original_wording: string
value: 0
date: '2019-03-20'
type: string
state: string
rdate: '2019-03-20'
coming: true
active: true
date_scraped: '2019-03-20T12:38:36Z'
id_account: 0

```

<h3 id="create-transactions-parameters">Parameters</h3>

|Name|In|Type|Required|Description|
|---|---|---|---|---|
|id_user|path|string|true|Hint: you can use 'me' or 'all'|
|expand|query|string|false|none|
|body|body|[postUsers_idUser_accounts_idAccount_transactions](#schemapostusers_iduser_accounts_idaccount_transactions)|false|none|
|» original_wording|body|string|true|label of the transaction|
|» value|body|integer|true|vallue of the transaction|
|» date|body|string(date)|true|date of the transaction|
|» type|body|string|false|type of the transaction (default: unknown)|
|» state|body|string|false|nature of the transaction (default: new)|
|» rdate|body|string(date)|false|realisation date of the transaction (default: value of date)|
|» coming|body|boolean|false|1 if the transaction has already been debited (default: 0)|
|» active|body|boolean|false|1 if the transaction should be taken into account by pfm services (default: 1)|
|» date_scraped|body|string(date-time)|false|date on which the transaction has been found for the first time. YYYY-MM-DD HH:MM:SS(default: now)|
|» id_account|body|integer|false|account of the transaction. If not supplied, it has to be given in the route|

> Example responses

> 200 Response

```json
{
  "id": 0,
  "id_account": 0,
  "webid": "",
  "application_date": "2019-03-20",
  "date": "2019-03-20",
  "value": 0,
  "gross_value": 0,
  "nature": "inconnu",
  "wording": "",
  "id_category": 0,
  "state": "new",
  "date_scraped": "2019-03-20 11:42:25.569614",
  "rdate": "2019-03-20",
  "vdate": "2019-03-20",
  "coming": false,
  "active": true,
  "id_cluster": 0,
  "comment": "",
  "last_update": "2019-03-20 11:42:25.570182",
  "deleted": "2019-03-20 11:42:25.570269",
  "nopurge": false,
  "original_value": 0,
  "original_currency": "",
  "commission": 0,
  "commission_currency": "",
  "country": "",
  "counterparty": "",
  "card": ""
}
```

<h3 id="create-transactions-responses">Responses</h3>

|Status|Meaning|Description|Schema|
|---|---|---|---|
|200|[OK](https://tools.ietf.org/html/rfc7231#section-6.3.1)|Successful POST on Transaction resource|[Transaction](#schematransaction)|

<aside class="success">
This operation does not require authentication
</aside>

## Delete transactions

> Code samples

```http
DELETE /demo.biapi.pro/2.0/users/{id_user}/transactions HTTP/1.1

Accept: application/json

```

```python
import requests
headers = {
  'Accept': 'application/json'
}

r = requests.delete('/demo.biapi.pro/2.0/users/{id_user}/transactions', params={

}, headers = headers)

print r.json()

```

`DELETE /users/{id_user}/transactions`

<h3 id="delete-transactions-parameters">Parameters</h3>

|Name|In|Type|Required|Description|
|---|---|---|---|---|
|id_user|path|string|true|Hint: you can use 'me' or 'all'|
|expand|query|string|false|none|

> Example responses

> 200 Response

```json
{
  "id": 0,
  "id_account": 0,
  "webid": "",
  "application_date": "2019-03-20",
  "date": "2019-03-20",
  "value": 0,
  "gross_value": 0,
  "nature": "inconnu",
  "wording": "",
  "id_category": 0,
  "state": "new",
  "date_scraped": "2019-03-20 11:42:25.569614",
  "rdate": "2019-03-20",
  "vdate": "2019-03-20",
  "coming": false,
  "active": true,
  "id_cluster": 0,
  "comment": "",
  "last_update": "2019-03-20 11:42:25.570182",
  "deleted": "2019-03-20 11:42:25.570269",
  "nopurge": false,
  "original_value": 0,
  "original_currency": "",
  "commission": 0,
  "commission_currency": "",
  "country": "",
  "counterparty": "",
  "card": ""
}
```

<h3 id="delete-transactions-responses">Responses</h3>

|Status|Meaning|Description|Schema|
|---|---|---|---|
|200|[OK](https://tools.ietf.org/html/rfc7231#section-6.3.1)|Successful DELETE on Transaction resource|[Transaction](#schematransaction)|

<aside class="success">
This operation does not require authentication
</aside>

## Edit a transaction meta-data

> Code samples

```http
PUT /demo.biapi.pro/2.0/users/{id_user}/transactions/{id_transaction} HTTP/1.1

Content-Type: multipart/form-data
Accept: application/json

```

```python
import requests
headers = {
  'Content-Type': 'multipart/form-data',
  'Accept': 'application/json'
}

r = requests.put('/demo.biapi.pro/2.0/users/{id_user}/transactions/{id_transaction}', params={

}, headers = headers)

print r.json()

```

`PUT /users/{id_user}/transactions/{id_transaction}`

> Body parameter

```yaml
wording: string
application_date: '2019-03-20'
id_category: 0
comment: string
active: true

```

<h3 id="edit-a-transaction-meta-data-parameters">Parameters</h3>

|Name|In|Type|Required|Description|
|---|---|---|---|---|
|id_user|path|string|true|Hint: you can use 'me' or 'all'|
|id_transaction|path|integer|true|none|
|expand|query|string|false|none|
|body|body|[putUsers_idUser_accounts_idAccount_transactions_idTransaction_](#schemaputusers_iduser_accounts_idaccount_transactions_idtransaction_)|false|none|
|» wording|body|string|false|user rewording of the transaction|
|» application_date|body|string(date)|false|change application date of the transaction|
|» id_category|body|integer|false|ID of the associated category|
|» comment|body|string|false|change comment|
|» active|body|boolean|false|if false, transaction isn't considered in analyzisis endpoints (like /balances)|

> Example responses

> 200 Response

```json
{
  "id": 0,
  "id_account": 0,
  "webid": "",
  "application_date": "2019-03-20",
  "date": "2019-03-20",
  "value": 0,
  "gross_value": 0,
  "nature": "inconnu",
  "wording": "",
  "id_category": 0,
  "state": "new",
  "date_scraped": "2019-03-20 11:42:25.569614",
  "rdate": "2019-03-20",
  "vdate": "2019-03-20",
  "coming": false,
  "active": true,
  "id_cluster": 0,
  "comment": "",
  "last_update": "2019-03-20 11:42:25.570182",
  "deleted": "2019-03-20 11:42:25.570269",
  "nopurge": false,
  "original_value": 0,
  "original_currency": "",
  "commission": 0,
  "commission_currency": "",
  "country": "",
  "counterparty": "",
  "card": ""
}
```

<h3 id="edit-a-transaction-meta-data-responses">Responses</h3>

|Status|Meaning|Description|Schema|
|---|---|---|---|
|200|[OK](https://tools.ietf.org/html/rfc7231#section-6.3.1)|Successful PUT on Transaction resource|[Transaction](#schematransaction)|

<aside class="success">
This operation does not require authentication
</aside>

## List all arbitrary key-value pairs on a transaction

> Code samples

```http
GET /demo.biapi.pro/2.0/users/{id_user}/transactions/{id_transaction}/informations HTTP/1.1

Accept: application/json

```

```python
import requests
headers = {
  'Accept': 'application/json'
}

r = requests.get('/demo.biapi.pro/2.0/users/{id_user}/transactions/{id_transaction}/informations', params={

}, headers = headers)

print r.json()

```

`GET /users/{id_user}/transactions/{id_transaction}/informations`

<h3 id="list-all-arbitrary-key-value-pairs-on-a-transaction-parameters">Parameters</h3>

|Name|In|Type|Required|Description|
|---|---|---|---|---|
|id_user|path|string|true|Hint: you can use 'me' or 'all'|
|id_transaction|path|integer|true|none|
|expand|query|string|false|none|

> Example responses

> 200 Response

```json
{
  "transactioninformations": [
    {
      "id": 0,
      "id_transaction": 0,
      "key": "",
      "value": ""
    }
  ]
}
```

<h3 id="list-all-arbitrary-key-value-pairs-on-a-transaction-responses">Responses</h3>

|Status|Meaning|Description|Schema|
|---|---|---|---|
|200|[OK](https://tools.ietf.org/html/rfc7231#section-6.3.1)|transactioninformations|Inline|

<h3 id="list-all-arbitrary-key-value-pairs-on-a-transaction-responseschema">Response Schema</h3>

Status Code **200**

|Name|Type|Required|Restrictions|Description|
|---|---|---|---|---|
|» transactioninformations|[[TransactionInformation](#schematransactioninformation)]|true|none|none|
|»» id|integer|true|none|ID of this transaction information|
|»» id_transaction|integer|true|none|ID of the related transaction|
|»» key|string|true|none|Key of the transaction information|
|»» value|string|false|none|Value of the transaction information|

<aside class="success">
This operation does not require authentication
</aside>

## Add or edit transaction arbitrary key-value pairs

> Code samples

```http
PUT /demo.biapi.pro/2.0/users/{id_user}/transactions/{id_transaction}/informations HTTP/1.1

Accept: application/json

```

```python
import requests
headers = {
  'Accept': 'application/json'
}

r = requests.put('/demo.biapi.pro/2.0/users/{id_user}/transactions/{id_transaction}/informations', params={

}, headers = headers)

print r.json()

```

`PUT /users/{id_user}/transactions/{id_transaction}/informations`

<h3 id="add-or-edit-transaction-arbitrary-key-value-pairs-parameters">Parameters</h3>

|Name|In|Type|Required|Description|
|---|---|---|---|---|
|id_user|path|string|true|Hint: you can use 'me' or 'all'|
|id_transaction|path|integer|true|none|
|expand|query|string|false|none|

> Example responses

> 200 Response

```json
{
  "id": 0,
  "id_transaction": 0,
  "key": "",
  "value": ""
}
```

<h3 id="add-or-edit-transaction-arbitrary-key-value-pairs-responses">Responses</h3>

|Status|Meaning|Description|Schema|
|---|---|---|---|
|200|[OK](https://tools.ietf.org/html/rfc7231#section-6.3.1)|Successful PUT on TransactionInformation resource|[TransactionInformation](#schematransactioninformation)|

<aside class="success">
This operation does not require authentication
</aside>

## Delete all arbitrary key-value pairs of a transaction

> Code samples

```http
DELETE /demo.biapi.pro/2.0/users/{id_user}/transactions/{id_transaction}/informations HTTP/1.1

Accept: application/json

```

```python
import requests
headers = {
  'Accept': 'application/json'
}

r = requests.delete('/demo.biapi.pro/2.0/users/{id_user}/transactions/{id_transaction}/informations', params={

}, headers = headers)

print r.json()

```

`DELETE /users/{id_user}/transactions/{id_transaction}/informations`

<h3 id="delete-all-arbitrary-key-value-pairs-of-a-transaction-parameters">Parameters</h3>

|Name|In|Type|Required|Description|
|---|---|---|---|---|
|id_user|path|string|true|Hint: you can use 'me' or 'all'|
|id_transaction|path|integer|true|none|
|expand|query|string|false|none|

> Example responses

> 200 Response

```json
{
  "id": 0,
  "id_transaction": 0,
  "key": "",
  "value": ""
}
```

<h3 id="delete-all-arbitrary-key-value-pairs-of-a-transaction-responses">Responses</h3>

|Status|Meaning|Description|Schema|
|---|---|---|---|
|200|[OK](https://tools.ietf.org/html/rfc7231#section-6.3.1)|Successful DELETE on TransactionInformation resource|[TransactionInformation](#schematransactioninformation)|

<aside class="success">
This operation does not require authentication
</aside>

## Get a particular arbitrary key-value pair on a transaction

> Code samples

```http
GET /demo.biapi.pro/2.0/users/{id_user}/transactions/{id_transaction}/informations/{id_information} HTTP/1.1

Accept: application/json

```

```python
import requests
headers = {
  'Accept': 'application/json'
}

r = requests.get('/demo.biapi.pro/2.0/users/{id_user}/transactions/{id_transaction}/informations/{id_information}', params={

}, headers = headers)

print r.json()

```

`GET /users/{id_user}/transactions/{id_transaction}/informations/{id_information}`

<h3 id="get-a-particular-arbitrary-key-value-pair-on-a-transaction-parameters">Parameters</h3>

|Name|In|Type|Required|Description|
|---|---|---|---|---|
|id_user|path|string|true|Hint: you can use 'me' or 'all'|
|id_transaction|path|integer|true|none|
|id_information|path|integer|true|none|
|expand|query|string|false|none|

> Example responses

> 200 Response

```json
{
  "id": 0,
  "id_transaction": 0,
  "key": "",
  "value": ""
}
```

<h3 id="get-a-particular-arbitrary-key-value-pair-on-a-transaction-responses">Responses</h3>

|Status|Meaning|Description|Schema|
|---|---|---|---|
|200|[OK](https://tools.ietf.org/html/rfc7231#section-6.3.1)|Successful GET on TransactionInformation resource|[TransactionInformation](#schematransactioninformation)|

<aside class="success">
This operation does not require authentication
</aside>

## Delete a particular key-value pair on a transaction.

> Code samples

```http
DELETE /demo.biapi.pro/2.0/users/{id_user}/transactions/{id_transaction}/informations/{id_information} HTTP/1.1

Accept: application/json

```

```python
import requests
headers = {
  'Accept': 'application/json'
}

r = requests.delete('/demo.biapi.pro/2.0/users/{id_user}/transactions/{id_transaction}/informations/{id_information}', params={

}, headers = headers)

print r.json()

```

`DELETE /users/{id_user}/transactions/{id_transaction}/informations/{id_information}`

<h3 id="delete-a-particular-key-value-pair-on-a-transaction.-parameters">Parameters</h3>

|Name|In|Type|Required|Description|
|---|---|---|---|---|
|id_user|path|string|true|Hint: you can use 'me' or 'all'|
|id_transaction|path|integer|true|none|
|id_information|path|integer|true|none|
|expand|query|string|false|none|

> Example responses

> 200 Response

```json
{
  "id": 0,
  "id_transaction": 0,
  "key": "",
  "value": ""
}
```

<h3 id="delete-a-particular-key-value-pair-on-a-transaction.-responses">Responses</h3>

|Status|Meaning|Description|Schema|
|---|---|---|---|
|200|[OK](https://tools.ietf.org/html/rfc7231#section-6.3.1)|Successful DELETE on TransactionInformation resource|[TransactionInformation](#schematransactioninformation)|

<aside class="success">
This operation does not require authentication
</aside>

## Get clustered transactions

> Code samples

```http
GET /demo.biapi.pro/2.0/users/{id_user}/transactionsclusters HTTP/1.1

Accept: application/json

```

```python
import requests
headers = {
  'Accept': 'application/json'
}

r = requests.get('/demo.biapi.pro/2.0/users/{id_user}/transactionsclusters', params={

}, headers = headers)

print r.json()

```

`GET /users/{id_user}/transactionsclusters`

<h3 id="get-clustered-transactions-parameters">Parameters</h3>

|Name|In|Type|Required|Description|
|---|---|---|---|---|
|id_user|path|string|true|Hint: you can use 'me' or 'all'|
|expand|query|string|false|none|

> Example responses

> 200 Response

```json
{
  "transactionsclusters": [
    {
      "id": 0,
      "id_account": 0,
      "mean_amount": 0,
      "median_increment": 0,
      "enabled": true,
      "next_date": "2019-03-20",
      "wording": "",
      "id_category": 0,
      "created_by": ""
    }
  ]
}
```

<h3 id="get-clustered-transactions-responses">Responses</h3>

|Status|Meaning|Description|Schema|
|---|---|---|---|
|200|[OK](https://tools.ietf.org/html/rfc7231#section-6.3.1)|transactionsclusters|Inline|

<h3 id="get-clustered-transactions-responseschema">Response Schema</h3>

Status Code **200**

|Name|Type|Required|Restrictions|Description|
|---|---|---|---|---|
|» transactionsclusters|[[TransactionsCluster](#schematransactionscluster)]|true|none|none|
|»» id|integer|true|none|none|
|»» id_account|integer|true|none|none|
|»» mean_amount|number(float)|true|none|none|
|»» median_increment|integer|false|none|none|
|»» enabled|boolean|true|none|none|
|»» next_date|string(date)|false|none|none|
|»» wording|string|true|none|none|
|»» id_category|integer|false|none|none|
|»» created_by|string|false|none|none|

<aside class="success">
This operation does not require authentication
</aside>

## Create clustered transaction

> Code samples

```http
POST /demo.biapi.pro/2.0/users/{id_user}/transactionsclusters HTTP/1.1

Accept: application/json

```

```python
import requests
headers = {
  'Accept': 'application/json'
}

r = requests.post('/demo.biapi.pro/2.0/users/{id_user}/transactionsclusters', params={

}, headers = headers)

print r.json()

```

`POST /users/{id_user}/transactionsclusters`

Form params : - next_date (date) required: Date of transaction - mean_amount (decimal) required: Mean Amount - wording (string) required: name of transaction - id_account (id) required: related account<br><br>

<h3 id="create-clustered-transaction-parameters">Parameters</h3>

|Name|In|Type|Required|Description|
|---|---|---|---|---|
|id_user|path|string|true|Hint: you can use 'me' or 'all'|
|expand|query|string|false|none|

> Example responses

> 200 Response

```json
{
  "id": 0,
  "id_account": 0,
  "mean_amount": 0,
  "median_increment": 0,
  "enabled": true,
  "next_date": "2019-03-20",
  "wording": "",
  "id_category": 0,
  "created_by": ""
}
```

<h3 id="create-clustered-transaction-responses">Responses</h3>

|Status|Meaning|Description|Schema|
|---|---|---|---|
|200|[OK](https://tools.ietf.org/html/rfc7231#section-6.3.1)|Successful POST on TransactionsCluster resource|[TransactionsCluster](#schematransactionscluster)|

<aside class="success">
This operation does not require authentication
</aside>

## Edit a clustered transaction

> Code samples

```http
PUT /demo.biapi.pro/2.0/users/{id_user}/transactionsclusters/{id_transactionscluster} HTTP/1.1

Accept: application/json

```

```python
import requests
headers = {
  'Accept': 'application/json'
}

r = requests.put('/demo.biapi.pro/2.0/users/{id_user}/transactionsclusters/{id_transactionscluster}', params={

}, headers = headers)

print r.json()

```

`PUT /users/{id_user}/transactionsclusters/{id_transactionscluster}`

Form params : - next_date (date): Date of transaction - mean_amount (decimal): Mean Amount - wording (string): name of transaction - id_account (id): related account - id_category (id): related category - enabled (bool): is enabled<br><br>

<h3 id="edit-a-clustered-transaction-parameters">Parameters</h3>

|Name|In|Type|Required|Description|
|---|---|---|---|---|
|id_user|path|string|true|Hint: you can use 'me' or 'all'|
|id_transactionscluster|path|integer|true|none|
|expand|query|string|false|none|

> Example responses

> 200 Response

```json
{
  "id": 0,
  "id_account": 0,
  "mean_amount": 0,
  "median_increment": 0,
  "enabled": true,
  "next_date": "2019-03-20",
  "wording": "",
  "id_category": 0,
  "created_by": ""
}
```

<h3 id="edit-a-clustered-transaction-responses">Responses</h3>

|Status|Meaning|Description|Schema|
|---|---|---|---|
|200|[OK](https://tools.ietf.org/html/rfc7231#section-6.3.1)|Successful PUT on TransactionsCluster resource|[TransactionsCluster](#schematransactionscluster)|

<aside class="success">
This operation does not require authentication
</aside>

## Delete a clustered transaction

> Code samples

```http
DELETE /demo.biapi.pro/2.0/users/{id_user}/transactionsclusters/{id_transactionscluster} HTTP/1.1

Accept: application/json

```

```python
import requests
headers = {
  'Accept': 'application/json'
}

r = requests.delete('/demo.biapi.pro/2.0/users/{id_user}/transactionsclusters/{id_transactionscluster}', params={

}, headers = headers)

print r.json()

```

`DELETE /users/{id_user}/transactionsclusters/{id_transactionscluster}`

<h3 id="delete-a-clustered-transaction-parameters">Parameters</h3>

|Name|In|Type|Required|Description|
|---|---|---|---|---|
|id_user|path|string|true|Hint: you can use 'me' or 'all'|
|id_transactionscluster|path|integer|true|none|
|expand|query|string|false|none|

> Example responses

> 200 Response

```json
{
  "id": 0,
  "id_account": 0,
  "mean_amount": 0,
  "median_increment": 0,
  "enabled": true,
  "next_date": "2019-03-20",
  "wording": "",
  "id_category": 0,
  "created_by": ""
}
```

<h3 id="delete-a-clustered-transaction-responses">Responses</h3>

|Status|Meaning|Description|Schema|
|---|---|---|---|
|200|[OK](https://tools.ietf.org/html/rfc7231#section-6.3.1)|Successful DELETE on TransactionsCluster resource|[TransactionsCluster](#schematransactionscluster)|

<aside class="success">
This operation does not require authentication
</aside>

## Get alerts

> Code samples

```http
GET /demo.biapi.pro/2.0/users/{id_user}/alerts HTTP/1.1

Accept: application/json

```

```python
import requests
headers = {
  'Accept': 'application/json'
}

r = requests.get('/demo.biapi.pro/2.0/users/{id_user}/alerts', params={

}, headers = headers)

print r.json()

```

`GET /users/{id_user}/alerts`

<h3 id="get-alerts-parameters">Parameters</h3>

|Name|In|Type|Required|Description|
|---|---|---|---|---|
|id_user|path|string|true|Hint: you can use 'me' or 'all'|
|expand|query|string|false|none|

> Example responses

> 200 Response

```json
{
  "alerts": [
    {
      "id": 0,
      "id_user": 0,
      "timestamp": "CURRENT_TIMESTAMP",
      "type": "",
      "id_transaction": 0,
      "id_account": 0,
      "value": 0,
      "id_investment": 0
    }
  ]
}
```

<h3 id="get-alerts-responses">Responses</h3>

|Status|Meaning|Description|Schema|
|---|---|---|---|
|200|[OK](https://tools.ietf.org/html/rfc7231#section-6.3.1)|alerts|Inline|

<h3 id="get-alerts-responseschema">Response Schema</h3>

Status Code **200**

|Name|Type|Required|Restrictions|Description|
|---|---|---|---|---|
|» alerts|[[Alert](#schemaalert)]|true|none|none|
|»» id|integer|true|none|none|
|»» id_user|integer|true|none|ID of the related user|
|»» timestamp|string(date-time)|true|none|Date of the alerts emission|
|»» type|string|true|none|Type of the alert|
|»» id_transaction|integer|false|none|ID of the related transaction|
|»» id_account|integer|false|none|ID of the related account|
|»» value|number(float)|true|none|Amount related to the alert|
|»» id_investment|integer|false|none|ID of the related investment|

<aside class="success">
This operation does not require authentication
</aside>

## Create a new transaction category

> Code samples

```http
POST /demo.biapi.pro/2.0/users/{id_user}/categories/full HTTP/1.1

Content-Type: multipart/form-data
Accept: application/json

```

```python
import requests
headers = {
  'Content-Type': 'multipart/form-data',
  'Accept': 'application/json'
}

r = requests.post('/demo.biapi.pro/2.0/users/{id_user}/categories/full', params={

}, headers = headers)

print r.json()

```

`POST /users/{id_user}/categories/full`

> Body parameter

```yaml
name: string
id_parent_category: 0
id_parent_category_in_menu: 0
color: string
income: true
refundable: true
accountant_account: string

```

<h3 id="create-a-new-transaction-category-parameters">Parameters</h3>

|Name|In|Type|Required|Description|
|---|---|---|---|---|
|id_user|path|string|true|Hint: you can use 'me' or 'all'|
|expand|query|string|false|none|
|body|body|object|false|none|
|» name|body|string|false|Name of the category.|
|» id_parent_category|body|integer|false|ID of the parent category.|
|» id_parent_category_in_menu|body|integer|false|ID of the parent category to be displayed.|
|» color|body|string|false|Color of the category.|
|» income|body|boolean|false|Is an income category. If null, this is both an income and an expense category.|
|» refundable|body|boolean|false|This category accepts opposite sign of transactions.|
|» accountant_account|body|string|false|Accountant account number.|

> Example responses

> 200 Response

```json
{
  "id": 0,
  "id_parent_category": 0,
  "name": "",
  "income": false,
  "color": "",
  "id_parent_category_in_menu": 0,
  "name_displayed": "",
  "refundable": false,
  "id_user": 0,
  "id_logo": 0
}
```

<h3 id="create-a-new-transaction-category-responses">Responses</h3>

|Status|Meaning|Description|Schema|
|---|---|---|---|
|200|[OK](https://tools.ietf.org/html/rfc7231#section-6.3.1)|Successful POST on Category resource|[Category](#schemacategory)|

<aside class="success">
This operation does not require authentication
</aside>

## Modify a user-created category

> Code samples

```http
PUT /demo.biapi.pro/2.0/users/{id_user}/categories/full/{id_full} HTTP/1.1

Content-Type: multipart/form-data
Accept: application/json

```

```python
import requests
headers = {
  'Content-Type': 'multipart/form-data',
  'Accept': 'application/json'
}

r = requests.put('/demo.biapi.pro/2.0/users/{id_user}/categories/full/{id_full}', params={

}, headers = headers)

print r.json()

```

`PUT /users/{id_user}/categories/full/{id_full}`

> Body parameter

```yaml
hide: string
accountant_account: string

```

<h3 id="modify-a-user-created-category-parameters">Parameters</h3>

|Name|In|Type|Required|Description|
|---|---|---|---|---|
|id_user|path|string|true|Hint: you can use 'me' or 'all'|
|id_full|path|integer|true|none|
|expand|query|string|false|none|
|body|body|object|false|none|
|» hide|body|string|false|Hide (but not delete) a category. Must be 0, 1 or toggle.|
|» accountant_account|body|string|false|Accountant account number.|

> Example responses

> 200 Response

```json
{
  "id": 0,
  "id_parent_category": 0,
  "name": "",
  "income": false,
  "color": "",
  "id_parent_category_in_menu": 0,
  "name_displayed": "",
  "refundable": false,
  "id_user": 0,
  "id_logo": 0
}
```

<h3 id="modify-a-user-created-category-responses">Responses</h3>

|Status|Meaning|Description|Schema|
|---|---|---|---|
|200|[OK](https://tools.ietf.org/html/rfc7231#section-6.3.1)|Successful PUT on Category resource|[Category](#schemacategory)|

<aside class="success">
This operation does not require authentication
</aside>

## Delete a user-created transaction category

> Code samples

```http
DELETE /demo.biapi.pro/2.0/users/{id_user}/categories/full/{id_full} HTTP/1.1

Accept: application/json

```

```python
import requests
headers = {
  'Accept': 'application/json'
}

r = requests.delete('/demo.biapi.pro/2.0/users/{id_user}/categories/full/{id_full}', params={

}, headers = headers)

print r.json()

```

`DELETE /users/{id_user}/categories/full/{id_full}`

<h3 id="delete-a-user-created-transaction-category-parameters">Parameters</h3>

|Name|In|Type|Required|Description|
|---|---|---|---|---|
|id_user|path|string|true|Hint: you can use 'me' or 'all'|
|id_full|path|integer|true|none|
|expand|query|string|false|none|

> Example responses

> 200 Response

```json
{
  "id": 0,
  "id_parent_category": 0,
  "name": "",
  "income": false,
  "color": "",
  "id_parent_category_in_menu": 0,
  "name_displayed": "",
  "refundable": false,
  "id_user": 0,
  "id_logo": 0
}
```

<h3 id="delete-a-user-created-transaction-category-responses">Responses</h3>

|Status|Meaning|Description|Schema|
|---|---|---|---|
|200|[OK](https://tools.ietf.org/html/rfc7231#section-6.3.1)|Successful DELETE on Category resource|[Category](#schemacategory)|

<aside class="success">
This operation does not require authentication
</aside>

## Get forecast

> Code samples

```http
GET /demo.biapi.pro/2.0/users/{id_user}/forecast HTTP/1.1

```

```python
import requests

r = requests.get('/demo.biapi.pro/2.0/users/{id_user}/forecast', params={

)

print r.json()

```

`GET /users/{id_user}/forecast`

<h3 id="get-forecast-parameters">Parameters</h3>

|Name|In|Type|Required|Description|
|---|---|---|---|---|
|id_user|path|string|true|Hint: you can use 'me' or 'all'|

<h3 id="get-forecast-responses">Responses</h3>

|Status|Meaning|Description|Schema|
|---|---|---|---|
|200|[OK](https://tools.ietf.org/html/rfc7231#section-6.3.1)|standard HTTP response|None|

<aside class="success">
This operation does not require authentication
</aside>

<h1 id="budgea-api-documentation-connections">Connections</h1>

## Get list of connectors

> Code samples

```http
GET /demo.biapi.pro/2.0/providers HTTP/1.1

Accept: application/json

```

```python
import requests
headers = {
  'Accept': 'application/json'
}

r = requests.get('/demo.biapi.pro/2.0/providers', params={

}, headers = headers)

print r.json()

```

`GET /providers`

<h3 id="get-list-of-connectors-parameters">Parameters</h3>

|Name|In|Type|Required|Description|
|---|---|---|---|---|
|expand|query|string|false|none|

> Example responses

> 200 Response

```json
{
  "providers": [
    {
      "id": 0,
      "name": "",
      "id_weboob": "",
      "hidden": false,
      "charged": true,
      "code": "",
      "beta": false,
      "color": "",
      "slug": "",
      "sync_frequency": 0,
      "months_to_fetch": 0,
      "auth_mechanism": ""
    }
  ]
}
```

<h3 id="get-list-of-connectors-responses">Responses</h3>

|Status|Meaning|Description|Schema|
|---|---|---|---|
|200|[OK](https://tools.ietf.org/html/rfc7231#section-6.3.1)|providers|Inline|

<h3 id="get-list-of-connectors-responseschema">Response Schema</h3>

Status Code **200**

|Name|Type|Required|Restrictions|Description|
|---|---|---|---|---|
|» providers|[[Connector](#schemaconnector)]|true|none|none|
|»» id|integer|true|none|ID of the connector|
|»» name|string|true|none|Name of the bank or provider|
|»» id_weboob|string|true|none|none|
|»» hidden|boolean|false|none|This connector is hidden from your users|
|»» charged|boolean|true|none|Usage of this connector is charged|
|»» code|string|false|none|Bank code|
|»» beta|boolean|true|none|If true, this connector is perhaps unstable :)|
|»» color|string|false|none|Main color of the bank or provider|
|»» slug|string|false|none|none|
|»» sync_frequency|number(float)|false|none|How many days to wait between syncs|
|»» months_to_fetch|integer|false|none|How many months of history to fetch|
|»» auth_mechanism|string|false|none|Authentication mechanism to use|

<aside class="success">
This operation does not require authentication
</aside>

## Get connections without a user

> Code samples

```http
GET /demo.biapi.pro/2.0/connections HTTP/1.1

Accept: application/json

```

```python
import requests
headers = {
  'Accept': 'application/json'
}

r = requests.get('/demo.biapi.pro/2.0/connections', params={

}, headers = headers)

print r.json()

```

`GET /connections`

<h3 id="get-connections-without-a-user-parameters">Parameters</h3>

|Name|In|Type|Required|Description|
|---|---|---|---|---|
|expand|query|string|false|none|

> Example responses

> 200 Response

```json
{
  "connections": [
    {
      "id": 0,
      "id_user": 0,
      "id_connector": 0,
      "last_update": "2019-03-20 11:42:25.591002",
      "error": "",
      "expire": "2019-03-20 11:42:25.591259",
      "active": true,
      "last_push": "2019-03-20 11:42:25.591429",
      "next_try": "2019-03-20 11:42:25.591684"
    }
  ]
}
```

<h3 id="get-connections-without-a-user-responses">Responses</h3>

|Status|Meaning|Description|Schema|
|---|---|---|---|
|200|[OK](https://tools.ietf.org/html/rfc7231#section-6.3.1)|connections|Inline|

<h3 id="get-connections-without-a-user-responseschema">Response Schema</h3>

Status Code **200**

|Name|Type|Required|Restrictions|Description|
|---|---|---|---|---|
|» connections|[[Connection](#schemaconnection)]|true|none|none|
|»» id|integer|true|none|ID of connection|
|»» id_user|integer|false|none|ID of the related user|
|»» id_connector|integer|true|none|ID of the related connector|
|»» last_update|string(date-time)|false|none|Last successful update|
|»» error|string|false|none|If the last update has failed, the error code|
|»» expire|string(date-time)|false|none|Expiration of the connection. Used during add of a two-factor authentication, to purge the connection if the user abort|
|»» active|boolean|true|none|This connection is active and will be automatically synced|
|»» last_push|string(date-time)|false|none|Last successful push|
|»» next_try|string(date-time)|false|none|Date of next synchronization|

<aside class="success">
This operation does not require authentication
</aside>

## Request a new connector

> Code samples

```http
POST /demo.biapi.pro/2.0/connectors HTTP/1.1

Content-Type: multipart/form-data
Accept: application/json

```

```python
import requests
headers = {
  'Content-Type': 'multipart/form-data',
  'Accept': 'application/json'
}

r = requests.post('/demo.biapi.pro/2.0/connectors', params={

}, headers = headers)

print r.json()

```

`POST /connectors`

Send a request to add a new connector<br><br>

> Body parameter

```yaml
api: string
name: string
url: string
email: string
login: string
password: string
types: string
comment: string
sendmail: true

```

<h3 id="request-a-new-connector-parameters">Parameters</h3>

|Name|In|Type|Required|Description|
|---|---|---|---|---|
|expand|query|string|false|none|
|body|body|object|false|none|
|» api|body|string|false|Name of the API|
|» name|body|string|true|Name of the bank or provider|
|» url|body|string|false|Url of the bank|
|» email|body|string|false|Email of the user|
|» login|body|string|true|Users login|
|» password|body|string|true|Users password|
|» types|body|string|false|Type of connector, eg. banks or providers|
|» comment|body|string|false|Optionnal comment|
|» sendmail|body|boolean|false|if set, send an email to user|

> Example responses

> 200 Response

```json
{
  "id": 0,
  "name": "",
  "id_weboob": "",
  "hidden": false,
  "charged": true,
  "code": "",
  "beta": false,
  "color": "",
  "slug": "",
  "sync_frequency": 0,
  "months_to_fetch": 0,
  "auth_mechanism": ""
}
```

<h3 id="request-a-new-connector-responses">Responses</h3>

|Status|Meaning|Description|Schema|
|---|---|---|---|
|200|[OK](https://tools.ietf.org/html/rfc7231#section-6.3.1)|Successful POST on Connector resource|[Connector](#schemaconnector)|

<aside class="success">
This operation does not require authentication
</aside>

## Get connection logs

> Code samples

```http
GET /demo.biapi.pro/2.0/users/{id_user}/logs HTTP/1.1

Accept: application/json

```

```python
import requests
headers = {
  'Accept': 'application/json'
}

r = requests.get('/demo.biapi.pro/2.0/users/{id_user}/logs', params={

}, headers = headers)

print r.json()

```

`GET /users/{id_user}/logs`

Get logs about connections.<br><br>

<h3 id="get-connection-logs-parameters">Parameters</h3>

|Name|In|Type|Required|Description|
|---|---|---|---|---|
|id_user|path|string|true|Hint: you can use 'me' or 'all'|
|limit|query|integer|false|limit number of results|
|offset|query|integer|false|offset of first result|
|min_date|query|string(date)|false|minimal date|
|max_date|query|string(date)|false|maximum date|
|state|query|integer|false|state of user|
|period|query|string|false|period to group logs|
|id_user|query|integer|false|ID of a user|
|id_connection|query|integer|false|ID of a connection|
|id_connector|query|integer|false|ID of a connector|
|charged|query|boolean|false|consider only logs for charged connectors|
|expand|query|string|false|none|

> Example responses

> 200 Response

```json
{
  "connectionlogs": [
    {
      "id": 0,
      "id_user": 0,
      "id_connection": 0,
      "id_connector": 0,
      "login": "",
      "error_uid": "",
      "timestamp": "CURRENT_TIMESTAMP",
      "next_try": "2019-03-20 11:42:25.606763",
      "error": "",
      "error_message": "",
      "statut": 0,
      "nb_accounts": 0,
      "start": "2019-03-20 11:42:25.607048",
      "worker": "",
      "session_folder_id": ""
    }
  ]
}
```

<h3 id="get-connection-logs-responses">Responses</h3>

|Status|Meaning|Description|Schema|
|---|---|---|---|
|200|[OK](https://tools.ietf.org/html/rfc7231#section-6.3.1)|connectionlogs|Inline|

<h3 id="get-connection-logs-responseschema">Response Schema</h3>

Status Code **200**

|Name|Type|Required|Restrictions|Description|
|---|---|---|---|---|
|» connectionlogs|[[ConnectionLog](#schemaconnectionlog)]|true|none|none|
|»» id|integer|true|none|ID of the log|
|»» id_user|integer|false|none|ID of the user|
|»» id_connection|integer|true|none|ID of the connection|
|»» id_connector|integer|false|none|ID of the connector|
|»» login|string|false|none|bcrypt hash of the login|
|»» error_uid|string|false|none|MD5 hash of the exception backtrace|
|»» timestamp|string(date-time)|true|none|Timestamp of log, when the synchronization has finished|
|»» next_try|string(date-time)|false|none|If fail, the date represents the next try to connect|
|»» error|string|false|none|If fail, contains the error code|
|»» error_message|string|false|none|If fail, error message received from connector|
|»» statut|integer|false|none|Status of user (1 = charged user)|
|»» nb_accounts|integer|false|none|In case of bank connection, number of accounts|
|»» start|string(date-time)|false|none|Timestamp when the synchronization has started|
|»» worker|string|false|none|Worker used to do synchronization|
|»» session_folder_id|string|false|none|Session folder uid|

<aside class="success">
This operation does not require authentication
</aside>

## Get connections

> Code samples

```http
GET /demo.biapi.pro/2.0/users/{id_user}/connections HTTP/1.1

Accept: application/json

```

```python
import requests
headers = {
  'Accept': 'application/json'
}

r = requests.get('/demo.biapi.pro/2.0/users/{id_user}/connections', params={

}, headers = headers)

print r.json()

```

`GET /users/{id_user}/connections`

<h3 id="get-connections-parameters">Parameters</h3>

|Name|In|Type|Required|Description|
|---|---|---|---|---|
|id_user|path|string|true|Hint: you can use 'me' or 'all'|
|expand|query|string|false|none|

> Example responses

> 200 Response

```json
{
  "connections": [
    {
      "id": 0,
      "id_user": 0,
      "id_connector": 0,
      "last_update": "2019-03-20 11:42:25.591002",
      "error": "",
      "expire": "2019-03-20 11:42:25.591259",
      "active": true,
      "last_push": "2019-03-20 11:42:25.591429",
      "next_try": "2019-03-20 11:42:25.591684"
    }
  ]
}
```

<h3 id="get-connections-responses">Responses</h3>

|Status|Meaning|Description|Schema|
|---|---|---|---|
|200|[OK](https://tools.ietf.org/html/rfc7231#section-6.3.1)|connections|Inline|

<h3 id="get-connections-responseschema">Response Schema</h3>

Status Code **200**

|Name|Type|Required|Restrictions|Description|
|---|---|---|---|---|
|» connections|[[Connection](#schemaconnection)]|true|none|none|
|»» id|integer|true|none|ID of connection|
|»» id_user|integer|false|none|ID of the related user|
|»» id_connector|integer|true|none|ID of the related connector|
|»» last_update|string(date-time)|false|none|Last successful update|
|»» error|string|false|none|If the last update has failed, the error code|
|»» expire|string(date-time)|false|none|Expiration of the connection. Used during add of a two-factor authentication, to purge the connection if the user abort|
|»» active|boolean|true|none|This connection is active and will be automatically synced|
|»» last_push|string(date-time)|false|none|Last successful push|
|»» next_try|string(date-time)|false|none|Date of next synchronization|

<aside class="success">
This operation does not require authentication
</aside>

## Add a new connection.

> Code samples

```http
POST /demo.biapi.pro/2.0/users/{id_user}/connections HTTP/1.1

Content-Type: multipart/form-data
Accept: application/json

```

```python
import requests
headers = {
  'Content-Type': 'multipart/form-data',
  'Accept': 'application/json'
}

r = requests.post('/demo.biapi.pro/2.0/users/{id_user}/connections', params={

}, headers = headers)

print r.json()

```

`POST /users/{id_user}/connections`

Create a new connection to a given bank or provider. You have to give all needed parameters (use /banks/ID/fields or /providers/ID/fields to get them).<br><br>

> Body parameter

```yaml
id_connector: 0

```

<h3 id="add-a-new-connection.-parameters">Parameters</h3>

|Name|In|Type|Required|Description|
|---|---|---|---|---|
|id_user|path|string|true|Hint: you can use 'me' or 'all'|
|expand|query|string|false|none|
|body|body|object|false|none|
|» id_connector|body|integer|false|ID of the connector|

> Example responses

> 200 Response

```json
{
  "id": 0,
  "id_user": 0,
  "id_connector": 0,
  "last_update": "2019-03-20 11:42:25.591002",
  "error": "",
  "expire": "2019-03-20 11:42:25.591259",
  "active": true,
  "last_push": "2019-03-20 11:42:25.591429",
  "next_try": "2019-03-20 11:42:25.591684"
}
```

<h3 id="add-a-new-connection.-responses">Responses</h3>

|Status|Meaning|Description|Schema|
|---|---|---|---|
|200|[OK](https://tools.ietf.org/html/rfc7231#section-6.3.1)|Successful POST on Connection resource|[Connection](#schemaconnection)|

<aside class="success">
This operation does not require authentication
</aside>

## Delete all connections

> Code samples

```http
DELETE /demo.biapi.pro/2.0/users/{id_user}/connections HTTP/1.1

Accept: application/json

```

```python
import requests
headers = {
  'Accept': 'application/json'
}

r = requests.delete('/demo.biapi.pro/2.0/users/{id_user}/connections', params={

}, headers = headers)

print r.json()

```

`DELETE /users/{id_user}/connections`

<h3 id="delete-all-connections-parameters">Parameters</h3>

|Name|In|Type|Required|Description|
|---|---|---|---|---|
|id_user|path|string|true|Hint: you can use 'me' or 'all'|
|expand|query|string|false|none|

> Example responses

> 200 Response

```json
{
  "id": 0,
  "id_user": 0,
  "id_connector": 0,
  "last_update": "2019-03-20 11:42:25.591002",
  "error": "",
  "expire": "2019-03-20 11:42:25.591259",
  "active": true,
  "last_push": "2019-03-20 11:42:25.591429",
  "next_try": "2019-03-20 11:42:25.591684"
}
```

<h3 id="delete-all-connections-responses">Responses</h3>

|Status|Meaning|Description|Schema|
|---|---|---|---|
|200|[OK](https://tools.ietf.org/html/rfc7231#section-6.3.1)|Successful DELETE on Connection resource|[Connection](#schemaconnection)|

<aside class="success">
This operation does not require authentication
</aside>

## Update a connection.

> Code samples

```http
POST /demo.biapi.pro/2.0/users/{id_user}/connections/{id_connection} HTTP/1.1

Content-Type: multipart/form-data
Accept: application/json

```

```python
import requests
headers = {
  'Content-Type': 'multipart/form-data',
  'Accept': 'application/json'
}

r = requests.post('/demo.biapi.pro/2.0/users/{id_user}/connections/{id_connection}', params={

}, headers = headers)

print r.json()

```

`POST /users/{id_user}/connections/{id_connection}`

Give new parameters to change on the configuration of this connection (for example "password").<br><br>It tests connection to website, and if it fails, a 400 response is given with the error code "wrongpass" or "websiteUnavailable".<br><br>You can also supply meta-parameters on connection, like 'active' or 'expire'.<br><br>

> Body parameter

```yaml
active: true
expire: '2019-03-20T12:38:36Z'
login: string
password: string

```

<h3 id="update-a-connection.-parameters">Parameters</h3>

|Name|In|Type|Required|Description|
|---|---|---|---|---|
|id_user|path|string|true|Hint: you can use 'me' or 'all'|
|id_connection|path|integer|true|none|
|expand|query|string|false|none|
|body|body|object|false|none|
|» active|body|boolean|false|Set if the connection synchronisation is active|
|» expire|body|string(date-time)|false|Set expiration of the connection to this date|
|» login|body|string|false|Set login to this new login|
|» password|body|string|false|Set password to this new password|

> Example responses

> 200 Response

```json
{
  "id": 0,
  "id_user": 0,
  "id_connector": 0,
  "last_update": "2019-03-20 11:42:25.591002",
  "error": "",
  "expire": "2019-03-20 11:42:25.591259",
  "active": true,
  "last_push": "2019-03-20 11:42:25.591429",
  "next_try": "2019-03-20 11:42:25.591684"
}
```

<h3 id="update-a-connection.-responses">Responses</h3>

|Status|Meaning|Description|Schema|
|---|---|---|---|
|200|[OK](https://tools.ietf.org/html/rfc7231#section-6.3.1)|Successful POST on Connection resource|[Connection](#schemaconnection)|

<aside class="success">
This operation does not require authentication
</aside>

## Force synchronisation of a connection.

> Code samples

```http
PUT /demo.biapi.pro/2.0/users/{id_user}/connections/{id_connection} HTTP/1.1

Accept: application/json

```

```python
import requests
headers = {
  'Accept': 'application/json'
}

r = requests.put('/demo.biapi.pro/2.0/users/{id_user}/connections/{id_connection}', params={

}, headers = headers)

print r.json()

```

`PUT /users/{id_user}/connections/{id_connection}`

We suggest to pass parameter expand=accounts[transactions] to get all *new* and *updated* transactions.<br><br>Query params: - expand (string): fields to expand - last_update (dateTime): if supplied, get transactions inserted since this date<br><br>

<h3 id="force-synchronisation-of-a-connection.-parameters">Parameters</h3>

|Name|In|Type|Required|Description|
|---|---|---|---|---|
|id_user|path|string|true|Hint: you can use 'me' or 'all'|
|id_connection|path|integer|true|none|
|expand|query|string|false|none|

> Example responses

> 200 Response

```json
{
  "id": 0,
  "id_user": 0,
  "id_connector": 0,
  "last_update": "2019-03-20 11:42:25.591002",
  "error": "",
  "expire": "2019-03-20 11:42:25.591259",
  "active": true,
  "last_push": "2019-03-20 11:42:25.591429",
  "next_try": "2019-03-20 11:42:25.591684"
}
```

<h3 id="force-synchronisation-of-a-connection.-responses">Responses</h3>

|Status|Meaning|Description|Schema|
|---|---|---|---|
|200|[OK](https://tools.ietf.org/html/rfc7231#section-6.3.1)|Successful PUT on Connection resource|[Connection](#schemaconnection)|

<aside class="success">
This operation does not require authentication
</aside>

## Delete a connection.

> Code samples

```http
DELETE /demo.biapi.pro/2.0/users/{id_user}/connections/{id_connection} HTTP/1.1

Accept: application/json

```

```python
import requests
headers = {
  'Accept': 'application/json'
}

r = requests.delete('/demo.biapi.pro/2.0/users/{id_user}/connections/{id_connection}', params={

}, headers = headers)

print r.json()

```

`DELETE /users/{id_user}/connections/{id_connection}`

This endpoint deletes a connection and all related accounts and transactions.<br><br>

<h3 id="delete-a-connection.-parameters">Parameters</h3>

|Name|In|Type|Required|Description|
|---|---|---|---|---|
|id_user|path|string|true|Hint: you can use 'me' or 'all'|
|id_connection|path|integer|true|none|
|expand|query|string|false|none|

> Example responses

> 200 Response

```json
{
  "id": 0,
  "id_user": 0,
  "id_connector": 0,
  "last_update": "2019-03-20 11:42:25.591002",
  "error": "",
  "expire": "2019-03-20 11:42:25.591259",
  "active": true,
  "last_push": "2019-03-20 11:42:25.591429",
  "next_try": "2019-03-20 11:42:25.591684"
}
```

<h3 id="delete-a-connection.-responses">Responses</h3>

|Status|Meaning|Description|Schema|
|---|---|---|---|
|200|[OK](https://tools.ietf.org/html/rfc7231#section-6.3.1)|Successful DELETE on Connection resource|[Connection](#schemaconnection)|

<aside class="success">
This operation does not require authentication
</aside>

## Get connection additionnal informations

> Code samples

```http
GET /demo.biapi.pro/2.0/users/{id_user}/connections/{id_connection}/informations HTTP/1.1

Accept: application/json

```

```python
import requests
headers = {
  'Accept': 'application/json'
}

r = requests.get('/demo.biapi.pro/2.0/users/{id_user}/connections/{id_connection}/informations', params={

}, headers = headers)

print r.json()

```

`GET /users/{id_user}/connections/{id_connection}/informations`

<br><br>

<h3 id="get-connection-additionnal-informations-parameters">Parameters</h3>

|Name|In|Type|Required|Description|
|---|---|---|---|---|
|id_user|path|string|true|Hint: you can use 'me' or 'all'|
|id_connection|path|integer|true|none|
|expand|query|string|false|none|

> Example responses

> 200 Response

```json
{
  "connections": [
    {
      "id": 0,
      "id_user": 0,
      "id_connector": 0,
      "last_update": "2019-03-20 11:42:25.591002",
      "error": "",
      "expire": "2019-03-20 11:42:25.591259",
      "active": true,
      "last_push": "2019-03-20 11:42:25.591429",
      "next_try": "2019-03-20 11:42:25.591684"
    }
  ]
}
```

> connections

<h3 id="get-connection-additionnal-informations-responses">Responses</h3>

|Status|Meaning|Description|Schema|
|---|---|---|---|
|200|[OK](https://tools.ietf.org/html/rfc7231#section-6.3.1)|connections|Inline|

<h3 id="get-connection-additionnal-informations-responseschema">Response Schema</h3>

Status Code **200**

|Name|Type|Required|Restrictions|Description|
|---|---|---|---|---|
|» connections|[[Connection](#schemaconnection)]|true|none|none|
|»» id|integer|true|none|ID of connection|
|»» id_user|integer|false|none|ID of the related user|
|»» id_connector|integer|true|none|ID of the related connector|
|»» last_update|string(date-time)|false|none|Last successful update|
|»» error|string|false|none|If the last update has failed, the error code|
|»» expire|string(date-time)|false|none|Expiration of the connection. Used during add of a two-factor authentication, to purge the connection if the user abort|
|»» active|boolean|true|none|This connection is active and will be automatically synced|
|»» last_push|string(date-time)|false|none|Last successful push|
|»» next_try|string(date-time)|false|none|Date of next synchronization|

<aside class="success">
This operation does not require authentication
</aside>

<h1 id="budgea-api-documentation-ocr">OCR</h1>

## Post an image with OCR

> Code samples

```http
POST /demo.biapi.pro/2.0/users/{id_user}/ocr HTTP/1.1

Content-Type: multipart/form-data

```

```python
import requests
headers = {
  'Content-Type': 'multipart/form-data'
}

r = requests.post('/demo.biapi.pro/2.0/users/{id_user}/ocr', params={

}, headers = headers)

print r.json()

```

`POST /users/{id_user}/ocr`

Post an image and apply OCR on it to obtain found meta-data.<br><br>

> Body parameter

```yaml
id_transaction: 0
file: string
name: string

```

<h3 id="post-an-image-with-ocr-parameters">Parameters</h3>

|Name|In|Type|Required|Description|
|---|---|---|---|---|
|id_user|path|string|true|Hint: you can use 'me' or 'all'|
|body|body|[postOcr](#schemapostocr)|false|none|
|» id_transaction|body|integer|false|Transaction used to help OCR to find data|
|» file|body|string(binary)|true|File of the document|
|» name|body|string|false|Name of the document|

<h3 id="post-an-image-with-ocr-responses">Responses</h3>

|Status|Meaning|Description|Schema|
|---|---|---|---|
|200|[OK](https://tools.ietf.org/html/rfc7231#section-6.3.1)|standard HTTP response|None|

<aside class="success">
This operation does not require authentication
</aside>

<h1 id="budgea-api-documentation-oidc">OIDC</h1>

## Adds an authorized redirect uri

> Code samples

```http
POST /demo.biapi.pro/2.0/oidc/whitelist HTTP/1.1

Content-Type: multipart/form-data
Accept: application/json

```

```python
import requests
headers = {
  'Content-Type': 'multipart/form-data',
  'Accept': 'application/json'
}

r = requests.post('/demo.biapi.pro/2.0/oidc/whitelist', params={

}, headers = headers)

print r.json()

```

`POST /oidc/whitelist`

It requires the authorized redirect uri to be created<br><br>

> Body parameter

```yaml
redirect_uri: string

```

<h3 id="adds-an-authorized-redirect-uri-parameters">Parameters</h3>

|Name|In|Type|Required|Description|
|---|---|---|---|---|
|expand|query|string|false|none|
|body|body|object|false|none|
|» redirect_uri|body|string|true|authorized redirect uri to be created|

> Example responses

> 200 Response

```json
{
  "id": 0,
  "redirect_uri": ""
}
```

<h3 id="adds-an-authorized-redirect-uri-responses">Responses</h3>

|Status|Meaning|Description|Schema|
|---|---|---|---|
|200|[OK](https://tools.ietf.org/html/rfc7231#section-6.3.1)|Successful POST on OidcWhitelist resource|[OidcWhitelist](#schemaoidcwhitelist)|

<aside class="success">
This operation does not require authentication
</aside>

## Edit a authorized redirect uri

> Code samples

```http
POST /demo.biapi.pro/2.0/oidc/whitelist/{id_whitelist} HTTP/1.1

Content-Type: multipart/form-data
Accept: application/json

```

```python
import requests
headers = {
  'Content-Type': 'multipart/form-data',
  'Accept': 'application/json'
}

r = requests.post('/demo.biapi.pro/2.0/oidc/whitelist/{id_whitelist}', params={

}, headers = headers)

print r.json()

```

`POST /oidc/whitelist/{id_whitelist}`

Edit the uri for the supplied authorized redirect uri.<br><br>

> Body parameter

```yaml
redirect_uri: string

```

<h3 id="edit-a-authorized-redirect-uri-parameters">Parameters</h3>

|Name|In|Type|Required|Description|
|---|---|---|---|---|
|id_whitelist|path|integer|true|none|
|expand|query|string|false|none|
|body|body|object|false|none|
|» redirect_uri|body|string|true|new authorized redirect uri|

> Example responses

> 200 Response

```json
{
  "id": 0,
  "redirect_uri": ""
}
```

<h3 id="edit-a-authorized-redirect-uri-responses">Responses</h3>

|Status|Meaning|Description|Schema|
|---|---|---|---|
|200|[OK](https://tools.ietf.org/html/rfc7231#section-6.3.1)|Successful POST on OidcWhitelist resource|[OidcWhitelist](#schemaoidcwhitelist)|

<aside class="success">
This operation does not require authentication
</aside>

## Delete the supplied authorized redirect uri

> Code samples

```http
DELETE /demo.biapi.pro/2.0/oidc/whitelist/{id_whitelist} HTTP/1.1

Accept: application/json

```

```python
import requests
headers = {
  'Accept': 'application/json'
}

r = requests.delete('/demo.biapi.pro/2.0/oidc/whitelist/{id_whitelist}', params={

}, headers = headers)

print r.json()

```

`DELETE /oidc/whitelist/{id_whitelist}`

<h3 id="delete-the-supplied-authorized-redirect-uri-parameters">Parameters</h3>

|Name|In|Type|Required|Description|
|---|---|---|---|---|
|id_whitelist|path|integer|true|none|
|expand|query|string|false|none|

> Example responses

> 200 Response

```json
{
  "id": 0,
  "redirect_uri": ""
}
```

<h3 id="delete-the-supplied-authorized-redirect-uri-responses">Responses</h3>

|Status|Meaning|Description|Schema|
|---|---|---|---|
|200|[OK](https://tools.ietf.org/html/rfc7231#section-6.3.1)|Successful DELETE on OidcWhitelist resource|[OidcWhitelist](#schemaoidcwhitelist)|

<aside class="success">
This operation does not require authentication
</aside>

<h1 id="budgea-api-documentation-pfm">PFM</h1>

## Confirm email address

> Code samples

```http
POST /demo.biapi.pro/2.0/auth/confirm HTTP/1.1

Content-Type: multipart/form-data
Accept: application/json

```

```python
import requests
headers = {
  'Content-Type': 'multipart/form-data',
  'Accept': 'application/json'
}

r = requests.post('/demo.biapi.pro/2.0/auth/confirm', params={

}, headers = headers)

print r.json()

```

`POST /auth/confirm`

<br><br>

> Body parameter

```yaml
token: string
application: string

```

<h3 id="confirm-email-address-parameters">Parameters</h3>

|Name|In|Type|Required|Description|
|---|---|---|---|---|
|body|body|object|false|none|
|» token|body|string|true|token received in email|
|» application|body|string|true|application in use|

> Example responses

> 200 Response

```json
{
  "token": "string",
  "user": {}
}
```

> OK

<h3 id="confirm-email-address-responses">Responses</h3>

|Status|Meaning|Description|Schema|
|---|---|---|---|
|200|[OK](https://tools.ietf.org/html/rfc7231#section-6.3.1)|OK|Inline|

<h3 id="confirm-email-address-responseschema">Response Schema</h3>

Status Code **200**

|Name|Type|Required|Restrictions|Description|
|---|---|---|---|---|
|» token|string|true|none|confirmed token|
|» user|object|true|none|user data object|

<aside class="success">
This operation does not require authentication
</aside>

## Confirm new email address

> Code samples

```http
POST /demo.biapi.pro/2.0/auth/confirmNewEmail HTTP/1.1

Content-Type: multipart/form-data

```

```python
import requests
headers = {
  'Content-Type': 'multipart/form-data'
}

r = requests.post('/demo.biapi.pro/2.0/auth/confirmNewEmail', params={

}, headers = headers)

print r.json()

```

`POST /auth/confirmNewEmail`

> Body parameter

```yaml
token: string

```

<h3 id="confirm-new-email-address-parameters">Parameters</h3>

|Name|In|Type|Required|Description|
|---|---|---|---|---|
|body|body|object|false|none|
|» token|body|string|true|token received by email|

<h3 id="confirm-new-email-address-responses">Responses</h3>

|Status|Meaning|Description|Schema|
|---|---|---|---|
|200|[OK](https://tools.ietf.org/html/rfc7231#section-6.3.1)|standard HTTP response|None|

<aside class="success">
This operation does not require authentication
</aside>

## Login with credentials and set as cookie

> Code samples

```http
POST /demo.biapi.pro/2.0/auth/cookie HTTP/1.1

Content-Type: multipart/form-data

```

```python
import requests
headers = {
  'Content-Type': 'multipart/form-data'
}

r = requests.post('/demo.biapi.pro/2.0/auth/cookie', params={

}, headers = headers)

print r.json()

```

`POST /auth/cookie`

> Body parameter

```yaml
username: string
password: string
application: string
scope: string

```

<h3 id="login-with-credentials-and-set-as-cookie-parameters">Parameters</h3>

|Name|In|Type|Required|Description|
|---|---|---|---|---|
|body|body|[postAuthCookie](#schemapostauthcookie)|false|none|
|» username|body|string|true|username|
|» password|body|string|true|password|
|» application|body|string|true|application name|
|» scope|body|string|false|scope requested for the token|

<h3 id="login-with-credentials-and-set-as-cookie-responses">Responses</h3>

|Status|Meaning|Description|Schema|
|---|---|---|---|
|200|[OK](https://tools.ietf.org/html/rfc7231#section-6.3.1)|standard HTTP response|None|

<aside class="success">
This operation does not require authentication
</aside>

## Register to API

> Code samples

```http
POST /demo.biapi.pro/2.0/auth/register HTTP/1.1

Content-Type: multipart/form-data
Accept: application/json

```

```python
import requests
headers = {
  'Content-Type': 'multipart/form-data',
  'Accept': 'application/json'
}

r = requests.post('/demo.biapi.pro/2.0/auth/register', params={

}, headers = headers)

print r.json()

```

`POST /auth/register`

Create a new user with his email address and password.<br><br><br><br>

> Body parameter

```yaml
email: string
password: string
application: string
sponsor: string
notification_token: string

```

<h3 id="register-to-api-parameters">Parameters</h3>

|Name|In|Type|Required|Description|
|---|---|---|---|---|
|body|body|object|false|none|
|» email|body|string|true|email address|
|» password|body|string|true|password|
|» application|body|string|true|application in use|
|» sponsor|body|string|false|sponsor code to get advantages|
|» notification_token|body|string|false|APNS or GCM token to send notifications to device|

> Example responses

> 200 Response

```json
{
  "profile": {},
  "token": "string",
  "user": {}
}
```

> OK

<h3 id="register-to-api-responses">Responses</h3>

|Status|Meaning|Description|Schema|
|---|---|---|---|
|200|[OK](https://tools.ietf.org/html/rfc7231#section-6.3.1)|OK|Inline|

<h3 id="register-to-api-responseschema">Response Schema</h3>

Status Code **200**

|Name|Type|Required|Restrictions|Description|
|---|---|---|---|---|
|» profile|object|true|none|the user profile data object|
|» token|string|true|none|the requested token|
|» user|object|true|none|the user data object|

<aside class="success">
This operation does not require authentication
</aside>

## Login to API with credentials

> Code samples

```http
POST /demo.biapi.pro/2.0/auth/token HTTP/1.1

Content-Type: multipart/form-data
Accept: application/json

```

```python
import requests
headers = {
  'Content-Type': 'multipart/form-data',
  'Accept': 'application/json'
}

r = requests.post('/demo.biapi.pro/2.0/auth/token', params={

}, headers = headers)

print r.json()

```

`POST /auth/token`

Request a new user token by giving an username and a password.<br><br><br><br>

> Body parameter

```yaml
username: string
password: string
application: string
scope: string

```

<h3 id="login-to-api-with-credentials-parameters">Parameters</h3>

|Name|In|Type|Required|Description|
|---|---|---|---|---|
|body|body|[postAuthCookie](#schemapostauthcookie)|false|none|
|» username|body|string|true|username|
|» password|body|string|true|password|
|» application|body|string|true|application name|
|» scope|body|string|false|scope requested for the token|

> Example responses

> 200 Response

```json
{
  "profile": {},
  "scope": "string",
  "token": "string",
  "expires_in": 0,
  "user": {}
}
```

> OK

<h3 id="login-to-api-with-credentials-responses">Responses</h3>

|Status|Meaning|Description|Schema|
|---|---|---|---|
|200|[OK](https://tools.ietf.org/html/rfc7231#section-6.3.1)|OK|Inline|

<h3 id="login-to-api-with-credentials-responseschema">Response Schema</h3>

Status Code **200**

|Name|Type|Required|Restrictions|Description|
|---|---|---|---|---|
|» profile|object|true|none|the user profile data object|
|» scope|string|true|none|the token scope|
|» token|string|true|none|the requested token|
|» expires_in|integer|false|none|duration in seconds of the token validity|
|» user|object|true|none|the user data object|

<aside class="success">
This operation does not require authentication
</aside>

## Get balances of accounts

> Code samples

```http
GET /demo.biapi.pro/2.0/users/{id_user}/connections/{id_connection}/balances HTTP/1.1

```

```python
import requests

r = requests.get('/demo.biapi.pro/2.0/users/{id_user}/connections/{id_connection}/balances', params={

)

print r.json()

```

`GET /users/{id_user}/connections/{id_connection}/balances`

Get balance (income/outcome/balance) of enabled accounts for the given period.<br><br>By default, min_date and max_date are the current month, and period is a single month.<br><br>The period is composed with units (days, months, years) and numbers. You can give for example "1month", "15days", "1year6months", etc.<br><br>

<h3 id="get-balances-of-accounts-parameters">Parameters</h3>

|Name|In|Type|Required|Description|
|---|---|---|---|---|
|id_user|path|string|true|Hint: you can use 'me' or 'all'|
|id_connection|path|integer|true|none|
|min_date|query|string(date)|false|minimal (inclusive) date|
|max_date|query|string(date)|false|maximal (inclusive) date|
|period|query|string|false|split output with the given period (default: month)|

<h3 id="get-balances-of-accounts-responses">Responses</h3>

|Status|Meaning|Description|Schema|
|---|---|---|---|
|200|[OK](https://tools.ietf.org/html/rfc7231#section-6.3.1)|standard HTTP response|None|

<aside class="success">
This operation does not require authentication
</aside>

## Get a list of configurated alerts

> Code samples

```http
GET /demo.biapi.pro/2.0/users/{id_user}/operationsalert HTTP/1.1

Accept: application/json

```

```python
import requests
headers = {
  'Accept': 'application/json'
}

r = requests.get('/demo.biapi.pro/2.0/users/{id_user}/operationsalert', params={

}, headers = headers)

print r.json()

```

`GET /users/{id_user}/operationsalert`

<br><br>

<h3 id="get-a-list-of-configurated-alerts-parameters">Parameters</h3>

|Name|In|Type|Required|Description|
|---|---|---|---|---|
|id_user|path|string|true|Hint: you can use 'me' or 'all'|
|expand|query|string|false|none|

> Example responses

> 200 Response

```json
{
  "useralerts": [
    {
      "id": 0,
      "income_max": 500,
      "expense_max": 500,
      "balance_min1": 500,
      "balance_min2": 0,
      "balance_max": 10000,
      "resume_enabled": true,
      "enabled": true,
      "value_type": "flat",
      "type": "transactions",
      "transaction_types": "",
      "date_range": 0,
      "apply": "",
      "resume_frequency": 7
    }
  ]
}
```

> useralerts

<h3 id="get-a-list-of-configurated-alerts-responses">Responses</h3>

|Status|Meaning|Description|Schema|
|---|---|---|---|
|200|[OK](https://tools.ietf.org/html/rfc7231#section-6.3.1)|useralerts|Inline|

<h3 id="get-a-list-of-configurated-alerts-responseschema">Response Schema</h3>

Status Code **200**

|Name|Type|Required|Restrictions|Description|
|---|---|---|---|---|
|» useralerts|[[UserAlert](#schemauseralert)]|true|none|[/!\ Careful we use default value from database if present      For more information see AlertsPlugin.init]|
|»» id|integer|true|none|none|
|»» income_max|number(float)|false|none|none|
|»» expense_max|number(float)|false|none|none|
|»» balance_min1|number(float)|false|none|none|
|»» balance_min2|number(float)|false|none|none|
|»» balance_max|number(float)|false|none|none|
|»» resume_enabled|boolean|false|none|none|
|»» enabled|boolean|false|none|none|
|»» value_type|string|true|none|none|
|»» type|string|true|none|none|
|»» transaction_types|string|false|none|none|
|»» date_range|integer|false|none|none|
|»» apply|string|false|none|none|
|»» resume_frequency|integer|true|none|none|

<aside class="success">
This operation does not require authentication
</aside>

## Create an alert on transactions or investemens of a given user

> Code samples

```http
POST /demo.biapi.pro/2.0/users/{id_user}/operationsalert HTTP/1.1

Content-Type: multipart/form-data
Accept: application/json

```

```python
import requests
headers = {
  'Content-Type': 'multipart/form-data',
  'Accept': 'application/json'
}

r = requests.post('/demo.biapi.pro/2.0/users/{id_user}/operationsalert', params={

}, headers = headers)

print r.json()

```

`POST /users/{id_user}/operationsalert`

> Body parameter

```yaml
type: string
income_max: 0
expense_max: 0
value_type: string
date_range: 0

```

<h3 id="create-an-alert-on-transactions-or-investemens-of-a-given-user-parameters">Parameters</h3>

|Name|In|Type|Required|Description|
|---|---|---|---|---|
|id_user|path|string|true|Hint: you can use 'me' or 'all'|
|expand|query|string|false|none|
|body|body|[postUsers_idUser_accounts_idAccount_operationsalert](#schemapostusers_iduser_accounts_idaccount_operationsalert)|false|none|
|» type|body|string|false|parameter to choose the scope of the alert. accepted: transactions, investements|
|» income_max|body|integer|false|capital gain thresholds|
|» expense_max|body|integer|false|capital loss thresholds|
|» value_type|body|string|false|whether the threshold is given in absolut value or percent. accepted: percent, flat|
|» date_range|body|integer|false|(number of days) range on which the analysis has to be done|

> Example responses

> 200 Response

```json
{
  "id": 0,
  "income_max": 500,
  "expense_max": 500,
  "balance_min1": 500,
  "balance_min2": 0,
  "balance_max": 10000,
  "resume_enabled": true,
  "enabled": true,
  "value_type": "flat",
  "type": "transactions",
  "transaction_types": "",
  "date_range": 0,
  "apply": "",
  "resume_frequency": 7
}
```

<h3 id="create-an-alert-on-transactions-or-investemens-of-a-given-user-responses">Responses</h3>

|Status|Meaning|Description|Schema|
|---|---|---|---|
|200|[OK](https://tools.ietf.org/html/rfc7231#section-6.3.1)|Successful POST on UserAlert resource|[UserAlert](#schemauseralert)|

<aside class="success">
This operation does not require authentication
</aside>

## Edit an alert on transactions or investemens

> Code samples

```http
PUT /demo.biapi.pro/2.0/users/{id_user}/operationsalert/{id_operationsalert} HTTP/1.1

Content-Type: multipart/form-data
Accept: application/json

```

```python
import requests
headers = {
  'Content-Type': 'multipart/form-data',
  'Accept': 'application/json'
}

r = requests.put('/demo.biapi.pro/2.0/users/{id_user}/operationsalert/{id_operationsalert}', params={

}, headers = headers)

print r.json()

```

`PUT /users/{id_user}/operationsalert/{id_operationsalert}`

> Body parameter

```yaml
type: string
income_max: 0
expense_max: 0
value_type: string
date_range: 0

```

<h3 id="edit-an-alert-on-transactions-or-investemens-parameters">Parameters</h3>

|Name|In|Type|Required|Description|
|---|---|---|---|---|
|id_user|path|string|true|Hint: you can use 'me' or 'all'|
|id_operationsalert|path|integer|true|none|
|expand|query|string|false|none|
|body|body|[postUsers_idUser_accounts_idAccount_operationsalert](#schemapostusers_iduser_accounts_idaccount_operationsalert)|false|none|
|» type|body|string|false|parameter to choose the scope of the alert. accepted: transactions, investements|
|» income_max|body|integer|false|capital gain thresholds|
|» expense_max|body|integer|false|capital loss thresholds|
|» value_type|body|string|false|whether the threshold is given in absolut value or percent. accepted: percent, flat|
|» date_range|body|integer|false|(number of days) range on which the analysis has to be done|

> Example responses

> 200 Response

```json
{
  "id": 0,
  "income_max": 500,
  "expense_max": 500,
  "balance_min1": 500,
  "balance_min2": 0,
  "balance_max": 10000,
  "resume_enabled": true,
  "enabled": true,
  "value_type": "flat",
  "type": "transactions",
  "transaction_types": "",
  "date_range": 0,
  "apply": "",
  "resume_frequency": 7
}
```

<h3 id="edit-an-alert-on-transactions-or-investemens-responses">Responses</h3>

|Status|Meaning|Description|Schema|
|---|---|---|---|
|200|[OK](https://tools.ietf.org/html/rfc7231#section-6.3.1)|Successful PUT on UserAlert resource|[UserAlert](#schemauseralert)|

<aside class="success">
This operation does not require authentication
</aside>

## Delete an alert on transactions or investemens

> Code samples

```http
DELETE /demo.biapi.pro/2.0/users/{id_user}/operationsalert/{id_operationsalert} HTTP/1.1

Accept: application/json

```

```python
import requests
headers = {
  'Accept': 'application/json'
}

r = requests.delete('/demo.biapi.pro/2.0/users/{id_user}/operationsalert/{id_operationsalert}', params={

}, headers = headers)

print r.json()

```

`DELETE /users/{id_user}/operationsalert/{id_operationsalert}`

<br><br>

<h3 id="delete-an-alert-on-transactions-or-investemens-parameters">Parameters</h3>

|Name|In|Type|Required|Description|
|---|---|---|---|---|
|id_user|path|string|true|Hint: you can use 'me' or 'all'|
|id_operationsalert|path|integer|true|none|
|expand|query|string|false|none|

> Example responses

> 200 Response

```json
{
  "id": 0,
  "income_max": 500,
  "expense_max": 500,
  "balance_min1": 500,
  "balance_min2": 0,
  "balance_max": 10000,
  "resume_enabled": true,
  "enabled": true,
  "value_type": "flat",
  "type": "transactions",
  "transaction_types": "",
  "date_range": 0,
  "apply": "",
  "resume_frequency": 7
}
```

> Successful DELETE on UserAlert resource

<h3 id="delete-an-alert-on-transactions-or-investemens-responses">Responses</h3>

|Status|Meaning|Description|Schema|
|---|---|---|---|
|200|[OK](https://tools.ietf.org/html/rfc7231#section-6.3.1)|Successful DELETE on UserAlert resource|[UserAlert](#schemauseralert)|

<aside class="success">
This operation does not require authentication
</aside>

## Get alert configuration of a specific account

> Code samples

```http
GET /demo.biapi.pro/2.0/users/{id_user}/accountsalert HTTP/1.1

Accept: application/json

```

```python
import requests
headers = {
  'Accept': 'application/json'
}

r = requests.get('/demo.biapi.pro/2.0/users/{id_user}/accountsalert', params={

}, headers = headers)

print r.json()

```

`GET /users/{id_user}/accountsalert`

<br><br>

<h3 id="get-alert-configuration-of-a-specific-account-parameters">Parameters</h3>

|Name|In|Type|Required|Description|
|---|---|---|---|---|
|id_user|path|string|true|Hint: you can use 'me' or 'all'|

> Example responses

> 200 Response

```json
{}
```

> OK

<h3 id="get-alert-configuration-of-a-specific-account-responses">Responses</h3>

|Status|Meaning|Description|Schema|
|---|---|---|---|
|200|[OK](https://tools.ietf.org/html/rfc7231#section-6.3.1)|OK|Inline|

<h3 id="get-alert-configuration-of-a-specific-account-responseschema">Response Schema</h3>

<aside class="success">
This operation does not require authentication
</aside>

## Update alert configuration of an account

> Code samples

```http
PUT /demo.biapi.pro/2.0/users/{id_user}/accountsalert HTTP/1.1

Content-Type: multipart/form-data
Accept: application/json

```

```python
import requests
headers = {
  'Content-Type': 'multipart/form-data',
  'Accept': 'application/json'
}

r = requests.put('/demo.biapi.pro/2.0/users/{id_user}/accountsalert', params={

}, headers = headers)

print r.json()

```

`PUT /users/{id_user}/accountsalert`

It updates the alert configuration of a specific account<br><br><br><br>

> Body parameter

```yaml
expense_max: 0
accounts: 0
income_max: 0
balance_min2: 0
enabled: true

```

<h3 id="update-alert-configuration-of-an-account-parameters">Parameters</h3>

|Name|In|Type|Required|Description|
|---|---|---|---|---|
|id_user|path|string|true|Hint: you can use 'me' or 'all'|
|body|body|object|false|none|
|» expense_max|body|integer|false|threshold from which an alert has to be sent for a high expense|
|» accounts|body|integer|false|list of accounts (id coma separated) on wich the alert has to be applied. If 'all' is given, it is applied on all accounts. default: all|
|» income_max|body|integer|false|threshold from which an alert has to be sent for a high income|
|» balance_min2|body|integer|false|second threshold from which an alert has to be sent for a low balance|
|» enabled|body|boolean|false|if false, the alert is not taken into account|

> Example responses

> 200 Response

```json
{}
```

> OK

<h3 id="update-alert-configuration-of-an-account-responses">Responses</h3>

|Status|Meaning|Description|Schema|
|---|---|---|---|
|200|[OK](https://tools.ietf.org/html/rfc7231#section-6.3.1)|OK|Inline|

<h3 id="update-alert-configuration-of-an-account-responseschema">Response Schema</h3>

<aside class="success">
This operation does not require authentication
</aside>

## Get devices

> Code samples

```http
GET /demo.biapi.pro/2.0/users/{id_user}/devices HTTP/1.1

Accept: application/json

```

```python
import requests
headers = {
  'Accept': 'application/json'
}

r = requests.get('/demo.biapi.pro/2.0/users/{id_user}/devices', params={

}, headers = headers)

print r.json()

```

`GET /users/{id_user}/devices`

<h3 id="get-devices-parameters">Parameters</h3>

|Name|In|Type|Required|Description|
|---|---|---|---|---|
|id_user|path|string|true|Hint: you can use 'me' or 'all'|
|expand|query|string|false|none|

> Example responses

> 200 Response

```json
{
  "devices": [
    {
      "id": 0,
      "id_token": 0,
      "type": "",
      "notification_token": "",
      "last_update": "CURRENT_TIMESTAMP",
      "version": "",
      "debug": false
    }
  ]
}
```

<h3 id="get-devices-responses">Responses</h3>

|Status|Meaning|Description|Schema|
|---|---|---|---|
|200|[OK](https://tools.ietf.org/html/rfc7231#section-6.3.1)|devices|Inline|

<h3 id="get-devices-responseschema">Response Schema</h3>

Status Code **200**

|Name|Type|Required|Restrictions|Description|
|---|---|---|---|---|
|» devices|[[Device](#schemadevice)]|true|none|none|
|»» id|integer|true|none|none|
|»» id_token|integer|true|none|none|
|»» type|string|true|none|none|
|»» notification_token|string|true|none|none|
|»» last_update|string(date-time)|true|none|none|
|»» version|string|true|none|none|
|»» debug|boolean|true|none|none|

<aside class="success">
This operation does not require authentication
</aside>

## Create a device linked to specified token.

> Code samples

```http
POST /demo.biapi.pro/2.0/users/{id_user}/devices HTTP/1.1

Content-Type: multipart/form-data
Accept: application/json

```

```python
import requests
headers = {
  'Content-Type': 'multipart/form-data',
  'Accept': 'application/json'
}

r = requests.post('/demo.biapi.pro/2.0/users/{id_user}/devices', params={

}, headers = headers)

print r.json()

```

`POST /users/{id_user}/devices`

> Body parameter

```yaml
notification_token: string
application: string
notification_version: 0

```

<h3 id="create-a-device-linked-to-specified-token.-parameters">Parameters</h3>

|Name|In|Type|Required|Description|
|---|---|---|---|---|
|id_user|path|string|true|Hint: you can use 'me' or 'all'|
|expand|query|string|false|none|
|body|body|[postUsers_idUser_devices](#schemapostusers_iduser_devices)|false|none|
|» notification_token|body|string|true|the GCM or APNS notification_token to use|
|» application|body|string|true|the device in use|
|» notification_version|body|integer|false|version of notifications|

> Example responses

> 200 Response

```json
{
  "id": 0,
  "id_token": 0,
  "type": "",
  "notification_token": "",
  "last_update": "CURRENT_TIMESTAMP",
  "version": "",
  "debug": false
}
```

<h3 id="create-a-device-linked-to-specified-token.-responses">Responses</h3>

|Status|Meaning|Description|Schema|
|---|---|---|---|
|200|[OK](https://tools.ietf.org/html/rfc7231#section-6.3.1)|Successful POST on Device resource|[Device](#schemadevice)|

<aside class="success">
This operation does not require authentication
</aside>

## Get a device

> Code samples

```http
GET /demo.biapi.pro/2.0/users/{id_user}/devices/{id_device} HTTP/1.1

Accept: application/json

```

```python
import requests
headers = {
  'Accept': 'application/json'
}

r = requests.get('/demo.biapi.pro/2.0/users/{id_user}/devices/{id_device}', params={

}, headers = headers)

print r.json()

```

`GET /users/{id_user}/devices/{id_device}`

<h3 id="get-a-device-parameters">Parameters</h3>

|Name|In|Type|Required|Description|
|---|---|---|---|---|
|id_user|path|string|true|Hint: you can use 'me' or 'all'|
|id_device|path|integer|true|none|
|expand|query|string|false|none|

> Example responses

> 200 Response

```json
{
  "id": 0,
  "id_token": 0,
  "type": "",
  "notification_token": "",
  "last_update": "CURRENT_TIMESTAMP",
  "version": "",
  "debug": false
}
```

<h3 id="get-a-device-responses">Responses</h3>

|Status|Meaning|Description|Schema|
|---|---|---|---|
|200|[OK](https://tools.ietf.org/html/rfc7231#section-6.3.1)|Successful GET on Device resource|[Device](#schemadevice)|

<aside class="success">
This operation does not require authentication
</aside>

## Update attributes of the device.

> Code samples

```http
PUT /demo.biapi.pro/2.0/users/{id_user}/devices/{id_device} HTTP/1.1

Content-Type: multipart/form-data
Accept: application/json

```

```python
import requests
headers = {
  'Content-Type': 'multipart/form-data',
  'Accept': 'application/json'
}

r = requests.put('/demo.biapi.pro/2.0/users/{id_user}/devices/{id_device}', params={

}, headers = headers)

print r.json()

```

`PUT /users/{id_user}/devices/{id_device}`

> Body parameter

```yaml
notification_token: string
application: string
notification_version: 0

```

<h3 id="update-attributes-of-the-device.-parameters">Parameters</h3>

|Name|In|Type|Required|Description|
|---|---|---|---|---|
|id_user|path|string|true|Hint: you can use 'me' or 'all'|
|id_device|path|integer|true|none|
|expand|query|string|false|none|
|body|body|[postUsers_idUser_devices](#schemapostusers_iduser_devices)|false|none|
|» notification_token|body|string|true|the GCM or APNS notification_token to use|
|» application|body|string|true|the device in use|
|» notification_version|body|integer|false|version of notifications|

> Example responses

> 200 Response

```json
{
  "id": 0,
  "id_token": 0,
  "type": "",
  "notification_token": "",
  "last_update": "CURRENT_TIMESTAMP",
  "version": "",
  "debug": false
}
```

<h3 id="update-attributes-of-the-device.-responses">Responses</h3>

|Status|Meaning|Description|Schema|
|---|---|---|---|
|200|[OK](https://tools.ietf.org/html/rfc7231#section-6.3.1)|Successful PUT on Device resource|[Device](#schemadevice)|

<aside class="success">
This operation does not require authentication
</aside>

## Delete device.

> Code samples

```http
DELETE /demo.biapi.pro/2.0/users/{id_user}/devices/{id_device} HTTP/1.1

Accept: application/json

```

```python
import requests
headers = {
  'Accept': 'application/json'
}

r = requests.delete('/demo.biapi.pro/2.0/users/{id_user}/devices/{id_device}', params={

}, headers = headers)

print r.json()

```

`DELETE /users/{id_user}/devices/{id_device}`

<h3 id="delete-device.-parameters">Parameters</h3>

|Name|In|Type|Required|Description|
|---|---|---|---|---|
|id_user|path|string|true|Hint: you can use 'me' or 'all'|
|id_device|path|integer|true|none|
|expand|query|string|false|none|

> Example responses

> 200 Response

```json
{
  "id": 0,
  "id_token": 0,
  "type": "",
  "notification_token": "",
  "last_update": "CURRENT_TIMESTAMP",
  "version": "",
  "debug": false
}
```

<h3 id="delete-device.-responses">Responses</h3>

|Status|Meaning|Description|Schema|
|---|---|---|---|
|200|[OK](https://tools.ietf.org/html/rfc7231#section-6.3.1)|Successful DELETE on Device resource|[Device](#schemadevice)|

<aside class="success">
This operation does not require authentication
</aside>

## Change settings of the profile.

> Code samples

```http
PUT /demo.biapi.pro/2.0/users/{id_user}/profiles/me HTTP/1.1

Content-Type: multipart/form-data
Accept: application/json

```

```python
import requests
headers = {
  'Content-Type': 'multipart/form-data',
  'Accept': 'application/json'
}

r = requests.put('/demo.biapi.pro/2.0/users/{id_user}/profiles/me', params={

}, headers = headers)

print r.json()

```

`PUT /users/{id_user}/profiles/me`

> Body parameter

```yaml
email: string
password: string
current_password: string
contact: string
conf: string
state: true
lang: string

```

<h3 id="change-settings-of-the-profile.-parameters">Parameters</h3>

|Name|In|Type|Required|Description|
|---|---|---|---|---|
|id_user|path|string|true|Hint: you can use 'me' or 'all'|
|expand|query|string|false|none|
|body|body|object|false|none|
|» email|body|string|false|change email of profile|
|» password|body|string|false|change password of profile|
|» current_password|body|string|false|needed when changing the password or the email|
|» contact|body|string|false|change contact information of a profile|
|» conf|body|string|false|change config of a profile|
|» state|body|boolean|false|state of the profile|
|» lang|body|string|false|change lang of the profile|

> Example responses

> 200 Response

```json
{
  "id": 0,
  "id_user": 0,
  "role": "admin",
  "statut": 0,
  "admin": false,
  "conf": "",
  "lang": ""
}
```

<h3 id="change-settings-of-the-profile.-responses">Responses</h3>

|Status|Meaning|Description|Schema|
|---|---|---|---|
|200|[OK](https://tools.ietf.org/html/rfc7231#section-6.3.1)|Successful PUT on Profile resource|[Profile](#schemaprofile)|

<aside class="success">
This operation does not require authentication
</aside>

## Get synthesis configuration of a specific user

> Code samples

```http
GET /demo.biapi.pro/2.0/users/{id_user}/resume HTTP/1.1

Accept: application/json

```

```python
import requests
headers = {
  'Accept': 'application/json'
}

r = requests.get('/demo.biapi.pro/2.0/users/{id_user}/resume', params={

}, headers = headers)

print r.json()

```

`GET /users/{id_user}/resume`

<br><br>

<h3 id="get-synthesis-configuration-of-a-specific-user-parameters">Parameters</h3>

|Name|In|Type|Required|Description|
|---|---|---|---|---|
|id_user|path|string|true|Hint: you can use 'me' or 'all'|

> Example responses

> 200 Response

```json
{}
```

> OK

<h3 id="get-synthesis-configuration-of-a-specific-user-responses">Responses</h3>

|Status|Meaning|Description|Schema|
|---|---|---|---|
|200|[OK](https://tools.ietf.org/html/rfc7231#section-6.3.1)|OK|Inline|

<h3 id="get-synthesis-configuration-of-a-specific-user-responseschema">Response Schema</h3>

<aside class="success">
This operation does not require authentication
</aside>

## Update synthesis configuration

> Code samples

```http
POST /demo.biapi.pro/2.0/users/{id_user}/resume HTTP/1.1

Content-Type: multipart/form-data
Accept: application/json

```

```python
import requests
headers = {
  'Content-Type': 'multipart/form-data',
  'Accept': 'application/json'
}

r = requests.post('/demo.biapi.pro/2.0/users/{id_user}/resume', params={

}, headers = headers)

print r.json()

```

`POST /users/{id_user}/resume`

<br><br>

> Body parameter

```yaml
resume_enabled: true
resume_frequency: 0

```

<h3 id="update-synthesis-configuration-parameters">Parameters</h3>

|Name|In|Type|Required|Description|
|---|---|---|---|---|
|id_user|path|string|true|Hint: you can use 'me' or 'all'|
|body|body|object|false|none|
|» resume_enabled|body|boolean|false|whether the synthesis is activated or not|
|» resume_frequency|body|integer|false|frequency of the synthesis given in days|

> Example responses

> 200 Response

```json
{}
```

> OK

<h3 id="update-synthesis-configuration-responses">Responses</h3>

|Status|Meaning|Description|Schema|
|---|---|---|---|
|200|[OK](https://tools.ietf.org/html/rfc7231#section-6.3.1)|OK|Inline|

<h3 id="update-synthesis-configuration-responseschema">Response Schema</h3>

<aside class="success">
This operation does not require authentication
</aside>

<h1 id="budgea-api-documentation-providers">Providers</h1>

## Get document types

> Code samples

```http
GET /demo.biapi.pro/2.0/documenttypes HTTP/1.1

Accept: application/json

```

```python
import requests
headers = {
  'Accept': 'application/json'
}

r = requests.get('/demo.biapi.pro/2.0/documenttypes', params={

}, headers = headers)

print r.json()

```

`GET /documenttypes`

<h3 id="get-document-types-parameters">Parameters</h3>

|Name|In|Type|Required|Description|
|---|---|---|---|---|
|expand|query|string|false|none|

> Example responses

> 200 Response

```json
{
  "documenttypes": [
    {
      "id": 0,
      "name": "",
      "attacheable": true
    }
  ]
}
```

<h3 id="get-document-types-responses">Responses</h3>

|Status|Meaning|Description|Schema|
|---|---|---|---|
|200|[OK](https://tools.ietf.org/html/rfc7231#section-6.3.1)|documenttypes|Inline|

<h3 id="get-document-types-responseschema">Response Schema</h3>

Status Code **200**

|Name|Type|Required|Restrictions|Description|
|---|---|---|---|---|
|» documenttypes|[[DocumentType](#schemadocumenttype)]|true|none|none|
|»» id|integer|true|none|none|
|»» name|string|true|none|none|
|»» attacheable|boolean|true|none|none|

<aside class="success">
This operation does not require authentication
</aside>

## Edit a document type

> Code samples

```http
PUT /demo.biapi.pro/2.0/documenttypes/{id_documenttype} HTTP/1.1

Content-Type: multipart/form-data
Accept: application/json

```

```python
import requests
headers = {
  'Content-Type': 'multipart/form-data',
  'Accept': 'application/json'
}

r = requests.put('/demo.biapi.pro/2.0/documenttypes/{id_documenttype}', params={

}, headers = headers)

print r.json()

```

`PUT /documenttypes/{id_documenttype}`

Change value of a document type.<br><br>

> Body parameter

```yaml
name: string
attacheable: 0

```

<h3 id="edit-a-document-type-parameters">Parameters</h3>

|Name|In|Type|Required|Description|
|---|---|---|---|---|
|id_documenttype|path|integer|true|none|
|expand|query|string|false|none|
|body|body|object|false|none|
|» name|body|string|true|Displayed name of document type|
|» attacheable|body|integer|true|If true, documents of this type can be attached to a transaction, and have amount related meta-data|

> Example responses

> 200 Response

```json
{
  "id": 0,
  "name": "",
  "attacheable": true
}
```

<h3 id="edit-a-document-type-responses">Responses</h3>

|Status|Meaning|Description|Schema|
|---|---|---|---|
|200|[OK](https://tools.ietf.org/html/rfc7231#section-6.3.1)|Successful PUT on DocumentType resource|[DocumentType](#schemadocumenttype)|

<aside class="success">
This operation does not require authentication
</aside>

## Get documents

> Code samples

```http
GET /demo.biapi.pro/2.0/users/{id_user}/transactionsclusters/{id_transactions_cluster}/documents HTTP/1.1

Accept: application/json

```

```python
import requests
headers = {
  'Accept': 'application/json'
}

r = requests.get('/demo.biapi.pro/2.0/users/{id_user}/transactionsclusters/{id_transactions_cluster}/documents', params={

}, headers = headers)

print r.json()

```

`GET /users/{id_user}/transactionsclusters/{id_transactions_cluster}/documents`

Get list of documents<br><br>

<h3 id="get-documents-parameters">Parameters</h3>

|Name|In|Type|Required|Description|
|---|---|---|---|---|
|id_user|path|string|true|Hint: you can use 'me' or 'all'|
|id_transactions_cluster|path|integer|true|none|
|limit|query|integer|false|limit number of results|
|offset|query|integer|false|offset of first result|
|min_date|query|string(date)|false|minimal (inclusive) date|
|max_date|query|string(date)|false|maximum (inclusive) date|
|min_amount|query|number(float)|false|minimal (inclusive) amount|
|max_amount|query|number(float)|false|maximumd (inclusive) amount|
|min_timestamp|query|number(float)|false|minimal (inclusive) timestamp|
|max_timestamp|query|number(float)|false|maximum (inclusive) timestamp|
|id_type|query|integer|false|filter with a document type|
|expand|query|string|false|none|

> Example responses

> 200 Response

```json
{
  "documents": [
    {
      "id": 0,
      "id_type": 0,
      "id_category": 0,
      "id_user": 0,
      "id_subscription": 0,
      "id_file": 0,
      "id_thumbnail": 0,
      "url": "",
      "thumb_url": "",
      "name": "",
      "timestamp": "CURRENT_TIMESTAMP",
      "date": "2019-03-20 11:42:25.678944",
      "duedate": "2019-03-20",
      "total_amount": 0,
      "untaxed_amount": 0,
      "vat": 0,
      "income": true,
      "readonly": true,
      "number": "",
      "issuer": "",
      "last_update": "2019-03-20 11:42:25.679749",
      "currency": ""
    }
  ]
}
```

<h3 id="get-documents-responses">Responses</h3>

|Status|Meaning|Description|Schema|
|---|---|---|---|
|200|[OK](https://tools.ietf.org/html/rfc7231#section-6.3.1)|documents|Inline|

<h3 id="get-documents-responseschema">Response Schema</h3>

Status Code **200**

|Name|Type|Required|Restrictions|Description|
|---|---|---|---|---|
|» documents|[[Document](#schemadocument)]|true|none|none|
|»» id|integer|true|none|none|
|»» id_type|integer|false|none|none|
|»» id_category|integer|false|none|none|
|»» id_user|integer|true|none|none|
|»» id_subscription|integer|false|none|none|
|»» id_file|integer|false|none|none|
|»» id_thumbnail|integer|false|none|none|
|»» url|string|false|none|none|
|»» thumb_url|string|false|none|none|
|»» name|string|false|none|none|
|»» timestamp|string(date-time)|true|none|none|
|»» date|string(date-time)|true|none|none|
|»» duedate|string(date)|false|none|none|
|»» total_amount|number(float)|false|none|none|
|»» untaxed_amount|number(float)|false|none|none|
|»» vat|number(float)|false|none|none|
|»» income|boolean|false|none|none|
|»» readonly|boolean|true|none|none|
|»» number|string|false|none|none|
|»» issuer|string|false|none|none|
|»» last_update|string(date-time)|false|none|Last successful update of the document|
|»» currency|object|false|none|Document currency|

<aside class="success">
This operation does not require authentication
</aside>

## Add a new document

> Code samples

```http
POST /demo.biapi.pro/2.0/users/{id_user}/transactionsclusters/{id_transactions_cluster}/documents HTTP/1.1

Content-Type: multipart/form-data
Accept: application/json

```

```python
import requests
headers = {
  'Content-Type': 'multipart/form-data',
  'Accept': 'application/json'
}

r = requests.post('/demo.biapi.pro/2.0/users/{id_user}/transactionsclusters/{id_transactions_cluster}/documents', params={

}, headers = headers)

print r.json()

```

`POST /users/{id_user}/transactionsclusters/{id_transactions_cluster}/documents`

Add a new document<br><br>

> Body parameter

```yaml
id_type: 0
id_category: 0
date: '2019-03-20'
duedate: '2019-03-20'
total_amount: 0
untaxed_amount: 0
vat: 0
income: true
readonly: true
file: string
id_ocr: 0
name: string

```

<h3 id="add-a-new-document-parameters">Parameters</h3>

|Name|In|Type|Required|Description|
|---|---|---|---|---|
|id_user|path|string|true|Hint: you can use 'me' or 'all'|
|id_transactions_cluster|path|integer|true|none|
|expand|query|string|false|none|
|body|body|[postUsers_idUser_accounts_idAccount_transactions_idTransaction_documents](#schemapostusers_iduser_accounts_idaccount_transactions_idtransaction_documents)|false|none|
|» id_type|body|integer|false|Type of this document|
|» id_category|body|integer|false|Related category|
|» date|body|string(date)|true|Date of document|
|» duedate|body|string(date)|true|Due date of document|
|» total_amount|body|number(float)|false|Taxed amount|
|» untaxed_amount|body|number(float)|false|Untaxed amount|
|» vat|body|number(float)|false|VAT amount|
|» income|body|boolean|false|Is an income or an outcome|
|» readonly|body|boolean|false|Is this file readonly|
|» file|body|string(binary)|false|File of the document|
|» id_ocr|body|integer|false|Related OCR process|
|» name|body|string|false|Name of the document|

> Example responses

> 200 Response

```json
{
  "id": 0,
  "id_type": 0,
  "id_category": 0,
  "id_user": 0,
  "id_subscription": 0,
  "id_file": 0,
  "id_thumbnail": 0,
  "url": "",
  "thumb_url": "",
  "name": "",
  "timestamp": "CURRENT_TIMESTAMP",
  "date": "2019-03-20 11:42:25.678944",
  "duedate": "2019-03-20",
  "total_amount": 0,
  "untaxed_amount": 0,
  "vat": 0,
  "income": true,
  "readonly": true,
  "number": "",
  "issuer": "",
  "last_update": "2019-03-20 11:42:25.679749",
  "currency": ""
}
```

<h3 id="add-a-new-document-responses">Responses</h3>

|Status|Meaning|Description|Schema|
|---|---|---|---|
|200|[OK](https://tools.ietf.org/html/rfc7231#section-6.3.1)|Successful POST on Document resource|[Document](#schemadocument)|

<aside class="success">
This operation does not require authentication
</aside>

## Attach an existing document to a transaction or a transactions_cluster

> Code samples

```http
PUT /demo.biapi.pro/2.0/users/{id_user}/transactionsclusters/{id_transactions_cluster}/documents HTTP/1.1

Content-Type: multipart/form-data
Accept: application/json

```

```python
import requests
headers = {
  'Content-Type': 'multipart/form-data',
  'Accept': 'application/json'
}

r = requests.put('/demo.biapi.pro/2.0/users/{id_user}/transactionsclusters/{id_transactions_cluster}/documents', params={

}, headers = headers)

print r.json()

```

`PUT /users/{id_user}/transactionsclusters/{id_transactions_cluster}/documents`

> Body parameter

```yaml
id_document: 0

```

<h3 id="attach-an-existing-document-to-a-transaction-or-a-transactions_cluster-parameters">Parameters</h3>

|Name|In|Type|Required|Description|
|---|---|---|---|---|
|id_user|path|string|true|Hint: you can use 'me' or 'all'|
|id_transactions_cluster|path|integer|true|none|
|expand|query|string|false|none|
|body|body|[putUsers_idUser_accounts_idAccount_transactions_idTransaction_documents](#schemaputusers_iduser_accounts_idaccount_transactions_idtransaction_documents)|false|none|
|» id_document|body|integer|true|id of the document you want to attach the file to|

> Example responses

> 200 Response

```json
{
  "id": 0,
  "id_type": 0,
  "id_category": 0,
  "id_user": 0,
  "id_subscription": 0,
  "id_file": 0,
  "id_thumbnail": 0,
  "url": "",
  "thumb_url": "",
  "name": "",
  "timestamp": "CURRENT_TIMESTAMP",
  "date": "2019-03-20 11:42:25.678944",
  "duedate": "2019-03-20",
  "total_amount": 0,
  "untaxed_amount": 0,
  "vat": 0,
  "income": true,
  "readonly": true,
  "number": "",
  "issuer": "",
  "last_update": "2019-03-20 11:42:25.679749",
  "currency": ""
}
```

<h3 id="attach-an-existing-document-to-a-transaction-or-a-transactions_cluster-responses">Responses</h3>

|Status|Meaning|Description|Schema|
|---|---|---|---|
|200|[OK](https://tools.ietf.org/html/rfc7231#section-6.3.1)|Successful PUT on Document resource|[Document](#schemadocument)|

<aside class="success">
This operation does not require authentication
</aside>

## Delete documents

> Code samples

```http
DELETE /demo.biapi.pro/2.0/users/{id_user}/transactionsclusters/{id_transactions_cluster}/documents HTTP/1.1

Accept: application/json

```

```python
import requests
headers = {
  'Accept': 'application/json'
}

r = requests.delete('/demo.biapi.pro/2.0/users/{id_user}/transactionsclusters/{id_transactions_cluster}/documents', params={

}, headers = headers)

print r.json()

```

`DELETE /users/{id_user}/transactionsclusters/{id_transactions_cluster}/documents`

<h3 id="delete-documents-parameters">Parameters</h3>

|Name|In|Type|Required|Description|
|---|---|---|---|---|
|id_user|path|string|true|Hint: you can use 'me' or 'all'|
|id_transactions_cluster|path|integer|true|none|
|expand|query|string|false|none|

> Example responses

> 200 Response

```json
{
  "id": 0,
  "id_type": 0,
  "id_category": 0,
  "id_user": 0,
  "id_subscription": 0,
  "id_file": 0,
  "id_thumbnail": 0,
  "url": "",
  "thumb_url": "",
  "name": "",
  "timestamp": "CURRENT_TIMESTAMP",
  "date": "2019-03-20 11:42:25.678944",
  "duedate": "2019-03-20",
  "total_amount": 0,
  "untaxed_amount": 0,
  "vat": 0,
  "income": true,
  "readonly": true,
  "number": "",
  "issuer": "",
  "last_update": "2019-03-20 11:42:25.679749",
  "currency": ""
}
```

<h3 id="delete-documents-responses">Responses</h3>

|Status|Meaning|Description|Schema|
|---|---|---|---|
|200|[OK](https://tools.ietf.org/html/rfc7231#section-6.3.1)|Successful DELETE on Document resource|[Document](#schemadocument)|

<aside class="success">
This operation does not require authentication
</aside>

## Edit a document

> Code samples

```http
PUT /demo.biapi.pro/2.0/users/{id_user}/transactionsclusters/{id_transactions_cluster}/documents/{id_document} HTTP/1.1

Content-Type: multipart/form-data
Accept: application/json

```

```python
import requests
headers = {
  'Content-Type': 'multipart/form-data',
  'Accept': 'application/json'
}

r = requests.put('/demo.biapi.pro/2.0/users/{id_user}/transactionsclusters/{id_transactions_cluster}/documents/{id_document}', params={

}, headers = headers)

print r.json()

```

`PUT /users/{id_user}/transactionsclusters/{id_transactions_cluster}/documents/{id_document}`

Edit meta-data of a specific document.

> Body parameter

```yaml
id_type: 0
id_category: 0
date: '2019-03-20'
duedate: '2019-03-20'
total_amount: 0
untaxed_amount: 0
vat: 0
income: 0
readonly: 0
file: string
name: string

```

<h3 id="edit-a-document-parameters">Parameters</h3>

|Name|In|Type|Required|Description|
|---|---|---|---|---|
|id_user|path|string|true|Hint: you can use 'me' or 'all'|
|id_transactions_cluster|path|integer|true|none|
|id_document|path|integer|true|none|
|expand|query|string|false|none|
|body|body|[putUsers_idUser_accounts_idAccount_transactions_idTransaction_documents_idDocument_](#schemaputusers_iduser_accounts_idaccount_transactions_idtransaction_documents_iddocument_)|false|none|
|» id_type|body|integer|false|Type of this document|
|» id_category|body|integer|false|Related category|
|» date|body|string(date)|false|Date of document|
|» duedate|body|string(date)|false|Due date of document|
|» total_amount|body|number(float)|false|Taxed amount|
|» untaxed_amount|body|number(float)|false|Untaxed amount|
|» vat|body|number(float)|false|VAT amount|
|» income|body|integer|false|Is an income or an outcome|
|» readonly|body|integer|false|Is this file readonly|
|» file|body|string(binary)|false|File of the document|
|» name|body|string|false|Name of the document|

> Example responses

> 200 Response

```json
{
  "id": 0,
  "id_type": 0,
  "id_category": 0,
  "id_user": 0,
  "id_subscription": 0,
  "id_file": 0,
  "id_thumbnail": 0,
  "url": "",
  "thumb_url": "",
  "name": "",
  "timestamp": "CURRENT_TIMESTAMP",
  "date": "2019-03-20 11:42:25.678944",
  "duedate": "2019-03-20",
  "total_amount": 0,
  "untaxed_amount": 0,
  "vat": 0,
  "income": true,
  "readonly": true,
  "number": "",
  "issuer": "",
  "last_update": "2019-03-20 11:42:25.679749",
  "currency": ""
}
```

<h3 id="edit-a-document-responses">Responses</h3>

|Status|Meaning|Description|Schema|
|---|---|---|---|
|200|[OK](https://tools.ietf.org/html/rfc7231#section-6.3.1)|Successful PUT on Document resource|[Document](#schemadocument)|

<aside class="success">
This operation does not require authentication
</aside>

## Delete a document

> Code samples

```http
DELETE /demo.biapi.pro/2.0/users/{id_user}/transactionsclusters/{id_transactions_cluster}/documents/{id_document} HTTP/1.1

Accept: application/json

```

```python
import requests
headers = {
  'Accept': 'application/json'
}

r = requests.delete('/demo.biapi.pro/2.0/users/{id_user}/transactionsclusters/{id_transactions_cluster}/documents/{id_document}', params={

}, headers = headers)

print r.json()

```

`DELETE /users/{id_user}/transactionsclusters/{id_transactions_cluster}/documents/{id_document}`

<h3 id="delete-a-document-parameters">Parameters</h3>

|Name|In|Type|Required|Description|
|---|---|---|---|---|
|id_user|path|string|true|Hint: you can use 'me' or 'all'|
|id_transactions_cluster|path|integer|true|none|
|id_document|path|integer|true|none|
|expand|query|string|false|none|

> Example responses

> 200 Response

```json
{
  "id": 0,
  "id_type": 0,
  "id_category": 0,
  "id_user": 0,
  "id_subscription": 0,
  "id_file": 0,
  "id_thumbnail": 0,
  "url": "",
  "thumb_url": "",
  "name": "",
  "timestamp": "CURRENT_TIMESTAMP",
  "date": "2019-03-20 11:42:25.678944",
  "duedate": "2019-03-20",
  "total_amount": 0,
  "untaxed_amount": 0,
  "vat": 0,
  "income": true,
  "readonly": true,
  "number": "",
  "issuer": "",
  "last_update": "2019-03-20 11:42:25.679749",
  "currency": ""
}
```

<h3 id="delete-a-document-responses">Responses</h3>

|Status|Meaning|Description|Schema|
|---|---|---|---|
|200|[OK](https://tools.ietf.org/html/rfc7231#section-6.3.1)|Successful DELETE on Document resource|[Document](#schemadocument)|

<aside class="success">
This operation does not require authentication
</aside>

## Update a subscription

> Code samples

```http
PUT /demo.biapi.pro/2.0/users/{id_user}/subscriptions/{id_subscription} HTTP/1.1

Content-Type: multipart/form-data
Accept: application/json

```

```python
import requests
headers = {
  'Content-Type': 'multipart/form-data',
  'Accept': 'application/json'
}

r = requests.put('/demo.biapi.pro/2.0/users/{id_user}/subscriptions/{id_subscription}', params={

}, headers = headers)

print r.json()

```

`PUT /users/{id_user}/subscriptions/{id_subscription}`

It updates a specific subscription<br><br>

> Body parameter

```yaml
name: string
disabled: true

```

<h3 id="update-a-subscription-parameters">Parameters</h3>

|Name|In|Type|Required|Description|
|---|---|---|---|---|
|id_user|path|string|true|Hint: you can use 'me' or 'all'|
|id_subscription|path|integer|true|none|
|expand|query|string|false|none|
|body|body|[putUsers_idUser_connections_idConnection_subscriptions_idSubscription_](#schemaputusers_iduser_connections_idconnection_subscriptions_idsubscription_)|false|none|
|» name|body|string|false|Label of the subscription|
|» disabled|body|boolean|false|If the subscription is disabled (not synchronized)|

> Example responses

> 200 Response

```json
{
  "id": 0,
  "id_connection": 0,
  "id_user": 0,
  "number": "",
  "label": "",
  "subscriber": "",
  "validity": "2019-03-20",
  "renewdate": "2019-03-20",
  "last_update": "2019-03-20 11:42:25.663641",
  "deleted": "2019-03-20 11:42:25.663720",
  "disabled": "2019-03-20 11:42:25.663804",
  "error": ""
}
```

<h3 id="update-a-subscription-responses">Responses</h3>

|Status|Meaning|Description|Schema|
|---|---|---|---|
|200|[OK](https://tools.ietf.org/html/rfc7231#section-6.3.1)|Successful PUT on Subscription resource|[Subscription](#schemasubscription)|

<aside class="success">
This operation does not require authentication
</aside>

## Delete a subscription.

> Code samples

```http
DELETE /demo.biapi.pro/2.0/users/{id_user}/subscriptions/{id_subscription} HTTP/1.1

Accept: application/json

```

```python
import requests
headers = {
  'Accept': 'application/json'
}

r = requests.delete('/demo.biapi.pro/2.0/users/{id_user}/subscriptions/{id_subscription}', params={

}, headers = headers)

print r.json()

```

`DELETE /users/{id_user}/subscriptions/{id_subscription}`

It deletes a specific subscription If this is the last synced subscription of a connection, it will be removed too.<br><br>

<h3 id="delete-a-subscription.-parameters">Parameters</h3>

|Name|In|Type|Required|Description|
|---|---|---|---|---|
|id_user|path|string|true|Hint: you can use 'me' or 'all'|
|id_subscription|path|integer|true|none|
|expand|query|string|false|none|

> Example responses

> 200 Response

```json
{
  "id": 0,
  "id_connection": 0,
  "id_user": 0,
  "number": "",
  "label": "",
  "subscriber": "",
  "validity": "2019-03-20",
  "renewdate": "2019-03-20",
  "last_update": "2019-03-20 11:42:25.663641",
  "deleted": "2019-03-20 11:42:25.663720",
  "disabled": "2019-03-20 11:42:25.663804",
  "error": ""
}
```

<h3 id="delete-a-subscription.-responses">Responses</h3>

|Status|Meaning|Description|Schema|
|---|---|---|---|
|200|[OK](https://tools.ietf.org/html/rfc7231#section-6.3.1)|Successful DELETE on Subscription resource|[Subscription](#schemasubscription)|

<aside class="success">
This operation does not require authentication
</aside>

<h1 id="budgea-api-documentation-recipients">Recipients</h1>

## Add a recipient.

> Code samples

```http
POST /demo.biapi.pro/2.0/users/{id_user}/recipients HTTP/1.1

Content-Type: multipart/form-data
Accept: application/json

```

```python
import requests
headers = {
  'Content-Type': 'multipart/form-data',
  'Accept': 'application/json'
}

r = requests.post('/demo.biapi.pro/2.0/users/{id_user}/recipients', params={

}, headers = headers)

print r.json()

```

`POST /users/{id_user}/recipients`

if config key transfer.recipient.iban_white_list.enabled is set to 1, recipients whose IBAN are not from countries codes contained in transfer.recipient.iban_white_list.entries will be deleted<br><br>

> Body parameter

```yaml
label: string
iban: string

```

<h3 id="add-a-recipient.-parameters">Parameters</h3>

|Name|In|Type|Required|Description|
|---|---|---|---|---|
|id_user|path|string|true|Hint: you can use 'me' or 'all'|
|background|query|boolean|false|if true, do synchronization in background|
|expand|query|string|false|none|
|body|body|[postUsers_idUser_accounts_idAccount_recipients](#schemapostusers_iduser_accounts_idaccount_recipients)|false|none|
|» label|body|string|false|label of recipient|
|» iban|body|string|false|iban of recipient|

> Example responses

> 200 Response

```json
{
  "id": 0,
  "id_account": 0,
  "id_target_account": 0,
  "label": "",
  "bank_name": "",
  "iban": "",
  "webid": "",
  "category": "",
  "last_update": "2019-03-20 11:42:25.676098",
  "time_scraped": "2019-03-20 11:42:25.676188",
  "deleted": "2019-03-20 11:42:25.676267",
  "expire": "2019-03-20 11:42:25.676349",
  "enabled_at": "2019-03-20 11:42:25.676426",
  "add_verified": false,
  "state": "",
  "error": "",
  "currency": ""
}
```

<h3 id="add-a-recipient.-responses">Responses</h3>

|Status|Meaning|Description|Schema|
|---|---|---|---|
|200|[OK](https://tools.ietf.org/html/rfc7231#section-6.3.1)|Successful POST on Recipient resource|[Recipient](#schemarecipient)|

<aside class="success">
This operation does not require authentication
</aside>

<h1 id="budgea-api-documentation-stet">STET</h1>

## Check authorization of TPP

> Code samples

```http
POST /demo.biapi.pro/2.0/stet/v1/authorize HTTP/1.1

```

```python
import requests

r = requests.post('/demo.biapi.pro/2.0/stet/v1/authorize', params={

)

print r.json()

```

`POST /stet/v1/authorize`

Check if calling TPP has a client_id in Budgea's database, if it is not the case, it creates one and returns new id<br><br>

<h3 id="check-authorization-of-tpp-responses">Responses</h3>

|Status|Meaning|Description|Schema|
|---|---|---|---|
|200|[OK](https://tools.ietf.org/html/rfc7231#section-6.3.1)|standard HTTP response|None|

<aside class="success">
This operation does not require authentication
</aside>

## Transform a temporary code to a access_token

> Code samples

```http
POST /demo.biapi.pro/2.0/stet/v1/token HTTP/1.1

Content-Type: multipart/form-data
Accept: application/json

```

```python
import requests
headers = {
  'Content-Type': 'multipart/form-data',
  'Accept': 'application/json'
}

r = requests.post('/demo.biapi.pro/2.0/stet/v1/token', params={

}, headers = headers)

print r.json()

```

`POST /stet/v1/token`

In order to register a new user with the OAuth 2 process, the client has to call this endpoint to request a granted access_token with the received temporary code.<br><br>

> Body parameter

```yaml
grant_type: string
client_id: string
code: string
redirect_uri: string

```

<h3 id="transform-a-temporary-code-to-a-access_token-parameters">Parameters</h3>

|Name|In|Type|Required|Description|
|---|---|---|---|---|
|body|body|object|false|none|
|» grant_type|body|string|false|default is "authorization_code"|
|» client_id|body|string|true|ID of the client|
|» code|body|string|true|user's temporary code|
|» redirect_uri|body|string|false|redirect uri used by user|

> Example responses

> 200 Response

```json
{}
```

> OK

<h3 id="transform-a-temporary-code-to-a-access_token-responses">Responses</h3>

|Status|Meaning|Description|Schema|
|---|---|---|---|
|200|[OK](https://tools.ietf.org/html/rfc7231#section-6.3.1)|OK|Inline|

<h3 id="transform-a-temporary-code-to-a-access_token-responseschema">Response Schema</h3>

<aside class="success">
This operation does not require authentication
</aside>

<h1 id="budgea-api-documentation-terms">Terms</h1>

## Return the current terms and the content of the associated file

> Code samples

```http
GET /demo.biapi.pro/2.0/terms HTTP/1.1

Accept: application/json

```

```python
import requests
headers = {
  'Accept': 'application/json'
}

r = requests.get('/demo.biapi.pro/2.0/terms', params={

}, headers = headers)

print r.json()

```

`GET /terms`

<h3 id="return-the-current-terms-and-the-content-of-the-associated-file-parameters">Parameters</h3>

|Name|In|Type|Required|Description|
|---|---|---|---|---|
|expand|query|string|false|none|

> Example responses

> 200 Response

```json
{
  "termsofservices": [
    {
      "id": 0,
      "version": "",
      "id_file": 0,
      "created": "2019-03-20 11:42:25.546553",
      "deleted": "2019-03-20 11:42:25.546629"
    }
  ]
}
```

<h3 id="return-the-current-terms-and-the-content-of-the-associated-file-responses">Responses</h3>

|Status|Meaning|Description|Schema|
|---|---|---|---|
|200|[OK](https://tools.ietf.org/html/rfc7231#section-6.3.1)|termsofservices|Inline|

<h3 id="return-the-current-terms-and-the-content-of-the-associated-file-responseschema">Response Schema</h3>

Status Code **200**

|Name|Type|Required|Restrictions|Description|
|---|---|---|---|---|
|» termsofservices|[[TermsOfService](#schematermsofservice)]|true|none|none|
|»» id|integer|true|none|none|
|»» version|string|true|none|none|
|»» id_file|integer|false|none|none|
|»» created|string(date-time)|true|none|none|
|»» deleted|string(date-time)|false|none|none|

<aside class="success">
This operation does not require authentication
</aside>

## Register a version of 'Terms of Service' in database

> Code samples

```http
POST /demo.biapi.pro/2.0/terms HTTP/1.1

Content-Type: multipart/form-data
Accept: application/json

```

```python
import requests
headers = {
  'Content-Type': 'multipart/form-data',
  'Accept': 'application/json'
}

r = requests.post('/demo.biapi.pro/2.0/terms', params={

}, headers = headers)

print r.json()

```

`POST /terms`

> Body parameter

```yaml
version: string
file_content: string

```

<h3 id="register-a-version-of-'terms-of-service'-in-database-parameters">Parameters</h3>

|Name|In|Type|Required|Description|
|---|---|---|---|---|
|expand|query|string|false|none|
|body|body|object|false|none|
|» version|body|string|false|Number of version|
|» file_content|body|string|false|file containing the terms, optional|

> Example responses

> 200 Response

```json
{
  "id": 0,
  "version": "",
  "id_file": 0,
  "created": "2019-03-20 11:42:25.546553",
  "deleted": "2019-03-20 11:42:25.546629"
}
```

<h3 id="register-a-version-of-'terms-of-service'-in-database-responses">Responses</h3>

|Status|Meaning|Description|Schema|
|---|---|---|---|
|200|[OK](https://tools.ietf.org/html/rfc7231#section-6.3.1)|Successful POST on TermsOfService resource|[TermsOfService](#schematermsofservice)|

<aside class="success">
This operation does not require authentication
</aside>

## Get active terms object for a specific user, only one terms can be active

> Code samples

```http
GET /demo.biapi.pro/2.0/users/{id_user}/terms HTTP/1.1

Accept: application/json

```

```python
import requests
headers = {
  'Accept': 'application/json'
}

r = requests.get('/demo.biapi.pro/2.0/users/{id_user}/terms', params={

}, headers = headers)

print r.json()

```

`GET /users/{id_user}/terms`

<h3 id="get-active-terms-object-for-a-specific-user,-only-one-terms-can-be-active-parameters">Parameters</h3>

|Name|In|Type|Required|Description|
|---|---|---|---|---|
|id_user|path|string|true|Hint: you can use 'me' or 'all'|
|expand|query|string|false|none|

> Example responses

> 200 Response

```json
{
  "termsofservices": [
    {
      "id": 0,
      "version": "",
      "id_file": 0,
      "created": "2019-03-20 11:42:25.546553",
      "deleted": "2019-03-20 11:42:25.546629"
    }
  ]
}
```

<h3 id="get-active-terms-object-for-a-specific-user,-only-one-terms-can-be-active-responses">Responses</h3>

|Status|Meaning|Description|Schema|
|---|---|---|---|
|200|[OK](https://tools.ietf.org/html/rfc7231#section-6.3.1)|termsofservices|Inline|

<h3 id="get-active-terms-object-for-a-specific-user,-only-one-terms-can-be-active-responseschema">Response Schema</h3>

Status Code **200**

|Name|Type|Required|Restrictions|Description|
|---|---|---|---|---|
|» termsofservices|[[TermsOfService](#schematermsofservice)]|true|none|none|
|»» id|integer|true|none|none|
|»» version|string|true|none|none|
|»» id_file|integer|false|none|none|
|»» created|string(date-time)|true|none|none|
|»» deleted|string(date-time)|false|none|none|

<aside class="success">
This operation does not require authentication
</aside>

## Register user's consent for a specific terms id

> Code samples

```http
POST /demo.biapi.pro/2.0/users/{id_user}/terms HTTP/1.1

Content-Type: multipart/form-data
Accept: application/json

```

```python
import requests
headers = {
  'Content-Type': 'multipart/form-data',
  'Accept': 'application/json'
}

r = requests.post('/demo.biapi.pro/2.0/users/{id_user}/terms', params={

}, headers = headers)

print r.json()

```

`POST /users/{id_user}/terms`

> Body parameter

```yaml
id_terms: 0

```

<h3 id="register-user's-consent-for-a-specific-terms-id-parameters">Parameters</h3>

|Name|In|Type|Required|Description|
|---|---|---|---|---|
|id_user|path|string|true|Hint: you can use 'me' or 'all'|
|expand|query|string|false|none|
|body|body|object|false|none|
|» id_terms|body|integer|false|terms id|

> Example responses

> 200 Response

```json
{
  "id": 0,
  "version": "",
  "id_file": 0,
  "created": "2019-03-20 11:42:25.546553",
  "deleted": "2019-03-20 11:42:25.546629"
}
```

<h3 id="register-user's-consent-for-a-specific-terms-id-responses">Responses</h3>

|Status|Meaning|Description|Schema|
|---|---|---|---|
|200|[OK](https://tools.ietf.org/html/rfc7231#section-6.3.1)|Successful POST on TermsOfService resource|[TermsOfService](#schematermsofservice)|

<aside class="success">
This operation does not require authentication
</aside>

<h1 id="budgea-api-documentation-transfers">Transfers</h1>

## Returns the list of recipients.

> Code samples

```http
GET /demo.biapi.pro/2.0/users/{id_user}/recipients HTTP/1.1

Accept: application/json

```

```python
import requests
headers = {
  'Accept': 'application/json'
}

r = requests.get('/demo.biapi.pro/2.0/users/{id_user}/recipients', params={

}, headers = headers)

print r.json()

```

`GET /users/{id_user}/recipients`

<h3 id="returns-the-list-of-recipients.-parameters">Parameters</h3>

|Name|In|Type|Required|Description|
|---|---|---|---|---|
|id_user|path|string|true|Hint: you can use 'me' or 'all'|
|expand|query|string|false|none|

> Example responses

> 200 Response

```json
{
  "recipients": [
    {
      "id": 0,
      "id_account": 0,
      "id_target_account": 0,
      "label": "",
      "bank_name": "",
      "iban": "",
      "webid": "",
      "category": "",
      "last_update": "2019-03-20 11:42:25.676098",
      "time_scraped": "2019-03-20 11:42:25.676188",
      "deleted": "2019-03-20 11:42:25.676267",
      "expire": "2019-03-20 11:42:25.676349",
      "enabled_at": "2019-03-20 11:42:25.676426",
      "add_verified": false,
      "state": "",
      "error": "",
      "currency": ""
    }
  ]
}
```

<h3 id="returns-the-list-of-recipients.-responses">Responses</h3>

|Status|Meaning|Description|Schema|
|---|---|---|---|
|200|[OK](https://tools.ietf.org/html/rfc7231#section-6.3.1)|recipients|Inline|

<h3 id="returns-the-list-of-recipients.-responseschema">Response Schema</h3>

Status Code **200**

|Name|Type|Required|Restrictions|Description|
|---|---|---|---|---|
|» recipients|[[Recipient](#schemarecipient)]|true|none|none|
|»» id|integer|true|none|ID of the recipient|
|»» id_account|integer|true|none|ID of the related account|
|»» id_target_account|integer|false|none|ID of the target account, in case of internal recipient|
|»» label|string|true|none|Label of the recipient|
|»» bank_name|string|false|none|Bank of the recipient|
|»» iban|string|false|none|IBAN of the recipient|
|»» webid|string|false|none|Webid of the recipient|
|»» category|string|true|none|Category in which the recipient is|
|»» last_update|string(date-time)|true|none|Last time we have fetched this recipient|
|»» time_scraped|string(date-time)|false|none|First time we've seen this recipient|
|»» deleted|string(date-time)|false|none|The recipient isn't found anymore on the bank|
|»» expire|string(date-time)|false|none|none|
|»» enabled_at|string(date-time)|false|none|It will be possible to do transfers to this recipient at this date|
|»» add_verified|boolean|false|none|Was the recipient adding authorized|
|»» state|string|false|none|State of recipient|
|»» error|string|false|none|Error message|
|»» fields|string|false|none|Fields for recipient with additionalInformationNeeded state|
|»» currency|object|false|none|Currency of the object|

<aside class="success">
This operation does not require authentication
</aside>

## Continue addition of a recipient.

> Code samples

```http
POST /demo.biapi.pro/2.0/users/{id_user}/recipients/{id_recipient} HTTP/1.1

Accept: application/json

```

```python
import requests
headers = {
  'Accept': 'application/json'
}

r = requests.post('/demo.biapi.pro/2.0/users/{id_user}/recipients/{id_recipient}', params={

}, headers = headers)

print r.json()

```

`POST /users/{id_user}/recipients/{id_recipient}`

<h3 id="continue-addition-of-a-recipient.-parameters">Parameters</h3>

|Name|In|Type|Required|Description|
|---|---|---|---|---|
|id_user|path|string|true|Hint: you can use 'me' or 'all'|
|id_recipient|path|integer|true|none|
|background|query|boolean|false|if true, do synchronization in background|
|expand|query|string|false|none|

> Example responses

> 200 Response

```json
{
  "id": 0,
  "id_account": 0,
  "id_target_account": 0,
  "label": "",
  "bank_name": "",
  "iban": "",
  "webid": "",
  "category": "",
  "last_update": "2019-03-20 11:42:25.676098",
  "time_scraped": "2019-03-20 11:42:25.676188",
  "deleted": "2019-03-20 11:42:25.676267",
  "expire": "2019-03-20 11:42:25.676349",
  "enabled_at": "2019-03-20 11:42:25.676426",
  "add_verified": false,
  "state": "",
  "error": "",
  "currency": ""
}
```

<h3 id="continue-addition-of-a-recipient.-responses">Responses</h3>

|Status|Meaning|Description|Schema|
|---|---|---|---|
|200|[OK](https://tools.ietf.org/html/rfc7231#section-6.3.1)|Successful POST on Recipient resource|[Recipient](#schemarecipient)|

<aside class="success">
This operation does not require authentication
</aside>

## Get transfers

> Code samples

```http
GET /demo.biapi.pro/2.0/users/{id_user}/transfers HTTP/1.1

Accept: application/json

```

```python
import requests
headers = {
  'Accept': 'application/json'
}

r = requests.get('/demo.biapi.pro/2.0/users/{id_user}/transfers', params={

}, headers = headers)

print r.json()

```

`GET /users/{id_user}/transfers`

<h3 id="get-transfers-parameters">Parameters</h3>

|Name|In|Type|Required|Description|
|---|---|---|---|---|
|id_user|path|string|true|Hint: you can use 'me' or 'all'|
|period|query|string|false|period to group logs|
|expand|query|string|false|none|

> Example responses

> 200 Response

```json
{
  "transfers": [
    {
      "id": 0,
      "id_account": 0,
      "id_user": 0,
      "id_recipient": 0,
      "account_iban": "",
      "recipient_iban": "",
      "exec_date": "2019-03-20",
      "register_date": "2019-03-20 11:42:25.597683",
      "amount": 0,
      "fees": 0,
      "webid": "",
      "state": "",
      "error": "",
      "label": "",
      "account_balance": 0,
      "id_transaction": 0,
      "currency": ""
    }
  ]
}
```

<h3 id="get-transfers-responses">Responses</h3>

|Status|Meaning|Description|Schema|
|---|---|---|---|
|200|[OK](https://tools.ietf.org/html/rfc7231#section-6.3.1)|transfers|Inline|

<h3 id="get-transfers-responseschema">Response Schema</h3>

Status Code **200**

|Name|Type|Required|Restrictions|Description|
|---|---|---|---|---|
|» transfers|[[Transfer](#schematransfer)]|true|none|[This is a representation of a transfer.]|
|»» id|integer|true|none|ID of transfer|
|»» id_account|integer|false|none|ID of the debited account|
|»» id_user|integer|false|none|ID of the related user|
|»» id_recipient|integer|false|none|ID of the recipient|
|»» account_iban|string|false|none|IBAN of the debited account|
|»» recipient_iban|string|false|none|IBAN of the recipient|
|»» exec_date|string(date)|true|none|Date when the transfer will be operated by the bank|
|»» register_date|string(date-time)|true|none|Date when the transfer has been registered|
|»» amount|number(float)|true|none|Amount of the transfer|
|»» fees|number(float)|false|none|Fees taken by the bank|
|»» webid|string|false|none|WebID of the transfer|
|»» state|string|true|none|State of the transfer (created, scheduled, validating, pending, done, canceled, error, bug)|
|»» error|string|false|none|Error message during transfer, if any|
|»» label|string|false|none|Label of the transfer|
|»» account_balance|number(float)|false|none|Balance of the account just before the transfer|
|»» id_transaction|integer|false|none|If found, ID of the related transaction|
|»» currency|object|false|none|Currency of the object|

<aside class="success">
This operation does not require authentication
</aside>

## Create a transfer object.

> Code samples

```http
POST /demo.biapi.pro/2.0/users/{id_user}/transfers HTTP/1.1

Content-Type: multipart/form-data
Accept: application/json

```

```python
import requests
headers = {
  'Content-Type': 'multipart/form-data',
  'Accept': 'application/json'
}

r = requests.post('/demo.biapi.pro/2.0/users/{id_user}/transfers', params={

}, headers = headers)

print r.json()

```

`POST /users/{id_user}/transfers`

> Body parameter

```yaml
amount: 0
label: string
exec_date: '2019-03-20'

```

<h3 id="create-a-transfer-object.-parameters">Parameters</h3>

|Name|In|Type|Required|Description|
|---|---|---|---|---|
|id_user|path|string|true|Hint: you can use 'me' or 'all'|
|expand|query|string|false|none|
|body|body|[postUsers_idUser_accounts_idAccount_recipients_idRecipient_transfers](#schemapostusers_iduser_accounts_idaccount_recipients_idrecipient_transfers)|false|none|
|» amount|body|number(float)|true|amount of transfer|
|» label|body|string|false|reason of transfer|
|» exec_date|body|string(date)|false|excution date of transfer|

> Example responses

> 200 Response

```json
{
  "id": 0,
  "id_account": 0,
  "id_user": 0,
  "id_recipient": 0,
  "account_iban": "",
  "recipient_iban": "",
  "exec_date": "2019-03-20",
  "register_date": "2019-03-20 11:42:25.597683",
  "amount": 0,
  "fees": 0,
  "webid": "",
  "state": "",
  "error": "",
  "label": "",
  "account_balance": 0,
  "id_transaction": 0,
  "currency": ""
}
```

<h3 id="create-a-transfer-object.-responses">Responses</h3>

|Status|Meaning|Description|Schema|
|---|---|---|---|
|200|[OK](https://tools.ietf.org/html/rfc7231#section-6.3.1)|Successful POST on Transfer resource|[Transfer](#schematransfer)|

<aside class="success">
This operation does not require authentication
</aside>

## Execute or edit a Transfer.

> Code samples

```http
POST /demo.biapi.pro/2.0/users/{id_user}/transfers/{id_transfer} HTTP/1.1

Content-Type: multipart/form-data
Accept: application/json

```

```python
import requests
headers = {
  'Content-Type': 'multipart/form-data',
  'Accept': 'application/json'
}

r = requests.post('/demo.biapi.pro/2.0/users/{id_user}/transfers/{id_transfer}', params={

}, headers = headers)

print r.json()

```

`POST /users/{id_user}/transfers/{id_transfer}`

> Body parameter

```yaml
validated: true
id_recipient: 0

```

<h3 id="execute-or-edit-a-transfer.-parameters">Parameters</h3>

|Name|In|Type|Required|Description|
|---|---|---|---|---|
|id_user|path|string|true|Hint: you can use 'me' or 'all'|
|id_transfer|path|integer|true|none|
|expand|query|string|false|none|
|body|body|[postUsers_idUser_accounts_idAccount_recipients_idRecipient_transfers_idTransfer_](#schemapostusers_iduser_accounts_idaccount_recipients_idrecipient_transfers_idtransfer_)|false|none|
|» validated|body|boolean|false|set it to initialize transfer on the bank website.|
|» id_recipient|body|integer|false|set the recipient of the transfer|

> Example responses

> 200 Response

```json
{
  "id": 0,
  "id_account": 0,
  "id_user": 0,
  "id_recipient": 0,
  "account_iban": "",
  "recipient_iban": "",
  "exec_date": "2019-03-20",
  "register_date": "2019-03-20 11:42:25.597683",
  "amount": 0,
  "fees": 0,
  "webid": "",
  "state": "",
  "error": "",
  "label": "",
  "account_balance": 0,
  "id_transaction": 0,
  "currency": ""
}
```

<h3 id="execute-or-edit-a-transfer.-responses">Responses</h3>

|Status|Meaning|Description|Schema|
|---|---|---|---|
|200|[OK](https://tools.ietf.org/html/rfc7231#section-6.3.1)|Successful POST on Transfer resource|[Transfer](#schematransfer)|

<aside class="success">
This operation does not require authentication
</aside>

## Cancel a transfer.

> Code samples

```http
DELETE /demo.biapi.pro/2.0/users/{id_user}/transfers/{id_transfer} HTTP/1.1

Accept: application/json

```

```python
import requests
headers = {
  'Accept': 'application/json'
}

r = requests.delete('/demo.biapi.pro/2.0/users/{id_user}/transfers/{id_transfer}', params={

}, headers = headers)

print r.json()

```

`DELETE /users/{id_user}/transfers/{id_transfer}`

It is possible to cancel only a transfer in state 'created'.<br><br>

<h3 id="cancel-a-transfer.-parameters">Parameters</h3>

|Name|In|Type|Required|Description|
|---|---|---|---|---|
|id_user|path|string|true|Hint: you can use 'me' or 'all'|
|id_transfer|path|integer|true|none|
|expand|query|string|false|none|

> Example responses

> 200 Response

```json
{
  "id": 0,
  "id_account": 0,
  "id_user": 0,
  "id_recipient": 0,
  "account_iban": "",
  "recipient_iban": "",
  "exec_date": "2019-03-20",
  "register_date": "2019-03-20 11:42:25.597683",
  "amount": 0,
  "fees": 0,
  "webid": "",
  "state": "",
  "error": "",
  "label": "",
  "account_balance": 0,
  "id_transaction": 0,
  "currency": ""
}
```

<h3 id="cancel-a-transfer.-responses">Responses</h3>

|Status|Meaning|Description|Schema|
|---|---|---|---|
|200|[OK](https://tools.ietf.org/html/rfc7231#section-6.3.1)|Successful DELETE on Transfer resource|[Transfer](#schematransfer)|

<aside class="success">
This operation does not require authentication
</aside>

<h1 id="budgea-api-documentation-users-management">Users management</h1>

## Delete the user's connections

> Code samples

```http
DELETE /demo.biapi.pro/2.0/hash HTTP/1.1

```

```python
import requests

r = requests.delete('/demo.biapi.pro/2.0/hash', params={

)

print r.json()

```

`DELETE /hash`

deletes all connections of the user given his hash<br><br>

<h3 id="delete-the-user's-connections-responses">Responses</h3>

|Status|Meaning|Description|Schema|
|---|---|---|---|
|200|[OK](https://tools.ietf.org/html/rfc7231#section-6.3.1)|standard HTTP response|None|

<aside class="success">
This operation does not require authentication
</aside>

## Get users

> Code samples

```http
GET /demo.biapi.pro/2.0/users HTTP/1.1

Accept: application/json

```

```python
import requests
headers = {
  'Accept': 'application/json'
}

r = requests.get('/demo.biapi.pro/2.0/users', params={

}, headers = headers)

print r.json()

```

`GET /users`

<h3 id="get-users-parameters">Parameters</h3>

|Name|In|Type|Required|Description|
|---|---|---|---|---|
|search|query|string|false|searches a user by mail (if it contains no '@', '@biapi.pro' will be added at the end)|
|expand|query|string|false|none|

> Example responses

> 200 Response

```json
{
  "users": [
    {
      "id": 0,
      "signin": "CURRENT_TIMESTAMP",
      "platform": ""
    }
  ]
}
```

<h3 id="get-users-responses">Responses</h3>

|Status|Meaning|Description|Schema|
|---|---|---|---|
|200|[OK](https://tools.ietf.org/html/rfc7231#section-6.3.1)|users|Inline|

<h3 id="get-users-responseschema">Response Schema</h3>

Status Code **200**

|Name|Type|Required|Restrictions|Description|
|---|---|---|---|---|
|» users|[[User](#schemauser)]|true|none|none|
|»» id|integer|true|none|none|
|»» signin|string(date-time)|true|none|none|
|»» platform|string|true|none|none|

#### Enumerated Values

|Property|Value|
|---|---|
|platform|web|
|platform|iPad|
|platform|iPhone|
|platform|Android|
|platform|CAstore|
|platform|requestAccess|
|platform|sharedAccess|
|platform|transfer|

<aside class="success">
This operation does not require authentication
</aside>

## Get a user

> Code samples

```http
GET /demo.biapi.pro/2.0/users/{id_user} HTTP/1.1

Accept: application/json

```

```python
import requests
headers = {
  'Accept': 'application/json'
}

r = requests.get('/demo.biapi.pro/2.0/users/{id_user}', params={

}, headers = headers)

print r.json()

```

`GET /users/{id_user}`

<h3 id="get-a-user-parameters">Parameters</h3>

|Name|In|Type|Required|Description|
|---|---|---|---|---|
|id_user|path|string|true|Hint: you can use 'me' or 'all'|
|expand|query|string|false|none|

> Example responses

> 200 Response

```json
{
  "id": 0,
  "signin": "CURRENT_TIMESTAMP",
  "platform": ""
}
```

<h3 id="get-a-user-responses">Responses</h3>

|Status|Meaning|Description|Schema|
|---|---|---|---|
|200|[OK](https://tools.ietf.org/html/rfc7231#section-6.3.1)|Successful GET on User resource|[User](#schemauser)|

<aside class="success">
This operation does not require authentication
</aside>

## Get configuration of a user.

> Code samples

```http
GET /demo.biapi.pro/2.0/users/{id_user}/config HTTP/1.1

Accept: application/json

```

```python
import requests
headers = {
  'Accept': 'application/json'
}

r = requests.get('/demo.biapi.pro/2.0/users/{id_user}/config', params={

}, headers = headers)

print r.json()

```

`GET /users/{id_user}/config`

<br><br>

<h3 id="get-configuration-of-a-user.-parameters">Parameters</h3>

|Name|In|Type|Required|Description|
|---|---|---|---|---|
|id_user|path|string|true|Hint: you can use 'me' or 'all'|

> Example responses

> 200 Response

```json
{}
```

> OK

<h3 id="get-configuration-of-a-user.-responses">Responses</h3>

|Status|Meaning|Description|Schema|
|---|---|---|---|
|200|[OK](https://tools.ietf.org/html/rfc7231#section-6.3.1)|OK|Inline|

<h3 id="get-configuration-of-a-user.-responseschema">Response Schema</h3>

<aside class="success">
This operation does not require authentication
</aside>

## Change configuration of a user. modifications on keys prefixed by 'biapi.' (except callback_url) are ignored

> Code samples

```http
POST /demo.biapi.pro/2.0/users/{id_user}/config HTTP/1.1

```

```python
import requests

r = requests.post('/demo.biapi.pro/2.0/users/{id_user}/config', params={

)

print r.json()

```

`POST /users/{id_user}/config`

<h3 id="change-configuration-of-a-user.-modifications-on-keys-prefixed-by-'biapi.'-(except-callback_url)-are-ignored-parameters">Parameters</h3>

|Name|In|Type|Required|Description|
|---|---|---|---|---|
|id_user|path|string|true|Hint: you can use 'me' or 'all'|

<h3 id="change-configuration-of-a-user.-modifications-on-keys-prefixed-by-'biapi.'-(except-callback_url)-are-ignored-responses">Responses</h3>

|Status|Meaning|Description|Schema|
|---|---|---|---|
|200|[OK](https://tools.ietf.org/html/rfc7231#section-6.3.1)|standard HTTP response|None|

<aside class="success">
This operation does not require authentication
</aside>

## Delete the given user configurations. deletions on keys prefixed by 'biapi.' (except callback_url) are ignored

> Code samples

```http
DELETE /demo.biapi.pro/2.0/users/{id_user}/config HTTP/1.1

```

```python
import requests

r = requests.delete('/demo.biapi.pro/2.0/users/{id_user}/config', params={

)

print r.json()

```

`DELETE /users/{id_user}/config`

- keys (string): list of coma separated keys to be deleted.<br><br>

<h3 id="delete-the-given-user-configurations.-deletions-on-keys-prefixed-by-'biapi.'-(except-callback_url)-are-ignored-parameters">Parameters</h3>

|Name|In|Type|Required|Description|
|---|---|---|---|---|
|id_user|path|string|true|Hint: you can use 'me' or 'all'|

<h3 id="delete-the-given-user-configurations.-deletions-on-keys-prefixed-by-'biapi.'-(except-callback_url)-are-ignored-responses">Responses</h3>

|Status|Meaning|Description|Schema|
|---|---|---|---|
|200|[OK](https://tools.ietf.org/html/rfc7231#section-6.3.1)|standard HTTP response|None|

<aside class="success">
This operation does not require authentication
</aside>

## Get profiles

> Code samples

```http
GET /demo.biapi.pro/2.0/users/{id_user}/profiles HTTP/1.1

Accept: application/json

```

```python
import requests
headers = {
  'Accept': 'application/json'
}

r = requests.get('/demo.biapi.pro/2.0/users/{id_user}/profiles', params={

}, headers = headers)

print r.json()

```

`GET /users/{id_user}/profiles`

<h3 id="get-profiles-parameters">Parameters</h3>

|Name|In|Type|Required|Description|
|---|---|---|---|---|
|id_user|path|string|true|Hint: you can use 'me' or 'all'|
|expand|query|string|false|none|

> Example responses

> 200 Response

```json
{
  "profiles": [
    {
      "id": 0,
      "id_user": 0,
      "role": "admin",
      "statut": 0,
      "admin": false,
      "conf": "",
      "lang": ""
    }
  ]
}
```

<h3 id="get-profiles-responses">Responses</h3>

|Status|Meaning|Description|Schema|
|---|---|---|---|
|200|[OK](https://tools.ietf.org/html/rfc7231#section-6.3.1)|profiles|Inline|

<h3 id="get-profiles-responseschema">Response Schema</h3>

Status Code **200**

|Name|Type|Required|Restrictions|Description|
|---|---|---|---|---|
|» profiles|[[Profile](#schemaprofile)]|true|none|none|
|»» id|integer|true|none|none|
|»» id_user|integer|true|none|none|
|»» role|string|true|none|none|
|»» email|string|true|none|none|
|»» statut|integer|true|none|none|
|»» admin|boolean|true|none|none|
|»» conf|string|false|none|none|
|»» lang|string|false|none|none|

#### Enumerated Values

|Property|Value|
|---|---|
|role|admin|
|role|ser|

<aside class="success">
This operation does not require authentication
</aside>

## Get the main profile

> Code samples

```http
GET /demo.biapi.pro/2.0/users/{id_user}/profiles/main HTTP/1.1

Accept: application/json

```

```python
import requests
headers = {
  'Accept': 'application/json'
}

r = requests.get('/demo.biapi.pro/2.0/users/{id_user}/profiles/main', params={

}, headers = headers)

print r.json()

```

`GET /users/{id_user}/profiles/main`

<h3 id="get-the-main-profile-parameters">Parameters</h3>

|Name|In|Type|Required|Description|
|---|---|---|---|---|
|id_user|path|string|true|Hint: you can use 'me' or 'all'|
|expand|query|string|false|none|

> Example responses

> 200 Response

```json
{
  "id": 0,
  "id_user": 0,
  "role": "admin",
  "statut": 0,
  "admin": false,
  "conf": "",
  "lang": ""
}
```

<h3 id="get-the-main-profile-responses">Responses</h3>

|Status|Meaning|Description|Schema|
|---|---|---|---|
|200|[OK](https://tools.ietf.org/html/rfc7231#section-6.3.1)|Successful GET on Profile resource|[Profile](#schemaprofile)|

<aside class="success">
This operation does not require authentication
</aside>

## Get my profile

> Code samples

```http
GET /demo.biapi.pro/2.0/users/{id_user}/profiles/me HTTP/1.1

Accept: application/json

```

```python
import requests
headers = {
  'Accept': 'application/json'
}

r = requests.get('/demo.biapi.pro/2.0/users/{id_user}/profiles/me', params={

}, headers = headers)

print r.json()

```

`GET /users/{id_user}/profiles/me`

<h3 id="get-my-profile-parameters">Parameters</h3>

|Name|In|Type|Required|Description|
|---|---|---|---|---|
|id_user|path|string|true|Hint: you can use 'me' or 'all'|
|expand|query|string|false|none|

> Example responses

> 200 Response

```json
{
  "id": 0,
  "id_user": 0,
  "role": "admin",
  "statut": 0,
  "admin": false,
  "conf": "",
  "lang": ""
}
```

<h3 id="get-my-profile-responses">Responses</h3>

|Status|Meaning|Description|Schema|
|---|---|---|---|
|200|[OK](https://tools.ietf.org/html/rfc7231#section-6.3.1)|Successful GET on Profile resource|[Profile](#schemaprofile)|

<aside class="success">
This operation does not require authentication
</aside>

## Get a profile

> Code samples

```http
GET /demo.biapi.pro/2.0/users/{id_user}/profiles/{id_profile} HTTP/1.1

Accept: application/json

```

```python
import requests
headers = {
  'Accept': 'application/json'
}

r = requests.get('/demo.biapi.pro/2.0/users/{id_user}/profiles/{id_profile}', params={

}, headers = headers)

print r.json()

```

`GET /users/{id_user}/profiles/{id_profile}`

<h3 id="get-a-profile-parameters">Parameters</h3>

|Name|In|Type|Required|Description|
|---|---|---|---|---|
|id_user|path|string|true|Hint: you can use 'me' or 'all'|
|id_profile|path|integer|true|none|
|expand|query|string|false|none|

> Example responses

> 200 Response

```json
{
  "id": 0,
  "id_user": 0,
  "role": "admin",
  "statut": 0,
  "admin": false,
  "conf": "",
  "lang": ""
}
```

<h3 id="get-a-profile-responses">Responses</h3>

|Status|Meaning|Description|Schema|
|---|---|---|---|
|200|[OK](https://tools.ietf.org/html/rfc7231#section-6.3.1)|Successful GET on Profile resource|[Profile](#schemaprofile)|

<aside class="success">
This operation does not require authentication
</aside>

## Create a token

> Code samples

```http
POST /demo.biapi.pro/2.0/users/{id_user}/token HTTP/1.1

Content-Type: multipart/form-data
Accept: application/json

```

```python
import requests
headers = {
  'Content-Type': 'multipart/form-data',
  'Accept': 'application/json'
}

r = requests.post('/demo.biapi.pro/2.0/users/{id_user}/token', params={

}, headers = headers)

print r.json()

```

`POST /users/{id_user}/token`

Create an access_token for this user and get it.<br><br>

> Body parameter

```yaml
application: string

```

<h3 id="create-a-token-parameters">Parameters</h3>

|Name|In|Type|Required|Description|
|---|---|---|---|---|
|id_user|path|string|true|Hint: you can use 'me' or 'all'|
|body|body|object|false|none|
|» application|body|string|true|application name|

> Example responses

> 200 Response

```json
{}
```

> OK

<h3 id="create-a-token-responses">Responses</h3>

|Status|Meaning|Description|Schema|
|---|---|---|---|
|200|[OK](https://tools.ietf.org/html/rfc7231#section-6.3.1)|OK|Inline|

<h3 id="create-a-token-responseschema">Response Schema</h3>

<aside class="success">
This operation does not require authentication
</aside>

<h1 id="budgea-api-documentation-wealth">Wealth</h1>

## Get securities

> Code samples

```http
GET /demo.biapi.pro/2.0/finance/securities HTTP/1.1

Accept: application/json

```

```python
import requests
headers = {
  'Accept': 'application/json'
}

r = requests.get('/demo.biapi.pro/2.0/finance/securities', params={

}, headers = headers)

print r.json()

```

`GET /finance/securities`

<h3 id="get-securities-parameters">Parameters</h3>

|Name|In|Type|Required|Description|
|---|---|---|---|---|
|expand|query|string|false|none|

> Example responses

> 200 Response

```json
{
  "securities": [
    {
      "id": 0,
      "code": "",
      "name": "",
      "id_type": 0,
      "last_update": "2019-03-20 11:42:25.559779"
    }
  ]
}
```

<h3 id="get-securities-responses">Responses</h3>

|Status|Meaning|Description|Schema|
|---|---|---|---|
|200|[OK](https://tools.ietf.org/html/rfc7231#section-6.3.1)|securities|Inline|

<h3 id="get-securities-responseschema">Response Schema</h3>

Status Code **200**

|Name|Type|Required|Restrictions|Description|
|---|---|---|---|---|
|» securities|[[Security](#schemasecurity)]|true|none|none|
|»» id|integer|true|none|ID of the security|
|»» code|string|false|none|ISIN code of the security|
|»» name|string|true|none|Name of the security|
|»» id_type|integer|false|none|ID of the security type|
|»» last_update|string(date-time)|false|none|Last update of the security|

<aside class="success">
This operation does not require authentication
</aside>

## Get connection logs

> Code samples

```http
GET /demo.biapi.pro/2.0/users/{id_user}/investments/{id_investment}/security/history HTTP/1.1

```

```python
import requests

r = requests.get('/demo.biapi.pro/2.0/users/{id_user}/investments/{id_investment}/security/history', params={

)

print r.json()

```

`GET /users/{id_user}/investments/{id_investment}/security/history`

Get logs about connections.<br><br>

<h3 id="get-connection-logs-parameters">Parameters</h3>

|Name|In|Type|Required|Description|
|---|---|---|---|---|
|id_user|path|string|true|Hint: you can use 'me' or 'all'|
|id_investment|path|integer|true|none|
|limit|query|integer|false|limit number of results|
|offset|query|integer|false|offset of first result|
|min_date|query|string(date)|false|minimal date|
|max_date|query|string(date)|false|maximum date|
|period|query|string|false|period to group logs|

<h3 id="get-connection-logs-responses">Responses</h3>

|Status|Meaning|Description|Schema|
|---|---|---|---|
|200|[OK](https://tools.ietf.org/html/rfc7231#section-6.3.1)|standard HTTP response|None|

<aside class="success">
This operation does not require authentication
</aside>

## Get investments

> Code samples

```http
GET /demo.biapi.pro/2.0/users/{id_user}/investments HTTP/1.1

Accept: application/json

```

```python
import requests
headers = {
  'Accept': 'application/json'
}

r = requests.get('/demo.biapi.pro/2.0/users/{id_user}/investments', params={

}, headers = headers)

print r.json()

```

`GET /users/{id_user}/investments`

<h3 id="get-investments-parameters">Parameters</h3>

|Name|In|Type|Required|Description|
|---|---|---|---|---|
|id_user|path|string|true|Hint: you can use 'me' or 'all'|
|expand|query|string|false|none|

> Example responses

> 200 Response

```json
{
  "investments": [
    {
      "id": 0,
      "id_account": 0,
      "label": "",
      "code": "",
      "code_type": "",
      "source": "",
      "description": "",
      "quantity": 0,
      "unitprice": 0,
      "unitvalue": 0,
      "valuation": 0,
      "diff": 0,
      "diff_percent": 0,
      "prev_diff": 0,
      "portfolio_share": 0,
      "vdate": "2019-03-20",
      "prev_vdate": "2019-03-20",
      "id_security": 0,
      "original_currency": "",
      "original_valuation": 0,
      "original_unitvalue": 0,
      "original_unitprice": 0,
      "original_diff": 0,
      "last_update": "2019-03-20 11:42:25.672607",
      "deleted": "2019-03-20 11:42:25.672676"
    }
  ]
}
```

<h3 id="get-investments-responses">Responses</h3>

|Status|Meaning|Description|Schema|
|---|---|---|---|
|200|[OK](https://tools.ietf.org/html/rfc7231#section-6.3.1)|investments|Inline|

<h3 id="get-investments-responseschema">Response Schema</h3>

Status Code **200**

|Name|Type|Required|Restrictions|Description|
|---|---|---|---|---|
|» investments|[[Investment](#schemainvestment)]|true|none|none|
|»» id|integer|true|none|ID of the investment|
|»» id_account|integer|true|none|ID of the related account|
|»» label|string|true|none|Label of the investment|
|»» code|string|true|none|Investment code|
|»» code_type|string|false|none|Code type (ISIN of AMF)|
|»» source|string|false|none|Source of the ISIN code (website, notFound)|
|»» description|string|false|none|Description of the investment|
|»» quantity|number(float)|false|none|Quantity|
|»» unitprice|number(float)|false|none|Average buy price|
|»» unitvalue|number(float)|false|none|Current unit value|
|»» valuation|number(float)|false|none|Current valuation|
|»» diff|number(float)|false|none|Capital gain|
|»» diff_percent|number(float)|false|none|Capital gain in percent (between 0 and 1)|
|»» prev_diff|number(float)|false|none|Capital gain from previous value|
|»» portfolio_share|number(float)|false|none|Percent of the portfolio|
|»» vdate|string(date)|false|none|Value date|
|»» prev_vdate|string(date)|false|none|Value date of the previous value (prev_diff)|
|»» id_security|integer|false|none|ID of the related security|
|»» original_currency|object|false|none|Original currency|
|»» original_valuation|number(float)|false|none|Valuation in original currency|
|»» original_unitvalue|number(float)|false|none|Average buy price in the original currency|
|»» original_unitprice|number(float)|false|none|Current unit value in the original currency|
|»» original_diff|number(float)|false|none|Capital gain in the original currency|
|»» last_update|string(date-time)|false|none|Last update of the investment|
|»» deleted|string(date-time)|false|none|If set, this investment has been removed from the website|

<aside class="success">
This operation does not require authentication
</aside>

## Get investment values

> Code samples

```http
GET /demo.biapi.pro/2.0/users/{id_user}/investments/{id_investment}/history HTTP/1.1

Accept: application/json

```

```python
import requests
headers = {
  'Accept': 'application/json'
}

r = requests.get('/demo.biapi.pro/2.0/users/{id_user}/investments/{id_investment}/history', params={

}, headers = headers)

print r.json()

```

`GET /users/{id_user}/investments/{id_investment}/history`

<h3 id="get-investment-values-parameters">Parameters</h3>

|Name|In|Type|Required|Description|
|---|---|---|---|---|
|id_user|path|string|true|Hint: you can use 'me' or 'all'|
|id_investment|path|integer|true|none|
|expand|query|string|false|none|

> Example responses

> 200 Response

```json
{
  "investmentvalues": [
    {
      "id": 0,
      "id_investment": 0,
      "vdate": "2019-03-20",
      "unitvalue": 0,
      "original_currency": "",
      "original_unitvalue": 0
    }
  ]
}
```

<h3 id="get-investment-values-responses">Responses</h3>

|Status|Meaning|Description|Schema|
|---|---|---|---|
|200|[OK](https://tools.ietf.org/html/rfc7231#section-6.3.1)|investmentvalues|Inline|

<h3 id="get-investment-values-responseschema">Response Schema</h3>

Status Code **200**

|Name|Type|Required|Restrictions|Description|
|---|---|---|---|---|
|» investmentvalues|[[InvestmentValue](#schemainvestmentvalue)]|true|none|none|
|»» id|integer|true|none|ID of the value|
|»» id_investment|integer|true|none|ID of the related investment|
|»» vdate|string(date)|true|none|Date of this value|
|»» unitvalue|number(float)|true|none|Value on this date|
|»» original_currency|object|false|none|Original currency|
|»» original_unitvalue|number(float)|false|none|Value on this date, in the original currency|

<aside class="success">
This operation does not require authentication
</aside>

# Schemas

<h2 id="tocSconnector">Connector</h2>

<a id="schemaconnector"></a>

```json
{
  "id": 0,
  "name": "",
  "id_weboob": "",
  "hidden": false,
  "charged": true,
  "code": "",
  "beta": false,
  "color": "",
  "slug": "",
  "sync_frequency": 0,
  "months_to_fetch": 0,
  "auth_mechanism": ""
}

```

### Properties

|Name|Type|Required|Restrictions|Description|
|---|---|---|---|---|
|id|integer|true|none|ID of the connector|
|name|string|true|none|Name of the bank or provider|
|id_weboob|string|true|none|none|
|hidden|boolean|false|none|This connector is hidden from your users|
|charged|boolean|true|none|Usage of this connector is charged|
|code|string|false|none|Bank code|
|beta|boolean|true|none|If true, this connector is perhaps unstable :)|
|color|string|false|none|Main color of the bank or provider|
|slug|string|false|none|none|
|sync_frequency|number(float)|false|none|How many days to wait between syncs|
|months_to_fetch|integer|false|none|How many months of history to fetch|
|auth_mechanism|string|false|none|Authentication mechanism to use|

<h2 id="tocSfield">Field</h2>

<a id="schemafield"></a>

```json
{
  "id_connector": 0,
  "id": 0,
  "name": "",
  "label": "",
  "regex": "",
  "type": "text",
  "ephemeral": false,
  "value": "",
  "required": true
}

```

### Properties

|Name|Type|Required|Restrictions|Description|
|---|---|---|---|---|
|id_connector|integer|true|none|ID of the related connector|
|id|integer|true|none|ID of the field|
|name|string|true|none|Name of the field|
|label|string|true|none|Label to display to user|
|regex|string|false|none|If set, the value must match this regexp|
|type|string|false|none|Type of field (text, password, list, hidden)|
|ephemeral|boolean|false|none|This field will not be saved in database|
|value|string|false|none|Default value of the field|
|required|boolean|false|none|If true, field has to be set to synchronize the connection|

<h2 id="tocSconnectorlogo">ConnectorLogo</h2>

<a id="schemaconnectorlogo"></a>

```json
{
  "id": 0,
  "id_connector": 0,
  "id_file": 0,
  "type": ""
}

```

### Properties

|Name|Type|Required|Restrictions|Description|
|---|---|---|---|---|
|id|integer|true|none|none|
|id_connector|integer|true|none|ID of the connector|
|id_file|integer|true|none|Id of the Bank/Provider Logo|
|type|string|false|none|Logo's type|

<h2 id="tocSuser">User</h2>

<a id="schemauser"></a>

```json
{
  "id": 0,
  "signin": "CURRENT_TIMESTAMP",
  "platform": ""
}

```

### Properties

|Name|Type|Required|Restrictions|Description|
|---|---|---|---|---|
|id|integer|true|none|none|
|signin|string(date-time)|true|none|none|
|platform|string|true|none|none|

#### Enumerated Values

|Property|Value|
|---|---|
|platform|web|
|platform|iPad|
|platform|iPhone|
|platform|Android|
|platform|CAstore|
|platform|requestAccess|
|platform|sharedAccess|
|platform|transfer|

<h2 id="tocStermsofservice">TermsOfService</h2>

<a id="schematermsofservice"></a>

```json
{
  "id": 0,
  "version": "",
  "id_file": 0,
  "created": "2019-03-20 11:42:25.546553",
  "deleted": "2019-03-20 11:42:25.546629"
}

```

### Properties

|Name|Type|Required|Restrictions|Description|
|---|---|---|---|---|
|id|integer|true|none|none|
|version|string|true|none|none|
|id_file|integer|false|none|none|
|created|string(date-time)|true|none|none|
|deleted|string(date-time)|false|none|none|

<h2 id="tocScategory">Category</h2>

<a id="schemacategory"></a>

```json
{
  "id": 0,
  "id_parent_category": 0,
  "name": "",
  "income": false,
  "color": "",
  "id_parent_category_in_menu": 0,
  "name_displayed": "",
  "refundable": false,
  "id_user": 0,
  "id_logo": 0
}

```

### Properties

|Name|Type|Required|Restrictions|Description|
|---|---|---|---|---|
|id|integer|true|none|ID of the category|
|id_parent_category|integer|true|none|ID of the parent category. If this is a parent category, it will be equal to its own ID|
|name|string|true|none|Name of the category|
|income|boolean|false|none|Is an income category. If null, this is both an income and an expense category|
|color|string|true|none|Color of the category|
|id_parent_category_in_menu|integer|true|none|ID of the parent category to be displayed|
|name_displayed|string|false|none|Displayed name, with HTML tags|
|refundable|boolean|true|none|This category accepts opposite sign of transactions|
|id_user|integer|false|none|If not null, this category is specific to a user|
|id_logo|integer|false|none|ID of the logo|

<h2 id="tocSclient">Client</h2>

<a id="schemaclient"></a>

```json
{
  "id": 0,
  "name": "",
  "secret": "",
  "public_key": "",
  "private_key": "",
  "redirect_uri": "",
  "primary_color": "",
  "secondary_color": "",
  "pro": false,
  "description": "",
  "description_banks": "",
  "description_providers": "",
  "id_logo": 0
}

```

### Properties

|Name|Type|Required|Restrictions|Description|
|---|---|---|---|---|
|id|integer|true|none|none|
|name|string|true|none|none|
|secret|string|true|none|none|
|public_key|string|false|none|none|
|private_key|string|false|none|none|
|redirect_uri|string|true|none|none|
|primary_color|string|false|none|Primary color of client|
|secondary_color|string|false|none|Secondary color of client|
|pro|boolean|true|none|Should the client display the company manager page.|
|description|string|false|none|Text to display as a default description.|
|description_banks|string|false|none|Text to display as a description for banks.|
|description_providers|string|false|none|Text to display as a description for providers.|
|id_logo|integer|false|none|none|
|information|string|false|none|customizable information|

<h2 id="tocSfile">File</h2>

<a id="schemafile"></a>

```json
{
  "id": 0,
  "content_type": "",
  "filename": "",
  "file_size": 0
}

```

### Properties

|Name|Type|Required|Restrictions|Description|
|---|---|---|---|---|
|id|integer|true|none|none|
|content_type|string|true|none|none|
|filename|string|true|none|none|
|file_size|integer|true|none|none|

<h2 id="tocSsecurity">Security</h2>

<a id="schemasecurity"></a>

```json
{
  "id": 0,
  "code": "",
  "name": "",
  "id_type": 0,
  "last_update": "2019-03-20 11:42:25.559779"
}

```

### Properties

|Name|Type|Required|Restrictions|Description|
|---|---|---|---|---|
|id|integer|true|none|ID of the security|
|code|string|false|none|ISIN code of the security|
|name|string|true|none|Name of the security|
|id_type|integer|false|none|ID of the security type|
|last_update|string(date-time)|false|none|Last update of the security|

<h2 id="tocSaccount">Account</h2>

<a id="schemaaccount"></a>

```json
{
  "id": 0,
  "id_connection": 0,
  "id_user": 0,
  "id_parent": 0,
  "number": "",
  "original_name": "",
  "balance": 0,
  "coming": 0,
  "display": true,
  "last_update": "2019-03-20 11:42:25.565293",
  "deleted": "2019-03-20 11:42:25.565379",
  "disabled": "2019-03-20 11:42:25.565461",
  "iban": "",
  "currency": "",
  "id_type": 0,
  "bookmarked": false,
  "name": "",
  "error": "",
  "usage": ""
}

```

### Properties

|Name|Type|Required|Restrictions|Description|
|---|---|---|---|---|
|id|integer|true|none|ID of the account|
|id_connection|integer|false|none|ID of the related connection|
|id_user|integer|false|none|ID of the related user|
|id_parent|integer|false|none|Id of the parent account|
|number|string|false|none|Account number|
|original_name|string|true|none|Original name of the account on the bank|
|balance|number(float)|true|none|Balance of the account|
|coming|number(float)|false|none|Amount of coming operations not yet debited|
|display|boolean|true|none|Display this account in accounts list|
|last_update|string(date-time)|false|none|Last successful update of the account|
|deleted|string(date-time)|false|none|This account is not found on the website anymore|
|disabled|string(date-time)|false|none|This account has been deleted by user and will not be synchronized anymore|
|iban|string|false|none|Account IBAN|
|currency|object|false|none|Account currency|
|id_type|integer|false|none|ID of the account type|
|bookmarked|integer|true|none|This account has been bookmarked by user|
|name|string|false|none|Name of the account|
|error|string|false|none|If the last update has failed, the error code|
|usage|string|false|none|Account usage|

<h2 id="tocStransaction">Transaction</h2>

<a id="schematransaction"></a>

```json
{
  "id": 0,
  "id_account": 0,
  "webid": "",
  "application_date": "2019-03-20",
  "date": "2019-03-20",
  "value": 0,
  "gross_value": 0,
  "nature": "inconnu",
  "wording": "",
  "id_category": 0,
  "state": "new",
  "date_scraped": "2019-03-20 11:42:25.569614",
  "rdate": "2019-03-20",
  "vdate": "2019-03-20",
  "coming": false,
  "active": true,
  "id_cluster": 0,
  "comment": "",
  "last_update": "2019-03-20 11:42:25.570182",
  "deleted": "2019-03-20 11:42:25.570269",
  "nopurge": false,
  "original_value": 0,
  "original_currency": "",
  "commission": 0,
  "commission_currency": "",
  "country": "",
  "counterparty": "",
  "card": ""
}

```

### Properties

|Name|Type|Required|Restrictions|Description|
|---|---|---|---|---|
|id|integer|true|none|ID of the transaction|
|id_account|integer|true|none|ID of the related account|
|webid|string|false|none|Webid of the transaction|
|application_date|string(date)|false|none|Date considered by PFM services. It is used to change the month of a transaction, for example.|
|date|string(date)|true|none|Debit date|
|value|number(float)|false|none|Value of the transaction|
|gross_value|number(float)|false|none|Gross value of the transaction|
|nature|string|true|none|Type of transaction|
|original_wording|string|true|none|Full label of the transaction|
|simplified_wording|string|true|none|Simplified label of the transaction|
|stemmed_wording|string|true|none|Do not use it|
|wording|string|false|none|Label set by the user|
|id_category|integer|false|none|ID of the related category|
|state|string|true|none|Internal state of the transaction|
|date_scraped|string(date-time)|true|none|Date when the transaction has been seen|
|rdate|string(date)|true|none|Realization of the transaction|
|vdate|string(date)|false|none|Value date of the transaction|
|coming|boolean|true|none|If true, this transaction hasn't been yet debited|
|active|boolean|true|none|If false, PFM services will ignore this transaction|
|id_cluster|integer|false|none|If the transaction is part of a cluster|
|comment|string|false|none|User comment|
|last_update|string(date-time)|false|none|Last update of the transaction|
|deleted|string(date-time)|false|none|If set, this transaction has been removed from the bank|
|nopurge|boolean|true|none|If set to true, this transaction will never be considered as deleted|
|original_value|number(float)|false|none|Value in the original currency|
|original_currency|object|false|none|Original currency|
|commission|number(float)|false|none|Commission taken on the transaction|
|commission_currency|object|false|none|Commission currency|
|country|string|false|none|Original country|
|counterparty|string|false|none|Counterparty|
|card|string|false|none|Card number associated to the transaction|

<h2 id="tocSwebhook">Webhook</h2>

<a id="schemawebhook"></a>

```json
{
  "id": 0,
  "created": "2019-03-20 11:42:25.574358",
  "updated": "2019-03-20 11:42:25.574467",
  "deleted": "2019-03-20 11:42:25.574548",
  "id_service": 0,
  "id_user": 0,
  "id_event": 0,
  "url": ""
}

```

### Properties

|Name|Type|Required|Restrictions|Description|
|---|---|---|---|---|
|id|integer|true|none|ID of the webhook|
|created|string(date-time)|true|none|Date of the webhook creation|
|updated|string(date-time)|true|none|Date of the webhook last update|
|deleted|string(date-time)|false|none|Date of the webhook deletion|
|id_service|integer|false|none|ID of the service|
|id_user|integer|false|none|ID of the emitter user|
|id_event|integer|false|none|ID of the webhook event|
|url|string|false|none|URL of the webhook|

<h2 id="tocSwebhooklog">WebhookLog</h2>

<a id="schemawebhooklog"></a>

```json
{
  "id": 0,
  "id_webhook_data": 0,
  "id_service": 0,
  "id_user": 0,
  "timestamp": "2019-03-20 11:42:25.576657",
  "response_date": "2019-03-20 11:42:25.576761",
  "response_code": 0,
  "next_try": "2019-03-20 11:42:25.576909"
}

```

### Properties

|Name|Type|Required|Restrictions|Description|
|---|---|---|---|---|
|id|integer|true|none|ID of the log|
|id_webhook_data|integer|false|none|ID of the webhook data|
|id_service|integer|false|none|ID of the service|
|id_user|integer|false|none|ID of the user|
|timestamp|string(date-time)|true|none|Timestamp when the hook was sent|
|response_date|string(date-time)|false|none|Timestamp of the reply to the hook|
|response_code|integer|false|none|Return code of the reply to the hook|
|next_try|string(date-time)|false|none|If the log is an error, do not retry to push before this timestamp|

<h2 id="tocSdocumenttype">DocumentType</h2>

<a id="schemadocumenttype"></a>

```json
{
  "id": 0,
  "name": "",
  "attacheable": true
}

```

### Properties

|Name|Type|Required|Restrictions|Description|
|---|---|---|---|---|
|id|integer|true|none|none|
|name|string|true|none|none|
|attacheable|boolean|true|none|none|

<h2 id="tocShashtable">HashTable</h2>

<a id="schemahashtable"></a>

```json
{
  "wording": "",
  "income": false,
  "display": true,
  "nature": ""
}

```

### Properties

|Name|Type|Required|Restrictions|Description|
|---|---|---|---|---|
|wording|string|true|none|none|
|income|boolean|true|none|none|
|display|boolean|true|none|none|
|nature|string|true|none|none|

<h2 id="tocSconnection">Connection</h2>

<a id="schemaconnection"></a>

```json
{
  "id": 0,
  "id_user": 0,
  "id_connector": 0,
  "last_update": "2019-03-20 11:42:25.591002",
  "error": "",
  "expire": "2019-03-20 11:42:25.591259",
  "active": true,
  "last_push": "2019-03-20 11:42:25.591429",
  "next_try": "2019-03-20 11:42:25.591684"
}

```

### Properties

|Name|Type|Required|Restrictions|Description|
|---|---|---|---|---|
|id|integer|true|none|ID of connection|
|id_user|integer|false|none|ID of the related user|
|id_connector|integer|true|none|ID of the related connector|
|last_update|string(date-time)|false|none|Last successful update|
|error|string|false|none|If the last update has failed, the error code|
|expire|string(date-time)|false|none|Expiration of the connection. Used during add of a two-factor authentication, to purge the connection if the user abort|
|active|boolean|true|none|This connection is active and will be automatically synced|
|last_push|string(date-time)|false|none|Last successful push|
|next_try|string(date-time)|false|none|Date of next synchronization|

<h2 id="tocStransfer">Transfer</h2>

<a id="schematransfer"></a>

```json
{
  "id": 0,
  "id_account": 0,
  "id_user": 0,
  "id_recipient": 0,
  "account_iban": "",
  "recipient_iban": "",
  "exec_date": "2019-03-20",
  "register_date": "2019-03-20 11:42:25.597683",
  "amount": 0,
  "fees": 0,
  "webid": "",
  "state": "",
  "error": "",
  "label": "",
  "account_balance": 0,
  "id_transaction": 0,
  "currency": ""
}

```

*This is a representation of a transfer.*

### Properties

|Name|Type|Required|Restrictions|Description|
|---|---|---|---|---|
|id|integer|true|none|ID of transfer|
|id_account|integer|false|none|ID of the debited account|
|id_user|integer|false|none|ID of the related user|
|id_recipient|integer|false|none|ID of the recipient|
|account_iban|string|false|none|IBAN of the debited account|
|recipient_iban|string|false|none|IBAN of the recipient|
|exec_date|string(date)|true|none|Date when the transfer will be operated by the bank|
|register_date|string(date-time)|true|none|Date when the transfer has been registered|
|amount|number(float)|true|none|Amount of the transfer|
|fees|number(float)|false|none|Fees taken by the bank|
|webid|string|false|none|WebID of the transfer|
|state|string|true|none|State of the transfer (created, scheduled, validating, pending, done, canceled, error, bug)|
|error|string|false|none|Error message during transfer, if any|
|label|string|false|none|Label of the transfer|
|account_balance|number(float)|false|none|Balance of the account just before the transfer|
|id_transaction|integer|false|none|If found, ID of the related transaction|
|currency|object|false|none|Currency of the object|

<h2 id="tocScertificate">Certificate</h2>

<a id="schemacertificate"></a>

```json
{
  "id": 0,
  "id_public_key_file": 0,
  "id_private_key_file": 0,
  "type": "",
  "created": "2019-03-20 11:42:25.604192"
}

```

### Properties

|Name|Type|Required|Restrictions|Description|
|---|---|---|---|---|
|id|integer|true|none|none|
|id_public_key_file|integer|true|none|none|
|id_private_key_file|integer|true|none|none|
|type|string|true|none|none|
|created|string(date-time)|true|none|none|

<h2 id="tocSconnectionlog">ConnectionLog</h2>

<a id="schemaconnectionlog"></a>

```json
{
  "id": 0,
  "id_user": 0,
  "id_connection": 0,
  "id_connector": 0,
  "login": "",
  "error_uid": "",
  "timestamp": "CURRENT_TIMESTAMP",
  "next_try": "2019-03-20 11:42:25.606763",
  "error": "",
  "error_message": "",
  "statut": 0,
  "nb_accounts": 0,
  "start": "2019-03-20 11:42:25.607048",
  "worker": "",
  "session_folder_id": ""
}

```

### Properties

|Name|Type|Required|Restrictions|Description|
|---|---|---|---|---|
|id|integer|true|none|ID of the log|
|id_user|integer|false|none|ID of the user|
|id_connection|integer|true|none|ID of the connection|
|id_connector|integer|false|none|ID of the connector|
|login|string|false|none|bcrypt hash of the login|
|error_uid|string|false|none|MD5 hash of the exception backtrace|
|timestamp|string(date-time)|true|none|Timestamp of log, when the synchronization has finished|
|next_try|string(date-time)|false|none|If fail, the date represents the next try to connect|
|error|string|false|none|If fail, contains the error code|
|error_message|string|false|none|If fail, error message received from connector|
|statut|integer|false|none|Status of user (1 = charged user)|
|nb_accounts|integer|false|none|In case of bank connection, number of accounts|
|start|string(date-time)|false|none|Timestamp when the synchronization has started|
|worker|string|false|none|Worker used to do synchronization|
|session_folder_id|string|false|none|Session folder uid|

<h2 id="tocSprojecttype">ProjectType</h2>

<a id="schemaprojecttype"></a>

```json
{
  "id": 0,
  "name": "",
  "icon_url": ""
}

```

### Properties

|Name|Type|Required|Restrictions|Description|
|---|---|---|---|---|
|id|integer|true|none|none|
|name|string|true|none|none|
|icon_url|string|false|none|none|

<h2 id="tocSaccess">Access</h2>

<a id="schemaaccess"></a>

```json
{
  "id": 0,
  "id_user": 0,
  "id_profile": 0,
  "id_role": 0,
  "email": ""
}

```

### Properties

|Name|Type|Required|Restrictions|Description|
|---|---|---|---|---|
|id|integer|true|none|none|
|id_user|integer|false|none|none|
|id_profile|integer|true|none|none|
|id_role|integer|false|none|none|
|email|string|false|none|none|

<h2 id="tocSprofile">Profile</h2>

<a id="schemaprofile"></a>

```json
{
  "id": 0,
  "id_user": 0,
  "role": "admin",
  "statut": 0,
  "admin": false,
  "conf": "",
  "lang": ""
}

```

### Properties

|Name|Type|Required|Restrictions|Description|
|---|---|---|---|---|
|id|integer|true|none|none|
|id_user|integer|true|none|none|
|role|string|true|none|none|
|email|string|true|none|none|
|statut|integer|true|none|none|
|admin|boolean|true|none|none|
|conf|string|false|none|none|
|lang|string|false|none|none|

#### Enumerated Values

|Property|Value|
|---|---|
|role|admin|
|role|ser|

<h2 id="tocStransactionscluster">TransactionsCluster</h2>

<a id="schematransactionscluster"></a>

```json
{
  "id": 0,
  "id_account": 0,
  "mean_amount": 0,
  "median_increment": 0,
  "enabled": true,
  "next_date": "2019-03-20",
  "wording": "",
  "id_category": 0,
  "created_by": ""
}

```

### Properties

|Name|Type|Required|Restrictions|Description|
|---|---|---|---|---|
|id|integer|true|none|none|
|id_account|integer|true|none|none|
|mean_amount|number(float)|true|none|none|
|median_increment|integer|false|none|none|
|enabled|boolean|true|none|none|
|next_date|string(date)|false|none|none|
|wording|string|true|none|none|
|id_category|integer|false|none|none|
|created_by|string|false|none|none|

<h2 id="tocSuseralert">UserAlert</h2>

<a id="schemauseralert"></a>

```json
{
  "id": 0,
  "income_max": 500,
  "expense_max": 500,
  "balance_min1": 500,
  "balance_min2": 0,
  "balance_max": 10000,
  "resume_enabled": true,
  "enabled": true,
  "value_type": "flat",
  "type": "transactions",
  "transaction_types": "",
  "date_range": 0,
  "apply": "",
  "resume_frequency": 7
}

```

*/!\ Careful we use default value from database if present

    For more information see AlertsPlugin.init*

### Properties

|Name|Type|Required|Restrictions|Description|
|---|---|---|---|---|
|id|integer|true|none|none|
|income_max|number(float)|false|none|none|
|expense_max|number(float)|false|none|none|
|balance_min1|number(float)|false|none|none|
|balance_min2|number(float)|false|none|none|
|balance_max|number(float)|false|none|none|
|resume_enabled|boolean|false|none|none|
|enabled|boolean|false|none|none|
|value_type|string|true|none|none|
|type|string|true|none|none|
|transaction_types|string|false|none|none|
|date_range|integer|false|none|none|
|apply|string|false|none|none|
|resume_frequency|integer|true|none|none|

<h2 id="tocSaccounttype">AccountType</h2>

<a id="schemaaccounttype"></a>

```json
{
  "id": 0,
  "name": "",
  "is_invest": false,
  "weboob_type_id": 0,
  "display_name_p": "",
  "display_name": "",
  "color": "",
  "id_parent": 0
}

```

### Properties

|Name|Type|Required|Restrictions|Description|
|---|---|---|---|---|
|id|integer|true|none|ID of the account type|
|name|string|true|none|Name of the account type|
|is_invest|boolean|true|none|Is it an investment account|
|weboob_type_id|integer|true|none|Map to the weboob_type_id|
|display_name_p|string|true|none|Name to display in plurial|
|display_name|string|true|none|Name to display in singular|
|color|string|false|none|Color of the account type (hexdecimal)|
|id_parent|integer|false|none|Id of the parent type|

<h2 id="tocSsubscription">Subscription</h2>

<a id="schemasubscription"></a>

```json
{
  "id": 0,
  "id_connection": 0,
  "id_user": 0,
  "number": "",
  "label": "",
  "subscriber": "",
  "validity": "2019-03-20",
  "renewdate": "2019-03-20",
  "last_update": "2019-03-20 11:42:25.663641",
  "deleted": "2019-03-20 11:42:25.663720",
  "disabled": "2019-03-20 11:42:25.663804",
  "error": ""
}

```

### Properties

|Name|Type|Required|Restrictions|Description|
|---|---|---|---|---|
|id|integer|true|none|ID of subscription|
|id_connection|integer|false|none|ID of related connection|
|id_user|integer|false|none|ID of related user|
|number|string|true|none|Subscription's number|
|label|string|true|none|Label of the subscription|
|subscriber|string|false|none|Name of the subscriber|
|validity|string(date)|false|none|The subscription is valid until this date, if any|
|renewdate|string(date)|false|none|Next renew date, if any|
|last_update|string(date-time)|false|none|Last successful update of the subscription|
|deleted|string(date-time)|false|none|This subscription is not found on the website anymore|
|disabled|string(date-time)|false|none|This subscription has been deleted by user and will not be synchronized anymore|
|error|string|false|none|If the last update has failed, the error code|

<h2 id="tocSinvestment">Investment</h2>

<a id="schemainvestment"></a>

```json
{
  "id": 0,
  "id_account": 0,
  "label": "",
  "code": "",
  "code_type": "",
  "source": "",
  "description": "",
  "quantity": 0,
  "unitprice": 0,
  "unitvalue": 0,
  "valuation": 0,
  "diff": 0,
  "diff_percent": 0,
  "prev_diff": 0,
  "portfolio_share": 0,
  "vdate": "2019-03-20",
  "prev_vdate": "2019-03-20",
  "id_security": 0,
  "original_currency": "",
  "original_valuation": 0,
  "original_unitvalue": 0,
  "original_unitprice": 0,
  "original_diff": 0,
  "last_update": "2019-03-20 11:42:25.672607",
  "deleted": "2019-03-20 11:42:25.672676"
}

```

### Properties

|Name|Type|Required|Restrictions|Description|
|---|---|---|---|---|
|id|integer|true|none|ID of the investment|
|id_account|integer|true|none|ID of the related account|
|label|string|true|none|Label of the investment|
|code|string|true|none|Investment code|
|code_type|string|false|none|Code type (ISIN of AMF)|
|source|string|false|none|Source of the ISIN code (website, notFound)|
|description|string|false|none|Description of the investment|
|quantity|number(float)|false|none|Quantity|
|unitprice|number(float)|false|none|Average buy price|
|unitvalue|number(float)|false|none|Current unit value|
|valuation|number(float)|false|none|Current valuation|
|diff|number(float)|false|none|Capital gain|
|diff_percent|number(float)|false|none|Capital gain in percent (between 0 and 1)|
|prev_diff|number(float)|false|none|Capital gain from previous value|
|portfolio_share|number(float)|false|none|Percent of the portfolio|
|vdate|string(date)|false|none|Value date|
|prev_vdate|string(date)|false|none|Value date of the previous value (prev_diff)|
|id_security|integer|false|none|ID of the related security|
|original_currency|object|false|none|Original currency|
|original_valuation|number(float)|false|none|Valuation in original currency|
|original_unitvalue|number(float)|false|none|Average buy price in the original currency|
|original_unitprice|number(float)|false|none|Current unit value in the original currency|
|original_diff|number(float)|false|none|Capital gain in the original currency|
|last_update|string(date-time)|false|none|Last update of the investment|
|deleted|string(date-time)|false|none|If set, this investment has been removed from the website|

<h2 id="tocSrecipient">Recipient</h2>

<a id="schemarecipient"></a>

```json
{
  "id": 0,
  "id_account": 0,
  "id_target_account": 0,
  "label": "",
  "bank_name": "",
  "iban": "",
  "webid": "",
  "category": "",
  "last_update": "2019-03-20 11:42:25.676098",
  "time_scraped": "2019-03-20 11:42:25.676188",
  "deleted": "2019-03-20 11:42:25.676267",
  "expire": "2019-03-20 11:42:25.676349",
  "enabled_at": "2019-03-20 11:42:25.676426",
  "add_verified": false,
  "state": "",
  "error": "",
  "currency": ""
}

```

### Properties

|Name|Type|Required|Restrictions|Description|
|---|---|---|---|---|
|id|integer|true|none|ID of the recipient|
|id_account|integer|true|none|ID of the related account|
|id_target_account|integer|false|none|ID of the target account, in case of internal recipient|
|label|string|true|none|Label of the recipient|
|bank_name|string|false|none|Bank of the recipient|
|iban|string|false|none|IBAN of the recipient|
|webid|string|false|none|Webid of the recipient|
|category|string|true|none|Category in which the recipient is|
|last_update|string(date-time)|true|none|Last time we have fetched this recipient|
|time_scraped|string(date-time)|false|none|First time we've seen this recipient|
|deleted|string(date-time)|false|none|The recipient isn't found anymore on the bank|
|expire|string(date-time)|false|none|none|
|enabled_at|string(date-time)|false|none|It will be possible to do transfers to this recipient at this date|
|add_verified|boolean|false|none|Was the recipient adding authorized|
|state|string|false|none|State of recipient|
|error|string|false|none|Error message|
|fields|string|false|none|Fields for recipient with additionalInformationNeeded state|
|currency|object|false|none|Currency of the object|

<h2 id="tocSdocument">Document</h2>

<a id="schemadocument"></a>

```json
{
  "id": 0,
  "id_type": 0,
  "id_category": 0,
  "id_user": 0,
  "id_subscription": 0,
  "id_file": 0,
  "id_thumbnail": 0,
  "url": "",
  "thumb_url": "",
  "name": "",
  "timestamp": "CURRENT_TIMESTAMP",
  "date": "2019-03-20 11:42:25.678944",
  "duedate": "2019-03-20",
  "total_amount": 0,
  "untaxed_amount": 0,
  "vat": 0,
  "income": true,
  "readonly": true,
  "number": "",
  "issuer": "",
  "last_update": "2019-03-20 11:42:25.679749",
  "currency": ""
}

```

### Properties

|Name|Type|Required|Restrictions|Description|
|---|---|---|---|---|
|id|integer|true|none|none|
|id_type|integer|false|none|none|
|id_category|integer|false|none|none|
|id_user|integer|true|none|none|
|id_subscription|integer|false|none|none|
|id_file|integer|false|none|none|
|id_thumbnail|integer|false|none|none|
|url|string|false|none|none|
|thumb_url|string|false|none|none|
|name|string|false|none|none|
|timestamp|string(date-time)|true|none|none|
|date|string(date-time)|true|none|none|
|duedate|string(date)|false|none|none|
|total_amount|number(float)|false|none|none|
|untaxed_amount|number(float)|false|none|none|
|vat|number(float)|false|none|none|
|income|boolean|false|none|none|
|readonly|boolean|true|none|none|
|number|string|false|none|none|
|issuer|string|false|none|none|
|last_update|string(date-time)|false|none|Last successful update of the document|
|currency|object|false|none|Document currency|

<h2 id="tocSproject">Project</h2>

<a id="schemaproject"></a>

```json
{
  "id": 0,
  "id_user": 0,
  "id_type": 0,
  "name": "",
  "target": 0,
  "saved": 0,
  "monthly_savings": 0,
  "comment": "",
  "active": true
}

```

### Properties

|Name|Type|Required|Restrictions|Description|
|---|---|---|---|---|
|id|integer|true|none|none|
|id_user|integer|true|none|none|
|id_type|integer|true|none|none|
|name|string|true|none|none|
|target|number(float)|true|none|none|
|saved|number(float)|true|none|none|
|monthly_savings|number(float)|true|none|none|
|comment|string|true|none|none|
|active|boolean|true|none|none|

<h2 id="tocSdevice">Device</h2>

<a id="schemadevice"></a>

```json
{
  "id": 0,
  "id_token": 0,
  "type": "",
  "notification_token": "",
  "last_update": "CURRENT_TIMESTAMP",
  "version": "",
  "debug": false
}

```

### Properties

|Name|Type|Required|Restrictions|Description|
|---|---|---|---|---|
|id|integer|true|none|none|
|id_token|integer|true|none|none|
|type|string|true|none|none|
|notification_token|string|true|none|none|
|last_update|string(date-time)|true|none|none|
|version|string|true|none|none|
|debug|boolean|true|none|none|

<h2 id="tocSpocket">Pocket</h2>

<a id="schemapocket"></a>

```json
{
  "id": 0,
  "id_account": 0,
  "id_investment": 0,
  "label": "",
  "value": 0,
  "quantity": 0,
  "availability_date": "2019-03-20",
  "condition": "inconnu",
  "last_update": "2019-03-20 11:42:25.691468",
  "deleted": "2019-03-20 11:42:25.691553"
}

```

### Properties

|Name|Type|Required|Restrictions|Description|
|---|---|---|---|---|
|id|integer|true|none|ID of the pocket|
|id_account|integer|true|none|ID of the related account|
|id_investment|integer|true|none|ID of the related investment|
|label|string|false|none|Label of the pocket|
|value|number(float)|true|none|Value of the pocket|
|quantity|number(float)|false|none|Quantity of stocks|
|availability_date|string(date)|false|none|Availability date of the pocket|
|condition|string|true|none|Withdrawal condition of the pocket|
|last_update|string(date-time)|false|none|Last update of the pocket|
|deleted|string(date-time)|false|none|If set, this pocket has been removed from the website|

<h2 id="tocSinvite">Invite</h2>

<a id="schemainvite"></a>

```json
{
  "id": 0,
  "id_unsubscribe": 0,
  "id_user_inviting": 0,
  "id_user_invited": 0,
  "email_invited": "",
  "timestamp": "2019-03-20 11:42:25.692796"
}

```

### Properties

|Name|Type|Required|Restrictions|Description|
|---|---|---|---|---|
|id|integer|true|none|none|
|id_unsubscribe|integer|false|none|none|
|id_user_inviting|integer|true|none|none|
|id_user_invited|integer|false|none|none|
|email_invited|string|false|none|none|
|timestamp|string(date-time)|false|none|none|

<h2 id="tocSalert">Alert</h2>

<a id="schemaalert"></a>

```json
{
  "id": 0,
  "id_user": 0,
  "timestamp": "CURRENT_TIMESTAMP",
  "type": "",
  "id_transaction": 0,
  "id_account": 0,
  "value": 0,
  "id_investment": 0
}

```

### Properties

|Name|Type|Required|Restrictions|Description|
|---|---|---|---|---|
|id|integer|true|none|none|
|id_user|integer|true|none|ID of the related user|
|timestamp|string(date-time)|true|none|Date of the alerts emission|
|type|string|true|none|Type of the alert|
|id_transaction|integer|false|none|ID of the related transaction|
|id_account|integer|false|none|ID of the related account|
|value|number(float)|true|none|Amount related to the alert|
|id_investment|integer|false|none|ID of the related investment|

<h2 id="tocSgroup">Group</h2>

<a id="schemagroup"></a>

```json
{
  "id": 0,
  "id_parent_group": 0,
  "id_logo": 0,
  "name": "",
  "url": "",
  "color": "",
  "email": "",
  "conf": ""
}

```

### Properties

|Name|Type|Required|Restrictions|Description|
|---|---|---|---|---|
|id|integer|true|none|none|
|id_parent_group|integer|false|none|none|
|id_logo|integer|false|none|none|
|name|string|false|none|none|
|url|string|false|none|none|
|color|string|false|none|none|
|email|string|false|none|none|
|conf|string|false|none|none|

<h2 id="tocStransferlog">TransferLog</h2>

<a id="schematransferlog"></a>

```json
{
  "id": 0,
  "id_transfer": 0,
  "id_file": 0,
  "request_data": "",
  "state": "",
  "error": "",
  "timestamp": "CURRENT_TIMESTAMP"
}

```

### Properties

|Name|Type|Required|Restrictions|Description|
|---|---|---|---|---|
|id|integer|true|none|ID of the transfer log entry|
|id_transfer|integer|false|none|ID of the related transfer|
|id_file|integer|false|none|ID of the related file|
|request_data|string|false|none|Data stored related to user who has requested the transfer|
|state|string|false|none|State of the transfer (created, scheduled, validating, pending, done, canceled, error, bug)|
|error|string|false|none|Error message during transfer, if any|
|timestamp|string(date-time)|true|none|Timestamp of the log|

<h2 id="tocStransactioninformation">TransactionInformation</h2>

<a id="schematransactioninformation"></a>

```json
{
  "id": 0,
  "id_transaction": 0,
  "key": "",
  "value": ""
}

```

### Properties

|Name|Type|Required|Restrictions|Description|
|---|---|---|---|---|
|id|integer|true|none|ID of this transaction information|
|id_transaction|integer|true|none|ID of the related transaction|
|key|string|true|none|Key of the transaction information|
|value|string|false|none|Value of the transaction information|

<h2 id="tocSinvestmentvalue">InvestmentValue</h2>

<a id="schemainvestmentvalue"></a>

```json
{
  "id": 0,
  "id_investment": 0,
  "vdate": "2019-03-20",
  "unitvalue": 0,
  "original_currency": "",
  "original_unitvalue": 0
}

```

### Properties

|Name|Type|Required|Restrictions|Description|
|---|---|---|---|---|
|id|integer|true|none|ID of the value|
|id_investment|integer|true|none|ID of the related investment|
|vdate|string(date)|true|none|Date of this value|
|unitvalue|number(float)|true|none|Value on this date|
|original_currency|object|false|none|Original currency|
|original_unitvalue|number(float)|false|none|Value on this date, in the original currency|

<h2 id="tocSrecipientlog">RecipientLog</h2>

<a id="schemarecipientlog"></a>

```json
{
  "id": 0,
  "id_recipient": 0,
  "id_file": 0,
  "request_data": "",
  "step": "",
  "error": "",
  "timestamp": "CURRENT_TIMESTAMP"
}

```

### Properties

|Name|Type|Required|Restrictions|Description|
|---|---|---|---|---|
|id|integer|true|none|ID of the transfer log entry|
|id_recipient|integer|false|none|ID of the related recipient|
|id_file|integer|false|none|ID of the related file|
|request_data|string|false|none|Data stored related to user who has requested the recipient addition|
|step|string|false|none|Step of recipient addition, (add_recipient, asking_field, recipient addition validated, creation, storing_files)|
|error|string|false|none|Error message during recipient addition, if any|
|timestamp|string(date-time)|true|none|Timestamp of the log|

<h2 id="tocSaccountlog">AccountLog</h2>

<a id="schemaaccountlog"></a>

```json
{
  "id_account": 0,
  "id_connector": 0,
  "balance": 0,
  "coming": 0,
  "timestamp": "CURRENT_TIMESTAMP",
  "error": "",
  "error_message": "",
  "id_connection_log": 0
}

```

### Properties

|Name|Type|Required|Restrictions|Description|
|---|---|---|---|---|
|id_account|integer|true|none|ID of the related account|
|id_connector|integer|false|none|provider id|
|balance|number(float)|true|none|Balanced recorded|
|coming|number(float)|false|none|Coming debit recorded|
|timestamp|string(date-time)|true|none|Timestamp of log|
|error|string|false|none|If fail, contains the error code|
|error_message|string|false|none|If fail, error message received from bank or provider|
|id_connection_log|integer|false|none|ID of the related connection log|

<h2 id="tocScurrency">Currency</h2>

<a id="schemacurrency"></a>

```json
{
  "id": "",
  "symbol": "",
  "prefix": false,
  "crypto": false,
  "precision": 2,
  "marketcap": 0,
  "datetime": "2019-03-20 11:42:25.990818"
}

```

### Properties

|Name|Type|Required|Restrictions|Description|
|---|---|---|---|---|
|id|string|true|none|ISO 4217 code used as ID|
|symbol|string|true|none|Symbol representing the currency|
|prefix|boolean|true|none|Amount is prefixed or not by the currency|
|crypto|boolean|false|none|It is a crypto currency or not|
|precision|integer|false|none|Numbers of significant digits|
|marketcap|number(float)|false|none|Market Capitalization in EUR|
|datetime|string(date-time)|false|none|Time and date of Market Cap (for cryptos)|

<h2 id="tocSconnectorcategory">ConnectorCategory</h2>

<a id="schemaconnectorcategory"></a>

```json
{
  "id": 0,
  "name": false
}

```

### Properties

|Name|Type|Required|Restrictions|Description|
|---|---|---|---|---|
|id|integer|true|none|ID of the bank category|
|name|string|true|none|Name of the category|

<h2 id="tocSlockeduser">LockedUser</h2>

<a id="schemalockeduser"></a>

```json
{
  "id_user": 0,
  "timestamp": "CURRENT_TIMESTAMP",
  "worker": ""
}

```

### Properties

|Name|Type|Required|Restrictions|Description|
|---|---|---|---|---|
|id_user|integer|true|none|none|
|timestamp|string(date-time)|true|none|none|
|worker|string|false|none|none|

<h2 id="tocSoidcwhitelist">OidcWhitelist</h2>

<a id="schemaoidcwhitelist"></a>

```json
{
  "id": 0,
  "redirect_uri": ""
}

```

### Properties

|Name|Type|Required|Restrictions|Description|
|---|---|---|---|---|
|id|integer|true|none|none|
|redirect_uri|string|true|none|authorized redirect uri|

