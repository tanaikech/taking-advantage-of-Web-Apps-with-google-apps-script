# Taking advantage of Web Apps with Google Apps Script

<a name="top"></a>
[![MIT License](http://img.shields.io/badge/license-MIT-blue.svg?style=flat)](LICENCE)

# Table of contents

- [Overview](#overview)
- [Description](#description)
- **Web App Deployment & Configuration**
  - [Deploy Web Apps](#deploywebApps)
  - [Redeploying Web Apps without Changing URL of Web Apps for new IDE](#redeploy)
  - [5 situations for Web Apps](#situationsforwebapps)
- **Understanding Access and Interaction**
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
  - [Understanding Flow of Request to Web Apps Created by Google Apps Script](#understandrequestwebapps)
- **Developing & Troubleshooting Web Apps**
  - [Event object of Web Apps](#eventobject)
  - [Understanding Logs in Google Apps Script Web Apps](#checklog)
  - [Error messages](#errormessages)
  - [Status code from Web Apps](#statuscodefromwebapps)
  - [CORS in Web Apps](#corsinwebapps)
  - [Confidentiality of scripts for Web Apps](#confidentialityofscripts)
- **Performance & Advanced Capabilities**
  - [Limitation of simultaneous connection to Web Apps](#limitationofsimultaneousconnection)
  - [Concurrent access to Web Apps](#concurrentaccesstowebapps)
  - [Implementing Pseudo 2FA for Web Apps](#2fatowebapps)
  - [Executing Google Apps Script with Service Account](#executingwithserviceaccount)
- **Code Examples**
  - [Sample script of server side](#samplescriptofserverside)
  - [Sample scripts of client side](#samplescriptofclientside)
  - [Sample scripts of client side by various languages](#samplescriptvariouslanguages)
  - [Request Web Apps using Fetch API of Javascript with the access token](#fetchapiwithaccesstoken)
- **Workarounds**
  - [1. Reflecting Latest Script to Deployed Web Apps without Redeploying](#workaround1)
- **Real-World Use Cases**
  - [Applications](#applications)
  - [Applications for AI](#applicationsforai)
  - [Sample situations](#samplesituations)
- **Appendices**
  - [References](#references)
  - [Licence](#licence)
  - [Author](#author)
  - [Update History](#updatehistory)

<a name="overview"></a>

# Overview

This repository provides a comprehensive guide to understanding and leveraging Web Apps with Google Apps Script (GAS).

<a name="description"></a>

# Description

Google Apps Script (GAS) provides a powerful feature called Web Apps, which transforms your server-side script functions (doGet, doPost) into accessible web services. Leveraging the streamlined deployment process available in the new Apps Script IDE, developers can effortlessly create custom web pages, robust backend APIs, or versatile webhooks.

The primary effect of Web Apps is to seamlessly bridge the gap between your Google Workspace environment and external users or applications, enabling dynamic and interactive solutions. Key advantages include unparalleled versatility from hosting responsive user interfaces to serving as powerful backend services for mobile applications or third-party platforms. Web Apps offer flexible access control, allowing precise definition of who can execute your scripts and under what permissions, enhancing both security and accessibility. This eliminates the need for complex server setup and ongoing maintenance, making it a highly cost-effective and rapid development solution for leveraging Google's cloud infrastructure. This repository aims to provide comprehensive insights and practical guidance for maximizing the potential of Web Apps across diverse scenarios.

# Web App Deployment & Configuration

<a name="deploywebApps"></a>

## Deploy Web Apps

For detailed information, refer to [the official documentation](https://developers.google.com/apps-script/guides/web#deploy_a_script_as_a_web_app).

To deploy a web app, open the script editor in Google Apps Script and click "Deploy" \> "New deployment." Then, select "Web app" under "Select type." This will display the following dialog:

![](images/DeployWebApps_fig1.png)

Follow these steps to deploy the web app in the script editor:

1.  In the script editor, click "Deploy" \> "New deployment" in the top right.
2.  Click "Select type" \> "Web App." The dialog shown above will appear.
3.  Enter the web app's information in the fields under "Deployment configuration."
4.  Select **"Execute as"**.
      * Details for each value are in the following section. [Ref](#situationsforwebapps)
5.  Select **"Who has access to the app:"**.
      * Details for each value are in the following section. [Ref](#situationsforwebapps)
6.  Click "Deploy."
7.  In the script editor, click "Deploy" \> "Test deployments" in the top right.
8.  Copy the web app URL, which will be similar to `https://script.google.com/macros/s/###/exec`. This URL can be accessed from outside of Google.

**It is important to note that whenever you modify the Google Apps Script for the web app, you must deploy it as a new version.** This ensures the changes are reflected in the web app. For more details on this, please refer to my report: "[Redeploying Web Apps without Changing URL of Web Apps for new IDE](https://gist.github.com/tanaikech/ebf92d8f427d02d53989d6c3464a9c43)."

---

<a name="redeploy"></a>

## Redeploying Web Apps without Changing Web Apps URL

Prior to March 15, 2021, each web app deployment generated a unique endpoint [Ref](https://developers.google.com/apps-script/releases/#march_15_2021). Consequently, redeploying a web app would change its URL due to a new deployment ID. This behavior appears to be the standard specification. This report outlines a method for redeploying web apps without altering their URL.

- When the script of Web Apps is changed and redeploy Web Apps, after March 15, 2021, the endpoint is changed. This is new specification. Because the deployment ID of `###` in `https://script.google.com/macros/s/###/exec` is changed.

When you want to use the same endpoint by reflected the latest script to Web Apps, please do the following flow.

1.  Create new Google Apps Script project and copy and paste the following script.

    ```javascript
    function doGet() {
      return ContentService.createTextOutput("sample1");
    }
    ```

2.  Deploy Web Apps as the following settings.

    - `Execute as: Me`
    - `Who has access: Anyone`

3.  Test the Web Apps using the following curl command.

    ```bash
    $ curl -L "https://script.google.com/macros/s/###/exec"
    ```

    - Please confirm that `sample1` is returned.

4.  Modify the script of Web Apps as follows.

    ```javascript
    function doGet() {
      return ContentService.createTextOutput("sample2"); // Modified
    }
    ```

5.  Update the deployment as follows.

    ![](images/RedeployingWebAppswithoutChangingWebAppsURL_fig1.png)

    1. Open "New deployment" dialog with "Deploy" -> "Manage deployments".
    2. For the 1st deployment, click the pencil icon for editing the deployment.
    3. Set "version" as "New version".
    4. When you want to change the description, please modify it.
    5. Click "Deploy" button.

6.  Testing.

    - Please use above sample curl command. By this, you can confirm that `sample2` is returned without changing the endpoint.

For detailed instructions, please refer to [https://gist.github.com/tanaikech/ebf92d8f427d02d53989d6c3464a9c43](https://gist.github.com/tanaikech/ebf92d8f427d02d53989d6c3464a9c43).

---

<a name="situationsforwebapps"></a>

## 5 situations for Web Apps

1. **Situation 1**
   - "Execute the app as:" : **Me**
   - "Who has access to the app:": **Only myself**
1. **Situation 2**
   - "Execute the app as:" : **Me**
   - "Who has access to the app:": **Anyone with Google account**
1. **Situation 3**
   - "Execute the app as:" : **Me**
   - "Who has access to the app:": **Anyone**
1. **Situation 4**
   - "Execute the app as:" : **User accessing the web app**
   - "Who has access to the app:": **Only myself**
1. **Situation 5**
   - "Execute the app as:" : **User accessing the web app**
   - "Who has access to the app:": **Anyone with Google account**

# Understanding Access and Interaction

<a name="howtoaccesstowebapps"></a>

## How to Access Web Apps

There are 5 deployment situations for Web Apps, as previously mentioned. The methods for accessing deployed Web Apps are broadly divided into two types: accessing via a browser, and accessing via tools like Curl or Google Apps Script that don't use a browser. Each method is used by both the owner of the Web App and client users. The summarized flow for these methods is as follows.

---

<a name="howtoaccesstowebapps1"></a>

### 1. Owner Accesses Web Apps Using a Browser

```mermaid
flowchart TD
    A[Owner] --> B[Browser]
    B --> C[Log in to Google and authorize scopes]
    C --> D[Situation 1]
    C --> E[Situation 2]
    C --> F[Situation 3]
    C --> G[Situation 4]
    C --> H[Situation 5]
````

  * **For Situations 1, 4:** The owner can access and run the Web App's script by logging in to Google.
  * **For Situations 2, 5:** The owner can access and run the Web App's script by logging in to Google.
  * **For Situation 3:** The owner can access and run the Web App's script without logging in to Google.

---

<a name="howtoaccesstowebapps2"></a>

### 2. Client Users Access Web Apps Using a Browser

```mermaid
flowchart TD
    A[Client users] --> B[Browser]
    B --> C[Log in to Google and authorize scopes]
    C --> D[Situation 1]
    C --> E[Situation 2]
    C --> F[Situation 3]
    C --> G[Situation 4]
    C --> H[Situation 5]
```

  * **For Situations 1, 4:** Client users cannot access and run the Web App's script.
  * **For Situations 2, 5:** Client users can access and run the Web App's script by logging in to Google.
  * **For Situation 3:** Client users can access and run the Web App's script without logging in to Google.

---

<a name="howtoaccesstowebapps3"></a>

### 3. Owner Accesses Web Apps Using Tools (e.g., Curl, Google Apps Script) Without a Browser

```mermaid
flowchart TD
    A[Owner] --> B[Curl, Google Apps Script, etc., not using a browser]
    B --> C{Are scopes used in Web App scripts?}
    C --> |Yes| D[Authorization of scopes with the browser]
    C --> |No| E[Situation 3]
    D --> E
    D --> F[Access token]
    F --> G[Situation 1]
    F --> H[Situation 2]
    F --> I[Situation 4]
    F --> J[Situation 5]
```

  * **For Situations 1, 4:** The owner can access and run the Web App's script by using an access token.
  * **For Situations 2, 5:** The owner can access and run the Web App's script by using an access token.
  * **For Situation 3:** If the Web App's script uses specific scopes, the owner must authorize those scopes via their browser. The owner can then access and run the Web App's script without logging in to Google for each access.

---

<a name="howtoaccesstowebapps4"></a>

### 4. Client Users Access Web Apps Using Tools (e.g., Curl, Google Apps Script) Without a Browser

```mermaid
flowchart TD
    A[Client users] --> B[Curl, Google Apps Script, etc., not using a browser]
    B --> C{Using 'Situation 3'?}
    C --> |No| D[Share Web Apps project with users]
    C --> |Yes| E{Are scopes used in Web App scripts?}
    D --> E
    E --> |Yes| F[Authorization of scopes using the browser]
    F --> G[Situation 3]
    E --> |No| G
    F --> H[Access token]
    H --> I[Situation 1]
    H --> J[Situation 2]
    H --> K[Situation 4]
    H --> L[Situation 5]
```

  * **For Situations 1, 4:** Client users cannot access and run the Web App's script.
  * **For Situations 2, 5:**
      * **Initially, the project where the Web App is deployed must be shared with the client users who will use the Web App.**
      * *I confirmed that as of April 11, 2018, sharing the project is required to access Web Apps. This might be due to a Google update.*
      * If the Web App's script uses specific scopes, client users must authorize those scopes via their browser.
      * Client users can then access and run the Web App's script without logging in to Google for each access.
  * **For Situation 3:**
      * If the Web App's script uses specific scopes, client users must authorize those scopes via their browser.
      * Client users can access and run the Web App's script without logging in to Google.

---

<a name="requiredparameters"></a>

## Required Parameters for Accessing Deployed Web Apps

The table below summarizes the requirements for accessing deployed Google Apps Script Web Apps in five different scenarios. "Owner" refers to the user who deployed the Web App, and "Client users" refers to the users who access and use the deployed Web App.

| Situations | Script of Web Apps | Authorization for Scopes | Owner                        | Client Users                 | Share Project |
| :---------- | :----------------- | :----------------------- | :--------------------------- | :--------------------------- | :------------ |
| Situation 1 | Run as owner       | Only owner               | Access with access token     | Cannot access                | No            |
| Situation 2 | Run as owner       | Only owner               | Access with access token     | Access with access token     | Yes           |
| Situation 3 | Run as owner       | Only owner               | Access without access token  | Access without access token  | No            |
| Situation 4 | Run as each user   | Each user                | Access with access token     | Cannot access                | No            |
| Situation 5 | Run as each user   | Each user                | Access with access token     | Access with access token     | Yes           |

For example, in **Situation 5**, the Web App's script runs as each user (both the owner and client users). Authorization for the scopes used by the Web App's scripts is required for them to run. This authorization must be performed by each user (owner and client users) through their own browser. An access token is required for each user (owner and client users) to access the Web App. To allow client users to access the Web App, the project from which the Web App was deployed must be shared with them.

---

<a name="authorizationforscopes"></a>

## Authorization for Scopes

* When a Web App is deployed with **`Execute the app as: Me`** by the owner, the authorization screen is automatically displayed. Once the owner authorizes it, both the owner and client users can run the Web App's scripts as the owner.
    * **Example**: If `Session.getEffectiveUser().getEmail()` is run by a client user in this situation, the retrieved email will be the owner's email, indicating that the script is running as the owner.
    * **Example**: If `DriveApp.createFile(blob)` is executed within the Web App's script, the file will be created in the owner's Google Drive.

* When a Web App is deployed with **`Execute the app as: User accessing the web app`** by the owner, the authorization screen is **not** automatically displayed. Therefore, before the owner and client users can access the Web App, they must manually authorize the necessary scopes using their own browsers.
    * If the Web App's scripts use specific scopes, the owner and client users only need to authorize them once via their own browser before accessing the Web App.
        * The authorization URL is `https://script.google.com/macros/s/#####/exec`, which is the same as the Web App's URL. When the owner and client users access this URL using their own browser, the following authorization screen appears:
            * ![](images/Authorizationforscopes_fig1.png)
        * On this screen, click "**REVIEW PERMISSIONS**", then select your account and authorize the scopes. After the owner and client users have authorized the scopes, they can run the Web App's script.
    * **Example**: If `Session.getEffectiveUser().getEmail()` is run by a client user in this situation, the retrieved email will be each individual user's email, indicating that the script is running as each user.
    * **Example**: If `DriveApp.createFile(blob)` is executed within the Web App's script, the file will be created in each individual user's Google Drive.

---

<a name="accesstokenforaccessingtowebapps"></a>

## Access Token for Accessing Web Apps

* When a Web App is deployed with **"`Who has access to the app: Only myself`"** or **"`Who has access to the app: Anyone with Google account`"** by the owner, both the owner and client users must access and run the Web App's script using their own access token.
    * At least one Drive API scope must be included in the access token. Examples include:
        * `https://www.googleapis.com/auth/drive.readonly`
        * `https://www.googleapis.com/auth/drive.files`
        * `https://www.googleapis.com/auth/drive`
    * Even if the server script uses scopes other than those for Drive API, these additional scopes are not required to be included in the access token. This is because the use of other scopes is authorized through the browser before accessing the Web App. This means that when making GET or POST requests to the Web App, only Drive API scopes are required.
    * For instance, when accessing a Web App using Google Apps Script, if you use `{"Authorization": "Bearer " + ScriptApp.getOAuthToken()}` in the headers and encounter an `<TITLE>Unauthorized</TITLE>` error, check the scopes defined in your script editor. If `https://www.googleapis.com/auth/drive` is not included, you can add it by, for example, placing `// DriveApp.getFiles()` in your script **as a comment**. The script editor will then automatically include the `https://www.googleapis.com/auth/drive` scope.

* If no scopes are used in the Web App's scripts, the owner and client users can run the script without browser-based scope authorization. However, an access token is still required to access the Web App if it's deployed as **"`Who has access to the app:` Only myself`"** or **"`Who has access to the app: Anyone with Google account`"**.

* Only when the Web App is deployed as **"`Who has access to the app: Anyone`"** can the owner and client users access the Web App without an access token.

---

<a name="shareproject"></a>

## Share Web App Project with Client Users

When client users access a Web App under **Situation 2** or **Situation 5**, the Web App owner must share the deployed project with them. If the project is not shared, an `<title>Google Drive - Access Denied</title>` error message will be returned. For example, if the project is a container-bound script linked to a Spreadsheet, share the Spreadsheet with the client users.

You can [manually share the project](https://support.google.com/drive/answer/2494822) by setting the permissions to "VIEW".

Alternatively, you can share the project using Google Apps Script. The following sample script demonstrates how to share the project with client users. Replace `"### Client user's email address ###"` with the actual email and `"### fileId of project ###"` with the project's file ID. For a container-bound script with a Spreadsheet, the file ID is that of the Spreadsheet. If you wish to send a notification email when the project is shared, change `sendNotificationEmail=false` to `sendNotificationEmail=true`.

```javascript
// DriveApp.getFiles() // This comment helps include the "[https://www.googleapis.com/auth/drive](https://www.googleapis.com/auth/drive)" scope. Alternatively, you can define scopes in the project's manifest file.
var email = "### Client user's email address ###";
var fileId = "### fileId of project ###";

var url =
  "[https://www.googleapis.com/drive/v3/files/](https://www.googleapis.com/drive/v3/files/)" +
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

---

<a name="howtousedevmode"></a>

## How to Use Dev Mode from Outside

When you deploy a Web App in Google Apps Script, you'll see a link labeled **latest code**, which typically looks like `https://script.google.com/macros/s/###/dev`. Google's official documentation states:

> This URL can only be accessed by users who have edit access to the script. This instance of the app always runs the most recently saved code — not necessarily a formal version — and is intended for quick testing during development.

If you're logged into Google in your browser, you can directly access the Web App in dev mode via this **latest code** link. However, accessing the Web App in dev mode from an external application or environment is not straightforward, and there isn't official documentation on how to do it. This section will guide you through the process.

When attempting to access the **latest code** URL from outside, you'll typically be redirected to a login screen. This indicates that an **access token is required** to authenticate your request. As an example, the following `curl` commands demonstrate how to achieve this. The required OAuth scope for this operation is `https://www.googleapis.com/auth/drive`.

### Requesting `doGet()`

Here's how to make a `GET` request to your Web App's `doGet()` function using an access token in the header:

```bash
curl -L \
  -H "Authorization: Bearer ### access token ###" \
  "https://script.google.com/macros/s/#####/dev"
```

Alternatively, as per the [Standard Query Parameters](https://developers.google.com/drive/api/v3/query-parameters) documentation, you can pass the access token as a query parameter:

```bash
curl -L "https://script.google.com/macros/s/#####/dev?access_token=### access token ###"
```

  * Replace `### access token ###` with the actual access token obtained using `ScriptApp.getOAuthToken()`.
  * Replace `https://script.google.com/macros/s/#####/dev` with the dev mode endpoint of your deployed Web App.

### Requesting `doPost()`

To make a `POST` request to your Web App's `doPost()` function, you can include the access token in the header:

```bash
curl -L \
  -H "Authorization: Bearer ### access token ###" \
  -d "key=value" \
  "https://script.google.com/macros/s/#####/dev"
```

Similarly, you can use the access token as a query parameter for `POST` requests:

```bash
curl -L \
  -d "key=value" \
  "https://script.google.com/macros/s/#####/dev?access_token=### access token ###"
```

  * Replace `### access token ###` with the actual access token obtained using `ScriptApp.getOAuthToken()`.
  * Replace `https://script.google.com/macros/s/#####/dev` with the dev mode endpoint of your deployed Web App.
  * The `-d "key=value"` flag is used to send data in the request body. If you don't need to send any key-value pairs, you can replace it with `-d ""`.

-----

### Note

  * By using the methods described above, you can effectively test the latest code of your Web App from external environments.

  * For simple testing purposes, you can retrieve an access token directly from within your Google Apps Script editor:

    1.  Log in to Google and open your script project in the Apps Script editor.
    2.  Paste the following function into your script:
        ```javascript
        function getAccesstoken() {
          Logger.log(ScriptApp.getOAuthToken());
          // DriveApp.getFiles(); // Uncomment if you need to authorize Drive scope
        }
        ```
    3.  Run the `getAccesstoken()` function. The access token will be printed in the Logger (View \> Logs).

    You can then use this retrieved access token for your testing.

  * This method was originally discussed and answered on Stack Overflow: [https://stackoverflow.com/a/54191688/7108653](https://stackoverflow.com/a/54191688/7108653).

  * For an explanation on why `-d` is used instead of `-X POST` in `curl` commands for `POST` requests, refer to [this Stack Overflow thread](https://stackoverflow.com/q/57103278).

---

<a name="understandrequestwebapps"></a>

## Understanding Flow of Request to Web Apps Created by Google Apps Script

This is from my gist. [https://gist.github.com/tanaikech/131ba814a1f6012fd6a5ffe11789971f](https://gist.github.com/tanaikech/131ba814a1f6012fd6a5ffe11789971f)

![](images/UnderstandingFlowofRequesttoWebApps_fig1.png)

Here, I would like to introduce a report for understanding the flow of the request to Web Apps created by Google Apps Script. There might be a case that various applications using the Web Apps are created and the Web Apps are used as the webhook. In that case, it is considered that when you have understood the flow of requests to the Web Apps, your goal might be able to be smoothly achieved. In this report, I would like to introduce the information about it.

### Sample situation

As a sample situation, the sample script for the Web Apps is as follows. And, please set a Spreadsheet ID to `work_`. In this sample, the data from the request is put into the Spreadsheet. And, the event object is directly returned from the Web Apps.

```javascript
function work_(e, method) {
  e[method] = method;
  const res = JSON.stringify(e);
  SpreadsheetApp.openById("###SpreadsheetId###").appendRow([new Date(), res]);
  return ContentService.createTextOutput(res);
}

const doGet = (e) => work_(e, "GET method");
const doPost = (e) => work_(e, "POST method");
```

And, please deploy this script as the Web Apps with the following condition.

- `Execute as: Me`
- `Who has access to the app: Anyone`

Please copy and paste your Web Apps URL like `https://script.google.com/macros/s/{deploymentId}/exec`. By this, this Web Apps can be accessed without the access token. This is a simple sample Web Apps.

### Request to Web Apps

#### 1. Simple curl command by manual operation

Here, in order to understand the flow of requests to the Web Apps, it tests to request the Web Apps using a simple curl command.

When it requests this Web Apps using curl command with the GET and POST methods, those are as follows.

```bash
curl -I -XGET "https://script.google.com/macros/s/{deploymentId}/exec"
```

```bash
curl -I -XPOST "https://script.google.com/macros/s/{deploymentId}/exec"
```

The status code 302 is returned. In this case, it indicates the redirect. And, the response header has the "location" property including the redirect URL. From this result, it is found that in the case of Web Apps, it is required to run the redirect.

When this redirect process is manually run, it becomes as follows.

##### For `doGet`

1. Request with the GET method to the Web Apps URL like `https://script.google.com/macros/s/{deploymentId}/exec`.

```bash
curl -I -XGET "https://script.google.com/macros/s/{deploymentId}/exec"
```

2. Retrieve the URL of "location" from the response header of the above request. It's like `https://script.googleusercontent.com/macros/echo?user_content_key=###id1###&lib=###id2###`.

3. Request the redirect URL with the GET method.

```bash
curl -XGET "https://script.googleusercontent.com/macros/echo?user_content_key=###id1###&lib=###id2###"
```

By this flow, the following value is returned from Web Apps.

```json
{
  "contentLength": -1,
  "queryString": "",
  "contextPath": "",
  "parameters": {},
  "parameter": {},
  "method": "GET method"
}
```

##### For `doPost`

1. Request with POST method to the Web Apps URL like `https://script.google.com/macros/s/{deploymentId}/exec`.

```bash
curl -I -XPOST "https://script.google.com/macros/s/{deploymentId}/exec"
```

2. Retrieve the URL of "location" from the response header of the above request. It's like `https://script.googleusercontent.com/macros/echo?user_content_key=###id1###&lib=###id2###`.

3. Request the redirect URL with the POST method.

```bash
curl -XPOST "https://script.googleusercontent.com/macros/echo?user_content_key=###id1###&lib=###id2###"
```

    - In this case, an error like "Page Not Found" (status code 405) occurs.

4. Request the redirect URL with the GET method.

```bash
curl -XGET "https://script.googleusercontent.com/macros/echo?user_content_key=###id1###&lib=###id2###"
```

- In this case, no error occurs. The correct value is returned from the Web Apps.

By this flow, the following value is returned from Web Apps.

```json
{
  "queryString": "",
  "parameters": {},
  "parameter": {},
  "contextPath": "",
  "contentLength": 0,
  "method": "POST method"
}
```

**From these 2 patterns, it is found that in the case of the Web Apps, it is required to request with the GET method to the redirect URL for both `doGet` and `doPost`.**

#### 2. Send values using the curl command by manual operation

Here, it tests sending values to the Web Apps using a curl command.

##### For `doGet`

1. Request with the GET method to the Web Apps URL like `https://script.google.com/macros/s/{deploymentId}/exec`. In this case, the value is included in the query parameter.

```bash
curl -I -XGET "https://script.google.com/macros/s/{deploymentId}/exec?key1=value1"
```

2. Retrieve the URL of "location" from the response header of the above request. It's like `https://script.googleusercontent.com/macros/echo?user_content_key=###id1###&lib=###id2###`.

   - Here, the most important point is that at the 1st request before the redirect process, the value of `key1=value1` has already been sent to the Web Apps. So, in this time, the values are put into the sheet with `SpreadsheetApp.openById("###").appendRow([new Date(), res])`.

3. Request the redirect URL with the GET method.

```bash
curl -XGET "https://script.googleusercontent.com/macros/echo?user_content_key=###id1###&lib=###id2###"
```

By this flow, the following value is returned from Web Apps. You can see that the returned values include the sending values.

```json
{
  "contextPath": "",
  "queryString": "key1=value1",
  "parameter": { "key1": "value1" },
  "contentLength": -1,
  "parameters": { "key1": ["value1"] },
  "method": "GET method"
}
```

##### For `doPost`

1. Request with POST method to the Web Apps URL like `https://script.google.com/macros/s/{deploymentId}/exec`. In this case, the values are included in both the query parameter and the request body.

```bash
curl -vso /dev/null -XPOST -d "key2=value" "https://script.google.com/macros/s/{deploymentId}/exec?key1=value1"
```

2. Retrieve the URL of "location" from the response header of the above request. It's like `https://script.googleusercontent.com/macros/echo?user_content_key=###id1###&lib=###id2###`.

   - Here, the most important point is that at the 1st request before the redirect process, the values of `key1=value1` and `key2=value` have already been sent to the Web Apps. So, this time, the values are put into the sheet with `SpreadsheetApp.openById("###").appendRow([new Date(), res])`.

3. Request the redirect URL with the GET method.

```bash
curl -XGET "https://script.googleusercontent.com/macros/echo?user_content_key=###id1###&lib=###id2###"
```

By this flow, the following value is returned from Web Apps. You can see that the returned values include the sending values.

```json
{
  "postData": {
    "contents": "key2=value",
    "length": 10,
    "name": "postData",
    "type": "application/x-www-form-urlencoded"
  },
  "contextPath": "",
  "queryString": "key1=value1",
  "contentLength": 10,
  "parameter": { "key2": "value", "key1": "value1" },
  "parameters": { "key1": ["value1"], "key2": ["value"] },
  "method": "POST method"
}
```

#### 4. Send values using the curl command by one command

When `--location` or `-L` is used with the curl command, the Web Apps can be requested by one command. [Ref](https://github.com/tanaikech/taking-advantage-of-Web-Apps-with-google-apps-script#sample-client-for-accessing-to-web-apps)

##### For `doGet`

```bash
curl -L -XGET "https://script.google.com/macros/s/{deploymentId}/exec?key1=value1"
```

##### For `doPost`

```bash
curl -L -d "key2=value" "https://script.google.com/macros/s/{deploymentId}/exec?key1=value1"
```

- Here, when `-XPOST` and `-L` are used, the request process is as follows. By this, the response values from Web Apps are not returned while the values are correctly sent to the Web Apps.

  1.  Request the Web Apps URL with the POST method.
  2.  Request the redirect URL with the POST method.

- On the other hand, when `-d` and `-L` are used, the request process is as follows. By this, the response values from Web Apps are correctly returned, and also the values are correctly sent to the Web Apps.

  1.  Request the Web Apps URL with the POST method.
  2.  Request the redirect URL with the GET method.

- The values, which you want to send to the Web Apps, are required to be included in the 1st request to the Web Apps URL. It is not required to include the values in the 2nd request to the redirect URL.

### Summary

From the experiment result, it was found the following important result.

1. In the case of a request to `doGet`, the following 2 requests are required to be run.

   1. Request the Web Apps URL with the GET method.
   2. Request the redirect URL with the GET method.

2. In the case of a request to `doPost`, the following 2 requests are required to be run.

   1. Request the Web Apps URL with the POST method.
   2. Request the redirect URL with the GET method.

3. For example, when in your situation of the POST method, when the values can be correctly sent to the Web Apps and the returned values are not correctly returned from the Web Apps, the request for the redirect might be the POST method. In that case, please change the request for the redirect URL from the POST method to the GET method. If your script cannot be directly changed, please try to do the above process with a script or a manual operation.

### Appendix

As an appendix, I would like to introduce the sample scripts of Google Apps Script for the above flow.

#### for `doGet`

```javascript
function requestToDoGet() {
  const webAppsUrl = "https://script.google.com/macros/s/{deploymentId}/exec";

  // 1st request.
  const res1 = UrlFetchApp.fetch(webAppsUrl + "?key=value", {
    method: "get",
    followRedirects: false,
  });
  const headers = res1.getAllHeaders();
  console.log(res1.getContentText()); // Here, HTML data including the redirect URL is returned. And, the query parameter of "key=value" is sent.
  const location = headers["Location"];

  // 2nd request.
  const res2 = UrlFetchApp.fetch(location, {
    method: "get",
    followRedirects: false,
  });
  console.log(res2.getContentText()); // Here, the values are returned from `doPost`.
}
```

#### for `doPost`

```javascript
function requestToDoPost() {
  const webAppsUrl = "https://script.google.com/macros/s/{deploymentId}/exec";

  // 1st request.
  const res1 = UrlFetchApp.fetch(webAppsUrl, {
    method: "post",
    payload: "sample",
    followRedirects: false,
  });
  const headers = res1.getAllHeaders();
  console.log(res1.getContentText()); // Here, HTML data including the redirect URL is returned. And, the request body of "sample" is sent.
  const location = headers["Location"];

  // 2nd request.
  const res2 = UrlFetchApp.fetch(location, {
    method: "get",
    followRedirects: false,
  });
  console.log(res2.getContentText()); // Here, the values are returned from `doPost`.
}
```

---

# Developing & Troubleshooting Web Apps

<a name="eventobject"></a>

## Event Object of Web Apps

When using Web Apps, `doGet(e)` and `doPost(e)` handle the GET and POST methods, respectively. This document introduces the event object `e`.

### Sample Web Apps Script

Here's a sample script for the GET and POST methods in Web Apps:

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

### Sample Client for Accessing Web Apps

We'll use `curl` as a sample client.

#### Pattern 1

##### Sample curl

```bash
$ curl -L "[https://script.google.com/macros/s/#####/exec?key1=value1&key2=value2&key3=value3](https://script.google.com/macros/s/#####/exec?key1=value1&key2=value2&key3=value3)"
```

##### Result

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

#### Pattern 2

##### Sample curl

```bash
$ curl -L -d "key1=value1" -d "key2=value2" "[https://script.google.com/macros/s/#####/exec?key3=value3](https://script.google.com/macros/s/#####/exec?key3=value3)"
```

##### Result

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

#### Pattern 3

##### Sample curl

```bash
$ curl -L -d '{"key1": "value1", "key2": "value2"}' "[https://script.google.com/macros/s/#####/exec?key3=value3](https://script.google.com/macros/s/#####/exec?key3=value3)"
```

##### Result

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

#### Pattern 4

##### Sample curl

```bash
$ curl -L -H "Content-Type: application/json" -d '{"key1": "value1", "key2": "value2"}' "[https://script.google.com/macros/s/#####/exec?key3=value3](https://script.google.com/macros/s/#####/exec?key3=value3)"
```

##### Result

**Although the JSON object is sent as `application/json` and Web Apps recognizes the data as `application/json`, the `contents` property is not automatically parsed as a JSON object. You need to parse it manually using `JSON.parse()`.**

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

#### Pattern 5

##### Sample curl

In this pattern, a filename is added to the URL:

```
[https://script.google.com/macros/s/#####/exec/fileName.txt](https://script.google.com/macros/s/#####/exec/fileName.txt)
```

**Note that authorization is required in this case.** When using the `curl` command, you must include an access token as follows:

```bash
$ curl -L \
  -H "Authorization: Bearer ###" \
  -H "Content-Type: application/json" \
  -d '{"key1": "value1", "key2": "value2"}' \
  "[https://script.google.com/macros/s/#####/exec/fileName.txt?key3=value3](https://script.google.com/macros/s/#####/exec/fileName.txt?key3=value3)"
```

Alternatively, if you are already logged into a Google account, you can directly access the URL. However, this will use the GET method.

When you run the above `curl` command, you will retrieve the following values. The `fileName.txt` can be accessed via `pathInfo`.

##### Result

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

##### `pathInfo`: Updated on February 14, 2023

Currently, `pathInfo` appears to be usable with an access token. Consider the following sample script:

```javascript
function doGet(e) {
  return ContentService.createTextOutput(JSON.stringify(e));
}
```

If you are logged into your Google account and access `https://script.google.com/macros/s/###/exec/sample.txt` with your browser, you will see `{"contextPath":"","parameter":{},"pathInfo":"sample.txt","contentLength":-1,"parameters":{},"queryString":""}`.

**If you try to access it without logging into a Google account, even if Web Apps is deployed as `Execute as: Me` and `Who has access to the app: Anyone`, a login screen will appear. Please be aware of this.**

If you wish to access `https://script.google.com/macros/s/###/exec/sample.txt` using a script, your request must include an access token. The access token can also be included as a query parameter. Ensure the access token includes at least one scope from the Drive API.

The sample `curl` command is as follows:

```bash
curl -L "[https://script.google.com/macros/s/###/exec/sample.txt?access_token=###](https://script.google.com/macros/s/###/exec/sample.txt?access_token=###)"
```

This will return the following result:

```json
{"contextPath":"","queryString":"access_token=###"},"pathInfo":"sample.txt","parameters":{"access_token":["###"]},"contentLength":-1}
```

---

<a name="checklog"></a>

## Understanding Logs in Google Apps Script Web Apps

[Gists](https://gist.github.com/tanaikech/3ccb4dd8ce43de21fdb764a68c14a4d7)

This document explains how to retrieve logs when requests are made to Google Apps Script Web Apps.

### Experimental Conditions

#### 1. Sample Web App Script

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

  - This Web App is deployed with `Execute the app as: Me` and `Who has access to the app: Anyone`.

#### 2. Sample Google Apps Script Project Types

1.  **Standalone Google Apps Script without a linked Google Cloud Platform (GCP) Project:**

      - You can create this standalone Google Apps Script directly.

2.  **Standalone Google Apps Script with a linked Google Cloud Platform (GCP) Project:**

      - You can achieve this by following [this flow](https://gist.github.com/tanaikech/e945c10917fac34a9d5d58cad768832c).

#### 3. Experimental Procedure

The Web App (both `doGet` and `doPost`) was requested using the following four patterns:

1.  **For `doGet`:**

    ```bash
    $ curl -L "https://script.google.com/macros/s/###/exec"
    ```

2.  **For `doPost`:**

    ```bash
    $ curl -L -d "key=value" "https://script.google.com/macros/s/###/exec"
    ```

3.  **For `doGet` using an access token:**

    ```bash
    $ curl -L -H "Authorization: Bearer ###" "https://script.google.com/macros/s/###/exec"
    ```

4.  **For `doPost` using an access token:**

    ```bash
    $ curl -L -H "Authorization: Bearer ###" -d "key=value" "https://script.google.com/macros/s/###/exec"
    ```

### Results and Discussions

The following table summarizes where logs can be found under different conditions:

| | Without Access Token | With Access Token |
| :---------------------- | :------------------- | :------------------------------------ |
| **Without linked GCP** | | Apps Script Dashboard |
| **With linked GCP** | Stackdriver | Apps Script Dashboard and Stackdriver |

From the above results, the following conclusions were drawn:

  - If you are using a default Google Apps Script project without linking a GCP project, to retrieve logs from requests to your Web App, you must access the Web App using an access token. This applies even when the Web App is deployed as `Execute the app as: Me` and `Who has access to the app: Anyone`.

  - If you are using a Google Apps Script project with a linked GCP project, all user access logs to the Web App can be retrieved in Stackdriver. This is true even when the Web App is deployed as `Execute the app as: Me` and `Who has access to the app: Anyone`.

  - In all experimental scenarios, logs generated by `Logger.log` were not visible.

### IMPORTANT

Currently, when an access token is used in the request headers for `XMLHttpRequest` and `fetch` in JavaScript, a CORS-related error occurs. Therefore, in [this report](https://gist.github.com/tanaikech/e27581278f8cb464dff1dd83d6f887a7), a workaround for this issue is proposed.

---

<a name="errormessages"></a>

## Error messages

When the error messages are returned from Web Apps, you can see the messages into the tag of `<title>### Error message ###</title>` including in HTML output. From the error messages, you can know the reason of the error using the following table. I think that there may be other errors. So if you found them, when you tell me them, I'm glad.

| Execute the app as                | Who has access to the app | Access          | Status code | Error messages                                   | Reason                                                                                                                 |
| :-------------------------------- | :------------------------ | :-------------- | :---------- | :----------------------------------------------- | :--------------------------------------------------------------------------------------------------------------------- |
| User accessing the web app        | Only myself,<br>Anyone with Google account    | Owner,<br>Users | 200         | Authorization needed                             | Scopes for scripts of Web Apps are not authorized.                                                                     |
| User accessing the web app,<br>Me | Only myself,<br>Anyone with Google account    | Owner,<br>Users | 200         | Meet Google Drive 窶・One place for all your files | No access token.                                                                                                       |
| For all settings                  | For all settings          | Owner,<br>Users | 200         | Error                                            | "Service invoked too many times in a short time: exec qps. Try Utilities.sleep(1000) between calls." is shown in Body. |
| User accessing the web app,<br>Me | Only myself,<br>Anyone with Google account    | Owner,<br>Users | 401         | Unauthorized                                     | Bad access token.<br>No required scopes.                                                                               |
| User accessing the web app,<br>Me | Anyone with Google account                    | Users           | 403         | Google Drive - Access Denied                     | Project of Web Apps is not shared with users.                                                                          |
| User accessing the web app,<br>Me | Only myself               | Users           | 404         | Google Drive -- Page Not Found                   | Users cannot access.                                                                                                   |

---

<a name="statuscodefromwebapps"></a>

## Status Codes from Web Apps

Here, I'll explain the status codes returned from Google Apps Script Web Apps.

### Preparation

For this experiment, a Web App was deployed with "Execute the app as" set to `Me` and "Who has access to the app" set to `Anyone`. The sample script for the Web App is as follows:

```javascript
function doGet(e) {
  return ContentService.createTextOutput("GET: Done.");
}
```

### Experiment

#### Using Google Apps Script

First, the status code was checked using Google Apps Script. The deployed Web App was requested with the following script:

```javascript
function myFunction() {
  var url_exec = "[https://script.google.com/macros/s/###/exec](https://script.google.com/macros/s/###/exec)";
  var url_dev = "[https://script.google.com/macros/s/###/dev](https://script.google.com/macros/s/###/dev)";
  var res = UrlFetchApp.fetchAll([{ url: url_exec }, { url: url_dev }]);
  res.forEach(function (e) {
    Logger.log(e.getResponseCode());
  });
}
```

In this case, a status code of `200` was obtained for both the `exec` and `dev` endpoints.

#### Using cURL

To retrieve the status code with cURL, I used `curl -s -o /dev/null -w "%{http_code}" http://www.example.org/`. This approach is derived from [this thread](https://superuser.com/questions/272265/getting-curl-to-output-http-status-code).

The status code was investigated using cURL, considering two types of requests based on the following options:

1.  [`--include`](https://curl.haxx.se/docs/manpage.html#-i): Includes the HTTP response headers in the output. This can include server name, cookies, document date, HTTP version, and more.
2.  [`--head`](https://curl.haxx.se/docs/manpage.html#-I): (HTTP FTP FILE) Fetches only the headers\! HTTP servers support the `HEAD` command, which curl uses to retrieve only the document's header. When used on an FTP or FILE file, curl displays only the file size and last modification time.

Using the above options, the following four patterns were investigated:

1.  Request to the `https://script.google.com/macros/s/###/exec` endpoint using the `--include` option.
      * `curl -sL --include -o /dev/null -w "%{http_code}" "https://script.google.com/macros/s/###/exec"`
      * `200` was returned.
2.  Request to the `https://script.google.com/macros/s/###/dev` endpoint using the `--include` option.
      * `curl -sL --include -o /dev/null -w "%{http_code}" "https://script.google.com/macros/s/###/dev"`
      * `200` was returned.
3.  Request to the `https://script.google.com/macros/s/###/exec` endpoint using the `--head` option.
      * `curl -sL --head -o /dev/null -w "%{http_code}" "https://script.google.com/macros/s/###/exec"`
      * **`403` was returned.**
4.  Request to the `https://script.google.com/macros/s/###/dev` endpoint using the `--head` option.
      * `curl -sL --head -o /dev/null -w "%{http_code}" "https://script.google.com/macros/s/###/dev"`
      * `200` was returned.

As a result, it was found that when the Web App at the `https://script.google.com/macros/s/###/exec` endpoint was requested with the `--head` option, a status code of `403` was obtained.

### Results and Discussion

When the `--head` option is used with the cURL command, according to its documentation, it requests only the header and not the body. This explains why a `403` status code was returned in that specific case.

Why was a status code of `200` returned for both `--include` and `--head` options when requesting the `dev` endpoint? It is likely because the login screen was returned. When the `dev` endpoint is accessed, an access token is typically required. If an access token is not used, the login screen is displayed, which often results in a `200` status code.

As a test case, when an access token is used for the `dev` endpoint with the following cURL command:

```bash
curl -sL --head -H "Authorization: Bearer ###" -o /dev/null -w "%{http_code}" "[https://script.google.com/macros/s/###/dev](https://script.google.com/macros/s/###/dev)"
```

A status code of `403` was returned. From these results, the following conclusions can be drawn:

  * When only the header is retrieved from a functional Web App, a `403` is returned.
  * When the login screen is returned, a `200` is returned.

### Applications for this Situation

One application where this situation is relevant is when registering a webhook with [Trello's REST API](https://developers.trello.com/reference/).

When a Web App deployed with "`Execute the app as: Me`" and "`Who has access to the app: Anyone`" is registered as a webhook, an error like `{"message":"URL (https://script.google.com/macros/s/###/exec) did not return 200 status code, got 403","error":"ERROR"}` occurs. This error is due to the situation described above.

As a workaround for registration, you can use the following flow:

  * **Before running the script**, set the Web App's permissions as follows:
      * "Execute the app as": `Me`
      * "Who has access to the app": `Only myself`
  * **After retrieving the response** (e.g., `{"id":"###","description":"sample","idModel":"###","callbackURL":"https://script.google.com/macros/s/###/exec","active":true}`), reset the Web App's permissions as follows:
      * "Execute the app as": `Me`
      * "Who has access to the app": `Anyone`

By following this flow, the webhook can be successfully used.

##### Sample Script

This is a sample Google Apps Script for registering a webhook URL:

```javascript
var url = "[https://api.trello.com/1/tokens/###/webhooks/?key=###](https://api.trello.com/1/tokens/###/webhooks/?key=###)";
var payload = {
  callbackURL: "[https://script.google.com/macros/s/###/exec](https://script.google.com/macros/s/###/exec)",
  idModel: "###",
  description: "sample",
};
var options = { method: "post", payload: payload };
var res = UrlFetchApp.fetch(url, options);
Logger.log(res.getContentText());
```

---

<a name="corsinwebapps"></a>

## CORS in Web Apps

This section explores **Cross-Origin Resource Sharing (CORS)** in Google Apps Script Web Apps. We'll examine various scenarios where Web Apps are accessed via `GET` and `POST` methods using JavaScript. For these examples, the Web App settings are configured as `Execute the app as: Me` and `Who has access to the app: Anyone`.

The following client-side JavaScript was used for testing:

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

### Sample Script 1

#### Web Apps Script

```javascript
const doGet = (e) => {};
const doPost = (e) => {};
```

#### Result

Both `GET` and `POST` requests return the following response, indicating a **CORS error**:

> Access to fetch at '[https://script.google.com/macros/s/\#\#\#/exec](https://script.google.com/macros/s/###/exec)' from origin 'https://\#\#\#script.googleusercontent.com' has been blocked by CORS policy: No 'Access-Control-Allow-Origin' header is present on the requested resource. If an opaque response serves your needs, set the request's mode to 'no-cors' to fetch the resource with CORS disabled.

### Sample Script 2

#### Web Apps Script

```javascript
const doGet = (e) => HtmlService.createHtmlOutput();
const doPost = (e) => HtmlService.createHtmlOutput();
```

#### Result

Both `GET` and `POST` requests return the same response as Sample Script 1, indicating a **CORS error**:

> Access to fetch at '[https://script.google.com/macros/s/\#\#\#/exec](https://script.google.com/macros/s/###/exec)' from origin 'https://\#\#\#script.googleusercontent.com' has been blocked by CORS policy: No 'Access-Control-Allow-Origin' header is present on the requested resource. If an opaque response serves your needs, set the request's mode to 'no-cors' to fetch the resource with CORS disabled.

### Sample Script 3

#### Web Apps Script

```javascript
const doGet = (e) => ContentService.createTextOutput();
const doPost = (e) => ContentService.createTextOutput();
```

#### Result

Both `GET` and `POST` requests return `200`. **No CORS error occurs.**

> 200

### Sample Script 4

#### Web Apps Script

```javascript
const doGet = (e) => ContentService.createTextOutput();
const doPost = (e) => ContentService.createTextOutput();
```

#### JavaScript Client Script

This example uses **Axios**. This sample was inspired by a Stack Overflow post: [Stackoverflow](https://stackoverflow.com/q/62587453).

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

In the `post()` function above, the `obj` is **not** converted to a string using `JSON.stringify`.

#### Result

  - **No error occurs for `get()`**.
  - **A CORS error occurs for `post()`**.

#### Resolving the `post()` Error

To resolve the CORS error for `post()`, the object must be converted to a string using `JSON.stringify`. With this change, **no error occurs**.

```javascript
async function post() {
  const obj = { key: "value" };
  const res = await axios.post(url, JSON.stringify(obj));
  console.log(res);
}
```

If the above script still doesn't work, try setting `"Content-Type": "text/plain"` in the request header.

On the other hand, if the following script is used for `post()`, **no error occurs**:

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

### Summary

To prevent CORS errors when working with Google Apps Script Web Apps, the following key points are crucial:

1.  Always return `ContentService.createTextOutput()` (or `ContentService.createTextOutput("done")`, etc.) from your `doGet` and `doPost` functions.
2.  For `POST` requests, ensure that the data sent to the Web App is converted to a string.

#### Reference

- [Class ContentService](https://developers.google.com/apps-script/reference/content/content-service)

---

<a name="confidentialityofscripts"></a>

## Confidentiality of Scripts for Web Apps

* If you don't want to publish the **scripts of Web Apps**, you can achieve this using **Situation 3**.
    * In this case, anyone can access the Web Apps.
* If you want to allow only specific users to access the **Web Apps**, you can achieve this using **Situation 2 and 5**.
    * In this case, the **scripts of Web Apps** can be viewed by the users because the script project must be shared with them.

---

# Performance & Advanced Capabilities

<a name="limitationofsimultaneousconnection"></a>

## Limitation of Simultaneous Connections to Web Apps

This section investigates the limitations of simultaneous connections to Web Apps. The `fetchAll` method, recently added by Google, was used for this investigation. [I have reported that the `fetchAll` method operates using asynchronous processing](https://gist.github.com/tanaikech/c0f383034045ab63c19604139ecb0728). This characteristic allows it to be used for measuring the limitation of simultaneous connections.

From the measurement results, it was found that the limitation of simultaneous connections to a single Web Apps server is under 30. [This aligns with the `scripts.run` method of the Apps Script API](https://github.com/tanaikech/RunAll).

According to ["Current limitations" in "Quotas for Google Services"](https://developers.google.com/apps-script/guides/services/quotas#current_limitations), the limit for simultaneous executions is 30. This result is the same as my measurement result.

---

<a name="concurrentaccesstowebapps"></a>

## Concurrent Access to Web Apps

- [Is there any limit on number of concurrent hits or simultaneous executions on Google App Script Web App](https://stackoverflow.com/q/17512194/7108653)
  - I answered this Stack Overflow thread. [Ref](https://stackoverflow.com/a/50033976/7108653)

- [Benchmark: Concurrent Writing to Google Spreadsheet using Form](https://gist.github.com/tanaikech/c2f3fccabbf4906a18fdc38463982f31)
  - When users attempt to write to a Google Sheet via a form, developers must account for concurrent submissions. For instance, if multiple users submit data simultaneously, not all data might be saved to the Sheet. Therefore, understanding concurrent writing behavior to Google Sheets using a form is crucial. This report investigates such a scenario.
  - The investigation into the success rate for concurrent writing to Google Sheets revealed that Web Apps created with Google Apps Script are more suitable than Google Forms. The threshold for successfully writing all data to the Sheet was 60 users for Web Apps and 35 for Google Forms. Furthermore, when using Web Apps for multiple submissions, it was found that **Lock Service** and sufficient **wait time** are absolutely essential.

---

<a name="2fatowebapps"></a>

## Implementing Pseudo 2FA for Web Apps

This is from my gist. [https://gist.github.com/tanaikech/7a15164b1227e2ec2231fce24ae9daf2](https://gist.github.com/tanaikech/7a15164b1227e2ec2231fce24ae9daf2)

![](images/2fatowebapps_fig1.png)

### Abstract

In Google Apps Script, there is the Web Apps. When Web Apps is used, the users can execute Google Apps Script using HTML and Javascript. This can be applied to various applications. When the Web Apps is deployed with "Anyone", anyone can access the Web Apps. And, there is the case that Web Apps deployed with "Anyone" is required to be used. Under this condition, when 2 Factor Authentication (2FA) can be implemented, it is considered that the security can be higher and it leads to giving various directions for the applications using Web Apps. In this report, I would like to introduce the method for implementing the pseud 2FA for Web Apps deployed with 窶廣nyone窶・using Google Apps Script.

### Introduction

This is a method for implementing the pseud 2 Factor Authentication (2FA) for Web Apps deployed with "Anyone" using Google Apps Script. About the Web Apps created by Google Apps Script, when the Web Apps is deployed as "Execute the app as: Anyone with Google account", the user is required to login to Google account. In this case, when the user has set 2FA in the Google account, that is used. But, when it is deployed as "Execute the app as: Anyone", the user is not required to login to Google account. Namely, all users can access the Web Apps. By this, when 2FA is required to be implemented by the developer. Recently, I have reported [Creating User's Dashboard by Inputting Name and Password using Web Apps with Google Apps Script](https://tanaikech.github.io/2022/08/01/creating-users-dashboard-by-inputting-name-and-password-using-web-apps-with-google-apps-script/). In this case, when the user name and the password are known, another user can access the user's dashboard. But, unfortunately, in the current stage, the Web Apps with "Execute the app as: Anyone" cannot directly use 2FA. So, in this post, I would like to introduce implementing the pseudo 2FA to the Web Apps deployed with "Execute the app as: Anyone". There is the case that it is required to use the Web Apps deployed as "Execute the app as: Anyone". This report might be useful for this situation.

In this sample, I implemented 2FA to this post "[Creating User's Dashboard by Inputting Name and Password using Web Apps with Google Apps Script](https://tanaikech.github.io/2022/08/01/creating-users-dashboard-by-inputting-name-and-password-using-web-apps-with-google-apps-script/)".

### Usage

#### 1. Prepare a standalone Google Apps Script project

In order to test this method, please create a standalone Google Apps Script project.

#### 2. Sample script

Please copy and paste the following script to the script editor of the created Google Apps Script project.

##### Google Apps Script: Code.gs

Please set `userObj`. In this script, an email including the authorization code for 2FA is sent to `email`. In this case, email addresses except for Gmail and Google email can be also used.

```javascript
// Please set the user's name and password and the sample value of the user.
// To use the sample value is a sample situation for explaining this method.
const userObj = [
  {
    email: "email address 1",
    password: "samplePassword1",
    sampleValue: "sample value 1",
  },
  {
    email: "email address 2",
    password: "samplePassword2",
    sampleValue: "sample value 2",
  },
  {
    email: "email address 3",
    password: "samplePassword3",
    sampleValue: "sample value 3",
  },
  ,
  ,
  ,
];

const limit = 600; // Expiration time of 2FA. Unit is seconds.

// Create key for each user.
const encodeValue_ = ({ email, pass }) =>
  Utilities.base64Encode(JSON.stringify([email, pass]));

// Show main HTML after the login and the confirmation of authorization code.
// If you want to change the returned value for each user, please modify "html.data = user.sampleValue;" of this function.
function showMainHTML_(user) {
  const htmlFilename = "showData";

  const html = HtmlService.createTemplateFromFile(htmlFilename);

  // This is a sample value. Please modify this for your actual situation.
  html.data = user.sampleValue;

  return html;
}

// Generate authorization code. A text of 5 letters.
function generateCode_(c, o) {
  const htmlFilename = "2FA";

  const newCode = Math.random().toString().slice(2, 7);
  c.put(encodeValue_(o), newCode, 3600);
  MailApp.sendEmail({
    to: o.email,
    subject: "Authorization code for 2FA of sample Web Apps.",
    body: `Authorization code is ${newCode}`,
  });
  const html = HtmlService.createTemplateFromFile(htmlFilename);
  html.url = ScriptApp.getService().getUrl();
  html.obj = JSON.stringify(o);
  return html;
}

// Show HTML of login.
function showLoginHTML_(error) {
  const htmlFilename = "login";

  const html = HtmlService.createTemplateFromFile(htmlFilename);
  html.url = ScriptApp.getService().getUrl();
  html.error = error;
  return html;
}

// This is a handler for using 2FA for Web Apps.
function handler_(e) {
  const { email, pass, code } = e.parameter;
  const c = CacheService.getScriptCache();
  let error = false;
  if (email && pass) {
    const user = userObj.find((o) => o.email == email && o.password == pass);
    if (user) {
      if (user.password == pass) {
        const currentCode = c.get(encodeValue_({ email, pass }));
        if (code && currentCode && currentCode == "AUTHORIZED") {
          return showMainHTML_(user);
        } else if (!code || code != "error") {
          return generateCode_(c, { email, pass });
        }
      }
    }
    error = true;
  }
  if (error && code == "error") {
    c.remove(encodeValue_({ email, pass }));
  }
  return showLoginHTML_(
    error
      ? code != "error"
        ? "Login error"
        : "Code error: Code is regenerated. Please log in and check email again."
      : ""
  );
}

// Get current stored code. This function is called from Javascript side.
function getCode(obj) {
  return CacheService.getScriptCache().get(encodeValue_(obj)) || "";
}

// Check inputted code. This function is called from Javascript side.
function checkCode({ email, pass, code }) {
  const c = CacheService.getScriptCache();
  const key = encodeValue_({ email, pass });
  const currentCode = c.get(key);
  if (currentCode && currentCode == code) {
    c.put(key, "AUTHORIZED");
    return true;
  }
  c.remove(key);
  return false;
}

// This is called as Web Apps.
function doGet(e) {
  return handler_(e)
    .evaluate()
    .setXFrameOptionsMode(HtmlService.XFrameOptionsMode.ALLOWALL);
}
```

- In this sample, the expiration time of the authorization code of 2FA is 10 minutes from `const limit = 600;`. So, after you logged in and authorized the code, the authorization code is not required to be used for 10 minutes. If you want to change this, please modify it.

##### HTML: login.html

```html
<p><?!= error ?></p>
<input type="email" id="email" placeholder="Please input your email address." />
<input type="password" id="pass" placeholder="Please input login password." />
<input type="button" value="login" onclick="sample()" />
<script>
  function sample() {
    const url = "<?!= url ?>";
    const [email, pass] = ["email", "pass"].map((e) =>
      document.getElementById(e).value.trim()
    );
    google.script.run
      .withSuccessHandler((code) => {
        window.open(
          code != ""
            ? `${url}?email=${email}&pass=${pass}&code=${code}`
            : `${url}?email=${email}&pass=${pass}`,
          "_top"
        );
      })
      .getCode({ email, pass });
  }
</script>
```

##### HTML: 2FA.html

```html
<p>
  An email including the authorization code was sent to your email address.
  Please check it and input the code to the following input tag.
</p>
<input type="text" id="code" placeholder="Please input your code." />
<input type="button" value="login" onclick="sample()" />
<script>
  function sample() {
    const url = "<?!= url ?>";
    const obj = <?!= obj ?>;
    const code = document.getElementById("code").value.trim();
    obj.code = code;
    google.script.run.withSuccessHandler(e => {
      if (e) {
        window.open(`${url}?code=${obj.code}&email=${obj.email}&pass=${obj.pass}`, "_top");
        return;
      }
      window.open(`${url}?code=error&email=${obj.email}&pass=${obj.pass}`, "_top");
    }).checkCode(obj);
  }
</script>
```

##### HTML: showData.html

```html
<?!= data ?>
```

#### 3. Deploy Web Apps.

The detailed information can be seen at [the official document](https://developers.google.com/apps-script/guides/web#deploy_a_script_as_a_web_app).

Please set this using the new IDE of the script editor.

1. On the script editor, at the top right of the script editor, please click "click Deploy" -> "New deployment".
2. Please click "Select type" -> "Web App".
3. Please input the information about the Web App in the fields under "Deployment configuration".
4. Please select **"Me"** for **"Execute as"**.
5. Please select **"Anyone"** for **"Who has access"**.
6. Please click the "Deploy" button.
7. Copy the URL of the Web App. It's like `https://script.google.com/macros/s/###/exec`.

- When you use this method, please access the retrieved URL using your browser. By this, you can see the login page.

- **When you modify the Google Apps Script, please modify the deployment as a new version. By this, the modified script is reflected in Web Apps. Please be careful about this.**

- You can see the detail of this in the report "[Redeploying Web Apps without Changing URL of Web Apps for new IDE](https://gist.github.com/tanaikech/ebf92d8f427d02d53989d6c3464a9c43)".

#### 4. Testing

When this method is used, the following sample situation is obtained.

![](images/2fatowebapps_fig2.gif)

In this sample, after you logged in and authorized the code, you can access your dashboard in Web Apps without the code for 10 minutes. After 10 minutes, the authorization code is reset. By this, it is required to authorize the code again.

When the malicious user accesses Web Apps without authorizing the code, even when the user knows your email and password, the user cannot authorize the code because the user cannot see your email box. By this, your dashboard cannot be seen by the malicious user.

### Note

- In this sample script, when the name, the password, and the authorization code are sent to the Google Apps Script side, those values are not encrypted as a sample. I selected this for the readability of the flow of this method. But, when you use this script in your actual situation, I would like to recommend encrypting the values.

## Reference

- [Creating User's Dashboard by Inputting Name and Password using Web Apps with Google Apps Script](https://tanaikech.github.io/2022/08/01/creating-users-dashboard-by-inputting-name-and-password-using-web-apps-with-google-apps-script/)

---

<a name="executingwithserviceaccount"></a>

## Executing Google Apps Script with Service Account

This is from my gist and Medium. [https://gist.github.com/tanaikech/304fea821ca36b8e9ccebe9814eaed82](https://gist.github.com/tanaikech/304fea821ca36b8e9ccebe9814eaed82) and [https://medium.com/@tanaike/executing-google-apps-script-with-service-account-3752f4e3df8c](https://medium.com/@tanaike/executing-google-apps-script-with-service-account-3752f4e3df8c)

![](images/executingwithserviceaccount_fig1.png)

### Abstract

One day, you might have a situation where it is required to run Google Apps Script using the service account. Unfortunately, in the current stage, Google Apps Script cannot be directly run with the service account because of the current specification. So, this report introduces a workaround for executing Google Apps Script using the service account.

### Introduction

When you want to execute Google Apps Script from outside of Google, as the basic approach, it can be achieved by Google Apps Script API. [Ref](https://developers.google.com/apps-script/api/reference/rest/v1/scripts/run) In order to use Google Apps Script, it is required to link the Google Apps Script project with the Google Cloud Platform project. [Ref](https://github.com/tanaikech/Linking-Google-Cloud-Platform-Project-to-Google-Apps-Script-Project-for-New-IDE) But, in the current stage, Google Apps Script can be executed by Google Apps Script API with only the access token obtained from OAuth2. Unfortunately, the access token obtained by the service account cannot used for executing Google Apps Script using Google Apps Script API. It seems that this is the current specification on the Google side. However, there might be a case that it is required to execute Google Apps Script using the service account. In this report, I would like to introduce a workaround for executing Google Apps Script using the service account. In this workaround, the Web Apps created by Google Apps Script is used. The Web Apps can be used for executing the preserved functions of `doGet` and `doPost` from outside of Google. [Ref](https://developers.google.com/apps-script/guides/web) In this workaround, this Web Apps is used for executing the various functions.

### Principle of this workaround

![](images/executingwithserviceaccount_fig2.png)

Unfortunately, the service account cannot directly execute Google Apps Script by the current specification. However, fortunately, the Web Apps can be accessed by the service account. This workaround uses this situation. The key factor for this workaround is to use the Web Apps deployed as `Execute as: Me` and `Who has access to the app: Anyone with Google account`. The service account is used for firing the Web Apps and the script of Google Apps Script is run as the owner of the Google Apps Script project (It's you.). You can see the figure of this as shown in the above figure.

For example, when the Web Apps deployed as `Execute as: Who has access` and `Who has access to the app: Anyone with Google account` is used, the service account cannot access the Web Apps.

Of course, it is considered that it is possible to use the Web Apps as the wrapper API of Google Apps Script API. But, in this case, the preparation process is a bit complicated. So, in this report, I proposed the above method.

### Usage

In this case, one Google Apps Script project as the server and a script as the client side are used.

#### 1. Create service account

In order to execute the Google Apps Script using the service account, please create your service account.

#### 2. Server-side

##### 2-1. Create a Google Apps Script project

On the server side, the Google Apps Script project is used. So, as a sample, please create a new Google Spreadsheet and open the script editor of Google Apps Script. This Google Apps Script project is used as the server side. Also, the functions in this project are executed by selecting the function.

Of course, you can use the standalone Google Apps Script project. But, in this case, in order to use the sample script, the container-bound script of Google Spreadsheet is used.

##### 2-2. Share Google Apps Script project with a service account

In this case, please share the Google Spreadsheet with the email of your service account.

##### 2-3. Prepare a sample script for Web Apps

This is used as a server. Please copy and paste the following script to the opened script editor and save the script.

```javascript
// Sample function. This is a sample user's function.
function setValue({ range, values }) {
  SpreadsheetApp.getActive().getRange(range).setValues(values);
  return "ok";
}

//
// Below functions are used for executing the script using Web Apps.
//
function process_(method, { parameter, postData }) {
  const lock = LockService.getScriptLock();
  if (lock.tryLock(350000)) {
    try {
      const { functionName } = parameter;
      let response = "No function.";
      if (functionName && !["doGet", "doPost"].includes(functionName)) {
        let args;
        if (method == "get") {
          const { arguments } = parameter;
          if (arguments) {
            args = JSON.parse(arguments);
          }
        } else if (method == "post" && postData) {
          const { contents } = postData;
          if (contents) {
            args = JSON.parse(contents);
          }
        }
        const res = this[functionName](args);
        response = typeof res == "object" ? JSON.stringify(res) : res;
      }
      return ContentService.createTextOutput(response);
    } catch ({ stack }) {
      return ContentService.createTextOutput(stack);
    } finally {
      lock.releaseLock();
    }
  } else {
    return ContentService.createTextOutput(Timeout);
  }
}

const doGet = (e) => process_("get", e);
const doPost = (e) => process_("post", e);

// This function is used for retrieving the Web Apps URL.
function getUrl() {
  console.log(ScriptApp.getService().getUrl());
}
```

In this script, the function of `setValue` is the sample script of the user's functions. Please add your script to this Google Apps Script project.

The functions `doGet`, `doPost`, `getUrl`, and `process_` are used for executing the user's functions.

##### 2-4. Deploy Web Apps

The detailed information can be seen in [the official document](https://developers.google.com/apps-script/guides/web#deploy_a_script_as_a_web_app).

1. On the script editor, at the top right of the script editor, please click "click Deploy" -> "New deployment".
2. Please click "Select type" -> "Web App".
3. Please input the information about the Web App in the fields under "Deployment configuration".
4. Please select **"Me"** for **"Execute as"**.
5. Please select **"Anyone with Google account"** for **"Who has access"**.
6. Please click "Deploy" button.
7. Please run the function `getUr()` of the above script. By this, you can see your Web Apps URL. It窶冱 like `https://script.google.com/macros/s/###/dev`. Please copy this URL. This URL is used on the client side. If the URL is not `dev`, please retrieve it on the script editor.

- In this case, the Web Apps is accessed as the developer mode with the endpoint of `https://script.google.com/macros/s/###/dev`. So, when you modify the Google Apps Script, it is not required to redeploy Web Apps. The latest script is always used when the Web Apps is accessed with the endpoint of `https://script.google.com/macros/s/###/dev`.

- As additional information, when `Execute as: Me` is changed to `Execute as: Anyone with Google account`, the service account cannot access the Web Apps. Please be careful about this.

By this flow, the setting of the server side was finished.

#### 3. Client-side

Here, as the sample clients, I would like to introduce the following 4 sample scripts. Please set your Web Apps URL retrieved by `getUr()` to the following scripts, and test it.

In order to access the Web Apps in the developer mode (`https://script.google.com/macros/s/###/dev`), the access token is used.

In the current stage, it seems that the access token can be used for the request header and the query parameter. [Ref](https://github.com/tanaikech/taking-advantage-of-Web-Apps-with-google-apps-script#how-to-use-dev-mode-from-outside) If you use the access token to the request header and an error occurs, please try to use it to the query parameter. For example, when the fetch API of Javascript is used, I have a case where an error occurs. At that time, the issue could be removed by using the access token to the query parameter instead of the request header.

##### 3-1. Google Apps Script

```javascript
// Ref: https://tanaikech.github.io/2018/12/07/retrieving-access-token-for-service-account-using-google-apps-script/
function getAccessToken_({ private_key, client_email, scopes }) {
  var url = "https://www.googleapis.com/oauth2/v4/token";
  var header = { alg: "RS256", typ: "JWT" };
  var now = Math.floor(Date.now() / 1000);
  var claim = {
    iss: client_email,
    scope: scopes.join(" "),
    aud: url,
    exp: (now + 3600).toString(),
    iat: now.toString(),
  };
  var signature =
    Utilities.base64Encode(JSON.stringify(header)) +
    "." +
    Utilities.base64Encode(JSON.stringify(claim));
  var jwt =
    signature +
    "." +
    Utilities.base64Encode(
      Utilities.computeRsaSha256Signature(signature, private_key)
    );
  var params = {
    payload: {
      assertion: jwt,
      grant_type: "urn:ietf:params:oauth:grant-type:jwt-bearer",
    },
  };
  var res = UrlFetchApp.fetch(url, params).getContentText();
  var { access_token } = JSON.parse(res);
  return access_token;
}

function sample() {
  const private_key =
    "-----BEGIN PRIVATE KEY-----\n###-----END PRIVATE KEY-----\n"; // Please set your value.
  const client_email = "###"; // Please set your value.

  const scopes = ["https://www.googleapis.com/auth/drive.readonly"];
  const accessToken = getAccessToken_({ private_key, client_email, scopes });

  const functionName = "setValue";
  const webAppsUrl = "https://script.google.com/macros/s/###/dev";

  // POST method
  const arguments1 = {
    range: "'Sheet1'!A1:A2",
    values: [["sample value 1"], ["sample value 2"]],
  };
  const res1 = UrlFetchApp.fetch(`${webAppsUrl}?functionName=${functionName}`, {
    headers: { Authorization: "Bearer " + accessToken },
    payload: JSON.stringify(arguments1),
  });
  console.log(res1.getContentText());

  // GET method
  const arguments2 = {
    range: "'Sheet1'!B1:B2",
    values: [["sample value 3"], ["sample value 4"]],
  };
  const res2 = UrlFetchApp.fetch(
    `${webAppsUrl}?functionName=${functionName}&arguments=${encodeURIComponent(
      JSON.stringify(arguments2)
    )}`,
    { headers: { Authorization: "Bearer " + accessToken } }
  );
  console.log(res2.getContentText());
}
```

##### 3-2. golang

```go
package main

import (
	"bytes"
	"encoding/json"
	"fmt"
	"io"
	"log"
	"net/http"
	"net/url"

	"github.com/tanaikech/go-gettokenbyserviceaccount"
)

// GET method
func getMethod(accessToken, webAppsURL string) {
	dataStr := struct {
		Range  string          `json:"range"`
		Values [][]interface{} `json:"values"`
	}{
		"'Sheet1'!B1:B2",
		[][]interface{}{{"sample value 3"}, {"sample value 4"}},
	}
	data, _ := json.Marshal(dataStr)
	u, err := url.Parse(webAppsURL)
	if err != nil {
		log.Fatal(err)
	}
	q := u.Query()
	q.Set("functionName", "setValue")
	q.Set("arguments", string(data))
	u.RawQuery = q.Encode()
	req, err := http.NewRequest("GET", u.String(), nil)
	if err != nil {
		log.Fatal(err)
	}
	req.Header.Set("Authorization", "Bearer "+accessToken)
	client := &http.Client{}
	res2, err := client.Do(req)
	if err != nil {
		log.Fatal(err)
	}
	defer res2.Body.Close()
	b, err := io.ReadAll(res2.Body)
	if err != nil {
		log.Fatal(err)
	}
	fmt.Println(string(b))
}

// POST method
func postMethod(accessToken, webAppsURL string) {
	url := webAppsURL + "?functionName=setValue"
	dataStr := struct {
		Range  string          `json:"range"`
		Values [][]interface{} `json:"values"`
	}{
		"'Sheet1'!A1:A2",
		[][]interface{}{{"sample value 1"}, {"sample value 2"}},
	}
	data, _ := json.Marshal(dataStr)
	req, err := http.NewRequest(
		"POST",
		url,
		bytes.NewReader(data),
	)
	if err != nil {
		log.Fatal(err)
	}
	req.Header.Set("Content-Type", "application/json")
	req.Header.Set("Authorization", "Bearer "+accessToken)
	client := &http.Client{}
	res2, err := client.Do(req)
	if err != nil {
		log.Fatal(err)
	}
	defer res2.Body.Close()
	b, err := io.ReadAll(res2.Body)
	if err != nil {
		log.Fatal(err)
	}
	fmt.Println(string(b))
}

func main() {
	privateKey := "-----BEGIN PRIVATE KEY-----\n###-----END PRIVATE KEY-----\n" // Please set your value.
	clientEmail := "###" // Please set your value.

	scopes := "https://www.googleapis.com/auth/drive.readonly"
	impersonateEmail := ""
	res1, err := gettokenbyserviceaccount.Do(privateKey, clientEmail, impersonateEmail, scopes)
	if err != nil {
		log.Fatal(err)
	}
	accessToken := res1.AccessToken
	webAppsURL := "https://script.google.com/macros/s/###/dev"
	postMethod(accessToken, webAppsURL)
	getMethod(accessToken, webAppsURL)
}
```

##### 3-3. Node.js

```javascript
const https = require("https");
const { google } = require("googleapis");

const auth = new google.auth.JWT({
  keyFile: "serviceAccount_credential.json", // Please set your value.

  scopes: ["https://www.googleapis.com/auth/drive.readonly"],
});

// POST method
function postMethod({ token, webAppsUrl, functionName }) {
  const arguments = {
    range: "'Sheet1'!A1:A2",
    values: [["sample value 1"], ["sample value 2"]],
  };
  const options = {
    method: "POST",
    headers: {
      "Content-Type": "application/json",
      Authorization: "Bearer " + token,
    },
  };
  const req1a = https.request(
    `${webAppsUrl}?functionName=${functionName}`,
    options,
    (res) => {
      const chunks = [];
      const req1b = https.request(res.headers["location"], (res2) => {
        res2
          .on("data", (chunk) => chunks.push(Buffer.from(chunk)))
          .on("end", () => console.log(Buffer.concat(chunks).toString()));
      });
      req1b.on("error", (err) => console.log(err));
      req1b.end();
    }
  );
  req1a.write(JSON.stringify(arguments));
  req1a.on("error", (err) => console.log(err));
  req1a.end();
}

// GET method
function getMethod({ token, webAppsUrl, functionName }) {
  const arguments = {
    range: "'Sheet1'!B1:B2",
    values: [["sample value 3"], ["sample value 4"]],
  };
  const options = {
    method: "GET",
    headers: { Authorization: "Bearer " + token },
  };
  const req2a = https.request(
    `${webAppsUrl}?functionName=${functionName}&arguments=${JSON.stringify(
      arguments
    )}`,
    options,
    (res) => {
      const chunks = [];
      const req2b = https.request(res.headers["location"], (res2) => {
        res2
          .on("data", (chunk) => chunks.push(Buffer.from(chunk)))
          .on("end", () => console.log(Buffer.concat(chunks).toString()));
      });
      req2b.on("error", (err) => console.log(err));
      req2b.end();
    }
  );
  req2a.on("error", (err) => console.log(err));
  req2a.end();
}

async function main() {
  const { token } = await auth.getAccessToken();
  const webAppsUrl = "https://script.google.com/macros/s/###/dev";
  const functionName = "setValue";
  postMethod({ token, webAppsUrl, functionName });
  getMethod({ token, webAppsUrl, functionName });
}

main();
```

##### 3-4. Python

```python
from google.oauth2 import service_account
import google.auth.transport.requests
import requests
import json


def getAccessToken():
    SERVICE_ACCOUNT_FILE = "serviceAccount_credential.json" # Please set your value.

    creds = service_account.Credentials.from_service_account_file(SERVICE_ACCOUNT_FILE, scopes=["https://www.googleapis.com/auth/drive.readonly"])
    creds.refresh(google.auth.transport.requests.Request())
    return creds.token

# Please set your Web Apps URL.
webApps_url = "https://script.google.com/macros/s/###/dev"

access_token = getAccessToken()

# POST method
functionName = "setValue"
arguments = {"range": "'Sheet1'!A1:A2", "values": [["sample value 1"], ["sample value 2"]]}
url = f'{webApps_url}?functionName={functionName}'
res = requests.post(url, json.dumps(arguments), headers={"Authorization": "Bearer " + access_token})
print(res.text)

# GET method
functionName = "setValue"
arguments = {"range": "'Sheet1'!B1:B2", "values": [["sample value 3"], ["sample value 4"]]}
url = f'{webApps_url}?functionName={functionName}&arguments={json.dumps(arguments)}'
res = requests.get(url, json.dumps(arguments), headers={"Authorization": "Bearer " + access_token})
print(res.text)
```

#### 4. Testing

All sample scripts obtain the same results.

- Values of "sample value 1", "sample value 2" are put into cells "A1:A2" by the POST method.
- Values of "sample value 3", "sample value 4" are put into cells "A1:A2" by the GET method.

---

# Code Examples

<a name="samplescriptofserverside"></a>

## Sample script of server side

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

---

<a name="samplescriptofclientside"></a>

## Sample scripts of client side

### Access with access token

#### GET

##### Curl

```bash
curl -GL \
  -H "Authorization: Bearer ### your access token ###" \
  -d "key1=value1" \
  -d "key2=value2" \
  "https://script.google.com/macros/s/#####/exec?key3=value3"
```

##### Google Apps Script

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

#### POST

##### Curl

```bash
curl -L \
  -H "Authorization: Bearer ### your access token ###" \
  -d "key1=value1" \
  -d "key2=value2" \
  "https://script.google.com/macros/s/#####/exec?key3=value3"
```

##### Google Apps Script

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

### Access without access token

#### GET

##### Curl

```bash
curl -GL \
  -d "key1=value1" \
  -d "key2=value2" \
  "https://script.google.com/macros/s/#####/exec?key3=value3"
```

##### Google Apps Script

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

#### POST

##### Curl

```bash
curl -L \
  -d "key1=value1" \
  -d "key2=value2" \
  "https://script.google.com/macros/s/#####/exec?key3=value3"
```

##### Google Apps Script

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

---

<a name="samplescriptvariouslanguages"></a>

# Sample scripts of client side by various languages

This is from my gist. [https://gist.github.com/tanaikech/a72aab0242012362c46ec69031c720d5](https://gist.github.com/tanaikech/a72aab0242012362c46ec69031c720d5)

These are the sample scripts by the various languages for requesting to [Web Apps](https://developers.google.com/apps-script/guides/web) created by Google Apps Script. Now, you can see the following 11 sample scripts.

1. [curl](#curl)
2. [Google Apps Script](#googleappsscript)
3. [Javascript](#javascript)
4. [ajax](#ajax)
5. [Node.js](#node)
6. [axios](#axios)
7. [angular](#angular)
8. [go](#go)
9. [python](#python)
10. [php](#php)
11. [powershell](#powershell)

## Sample script for Web Apps

- Sample script for Web Apps is as follows.

  ```javascript
  const doGet = (e) =>
    ContentService.createTextOutput(
      JSON.stringify({ method: "GET", eventObject: e })
    ).setMimeType(ContentService.MimeType.JSON);
  const doPost = (e) =>
    ContentService.createTextOutput(
      JSON.stringify({ method: "POST", eventObject: e })
    ).setMimeType(ContentService.MimeType.JSON);
  ```

- Setting for Web Apps is as follows.

  - **`Execute the app as: Me`**
  - **`Who has access to the app: Anyone`**
  - In this settings, no access token is required. When you use other settings, it might be required to use the access token. About this, please check [here](https://github.com/tanaikech/taking-advantage-of-Web-Apps-with-google-apps-script#5-situations-for-web-apps).

<a name="curl"></a>

### 1. curl

#### To doGet

```bash
$ curl -L "https://script.google.com/macros/s/#####/exec?key1=value1&key2=value2&key3=value3"
```

<a name="resultdoget"></a>

##### Result

```json
{
  "method": "GET",
  "eventObject": {
    "contentLength": -1,
    "parameters": {
      "key2": ["value2"],
      "key1": ["value1"],
      "key3": ["value3"]
    },
    "contextPath": "",
    "parameter": {
      "key1": "value1",
      "key3": "value3",
      "key2": "value2"
    },
    "queryString": "key1=value1&key2=value2&key3=value3"
  }
}
```

#### To doPost

```bash
$ curl -L -H "application/json" -d '{"key1": "value1", "key2": "value2"}' "https://script.google.com/macros/s/#####/exec?key3=value3"
```

<a name="resultdopost"></a>

##### Result

```json
{
  "method": "POST",
  "eventObject": {
    "contextPath": "",
    "postData": {
      "contents": "{\"key1\": \"value1\", \"key2\": \"value2\"}",
      "length": 36,
      "name": "postData",
      "type": "application/x-www-form-urlencoded"
    },
    "parameters": {
      "key3": ["value3"],
      "{\"key1\": \"value1\", \"key2\": \"value2\"}": [""]
    },
    "contentLength": 36,
    "queryString": "key3=value3",
    "parameter": {
      "{\"key1\": \"value1\", \"key2\": \"value2\"}": "",
      "key3": "value3"
    }
  }
}
```

- When you don't want to show `"{\"key1\": \"value1\", \"key2\": \"value2\"}": [""]` and `{\"key1\": \"value1\", \"key2\": \"value2\"}": ""`, please include the content type of `application/json`. But when this content type is added, there is the case that an error occurs by the language. So please be careful this.

<a name="googleappsscript"></a>

### 2. Google Apps Script

#### To doGet

```javascript
let url = "https://script.google.com/macros/s/###/exec";
url = `${url}?key1=value1&key2=value2&key3=value3`;
const res = UrlFetchApp.fetch(url);
console.log(res.getContentText());
```

##### Result

[Same as above.](#resultdoget)

#### To doPost

```javascript
let url = "https://script.google.com/macros/s/###/exec";
url = `${url}?key3=value3`;
const data = { key1: "value1", key2: "value2" };
const params = {
  method: "POST",
  payload: JSON.stringify(data),
};
const res = UrlFetchApp.fetch(url, params);
console.log(res.getContentText());
```

##### Result

[Same as above.](#resultdopost)

<a name="javascript"></a>

### 3. Javascript

#### To doGet

##### Using fetch

```javascript
let url = "https://script.google.com/macros/s/###/exec";
url = `${url}?key1=value1&key2=value2&key3=value3`;
fetch(url)
  .then((res) => res.json())
  .then((res) => console.log(res));
```

##### Using XMLHttpRequest

```javascript
let url = "https://script.google.com/macros/s/###/exec";
url = `${url}?key1=value1&key2=value2&key3=value3`;
const xhr = new XMLHttpRequest();
xhr.open("GET", url);
xhr.onload = () => console.log(xhr.responseText);
xhr.onerror = () => console.log(xhr.response);
xhr.send(null);
```

##### Result

[Same as above.](#resultdoget)

In above case, the result value can be retrieved as JSON object with `res.json()`.

#### To doPost using fetch

##### Using fetch

```javascript
let url = "https://script.google.com/macros/s/###/exec";
url = `${url}?key3=value3`;
const data = { key1: "value1", key2: "value2" };
fetch(url, { method: "POST", body: JSON.stringify(data) })
  .then((res) => res.json())
  .then((res) => console.log(res));
```

##### Using XMLHttpRequest

```javascript
let url = "https://script.google.com/macros/s/###/exec";
url = `${url}?key3=value3`;
const data = { key1: "value1", key2: "value2" };
var xhr = new XMLHttpRequest();
xhr.open("POST", url);
xhr.onload = () => console.log(xhr.responseText);
xhr.onerror = () => console.log(xhr.response);
xhr.send(JSON.stringify(data));
```

##### Result

```json
{
  "method": "POST",
  "eventObject": {
    "parameters": { "key3": ["value3"] },
    "parameter": { "key3": "value3" },
    "postData": {
      "contents": "{\"key1\":\"value1\",\"key2\":\"value2\"}",
      "length": 33,
      "name": "postData",
      "type": "text/plain"
    },
    "contextPath": "",
    "contentLength": 33,
    "queryString": "key3=value3"
  }
}
```

- In this case, the content type is `text/plain` and the properties without the value are not seen.

<a name="ajax"></a>

### 4. ajax

#### To doGet

```javascript
let url = "https://script.google.com/macros/s/###/exec";
url = `${url}?key1=value1&key2=value2&key3=value3`;
$.ajax({
  url: url,
  method: "GET",
  dataType: "json",
})
  .done((res) => console.log(res))
  .fail((err) => console.log(err));
```

##### Result

[Same as above.](#resultdoget)

#### To doPost

```javascript
let url = "https://script.google.com/macros/s/###/exec";
url = `${url}?key3=value3`;
const data = { key1: "value1", key2: "value2" };
$.ajax({
  url: url,
  method: "POST",
  dataType: "json",
  data: JSON.stringify(data),
})
  .done((res) => console.log(JSON.stringify(res)))
  .fail((err) => console.log(err));
```

##### Result

[Same as above.](#resultdopost)

<a name="node"></a>

### 5. Node.js

#### To doGet

```javascript
const request = require("request");
let url = "https://script.google.com/macros/s/###/exec";
url = `${url}?key1=value1&key2=value2&key3=value3`;
request(
  {
    method: "GET",
    url: url,
    json: true,
  },
  (err, res, body) => {
    if (err) {
      console.log(err);
      return;
    }
    console.log(body);
  }
);
```

##### Result

[Same as above.](#resultdoget)

#### To doPost

```javascript
let url = "https://script.google.com/macros/s/###/exec";
url = `${url}?key3=value3`;
const data = { key1: "value1", key2: "value2" };
request(
  {
    method: "POST",
    url: url,
    body: JSON.stringify(data),
    json: true,
    followAllRedirects: true,
  },
  (err, res, body) => {
    if (err) {
      console.log(err);
      return;
    }
    console.log(body);
  }
);
```

##### Result

```json
{
  "method": "POST",
  "eventObject": {
    "queryString": "key3=value3",
    "contextPath": "",
    "parameters": { "key3": ["value3"] },
    "postData": {
      "contents": "\"{\\\"key1\\\":\\\"value1\\\",\\\"key2\\\":\\\"value2\\\"}\"",
      "length": 43,
      "name": "postData",
      "type": "application/json"
    },
    "parameter": { "key3": "value3" },
    "contentLength": 43
  }
}
```

- In this case, `eventObject.postData.contents` can be retrieved by `JSON.parse(JSON.parse(body.eventObject.postData.contents))`. Please be careful this.

<a name="axios"></a>

### 6. axios

#### To doGet

```javascript
let url = "https://script.google.com/macros/s/###/exec";
url = `${url}?key1=value1&key2=value2&key3=value3`;
axios
  .get(url)
  .then((res) => console.log(res.data))
  .catch((err) => console.log(err));
```

##### Result

[Same as above.](#resultdoget)

#### To doPost

```javascript
let url = "https://script.google.com/macros/s/###/exec";
url = `${url}?key3=value3`;
const data = { key1: "value1", key2: "value2" };
axios
  .post(url, JSON.stringify(data))
  .then((res) => console.log(res.data))
  .catch((err) => console.log(err));
```

##### Result

[Same as above.](#resultdopost)

<a name="angular"></a>

### 7. angular

#### To doGet

```javascript
let url = "https://script.google.com/macros/s/###/exec";
url = `${url}?key1=value1&key2=value2&key3=value3`;
$http
  .get(url)
  .success((res) => console.log(res))
  .error((err) => console.log(err));
```

##### Result

[Same as above.](#resultdoget)

#### To doPost

```javascript
let url = "https://script.google.com/macros/s/###/exec";
url = `${url}?key3=value3`;
const data = { key1: "value1", key2: "value2" };
$http
  .post(url, JSON.stringify(data), {
    headers: { "Content-Type": "text/plain" },
  })
  .success((res) => console.log(res))
  .error((err) => console.log(err));
```

- In this case, when `"Content-Type": "text/plain"` is not used, the error related to CORS occurs. Please be careful this.
- Also, when `"Content-Type": "application/json"` is used, the error related to CORS occurs. Please be also careful this.

##### Result

```json
{
  "method": "POST",
  "eventObject": {
    "queryString": "key3=value3",
    "contentLength": 33,
    "contextPath": "",
    "parameters": { "key3": ["value3"] },
    "parameter": { "key3": "value3" },
    "postData": {
      "contents": "{\"key1\":\"value1\",\"key2\":\"value2\"}",
      "length": 33,
      "name": "postData",
      "type": "text/plain"
    }
  }
}
```

- In this case, the content type is `text/plain` and the properties without the value are not seen.

<a name="go"></a>

### 8. go

#### To doGet

```go
package main

import (
	"fmt"
	"io/ioutil"
	"log"
	"net/http"
)

func main() {
	url := "https://script.google.com/macros/s/###/exec"
	url += "?key1=value1&key2=value2&key3=value3"
	res, err := http.Get(url)
	if err != nil {
		log.Fatal(err)
	}
	b, err := ioutil.ReadAll(res.Body)
	if err != nil {
		log.Fatal(err)
	}
	fmt.Println(string(b))
    defer res.Body.Close()
}
```

##### Result

[Same as above.](#resultdoget)

#### To doPost

```go
package main

import (
	"fmt"
	"io/ioutil"
	"log"
	"net/http"
	"strings"
)

func main() {
	url := "https://script.google.com/macros/s/###/exec"
	url += "?key3=value3"
	dataStr := struct {
		Key1 string `json:"key1"`
		Key2 string `json:"key2"`
	}{
		"value1",
		"value2",
	}
	data, _ := json.Marshal(dataStr)
	res, err := http.Post(url, "application/json", bytes.NewReader(data))
	if err != nil {
		log.Fatal(err)
	}
	b, err := ioutil.ReadAll(res.Body)
	if err != nil {
		log.Fatal(err)
	}
	fmt.Println(string(b))
	defer res.Body.Close()
}
```

##### Result

```json
{
  "method": "POST",
  "eventObject": {
    "parameters": { "key3": ["value3"] },
    "contentLength": 33,
    "contextPath": "",
    "queryString": "key3=value3",
    "parameter": { "key3": "value3" },
    "postData": {
      "contents": "{\"key1\":\"value1\",\"key2\":\"value2\"}",
      "length": 33,
      "name": "postData",
      "type": "application/json"
    }
  }
}
```

- In this case, the content type is `application/json` and the properties without the value are not seen. By the way, the returned values from `application/json` and `text/plain` were the same.

<a name="python"></a>

### 9. python

#### To doGet

```python
import requests

url = "https://script.google.com/macros/s/###/exec"
url += "?key1=value1&key2=value2&key3=value3"
res = requests.get(url)
print(res.text)
```

##### Result

[Same as above.](#resultdoget)

#### To doPost

```python
import json
import requests

url = "https://script.google.com/macros/s/###/exec"
url += "?key3=value3"
data = {"key1": "value1", "key2": "value2"}
res = requests.post(url, json.dumps(data), headers={"Content-Type": "application/json"})
print(res.text)
```

##### Result

```json
{
  "method": "POST",
  "eventObject": {
    "contentLength": 36,
    "parameters": { "key3": ["value3"] },
    "parameter": { "key3": "value3" },
    "postData": {
      "contents": "{\"key2\": \"value2\", \"key1\": \"value1\"}",
      "length": 36,
      "name": "postData",
      "type": "application/json"
    },
    "queryString": "key3=value3",
    "contextPath": ""
  }
}
```

- In this case, when `"Content-Type": "application/json"` is not used, `postData` has no property of `contents`. Please be careful this.

<a name="php"></a>

### 10. php

#### To doGet

```php
$curl_handle = curl_init();
$url = 'https://script.google.com/macros/s/###/exec';
$url = $url.'?key1=value1&key2=value2&key3=value3';
curl_setopt($curl_handle, CURLOPT_URL, $url);
curl_setopt($curl_handle, CURLOPT_FOLLOWLOCATION, true);
$curl_data = curl_exec($curl_handle);
echo $curl_data;
```

##### Result

[Same as above.](#resultdoget)

#### To doPost

```php
$curl_handle = curl_init();
$url = 'https://script.google.com/macros/s/###/exec';
$url = $url.'?key3=value3';
$data = array(
    'key1' => 'value1',
    'key2' => 'value2'
);
curl_setopt($curl_handle, CURLOPT_URL, $url);
curl_setopt($curl_handle, CURLOPT_POST, TRUE);
curl_setopt($curl_handle, CURLOPT_FOLLOWLOCATION, true);
curl_setopt($curl_handle, CURLOPT_POSTFIELDS, json_encode($data));
$curl_data = curl_exec($curl_handle);
echo $curl_data;
```

##### Result

[Same as above.](#resultdopost)

<a name="powershell"></a>

### 11. powershell

#### To doGet

```powershell
$url = "https://script.google.com/macros/s/###/exec"
$url = "${url}?key1=value1&key2=value2&key3=value3"
$resp = Invoke-WebRequest -Uri $url -Method Get
$resp.Content
```

##### Result

[Same as above.](#resultdoget)

#### To doPost

```powershell
$url = "https://script.google.com/macros/s/###/exec"
$url = "${url}?key1=value1&key2=value2&key3=value3"
$json = @"
{key1: "value1", key2: "value2"}
"@
$resp = Invoke-WebRequest -Uri $url -Method Post -ContentType "application/json" -Body $json
$resp.Content
```

##### Result

```json
{
  "method": "POST",
  "eventObject": {
    "postData": {
      "contents": "{key1: \"value1\", key2: \"value2\"}",
      "length": 32,
      "name": "postData",
      "type": "application/json"
    },
    "parameters": { "key3": ["value3"] },
    "contextPath": "",
    "queryString": "key3=value3",
    "contentLength": 32,
    "parameter": { "key3": "value3" }
  }
}
```

- When `-ContentType "application/json"` is removed, the result is [the same as above.](#resultdopost)

---

<a name="fetchapiwithaccesstoken"></a>

## Request Web Apps using Fetch API of Javascript with the access token

In the current stage, when you want to request Web Apps with the access token, when the access token is including the request header, an error related to CORS occurs. Please be careful about this.

So, in this case, please include the access token in the query parameter. By this, the request can be worked. The sample script is as follows.

```javascript
const accessToken = "###"; // Please set your access token.
const url = "https://script.google.com/macros/s/###/exec?access_token=" + accessToken;
fetch(url)
  .then((res) => res.text())
  .then((res) => console.log(res));
```

By this modification, you can access Web Apps with the access token.

By the way, for example, when the curl command is used, the access token can be included in the request header as follows.

```bash
$ curl -L \
-H "Authorization: Bearer ###" \
"https://script.google.com/macros/s/###/exec"
```

And also, the following curl command can be used.

```bash
$ curl -L "https://script.google.com/macros/s/###/exec?access_token=###"
```

---

# Workarounds

<a name="workaround1"></a>

## 1. Reflecting Latest Script to Deployed Web Apps without Redeploying

This is from my gist. [https://gist.github.com/tanaikech/e46def22cf106b012dfa5ad359b93d24](https://gist.github.com/tanaikech/e46def22cf106b012dfa5ad359b93d24)

This report is a workaround for reflecting the latest Google Apps Script to the deployed Web Apps without redeploying.

### Pattern 1

Of course, when the developer mode of `https://script.google.com/macros/s/###/dev` is used, the latest script can be used without redeploying.

But, in this case, only the permitted users can use it using the access token. when you want to achieve this using the endpoint of `https://script.google.com/macros/s/###/exec` without the access token, in order to reflect the latest script to Web Apps, it is required to redeploy. As another pattern, I would like to introduce a workaround for this situation.

### Pattern 2

In this pattern, I would like to introduce a workaround that the latest script is reflected to the endpoint of `https://script.google.com/macros/s/###/exec`.

#### Usage

##### 1. Create 2 Google Apps Script projects

Please create the following 2 Google Apps Script projects.

1. For Web Apps. It supposes that the filename is "**project1**".

2. For Google Apps Script library. It supposes that the filename is "**project2**".

##### 2. Prepare sample script

Please copy and paste the following script to the script editor of "project1".

```javascript
const doGet = (e) => Lib.sample(e);
```

Please copy and paste the following script to the script editor of "project2".

```javascript
var sample = (obj) =>
  ContentService.createTextOutput(
    JSON.stringify({ obj, message: "Updated sample message." })
  );
```

##### 3. Deploy library

Please deploy "project2" as a library. [Ref](https://developers.google.com/apps-script/guides/libraries#create_and_share_a_library) Here, please copy the script ID. This script ID is the same as the library key for installation.

Please install the library to "project1" using the retrieved library key. [Ref](https://developers.google.com/apps-script/guides/libraries#add_a_library_to_your_script_project) In this case, please install the library as follows. The following JSON data is the manifest file (`appsscript.json`).

```json
{
  "timeZone": "### your timeZone ###",
  "dependencies": {
    "libraries": [
      {
        "userSymbol": "Lib",
        "version": "0",
        "libraryId": "### your library key ###",
        "developmentMode": true
      }
    ]
  },
  "exceptionLogging": "STACKDRIVER",
  "runtimeVersion": "V8",
  "webapp": {
    "executeAs": "USER_DEPLOYING",
    "access": "ANYONE_ANONYMOUS"
  }
}
```

##### 4. Deploy Web Apps

Please deploy Web Apps at "project1" as `Execute as: Me` and `Who has access to the app: Anyone`. [Ref](https://developers.google.com/apps-script/guides/web#deploy_a_script_as_a_web_app) And, please retrieve the Web Apps URL like `https://script.google.com/macros/s/###/exec`.

#### 5. Testing

As a test, a curl command is used. The sample curl command is as follows. Please use your Web Apps URL.

```bash
$ curl -L "https://script.google.com/macros/s/###/exec"
```

When this curl command is run, the following value is returned.

```json
{
  "obj": {
    "contextPath": "",
    "queryString": "",
    "parameters": {},
    "parameter": {},
    "contentLength": -1
  },
  "message": "Sample message."
}
```

Here, please modify "project2" as follows. "Sample message." was modified to "Updated sample message." Please save the script.

```javascript
var sample = (obj) =>
  ContentService.createTextOutput(
    JSON.stringify({ obj, message: "Updated sample message." })
  );
```

And, please run the above curl command again. By this, the following value is returned.

```json
{
  "obj": {
    "contentLength": -1,
    "parameters": {},
    "queryString": "",
    "contextPath": "",
    "parameter": {}
  },
  "message": "Updated sample message."
}
```

You can see the value of `"message":"Updated sample message."` is changed from `"message":"Sample message."`. By this, the latest script could be reflected in the Web Apps with the endpoint of `https://script.google.com/macros/s/###/exec`.

### Note

I posted this to the Japanese site on March 30, 2017. [Ref](https://qiita.com/tanaike/items/3087d67c5aaf4dede50a) But, I noticed that this has never been posted as my blog. So, I posted it here.

---

# Real-World Use Cases

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

<a name="applicationsforai"></a>

## Applications for AI

In May and June 2025, it was confirmed that Google Apps Script Web Apps can be effectively utilized for open protocols (MCP and A2A) in AI. The following reports provide more details:

### Model Context Protocol (MCP) Server

* [MCPApp](https://github.com/tanaikech/MCPApp): GitHub
  ```mermaid
  flowchart TD
      A[MCPClient] -->C{AI agent by Gemini}
      C -->|For managing Gmail| D[MCPServer1]
      C -->|For managing Google Calendar| E[MCPServer2]
  ```

* [Building Model Context Protocol (MCP) Server with Google Apps Script](https://medium.com/google-cloud/building-model-context-protocol-mcp-server-with-google-apps-script-9ff1fe58653c)
* [Image Transfer: MCP Server (Web Apps/Google Apps Script) to MCP Client (Gemini/Python)](https://medium.com/google-cloud/image-transfer-mcp-server-web-apps-google-apps-script-to-mcp-client-gemini-python-1fb22eb89dd2)
* [Gmail Processing using MCP Network Powered by Google Apps Script](https://medium.com/google-cloud/gmail-processing-using-mcp-network-powered-by-google-apps-script-5ede2a25c94e)

### Agent2Agent (A2A) Protocol Server

* [A2AApp](https://github.com/tanaikech/A2AApp): GitHub

  ```mermaid
  flowchart TD
    A[A2A client]
    A --> |Google Sheets Manager Agent| B[A2A server 1]
    A --> |Google Drive Manager Agent| C[A2A server 2]
    A --> |Google Calendar Manager Agent| D[A2A server 3]
    A --> |APIs Manager Agent| E[A2A server 4]
  ```

* [Building Agent2Agent (A2A) Server with Google Apps Script](https://medium.com/google-cloud/building-agent2agent-a2a-server-with-google-apps-script-d3efd32c7ca7)
* [Enabling Collaborative Agent Systems through Google Apps Script-based Agent2Agent (A2A) Network](https://medium.com/google-cloud/enabling-collaborative-agent-systems-through-google-apps-script-based-agent2agent-a2a-network-19e3d0472eaa)

### Both MCP and A2A Servers

* [Consolidating Generative AI Protocols: A Single Server Solution for MCP and A2A](https://medium.com/google-cloud/consolidating-generative-ai-protocols-a-single-server-solution-for-mcp-and-a2a-84e1e75ccbcf)

  ```mermaid
  flowchart TD
      subgraph Server
          direction TB
          functions[Functions] --> mvp[MCP server] & a2a[A2A server]
      end
      A[MCP Client] -- Protocol for MCP --> Server
      B[A2A Client] -- Protocol for A2A --> Server
  ```

---

<a name="samplesituations"></a>

## Sample situations

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
- [How to create webhook from Google Apps Script using the 窶彳xec窶・url rather than 窶彭ev窶・url. Exec returns 403 Error](https://stackoverflow.com/a/57426103)
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
- [Error when running Youtube Data Service in App Scripts (js) 窶・Daily Limit for Unauthenticated Use Exceeded](https://stackoverflow.com/q/62205747)
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
- [How to authenticate programmatically when running an Apps Script Web App from another Apps Script project](https://stackoverflow.com/q/64858395/7108653)
- [Using triggers with custom functions on Google Apps Script](https://stackoverflow.com/q/64897709/7108653)
- [how upload image & text input to same Row to google sheets](https://stackoverflow.com/q/65013982/7108653)
- [TextRotation angle is never set in Google Sheet API](https://stackoverflow.com/q/65229316/7108653)
- [Create one folder for all files uploaded with google apps script and web app form](https://stackoverflow.com/q/65292899/7108653)
- [Google sheets: is it possible to protect a range of cell such that it can only be modify with a script?](https://stackoverflow.com/q/66020192/7108653)
- [Uploading photo using Google Apps Script](https://stackoverflow.com/q/66053890/7108653)
- [How do I get/create a hyperlink in only part of a cell?](https://stackoverflow.com/q/66104825/7108653)
- [How do I get sorted results from the Google Photos search API?](https://stackoverflow.com/q/66354884/7108653)
- [Google Script HTML form from Library throws error Uncaught](https://stackoverflow.com/q/66711782/7108653)
- [Import XML content from a cell 窶・Google spreadsheets](https://stackoverflow.com/q/67053567/7108653)
- [How to update Chart Title via Google Sheet API (python)](https://stackoverflow.com/q/67377609/7108653)
- [How to serve (PDF,binary) file from GAS web to a browser?](https://stackoverflow.com/q/67545478/7108653)
- [How do I create an export to Google Sheet button on a web page?](https://stackoverflow.com/q/69465833/7108653)
- [How to get raw numbers from Google Sheets API](https://stackoverflow.com/q/69537680/7108653)
- [How to access data on different Google Spreadsheet through Google Apps Script for user without Editor access?](https://stackoverflow.com/q/69879044/7108653)
- [I need to make the end user able to run some functions without google apps script without giving him access to the code](https://stackoverflow.com/q/69967621/7108653)
- [Restrict Editors to Specific Ranges Script](https://stackoverflow.com/q/70342492/7108653)
- [How to POST JSON to a Google Apps Script with DEV Link](https://stackoverflow.com/q/70243094/7108653)
- [Creating an onEdit trigger on a sheet owned by a service account](https://stackoverflow.com/q/70367955/7108653)
- [Is there a way where I can minimize execution time of a google apps script through triggers?](https://stackoverflow.com/q/70415876/7108653)
- [If LockService works for a common script for multiple users, is there an option when dealing with multiple scripts sending data to the same column?](https://stackoverflow.com/q/71446597)
- [Convert Google doc to Word document with reduced oAuth scopes](https://stackoverflow.com/q/71556960)
- [How to use Web App approach when having functions run by editors on protected ranges, using Google Apps Scritps?](https://stackoverflow.com/q/71594546)
- [google apps script web app call differents functions](https://stackoverflow.com/q/71729091)
- [Multiple bulk import entries using Google Apps Script](https://stackoverflow.com/q/71821794)
- [How to add a status bar for file upload? (Google Drive/ Apps Script)](https://stackoverflow.com/q/71995596)
- [Inserting row into google sheet via external WebApp](https://stackoverflow.com/q/72088769)
- [Need to make simple read and update webapp using appscript](https://stackoverflow.com/q/72139734)
- [Can Google App Scripts create Web Apps with Multiple User Accounts?](https://stackoverflow.com/q/73179391)
- [How to add a list of hyperlinks in a CSV field to a cell in Google Sheets?](https://stackoverflow.com/q/73347973)
- [Dynamic hyperlink & Text in the same cell with a formula in google sheets](https://stackoverflow.com/q/73367517)
- [Getting JavaScript file from Google Sheets](https://stackoverflow.com/q/74274721)
- [How to track user clicks on Phaser web-game?](https://stackoverflow.com/q/74925713)
- [Exception: Access denied: DriveApp while running script from separate Google Account](https://stackoverflow.com/q/75044586)
- [Display google classroom data on the website using the app script](https://stackoverflow.com/q/75205210)
- [Unable to run apps script function via node.js api - Requested entity was not found](https://stackoverflow.com/q/76532430)
- [Python Google Slides API How to replace rectangle with an image?](https://stackoverflow.com/q/76548387)
- [Python chart scale in google sheet api](https://stackoverflow.com/q/77160660)
- [Calling Google Apps Script functions with Python: "Requested entity was not found"](https://stackoverflow.com/q/77256994)
- [How to duplicate (copy) a row keeping the protected ranges](https://stackoverflow.com/q/77345055)
- [How to automatic add timestamp to duplicated google sheet?](https://stackoverflow.com/q/77382145)
- [How to allow to run google script on a protected sheet (in a spreadsheet)](https://stackoverflow.com/q/77526757)
- [Create Gmail draft in other user account (GmailApp.createDraft)](https://stackoverflow.com/q/77615828)
- [Insert images in Google Sheets specific cell with gspread](https://stackoverflow.com/q/78423513)
- [google sheets v4 API. how to add error bars to column chart?](https://stackoverflow.com/q/78452563)
- [Google sheets cells jumped all over the place](https://stackoverflow.com/q/78601075)


---

# Appendices

<a name="references"></a>

# References

- [Web Apps](https://developers.google.com/apps-script/guides/web)
- [`google-apps-script` tag on Stackoverflow](https://stackoverflow.com/tags/gas-web-app/info)

---

<a name="licence"></a>

# Licence

[MIT](licence)

<a name="author"></a>

# Author

[Tanaike](https://tanaikech.github.io/about/)

<a name="updatehistory"></a>

# Update History

This repository was created on April 26, 2018. After this repository was created, the information has continued to grow.

- v1.0.0 (April 26, 2018)
  - Initial release.

- 2018 - 2025
  - Updated regularly.

- v2.0.0 (June 24, 2025)
  - Major updated.

I will continue to update this repository.

[TOP](#top)

