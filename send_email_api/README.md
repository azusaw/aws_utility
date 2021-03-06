
# about
Send an email when the contact form on the website submitted

# constitution
![SendEmail](https://user-images.githubusercontent.com/72424558/109382684-4d25b600-7925-11eb-9cbe-4a656bf6c71d.png)


# how to setup

## :pushpin: SES
#### Verify the email address

[Email Adresses]-[Verify a New Email Address]

Register the email address you want to send from.

## :pushpin: AWS Lambda
#### 1. Create Lambda function
Set a descriptive name. Like: sendContactEmail

Select 'Node.js 12.x' for runntime.

```javascript:index.js
'use strict'
const SDK = require('aws-sdk');

exports.handler = (event, context, callback) => {
	// set your region
    const ses = new SDK.SES({ region: 'us-west-2' });
    const email = {
    	// From
        Source: "no-reply@hoge.hoge",
        // To
        Destination: { ToAddresses: ["your-address@hoge.hoge"] },
        
        Message: {
            Subject: { Data: "New inquiry from the website" },
            Body: {
                Text: { Data: [
                    '[contact type] ' + event.contactType,
                    '[email] ' + event.email,
                    '[name] ' + event.name,
                    '[phone number] ' + event.phoneNumber,
                    '[company] ' + event.company,
                    '[department] ' + event.department,
                    '[details] : ' + event.details
                ].join("\n")}
            },
        }
    };
    ses.sendEmail(email, callback);
};

```

#### 2. Set API Gateway resource as the trigger
Add created API Gateway resource as the trigger.

(It may be set automatically when creating an API gateway..?)


## :pushpin: Amazon API Gateway
#### 1. Create the　REST API
Set a descriptive name. Like: sendContactEmail

#### 2. Create OPTIONS method
OPTIONS method is used pre-flight request.

:pencil: Setting [Method Response] as follow.

<strong>Response Headers for 200</strong>
|Name|
|---|
|Access-Control-Allow-Origin|

<b>Response Body for 200</b>
|Content type|Models|
|---|---|
|application/json|Empty|

#### 3. Create POST method
:pencil:Setting [Integration Request] as follow.

|Name|Value|
|---|---|
|Integration type|Lambda Function|
|Lambda Region|your region|
|Lambda Function|your Lambda function|

:pencil: Setting [Integration Response] as follow.

<strong>Header Mapping</strong>

|Response header|
|---|
|Access-Control-Allow-Origin|

<strong>Mapping Templates</strong>

|Content-Type|
|---|
|application/json|

<strong>Response Headers for 200</strong>

:pencil:Setting [Method Response] as follow.

|Name|
|---|
|Access-Control-Allow-Origin|

<strong>Response Body for 200</strong>
|Content type|Models|
|---|---|
|application/json|Empty|


# call by program
Vue example

```
send() {
      this.isSendButtonClicked = true;
      const formData = {
        page: 'contact',
        name: this.name,
        email: this.email,
        phoneNumber: this.phoneNumber,
        contactType: this.select,
        company: this.company,
        department: this.department,
        details: this.text
      }

      fetch('https://hogehoge.execute-api.your-region.amazonaws.com/v1/sendContactEmail',
        {
          method: 'POST',
          mode: 'cors',
          headers: {'Content-Type': 'application/json'},
          body: JSON.stringify(formData)
         })
        .then((response) => {
          this.$router.push('/contact-success');
        })
        .catch((error) => {
          this.$router.push('/contact-failure');
        });
    }
```
