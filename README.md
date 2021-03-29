# Taking advantage of Web Apps with Google Apps Script

<a name="TOP"></a>
[![MIT License](http://img.shields.io/badge/license-MIT-blue.svg?style=flat)](LICENCE)

# Table of contents

- [Overview](#overview)
- [Description](#description)
- [Deploy Web Apps](#deploywebApps)
- [Redeploying Web Apps without Changing URL of Web Apps for new IDE](#redeploy) <sup><font color="Red">Added at March 29, 2021</font></sup>
- [5 situations for Web Apps](#situationsforwebapps)
  - [How to access to Web Apps](#howtoaccesstowebapps)
    - [1. Owner accesses to Web Apps using browser](#howtoaccesstowebapps1)
    - [2. Client users access to Web Apps using browser](#howtoaccesstowebapps2)
    - [3. Owner accesses to Web Apps using Curl, Google Apps Script and so on which don't use browser](#howtoaccesstowebapps3)
    - [4. Client users access to Web Apps using Curl, Google Apps Script and so on which don't use browser](#howtoaccesstowebapps4)
  - [Required parameters for accessing to deployed Web Apps](#requiredparameters)
  - [Authorization for scopes](#authorizationforscopes)
  - [Access token for accessing to Web Apps](#accesstokenforaccessingtowebapps)
  - [Share project of Web Apps with client users](#shareproject)
- [How to use dev mode from outside](#howtousedevmode)
- [Event object of Web Apps](#eventobject) <sup><font color="Red">Added at May 21, 2020</font></sup>
- [Logs in Web Apps for Google Apps Script](#checklog) <sup><font color="Red">Added at July 26, 2020</font></sup>
- [Limitation of simultaneous connection to Web Apps](#limitationofsimultaneousconnection)
- [Error messages](#errormessages)
- [CORS in Web Apps](#corsinwebapps) <sup><font color="Red">Added at July 1, 2020</font></sup>
- [Confidentiality of scripts for Web Apps](#confidentialityofscripts)
- [Sample script of server side](#samplescriptofserverside)
- [Sample scripts of client side](#samplescriptofclientside)
- [Sample scripts of client side by various languages](#samplescriptvariouslanguages) <sup><font color="Red">Added at July 15, 2020</font></sup>
- [Status code from Web Apps](#statuscodefromwebapps)
- [Applications](#applications)
- [Sample situations](#samplesituations)
- [References](#references)

<a name="overview"></a>

# Overview

This is a report to take advantage of Web Apps with Google Apps Script (GAS).

<a name="description"></a>

# Description

There is [Web Apps](https://developers.google.com/apps-script/guides/web) as one of applications using Google Apps Script (GAS). I sometimes use this Web Apps. But I have only a little the information for the specification of Web Apps. So in order to take more advantage of Web Apps, I investigated and summarized about this. The aim of this report is to become one of the basic information for creating various applications using Web Apps with GAS.

<a name="deploywebapps"></a>

# Deploy Web Apps

When Web Apps is deployed, you can see the following window.

New IDE for Google Apps Script has finally been released at December 7, 2020. [Ref](https://developers.google.com/apps-script/releases/#december_7_2020) For new IDE, please open a dialog by "Deploy" -> "New deployment". And, please select "Web app" at "Select type. By this, you can see the following dialog.

![](images/newide1.png)

On the other hand, when you use old IDE, you can see the following dialog.

![](images/fig1.png)

For setting, generally, the following flow is used.

- On script editor
- Publish -> Deploy as web app...

  - Project version:
    - "New" and input the description.
  - Pattern 1: <u>Execute the app as:</u> (old IDE), <u>Execute as:</u> (new IDE)
    - **Me** (old IDE and new IDE)
      - <u>Who has access to the app:</u>
        - **Only myself** (old IDE and new IDE)
        - **Anyone** (old IDE), **Anyone with Google account** (new IDE)
        - **Anyone, even anonymous** (old IDE), **Anyone** (new IDE)
  - Pattern 2: <u>Execute the app as:</u> (old IDE), <u>Execute as:</u> (new IDE)
    - **User accessing the web app** (old IDE and new IDE)
      - <u>Who has access to the app:</u> (old IDE), <u>Who has access</u> (new IDE)
        - **Only myself** (old IDE and new IDE)
        - **Anyone** (old IDE), **Anyone with Google account** (new IDE)
  - Click Deploy.

- There are **Me** and **User accessing the web app** for "Execute the app as:" and **Only myself**, **Anyone** and **Anyone, even anonymous** for "Who has access to the app:" as the options. When **Me** and **User accessing the web app** are selected for "Execute the app as:", each "Who has access to the app:" has 3 and 2 options, respectively. From above flow, It is found that there are 5 situations for deploying Web Apps. For each situation, there are 2 methods of GET and POST.

- At new IDE,

  - **Execute the app as:** became **Execute as**. And the values are **Me** and **User accessing the web app**. These are the same with the old IDE.
  - **Who has access to the app:** became **Who has access**. About the values, please check the following list.

    - When **Execute as** is **Me**, the values of **Who has access** are **Only myself**, **Anyone with Google account** and **Anyone**. In this case, the values of **Only myself**, **Anyone with Google account** and **Anyone** of new IDE are the same with **Only myself**, **Anyone** and **Anyone, even anonymous**, respectively.
    - When **Execute as** is **User accessing the web app**, the values of **Who has access** are **Only myself** and **Anyone with Google account**. These are the same with **Only myself** and **Anyone** of the old IDE, respectively.

In this report, I would like to introduce about the specification for Web Apps deployed by 5 situations.

<a name="redeploy"></a>

# Redeploying Web Apps without Changing URL of Web Apps for new IDE

At March 15, 2021, one endpoint is created for one deployment. [Ref](https://developers.google.com/apps-script/releases/#march_15_2021) By this, when you redeploy "Web Apps", the endpoint is changed. Because the deployment ID is changed. It seems that this it the new specification. In this report, I would like to introduce the method for redeploying Web Apps without changing the URL of Web Apps for new IDE.

The detail document can be seen at [https://gist.github.com/tanaikech/ebf92d8f427d02d53989d6c3464a9c43](https://gist.github.com/tanaikech/ebf92d8f427d02d53989d6c3464a9c43).

<a name="situationsforwebapps"></a>

# 5 situations for Web Apps

1. **Situation 1**
   - "Execute the app as:" : **Me**
   - "Who has access to the app:": **Only myself**
1. **Situation 2**
   - "Execute the app as:" : **Me**
   - "Who has access to the app:": **Anyone**
1. **Situation 3**
   - "Execute the app as:" : **Me**
   - "Who has access to the app:": **Anyone, even anonymous**
1. **Situation 4**
   - "Execute the app as:" : **User accessing the web app**
   - "Who has access to the app:": **Only myself**
1. **Situation 5**
   - "Execute the app as:" : **User accessing the web app**
   - "Who has access to the app:": **Anyone**

<a name="howtoaccesstowebapps"></a>

## How to access to Web Apps

There are 5 situations for the deployed Web Apps, as mentioned above. And the methods for accessing to the deployed Web Apps are divided into 2 types. Those are the method which accesses using browser and the method which accesses using Curl, Google Apps Script and so on which don't use browser. Each method is used by the owner of Web Apps and the client users. The flow which summarized them is as follows.

<a name="howtoaccesstowebapps1"></a>

### 1. Owner accesses to Web Apps using browser

![](images/Browser_owner.png)

- For Situation 1, 4
  - Owner can access and run the script of Web Apps by the login to Google.
- For Situation 2, 5
  - Owner can access and run the script of Web Apps by the login to Google.
- For Situation 3
  - Owner can access and run the script of Web Apps without the login to Google.

<a name="howtoaccesstowebapps2"></a>

### 2. Client users access to Web Apps using browser

![](images/Browser_users.png)

- For Situation 1, 4
  - Client users cannot access and run the script of Web Apps.
- For Situation 2, 5
  - Client users can access and run the script of Web Apps by the login to Google.
- For Situation 3
  - Client users can access and run the script of Web Apps without the login to Google.

<a name="howtoaccesstowebapps3"></a>

### 3. Owner accesses to Web Apps using Curl, Google Apps Script and so on which don't use browser

![](images/Curl_owner.png)

- For Situation 1, 4
  - Owner can access and run the script of Web Apps by using the access token.
- For Situation 2, 5
  - Owner can access and run the script of Web Apps by using the access token.
- For Situation 3
  - If the script of Web Apps uses some scopes, owner has to authorize the scopes by own browser.
  - Owner can access and run the script of Web Apps without the login to Google.

<a name="howtoaccesstowebapps4"></a>

### 4. Client users access to Web Apps using Curl, Google Apps Script and so on which don't use browser

![](images/Curl_users.png)

- For Situation 1, 4
  - Client users cannot access and run the script of Web Apps.
- For Situation 2, 5
  - **At first, the project that Web Apps was deployed has to be shared with client users who use Web Apps.**
    - **I confirmed that from April 11, 2018, it is required to be shared the project to access to Web Apps. This might be due to the update of Google.**
  - If the script of Web Apps uses some scopes, client users have to authorize the scopes by own browser.
  - Client users can access and run the script of Web Apps without the login to Google.
- For Situation 3
  - If the script of Web Apps uses some scopes, client users have to authorize the scopes by own browser.
  - Client users can access and run the script of Web Apps without the login to Google.

<a name="requiredparameters"></a>

## Required parameters for accessing to deployed Web Apps

The simple explanations for above 5 situations were summarized as the following table. Owner and client users mean the owner who deployed Web Apps and the users who use the deployed Web Apps, respectively.

| Situations  | Script of Web Apps | Authorization for scopes | Owner                        | Client users                 | Share project |
| :---------- | :----------------- | :----------------------- | :--------------------------- | :--------------------------- | :------------ |
| Situation 1 | Run as owner.      | Only owner               | Access with access token.    | Cannot access.               | No            |
| Situation 2 | Run as owner.      | Only owner               | Access with access token.    | Access with access token.    | Yes           |
| Situation 3 | Run as owner.      | Only owner               | Access without access token. | Access without access token. | No            |
| Situation 4 | Run as each user.  | Each user                | Access with access token.    | Cannot access.               | No            |
| Situation 5 | Run as each user.  | Each user                | Access with access token.    | Access with access token.    | Yes           |

For example, at situation 5, the script of Web Apps is run as each user (owner and client users). The authorization for the scopes of scripts of Web Apps is required to run the scripts. This authorization has to be done by each user (owner and client users) using own browser. The access token is required for each user (owner and client users) to access to Web Apps. In order to access to Web Apps from client users, the project deployed Web Apps has to be shared with the users.

<a name="authorizationforscopes"></a>

## Authorization for scopes

- When Web Apps is deployed as **`"Execute the app as:" : Me`** by owner, the authorization screen is automatically opened. When the owner authorizes it, owner and client users can run the scripts of Web Apps as the owner.

  - <u>**For example**</u>, in this situation, when `Session.getEffectiveUser().getEmail()` is run in this situation by the client users, the retrieved email is the owner's email. From this, it is found that the script is run as the owner.

  - <u>**For example**</u>, in this situation, when `DriveApp.createFile(blob)` is run at the script of Web Apps, the file is created in the owner's Google Drive.

- When Web Apps is deployed as **`"Execute the app as:" : User accessing the web app`** by owner, the authorization screen is **NOT** opened automatically. So before owner and client users access to Web Apps, they have to manually authorize to use the scopes by own browser.

  - If some scopes are used for the scripts of Web Apps, owner and client users have to authorize them only one time using own browser, before it accesses to Web Apps.

    - The URL for authorizing is `https://script.google.com/macros/s/#####/exec` which is the same to URL of Web Apps. When owner and client users access to the URL using own browser, the following authorization screen is displayed.

      - ![](images/fig2.png)

    - When you see above screen, please click "REVIEW PERMISSIONS". And select account and authorize to use the scopes. After owner and client users authorized the scopes, they can run the script of Web Apps.

  - <u>**For example**</u>, in this situation, when `Session.getEffectiveUser().getEmail()` is run in this situation by the client users, the retrieved email is the each user's email. From this, it is found that the script is run as each user.

  - <u>**For example**</u>, in this situation, when `DriveApp.createFile(blob)` is run at the script of Web Apps, the file is created in the each user's Google Drive.

<a name="accesstokenforaccessingtowebapps"></a>

## Access token for accessing to Web Apps

- When Web Apps is deployed as **`"Who has access to the app:": Only myself`** or **`"Who has access to the app:": Anyone`** by owner, the owner and client users have to access and run the script of Web Apps with own access token.

  - At least, one of scopes for using Drive API is required to be included in the access token. For example, those are

    - `https://www.googleapis.com/auth/drive.readonly`
    - `https://www.googleapis.com/auth/drive.files`
    - `https://www.googleapis.com/auth/drive`

  - Even if the server script uses the scopes except for scripts to use Drive API, the scopes except for scopes for Drive API are not required to be included. Because the use of other scopes is authorized by the browser, before it accesses to Web Apps. This means that when it accesses to Web Apps by GET or POST request, only the scopes for using Drive API are required.

  - For example, when you want to access to Web Apps using GAS, even if you use `{"Authorization": "Bearer " + ScriptApp.getOAuthToken()}` to the headers, when the error of `<TITLE>Unauthorized</TITLE>` occurs, please confirm the scopes of the scripts at the script editor. If `https://www.googleapis.com/auth/drive` is not included in the scopes, please add it. In order to add it, for example, you can put `// DriveApp.getFiles()` in the scripts **as a comment**. By this, the script editor will automatically include `https://www.googleapis.com/auth/drive`.

- If no scopes are used for the scripts of Web Apps, owner and client users can run the script of Web Apps without the authorization of scopes using own browser. But the access token is required to access to Web Apps, if it is **`"Who has access to the app:": Only myself`** or **`"Who has access to the app:": Anyone`**.

- Only when it is **`"Who has access to the app:": Anyone, even anonymous`**, owner and client users can access to Web Apps without the access token.

<a name="shareproject"></a>

## Share project of Web Apps with client users

When client users access to Web Apps with Situation 2 and 5, the owner of Web Apps is required to share the project deployed Web Apps with the client users. If the project is not shared, the error message of `<title>Google Drive - Access Denied</title>` is returned. For example, if the project is the container-bound script with Spreadsheet, please share the spreadsheet with the client users.

For sharing the project, you can [manually share it](https://support.google.com/drive/answer/2494822) and use "VIEW" to the permissions.

Also you can share the project using GAS scripts. The sample GAS script for sharing the project with the client users is as follows. When you use this, please input email and fileId. For example, if the project is the container-bound script with Spreadsheet, the fileId is that of spreadsheet. If you want to send a notification email when the project is shared, please modify from `sendNotificationEmail=false` to `sendNotificationEmail=true`.

```javascript
// DriveApp.getFiles() // This comment is used for including a scope of https://www.googleapis.com/auth/drive. Of course, you can use Manifests for the project.
var email = "### Client user's email address ###";
var fileId = "### fileId of project ###";

var url =
  "https://www.googleapis.com/drive/v3/files/" +
  fileId +
  "/permissions?sendNotificationEmail=false";
var params = {
  method: "post",
  headers: { Authorization: "Bearer " + ScriptApp.getOAuthToken() },
  contentType: "application/json",
  payload: JSON.stringify({
    role: "reader",
    type: "user",
    emailAddress: email,
  }),
  muteHttpExceptions: true,
};
var res = UrlFetchApp.fetch(url, params);
Logger.log(res);
```

<a name="howtousedevmode"></a>

# How to use dev mode from outside

When you deploy Web Apps, you can see the link labeled **latest code**. The link is like `https://script.google.com/macros/s/###/dev`. The official document of Google is as follows.

> This URL can only be accessed by users who have edit access to the script. This instance of the app always runs the most recently saved code — not necessarily a formal version — and is intended for quick testing during development.

When you access to the link of **latest code** using your browser under you login to Google, you can access to Web Apps with the dev mode. But if you want to access to Web Apps with the dev mode from outside, there are no documents for the method. Here, it introduces such method.

When it accesses to the URL of **latest code** from outside, the login screen is returned. **This situation indicates that it is required to use the access token for accessing to the URL.** So as a sample, I show you the sample curl command as follows. The requirement scope is `https://www.googleapis.com/auth/drive`.

### Requesting to doGet()

```bash
curl -L \
  -H "Authorization: Bearer ### access token ###" \
  "https://script.google.com/macros/s/#####/dev"
```

Also from [Standard Query Parameters](https://developers.google.com/drive/api/v3/query-parameters), you can use the following method using the access token as the query parameter.

```bash
curl -L "https://script.google.com/macros/s/#####/dev?access_token=### access token ###"
```

- Replace `### access token ###` to the value retrieved by ScriptApp.getOAuthToken().
- Replace `https://script.google.com/macros/s/#####/dev` to your endpoint retrieved by deploying Web Apps.

### Requesting to doPost()

```bash
curl -L \
  -H "Authorization: Bearer ### access token ###" \
  -d "key=value" \
  "https://script.google.com/macros/s/#####/dev"
```

Also from [Standard Query Parameters](https://developers.google.com/drive/api/v3/query-parameters), you can use the following method using the access token as the query parameter.

```bash
curl -L \
  -d "key=value" \
  "https://script.google.com/macros/s/#####/dev?access_token=### access token ###"
```

- Replace `### access token ###` to the value retrieved by ScriptApp.getOAuthToken().
- Replace `https://script.google.com/macros/s/#####/dev` to your endpoint retrieved by deploying Web Apps.
- I used `-d "key=value"` for the post method. If you don't want to put the values, please replace to `-d ""`.

## Note

- By using above method, you can use the latest script of Web Apps from outside.
- If you want to use the dev mode as simple test, there is a simple method for retrieving the access token.
  1. Login to Google and open the script editor.
  1. Copy and paste the following script.
     - `function getAccesstoken() {Logger.log(ScriptApp.getOAuthToken())} // DriveApp.getFiles()`
  1. Run the function `getAccesstoken()` and retrieve the access token.

By this, you can test it using the retrieved access token.

- This method was answered at [https://stackoverflow.com/a/54191688/7108653](https://stackoverflow.com/a/54191688/7108653).

- The reason that `-d` is used instead of `-X POST` can be seen at [this thread](https://stackoverflow.com/q/57103278).

<a name="eventobject"></a>

# Event object of Web Apps

When Web Apps is used, `doGet(e)` and `doPost(e)` are used for the GET and POST method, respectively. Here, it introduces the event object of `e`.

## Sample script of Web Apps

The sample script for GET and POST method of Web Apps is as follows.

```javascript
function doGet(e) {
  e.method = "GET";
  return ContentService.createTextOutput(JSON.stringify(e)).setMimeType(
    ContentService.MimeType.JSON
  );
}

function doPost(e) {
  e.method = "POST";
  return ContentService.createTextOutput(JSON.stringify(e)).setMimeType(
    ContentService.MimeType.JSON
  );
}
```

## Sample client for accessing to Web Apps.

As a sample client, curl is used.

### Pattern 1

#### Sample curl

```bash
$ curl -L "https://script.google.com/macros/s/#####/exec?key1=value1&key2=value2&key3=value3"
```

#### Result

```json
{
  "parameter": {
    "key1": "value1",
    "key2": "value2",
    "key3": "value3"
  },
  "contextPath": "",
  "contentLength": -1,
  "queryString": "key1=value1&key2=value2&key3=value3",
  "parameters": {
    "key1": ["value1"],
    "key2": ["value2"],
    "key3": ["value3"]
  },
  "method": "GET"
}
```

### Pattern 2

#### Sample curl

```bash
$ curl -L -d "key1=value1" -d "key2=value2" "https://script.google.com/macros/s/#####/exec?key3=value3"
```

#### Result

```json
{
  "parameter": {
    "key1": "value1",
    "key2": "value2",
    "key3": "value3"
  },
  "contextPath": "",
  "contentLength": 23,
  "queryString": "key3=value3",
  "parameters": {
    "key1": ["value1"],
    "key2": ["value2"],
    "key3": ["value3"]
  },
  "postData": {
    "type": "application/x-www-form-urlencoded",
    "length": 23,
    "contents": "key1=value1&key2=value2",
    "name": "postData"
  },
  "method": "POST"
}
```

### Pattern 3

#### Sample curl

```bash
$ curl -L -d '{"key1": "value1", "key2": "value2"}' "https://script.google.com/macros/s/#####/exec?key3=value3"
```

#### Result

```json
{
  "parameter": {
    "key3": "value3",
    "{\"key1\": \"value1\", \"key2\": \"value2\"}": ""
  },
  "contextPath": "",
  "contentLength": 36,
  "queryString": "key3=value3",
  "parameters": {
    "key3": ["value3"],
    "{\"key1\": \"value1\", \"key2\": \"value2\"}": [""]
  },
  "postData": {
    "type": "application/x-www-form-urlencoded",
    "length": 36,
    "contents": "{\"key1\": \"value1\", \"key2\": \"value2\"}",
    "name": "postData"
  },
  "method": "POST"
}
```

### Pattern 4

#### Sample curl

```bash
$ curl -L -H "Content-Type: application/json" -d '{"key1": "value1", "key2": "value2"}' "https://script.google.com/macros/s/#####/exec?key3=value3"
```

#### Result

**Although the JSON object is sent as `application/json` and Web Apps recognizes the data as `application/json`, the JSON parse of `contents` is not parsed. Please be careful this. In this case, it is required to parse using `JSON.parse()`.**

```json
{
  "parameter": {
    "key3": "value3"
  },
  "contextPath": "",
  "contentLength": 36,
  "queryString": "key3=value3",
  "parameters": {
    "key3": ["value3"]
  },
  "postData": {
    "type": "application/json",
    "length": 36,
    "contents": "{\"key1\": \"value1\", \"key2\": \"value2\"}",
    "name": "postData"
  },
  "method": "POST"
}
```

### Pattern 5

#### Sample curl

In this pattern, the filename is added to the URL as follows.

```
https://script.google.com/macros/s/#####/exec/fileName.txt
```

**In this case, the authorization is required. Please be careful this.** So when the curl command is used, the access token is required to be used as follows.

```bash
$ curl -L \
  -H "Authorization: Bearer ###"  \
  -H "Content-Type: application/json" \
  -d '{"key1": "value1", "key2": "value2"}' \
  "https://script.google.com/macros/s/#####/exec/fileName.txt?key3=value3""
```

Also, when you have already logged in Google account, you can directly access to the URL. But in this case, it is the GET method.

When you run above curl command, you can retrieve the following values. `fileName.txt` can be retrieved by `pathInfo`.

#### Result

```json
{
  "pathInfo": "fileName.txt",
  "contextPath": "",
  "postData": {
    "contents": "{\"key1\": \"value1\", \"key2\": \"value2\"}",
    "length": 36,
    "name": "postData",
    "type": "application/json"
  },
  "contentLength": 36,
  "parameter": {
    "key3": "value3"
  },
  "parameters": {
    "key3": ["value3"]
  },
  "queryString": "key3=value3",
  "method": "POST"
}
```

<a name="checklog"></a>

# Logs in Web Apps for Google Apps Script

[Gists](https://gist.github.com/tanaikech/3ccb4dd8ce43de21fdb764a68c14a4d7)

This is a report for retrieving the logs in Web Apps for Google Apps Script, when it requests to the Web Apps.

## Experimental condition

### 1. Sample script for Web Apps

```javascript
const doGet = (e) => {
  Logger.log(`GET method: ${JSON.stringify(e)}`);
  console.log(`GET method: ${JSON.stringify(e)}`);
  return ContentService.createTextOutput(
    JSON.stringify({ method: "GET", e: e })
  );
};
const doPost = (e) => {
  Logger.log(`POST method: ${JSON.stringify(e)}`);
  console.log(`POST method: ${JSON.stringify(e)}`);
  return ContentService.createTextOutput(
    JSON.stringify({ method: "POST", e: e })
  );
};
```

- This Web Apps is deployed as `Execute the app as: Me` and `Who has access to the app: Anyone, even anonymous`.

### 2. Sample Google Apps Script project

1. Google Apps Script of standalone type WITHOUT linking Google Cloud Platform (GCP) Project

   - In this case, you can retrieve this standalone Google Apps Script by directly creating.

2. Google Apps Script of standalone type WITH linking Google Cloud Platform (GCP) Project
   - In this case, you can retrieve this standalone Google Apps Script by [this flow](https://gist.github.com/tanaikech/e945c10917fac34a9d5d58cad768832c).

### 3. Experimental procedure

To above Web Apps of `doGet` and `doPost`, it requests with the following 4 patterns.

1. For `doGet`.

   ```bash
   $ curl -L "https://script.google.com/macros/s/###/exec"
   ```

2. For `doPost`.

   ```bash
   $ curl -L -d "key=value" "https://script.google.com/macros/s/###/exec"
   ```

3. For `doGet`. Access token is used.

   ```bash
   $ curl -L -H "Authorization: Bearer ###" "https://script.google.com/macros/s/###/exec"
   ```

4. For `doPost`. Access token is used.

   ```bash
   $ curl -L -H "Authorization: Bearer ###" -d "key=value" "https://script.google.com/macros/s/###/exec"
   ```

## Result and discussions

The conditions which can confirm the logs are as follows.

<br>
<br>

|                     | Without access token | With access token                     |
| :------------------ | :------------------- | :------------------------------------ |
| Without linking GCP |                      | Apps Script Dashboard                 |
| With linking GCP    | Stackdriver          | Apps Script Dashboard and Stackdriver |

<br>
<br>

From above results, it was found as follows.

- If you use the default Google Apps Script project without linking GCP, in order to retrieve the logs which requested to the Web Apps, please access to the Web Apps using the access token, even when the Web Apps is deployed as `Execute the app as: Me` and `Who has access to the app: Anyone, even anonymous`.

- If you use the Google Apps Script project with linking GCP, you can retrieve all logs of users who accessed to the Web Apps at Stackdriver, even when the Web Apps is deployed as `Execute the app as: Me` and `Who has access to the app: Anyone, even anonymous`.

- In this case, the logs couldn't be seen with `Logger.log` for above all situations.

## IMPORTANT

In the current stage, when the access token is used for XMLHttpRequest and fetch of Javascript in the request headers, the error related to CORS occurs. So, in [this report](https://gist.github.com/tanaikech/e27581278f8cb464dff1dd83d6f887a7), I would like to propose the workaround for resolving this issue.

<a name="limitationofsimultaneousconnection"></a>

# Limitation of simultaneous connection to Web Apps

The limitation of simultaneous connection is also investigated. The fetchAll method added by recent Google's update was used for this investigation. [I have reported that the fetchAll method is worked by the asynchronous processing.](https://gist.github.com/tanaikech/c0f383034045ab63c19604139ecb0728) This can be used for measuring the limitation of simultaneous connection. From the result of measurement, it was found that the limitation of simultaneous connection to one Web Apps server is under 30. [This is the same with scripts.run method of Apps Script API.](https://github.com/tanaikech/RunAll)

When ["Current limitations" of "Quotas for Google Services"](https://developers.google.com/apps-script/guides/services/quotas#current_limitations), it says that the simultaneous executions is 30.

<a name="errormessages"></a>

# Error messages

When the error messages are returned from Web Apps, you can see the messages into the tag of `<title>### Error message ###</title>` including in HTML output. From the error messages, you can know the reason of the error using the following table. I think that there may be other errors. So if you found them, when you tell me them, I'm glad.

| Execute the app as                | Who has access to the app | Access          | Status code | Error messages                                   | Reason                                                                                                                 |
| :-------------------------------- | :------------------------ | :-------------- | :---------- | :----------------------------------------------- | :--------------------------------------------------------------------------------------------------------------------- |
| User accessing the web app        | Only myself,<br>Anyone    | Owner,<br>Users | 200         | Authorization needed                             | Scopes for scripts of Web Apps are not authorized.                                                                     |
| User accessing the web app,<br>Me | Only myself,<br>Anyone    | Owner,<br>Users | 200         | Meet Google Drive – One place for all your files | No access token.                                                                                                       |
| For all settings                  | For all settings          | Owner,<br>Users | 200         | Error                                            | "Service invoked too many times in a short time: exec qps. Try Utilities.sleep(1000) between calls." is shown in Body. |
| User accessing the web app,<br>Me | Only myself,<br>Anyone    | Owner,<br>Users | 401         | Unauthorized                                     | Bad access token.<br>No required scopes.                                                                               |
| User accessing the web app,<br>Me | Anyone                    | Users           | 403         | Google Drive - Access Denied                     | Project of Web Apps is not shared with users.                                                                          |
| User accessing the web app,<br>Me | Only myself               | Users           | 404         | Google Drive -- Page Not Found                   | Users cannot access.                                                                                                   |

<a name="corsinwebapps"></a>

# CORS in Web Apps

In this section, I would like to introduce CORS in Web Apps. As a sample situation, it supposes that Web Apps is requested using GET and POST method with Javascript. The settings of Web Apps are `Execute the app as: Me` and `Who has access to the app: Anyone, even anonymous`.

The Javascript of the client side, which was used for testing, is as follows.

```javascript
const url = "https://script.google.com/macros/s/###/exec";

function get() {
  fetch(url)
    .then((res) => {
      console.log(res.status);
      return res.text();
    })
    .then((res) => console.log(res));
}

function post() {
  const obj = { key: "value" };
  fetch(url, { method: "POST", body: JSON.stringify(obj) })
    .then((res) => {
      console.log(res.status);
      return res.text();
    })
    .then((res) => console.log(res));
}
```

## Sample script 1

### Script for Web Apps

```javascript
const doGet = (e) => {};
const doPost = (e) => {};
```

### Result

For both requests, the following response is returned. **Error related to CORS occurs.**

> Access to fetch at 'https://script.google.com/macros/s/###/exec' from origin 'https://###script.googleusercontent.com' has been blocked by CORS policy: No 'Access-Control-Allow-Origin' header is present on the requested resource. If an opaque response serves your needs, set the request's mode to 'no-cors' to fetch the resource with CORS disabled.

## Sample script 2

### Script for Web Apps

```javascript
const doGet = (e) => HtmlService.createHtmlOutput();
const doPost = (e) => HtmlService.createHtmlOutput();
```

### Result

For both requests, the following response is returned. **Error related to CORS occurs.**

> Access to fetch at 'https://script.google.com/macros/s/###/exec' from origin 'https://###script.googleusercontent.com' has been blocked by CORS policy: No 'Access-Control-Allow-Origin' header is present on the requested resource. If an opaque response serves your needs, set the request's mode to 'no-cors' to fetch the resource with CORS disabled.

## Sample script 3

### Script for Web Apps

```javascript
const doGet = (e) => ContentService.createTextOutput();
const doPost = (e) => ContentService.createTextOutput();
```

### Result

For both requests, the following response is returned. **No error occurs.**

> 200

## Sample script 4

### Script for Web Apps

```javascript
const doGet = (e) => ContentService.createTextOutput();
const doPost = (e) => ContentService.createTextOutput();
```

### Script for Javascript

In this case, axios is used. This sample referred [Stackoverflow](https://stackoverflow.com/q/62587453).

```javascript
const url = "https://script.google.com/macros/s/###/exec";

async function get() {
  const res = await axios.get(url);
  console.log(res);
}

async function post() {
  const obj = { key: "value" };
  const res = await axios.post(url, obj);
  console.log(res);
}
```

In this case, at `post()`, the object is not converted to the string with `JSON.stringify`.

### Result

- **No error occurs at `get()`.**
- **Error related to CORS occurs at `post()`.**

### Removing this error

Please convert the object to the string with `JSON.stringify`. By this, the error can be removed. **No error occurs.**

```javascript
async function post() {
  const obj = { key: "value" };
  const res = await axios.post(url, JSON.stringify(obj));
  console.log(res);
}
```

When above script didn't work, please set `"Content-Type": "text/plain"` to the header and test it again. On the other hand, when the following script is used, **No error occurs at `post()`.**

```javascript
function post() {
  const obj = { key: "value" };
  fetch(url, { method: "POST", body: obj })
    .then((res) => {
      console.log(res.status);
      return res.text();
    })
    .then((res) => console.log(res));
}
```

## Summary

As the result, in order to prevent the error related to CORS, it was found the following important points.

1. It is required to return `ContentService.createTextOutput()` and `ContentService.createTextOutput("done")` from `doGet` and `doPost`.

2. For POST method, the data is required to be converted to the string and sent to Web Apps.

### Reference

- [Class ContentService](https://developers.google.com/apps-script/reference/content/content-service)

<a name="confidentialityofscripts"></a>

# Confidentiality of scripts for Web Apps

- If you don't want to publish the scripts of Web Apps, you can achieve it using Situation 3.

  - In this case, anybody can access the Web Apps.

- If you want to make only the special users access to Web Apps, you can achieve it using Situation 2 and 5.
  - In this case, the scripts of Web Apps can be seen by the users, because the project of scripts has to be shared with the users.

<a name="samplescriptofserverside"></a>

# Sample script of server side

The simple sample-script of server side (Web Apps) is as follows. When the client users access to this server, the server returns URL query and the request body which were sent by the client. And also a text file is created. In this script, the scope of `https://www.googleapis.com/auth/drive` is used for creating a text file. So when you use this script at Situation 4 and Situation 5 as mentioned above, at first, please authorize the scope by accessing to Web Apps using your browser. After the authorization was done, the text file is created by requesting the following scripts for the client side.

```javascript
function feedback(e, method) {
  var id = "";
  var user = Session.getEffectiveUser().getEmail();
  if (e.parameter.key1 == "value1") {
    id = DriveApp.createFile(
      "sample.txt",
      "Created by " + user,
      MimeType.PLAIN_TEXT
    ).getId();
  }
  return ContentService.createTextOutput(
    JSON.stringify({
      result: Object.keys(e.parameters).map(function (f) {
        return f + "," + e.parameters[f];
      }),
      method: method,
      createdFile: id ? id + " was created by " + user : "Not created.",
    })
  ).setMimeType(ContentService.MimeType.JSON);
}

function doGet() {
  return feedback(e, "GET");
}

function doPost() {
  return feedback(e, "POST");
}
```

<a name="samplescriptofclientside"></a>

# Sample scripts of client side

## Access with access token

### GET

#### Curl

```bash
curl -GL \
  -H "Authorization: Bearer ### your access token ###" \
  -d "key1=value1" \
  -d "key2=value2" \
  "https://script.google.com/macros/s/#####/exec?key3=value3"
```

#### Google Apps Script

```javascript
// DriveApp.getFiles() // This comment is used for including a scope of https://www.googleapis.com/auth/drive.
var params = {
  method: "GET",
  headers: { Authorization: "Bearer " + ScriptApp.getOAuthToken() },
  muteHttpExceptions: true,
};
var url =
  "https://script.google.com/macros/s/#####/exec?key1=value1&key2=value2&key3=value3";
var res = UrlFetchApp.fetch(url, params);
Logger.log(res);
```

### POST

#### Curl

```bash
curl -L \
  -H "Authorization: Bearer ### your access token ###" \
  -d "key1=value1" \
  -d "key2=value2" \
  "https://script.google.com/macros/s/#####/exec?key3=value3"
```

#### Google Apps Script

```javascript
// DriveApp.getFiles() // This comment is used for including a scope of https://www.googleapis.com/auth/drive.
var params = {
  method: "POST",
  headers: { Authorization: "Bearer " + ScriptApp.getOAuthToken() },
  payload: { key1: "value1", key2: "value2" },
  muteHttpExceptions: true,
};
var url = "https://script.google.com/macros/s/#####/exec?key3=value3";
var res = UrlFetchApp.fetch(url, params);
Logger.log(res);
```

## Access without access token

### GET

#### Curl

```bash
curl -GL \
  -d "key1=value1" \
  -d "key2=value2" \
  "https://script.google.com/macros/s/#####/exec?key3=value3"
```

#### Google Apps Script

```javascript
var params = {
  method: "GET",
  muteHttpExceptions: true,
};
var url =
  "https://script.google.com/macros/s/#####/exec?key1=value1&key2=value2&key3=value3";
var res = UrlFetchApp.fetch(url, params);
Logger.log(res);
```

### POST

#### Curl

```bash
curl -L \
  -d "key1=value1" \
  -d "key2=value2" \
  "https://script.google.com/macros/s/#####/exec?key3=value3"
```

#### Google Apps Script

```javascript
var params = {
  method: "POST",
  payload: { key1: "value1", key2: "value2" },
  muteHttpExceptions: true,
};
var url = "https://script.google.com/macros/s/#####/exec?key3=value3";
var res = UrlFetchApp.fetch(url, params);
Logger.log(res);
```

<a name="samplescriptvariouslanguages"></a>

# Sample scripts of client side by various languages

Now, you can see the following 11 sample scripts at **[https://gist.github.com/tanaikech/a72aab0242012362c46ec69031c720d5](https://gist.github.com/tanaikech/a72aab0242012362c46ec69031c720d5)**.

1. curl
2. Google Apps Script
3. Javascript
4. ajax
5. Node.js
6. axios
7. angular
8. go
9. python
10. php
11. powershell

<a name="statuscodefromwebapps"></a>

# Status code from Web Apps

Here, I would like to introduce the status code returned from Web Apps.

## Preparation:

As the preparation, Web Apps was deployed with "Execute the app as" and "Who has access to the app" as `Me` and `Anyone, even anonymous`, respectively. The sample script for Web Apps is as follows.

```javascript
function doGet(e) {
  return ContentService.createTextOutput("GET: Done.");
}
```

## Experiment:

### Using Google Apps Script:

At first, the status code was checked using Google Apps Script. The deployed Web Apps was requested with the following script.

```javascript
function myFunction() {
  var url_exec = "https://script.google.com/macros/s/###/exec";
  var url_dev = "https://script.google.com/macros/s/###/dev";
  var res = UrlFetchApp.fetchAll([{ url: url_exec }, { url: url_dev }]);
  res.forEach(function (e) {
    Logger.log(e.getResponseCode());
  });
}
```

In this case, the status code of `200` was obtained for both endpoints of `exec` and `dev`.

### using Curl:

In order to retrieve the status code with curl, `curl -s -o /dev/null -w "%{http_code}" http://www.example.org/` was used. This is from [this thread](https://superuser.com/questions/272265/getting-curl-to-output-http-status-code). Here, the status code was investigated using the curl command. Because the curl can access by 2 kinds of request by the options as showing below.

1. [`--include`](https://curl.haxx.se/docs/manpage.html#-i): Include the HTTP response headers in the output. The HTTP response headers can include things like server name, cookies, date of the document, HTTP version and more...

2. [`--head`](https://curl.haxx.se/docs/manpage.html#-I): (HTTP FTP FILE) Fetch the headers only! HTTP-servers feature the command HEAD which this uses to get nothing but the header of a document. When used on an FTP or FILE file, curl displays the file size and last modification time only.

Using above options, the following 4 patterns were investigated.

1. Request to the endpoint of `https://script.google.com/macros/s/###/exec` using the option `--include`.
   - `curl -sL --include -o /dev/null -w "%{http_code}" "https://script.google.com/macros/s/###/exec"`
   - `200` was returned.
1. Request to the endpoint of `https://script.google.com/macros/s/###/dev` using the option `--include`.
   - `curl -sL --include -o /dev/null -w "%{http_code}" "https://script.google.com/macros/s/###/dev"`
   - `200` was returned.
1. Request to the endpoint of `https://script.google.com/macros/s/###/exec` using the option `--head`.
   - `curl -sL --head -o /dev/null -w "%{http_code}" "https://script.google.com/macros/s/###/exec"`
   - **`403` was returned.**
1. Request to the endpoint of `https://script.google.com/macros/s/###/dev` using the option `--head`.
   - `curl -sL --head -o /dev/null -w "%{http_code}" "https://script.google.com/macros/s/###/dev"`
   - `200` was returned.

As the result, it was found that when the Web Apps of the endpoint of `https://script.google.com/macros/s/###/exec` was requested with the option `--head`, the status code of `403` was obtained.

## Result and discussions:

When the option `--head` is used for the curl command, from the document, this means that it requests only header and doesn't request the body. By this, it was found that the status code of `403` was returned.

Here, why was the status code of `200` returned for both options of `--include` and `--head` when it requests to the endpoint of `dev`? It is considered that the reason of this is due to that the login screen was returned. When the endpoint of `dev` is accessed, it is required to use the access token. When the access token is not used, the login screen is returned. In this case, the status code of `200` is returned. As the test case, when the access token is used for the endpoint of `dev` using below curl command,

```bash
curl -sL --head -H "Authorization: Bearer ###" -o /dev/null -w "%{http_code}" "https://script.google.com/macros/s/###/dev"
```

The status code of `403` was returned. From this result, the following results were obtained.

- When only the header is retrieved under that the Web Apps works fine, `403` is returned.
- When the login screen is returned, `200` is returned.

## Applications for this situation

As one of applications using this situation, there is the registration of webhook at [Trello's REST API](https://developers.trello.com/reference/).

When the Web Apps which was deployed as `Execute the app as: Me` and `Who has access to the app: Anyone, even anonymous` is registered as the webhook, an error of `{"message":"URL (https://script.google.com/macros/s/###/exec) did not return 200 status code, got 403","error":"ERROR"}` occurs. The reason of this error is due to above situation. So as the workaround for the registration, you can use the following flow.

- Before you run the script, please set the Web Apps as follows. - "Execute the app as" and "Who has access to the app" are `Me` and `Only myself`, respectively.
- After the response of like `{"id":"###","description":"sample","idModel":"###","callbackURL":"https://script.google.com/macros/s/###/exec","active":true}` was retrieved, please set the Web Apps as follows. - "Execute the app as" and "Who has access to the app" are `Me` and `Anyone, even anonymous`, respectively.

By this flow, the webhook can be used.

#### Sample script

This is a sample script of Google Apps Script for the registration of webhook URL.

```javascript
var url = "https://api.trello.com/1/tokens/###/webhooks/?key=###";
var payload = {
  callbackURL: "https://script.google.com/macros/s/###/exec",
  idModel: "###",
  description: "sample",
};
var options = { method: "post", payload: payload };
var res = UrlFetchApp.fetch(url, options);
Logger.log(res.getContentText());
```

---

<a name="applications"></a>

## Applications

Here, I would like to introduce the applications of Web Apps. If you can introduce your applications, please tell me. I would like to introduce them here.

### 1. Converting from a1Notation to GridRange

This is a sample API for [converting from a1Notation to GridRange](https://gist.github.com/tanaikech/95c7cd650837f33a564babcaf013cae0). The GridRange is often used at Sheets API. Although I have thought that a1Notation is easy to use, there are no methods for converting it. So I created this as an API. I would like to introduce this as a sample for this report.

The sample script is as follows. At first, please copy and paste it to new standalone project.

```javascript
function doGet(e) {
  var output = {};
  if (e.parameter.sheetid && e.parameter.a1notation) {
    output = {
      result: a1notation2gridrange1(
        e.parameter.sheetid,
        e.parameter.a1notation
      ),
    };
  } else {
    output = { error: "Wrong parameters." };
  }
  return ContentService.createTextOutput(JSON.stringify(output));
}

// https://gist.github.com/tanaikech/95c7cd650837f33a564babcaf013cae0
function a1notation2gridrange1(sheetid, a1notation) {
  try {
    var data = a1notation.match(/(^.+)!(.+):(.+$)/);
    var ss = SpreadsheetApp.openById(sheetid).getSheetByName(data[1]);
    var range = ss.getRange(data[2] + ":" + data[3]);
    var gridRange = {
      sheetId: ss.getSheetId(),
      startRowIndex: range.getRow() - 1,
      endRowIndex: range.getRow() - 1 + range.getNumRows(),
      startColumnIndex: range.getColumn() - 1,
      endColumnIndex: range.getColumn() - 1 + range.getNumColumns(),
    };
    if (!data[2].match(/[0-9]/)) delete gridRange.startRowIndex;
    if (!data[3].match(/[0-9]/)) delete gridRange.endRowIndex;
    return gridRange;
  } catch (e) {
    return e;
  }
}
```

And then, please deploy this as Web Apps. The condition is as follows.

- "Execute the app as:" : **User accessing the web app**
- "Who has access to the app:": **Only myself**

By this condition, owner and each client user can use Spreadsheets on own Google Drive. Before you access to Web Apps, please don't forget to share the project to users. The usage is as follows. This is a curl sample. But of course, you can also use this API using Google Apps Script.

```bash
curl -GL \
  -H "Authorization: Bearer ### your access token ###" \
  -d "sheetid=### spreadsheet ID ###" \
  -d "a1notation=Sheet1%21A1%3aB2" \
  "https://script.google.com/macros/s/#####/exec"
```

When you use this, please do URL encode for the range.

### 2. URL to upload to Google Drive

[https://stackoverflow.com/questions/47833728/url-to-upload-to-google-drive](https://stackoverflow.com/questions/47833728/url-to-upload-to-google-drive)

### 3. How to get a data range of the Sheet

[https://stackoverflow.com/questions/48917258/how-to-get-a-data-range-of-the-sheet](https://stackoverflow.com/questions/48917258/how-to-get-a-data-range-of-the-sheet)

### 4. Enhanced Custom Function for Google Spreadsheet using Web Apps as Wrapper

[https://github.com/tanaikech/Enhanced-Custom-Function-for-Google-Spreadsheet-using-Web-Apps-as-Wrapper](https://github.com/tanaikech/Enhanced-Custom-Function-for-Google-Spreadsheet-using-Web-Apps-as-Wrapper)

---

<a name="samplesituations"></a>

# Sample situations

Here, I would like to introduce the sample situations with Web Apps. Those are threads of Stackoverflow. You can see the actual situations using Web Apps from them.

- [How to upload a file via POST (doPost) to a Google Script's Web App?](https://stackoverflow.com/a/42217817)
- [How to use importXML function with a file from Google Drive?](https://stackoverflow.com/a/45991519)
- [Retrieving the output of a Google Apps script to local computer](https://stackoverflow.com/a/46106339)
- [URL to upload to Google Drive](https://stackoverflow.com/a/47841498)
- [Using AuthToken obtained via ScriptApp.getAuthToken() to call web apps in GAS](https://stackoverflow.com/a/48840870)
- [Is there any limit on number of concurrent hits on Google App Script Web App](https://stackoverflow.com/a/50033976)
- [Uploading files to pre-set Google Drive account using API?](https://stackoverflow.com/a/50206254)
- [can't open Slack dialog through google apps scripts](https://stackoverflow.com/a/51093107)
- [using an http request to update google spreadsheet](https://stackoverflow.com/a/53383507)
- [Trying to use dev mode with GAS doPOST](https://stackoverflow.com/a/54191688)
- [Google Sheets View Only Protection for User Running App Script](https://stackoverflow.com/a/54452680)
- [How to assign UrlFetchApp with basic authorization to button?](https://stackoverflow.com/a/54622357)
- [Protecting Cells of Spreadsheet that Users Copied from Your Google Drive to User's Google Drive using Google Apps Script](https://gist.github.com/tanaikech/847ea7e3e27a4a22004faa88d7b4dedb)
- [Maintaining protected ranges while copying a file](https://stackoverflow.com/a/57066732)
- [How to create webhook from Google Apps Script using the “exec” url rather than “dev” url. Exec returns 403 Error](https://stackoverflow.com/a/57426103)
- [One Time Download for Google Drive](https://github.com/tanaikech/One_Time_Download_for_Google_Drive)
- [Open a HTML file with data using Google Script](https://stackoverflow.com/q/58055279)
- [Receiving Access-Control-Allow-Origin errors when submiting a form via Google Apps Script](https://stackoverflow.com/q/58773436)
- [How to do curl POST of CSV data?](https://stackoverflow.com/q/58847822)
- [How to POST from JS or node.js?](https://stackoverflow.com/q/58849644)
- [Google Drive API: Create Folder: Allow conflict error if folder by name exists](https://stackoverflow.com/q/59184790)
- [How to display classic Google Sites search box query in home page?](https://stackoverflow.com/q/57257600)
- [Is it possible to insert a Horizontal Rule with Google Docs API?](https://stackoverflow.com/q/59501947)
- [How to send an e.parameter to Google Apps Script using PHP and cURL](https://stackoverflow.com/q/59910628)
- [Apps Script: Import and parse a HTML file from Drive to Sheets](https://stackoverflow.com/q/59933492)
- [How to get all tab names/gid's with google.visualization from google spreadsheet?](https://stackoverflow.com/q/59933529)
- [Google spreadsheet how to insert column after the last column](https://stackoverflow.com/q/60007252)
- [Is it possible to set Chart options through the Google Sheets API?](https://stackoverflow.com/q/60309878)
- [Upload a word doc and an google app script. Convert the word doc to a google doc and then run the app script clean up on it?](https://stackoverflow.com/q/60399008)
- [Which are the right scopes to run request on Apps Script API](https://stackoverflow.com/q/60650940)
- [google sheet REST API : how find location of cell with some value](https://stackoverflow.com/q/60763776)
- [Convert Google Sheet to PDF and store in same Google Drive folder as sheet](https://stackoverflow.com/q/60933742)
- [Can I download a Google Document, via the Google Docs API without an OAuth consent screen?](https://stackoverflow.com/q/61399708)
- [Link to a specific sheet in published Google Sheet](https://stackoverflow.com/q/61437687)
- [How can one write to a publicly available Google sheet (without authorization) in Python?](https://stackoverflow.com/q/61484573)
- [Use path/slug after Web App's base url in Google Apps Script](https://stackoverflow.com/q/61729358)
- [Is it possible to get fresh Google Slides presentation data in AppsScript?](https://stackoverflow.com/q/62166790)
- [Error when running Youtube Data Service in App Scripts (js) – Daily Limit for Unauthenticated Use Exceeded](https://stackoverflow.com/q/62205747)
- [Trouble accessing Google Sheet as a TSV file](https://stackoverflow.com/q/62611127/7108653)
- [Get data from Google Sheets without Sheets API](https://stackoverflow.com/q/62732791)
- [Retrieve text in namedRange with Google Docs API](https://stackoverflow.com/q/62797150/7108653)
- [Using private sheets with tabletop.js](https://stackoverflow.com/q/63135606/7108653)
- [I'm trying to get the direct download link for a file using the Google Drive API](https://stackoverflow.com/q/63363012)
- [Upload file to my google drive with Google Apps Script (NO FORM IN GOOGLE)](https://stackoverflow.com/q/63374794)
- [Upload image from URL to Google Drive through Google Drive API Javascript](https://stackoverflow.com/q/63461593)
- [Google Sheets API with php - Find a cell value](https://stackoverflow.com/q/63496284)
- [calling sheet function in GAS](https://stackoverflow.com/q/63553532)
- [Possible to send mail from Google by URL?](https://stackoverflow.com/q/63817399)
- [Formula Works in Google App Script Console, But Returns Authentication Error in Spreadsheets](https://stackoverflow.com/q/63879205/7108653)
- [Scrape data from someone else's Google Sheets page using node.js?](https://stackoverflow.com/q/64001024/7108653)
- [How to enable not authorized users to protect the spreadsheet](https://stackoverflow.com/q/64086095/7108653)
- [Retrieving All Values from All Sheets from URL of 2PACX- of Web Published Google Spreadsheet using Google Apps Script and Javascript](https://tanaikech.github.io/2020/10/12/retrieving-all-values-from-all-sheets-from-url-of-2pacx-of-web-published-google-spreadsheet-using-google-apps-script-and-javascript/)
- [Changing Owner of the Sheet irrespective of the duplicator](https://stackoverflow.com/q/64666605/7108653)
- [Google Sheets - Run script with Sheet Protection](https://stackoverflow.com/q/64673622/7108653)
- [User Runs Script for Range Protected by Owner using Google Apps Script](https://gist.github.com/tanaikech/5ebf492b53de40fe254dba63c8520391)

---

<a name="references"></a>

# References

- [Web Apps](https://developers.google.com/apps-script/guides/web)
- [`gas-web-app` tag on Stackoverflow](https://stackoverflow.com/tags/gas-web-app/info)

---

<a name="licence"></a>

# Licence

[MIT](LICENCE)

<a name="author"></a>

# Author

[Tanaike](https://tanaikech.github.io/about/)

If you have any questions, feel free to tell me using e-mail tanaike@hotmail.com Also if you found new information related to this report and tell me them, I am glad. I would like to update this report.

<a name="update_history"></a>

# Update History

- v1.0.0 (April 26, 2018)

  Initial release.

[TOP](#TOP)
