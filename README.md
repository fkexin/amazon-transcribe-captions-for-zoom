## Amazon Transcribe Close Captioning for Zoom Proof of Concept.

This PoC demonstrates how you can use Amazon Transcribe as a third party close captioning service for Zoom. The concept was initiated to find solution a for instructors to provide near real time close captions for traidtional style lectures taking place in Zoom. The foundation of this PoC is based of the [AWS Samples - Amazon Transcribe Websocket Static](https://github.com/aws-samples/amazon-transcribe-websocket-static) project. 


## Zoom Requirements
1) The Account admin has enabled the ability for users to enable close captions.

2) Users to enable the closed captioning for your own use:
    * Sign in to the Zoom web portal.
    * In the navigation panel, click Settings.
    * Click the Meeting tab.
    * Verify that Closed Caption is enabled.
    * If the setting is disabled, click the toggle to enable it. 
    * If a verification dialog displays, click Turn On to verify the change.
    * Enable Save Captions to allow participants to save fully closed captions or transcripts 

## Building and Deploying 

### Step 1. 
Create an IAM user with programmatic access take note of security credntials. Attach the TranscribeStreamingWebSockets Managed policy to the IAM user. 

```
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "transcribestreaming",
            "Effect": "Allow",
            "Action": "transcribe:StartStreamTranscriptionWebSocket",
            "Resource": "*"
        }
    ]
}
```

### Step 2.
This application requires an API Gateway endpoint and Lambda to Pre-Signed URL 

Run the cloudformation script:

2.1 create a s3 deployment bucket
```
aws s3api create-bucket --bucket {YOUR_BUCKET_NAME} --create-bucket-configuration LocationConstraint={YOUR REGION} --region {YOUR REGION} --profile  {YOUR PROFILE}

```

```
aws --profile  {YOUR_PROFILE}  cloudformation package --template-file /path_to_template/transcribe2zoom_cfn.yaml  --s3-bucket {YOUR_BUCKET_NAME}  --output-template-file output.yml 
```

2.2 Deploy the template package.

The paramater values for the following paramater keys must be set:<br/><br/>
IAM user credentials from Step 1:

ACCESSKEYID  
SECRET


and the region you want the transcriptions to occur in:

REGION

```
aws cloudformation deploy --template /path_to_template/output.yml --stack-name {YOUR_STACK_NAME} --parameter-overrides REGION={REGION_TO_RUN_TRANSCRIBE_IN} ACCESSKEYID={IAM ACCESS KEY ID STEP 1}  SECRET={IAM SECRET STEP 1}
```

## Step 3. Configure the App.

3.1 After deploying the CloudFormation Stack get the API Gateway Invoke URL that is required to communicate with the backend Lambda and create a Pre-Signed URL.

Goto the AWS Console -> API Gateway -> transcribe2zoom-api -> Stages ->

![alt text](images/aws_apigateway_invokeurl.png "API Gateway Endpoint")

3.2 Copy the API Gateway Invoke URL to the application config file config/config.json.tpl  :
```
    'api_gateway_endpoint': 'https://YOUR-API-URL.execute-api.ca-central-1.amazonaws.com/v1/presign',

```

3.3 The application requires a CORS proxy add a proxy end point to the config/config.json.tpl file:

```

    'cors_proxy_url': 'https://cors-anywhere.herokuapp.com/'

```
*for this PoC we are using the CORS Anywhere Heroku app but for production would recommend replacing with your own managed service.  


3.4 Rename the config.json.tpl to config.json 

```
 cp config.json.tpl config.json
```


## Step 4. Launch the Amplify App:

Since this app is essentially enitely a static front end application hosting via Amplify is a viable option. To do so create a *private Github (saying private just because you have your API endpoints in the repo which you may not want commited) repo and copy this code over. 

Replace your region and add your repo URL to the following example Amplify Deploy URL:

```
https://ca-central-1.console.aws.amazon.com/amplify/home?region=ca-central-1#/deploy?repo=https://github.com/YOUR-GITHUB-REPO
```
## Alternative how to test / run the front end client locally

Even though this is a static site there is a build step required. Some of the modules used were originally made for server-side code and do not work natively in the browser.

We use [browserify](https://github.com/browserify/browserify) to enable browser support for the JavaScript modules we `require()`.

1. Clone the repo
2. run `npm install`
3. run `npm run-script build` to generate `dist/main.js`.

Once you've bundled the JavaScript, all you need is a webserver. For example, from your project directory: 

```
npm install --global local-web-server
ws --log.format dev
```

## Use the Application. 

After meeting the Zoom requirements. 

Launching a new Zoom meeting on the meeting room task bar bar click on the Close Caption icon and "Copy the API token" this will be used in the AWS Real Time Transcribe app.


![alt text](images/zoom_enable_cc.png "Use a 3rd party CC service")

Next goto the Real-time Audio Transcription Close Captioning for Zoom URL

Paste the token and select language:
![alt text](images/zoom_paste_token.png "Paste Zoom token")


Hit the Start button when you are ready to start recording audio to be transcribed. This will prompt a browser response to grant access to your microphone click Allow.

![alt text](images/allow_microphone_access.png "Allow Microphone Access")

### Further Recommendations

This is a Proof of Concept it would be highly recommended to add an authentication layer to this if deploying on campus, one simple solution maybe be an Apache Webhost / Shibboleth and host the static files in the protected directory. 

### Credits

This project is largely based on code written by Karan Grover from the Amazon Transcribe team, who did the hard work (audio encoding, event stream marshalling) and modified for use with Zoom by the team at the UBC Cloud Innovation Centre. 

## License

This library is licensed under the Apache 2.0 License. 