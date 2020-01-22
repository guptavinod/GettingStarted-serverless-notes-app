# Serverless Notes Application

# Notes Application : COMPONENTS

There are 7 Components:
1. Amazon API Gateway
2. Amazon S3 bucket
3. Google Cloud Platform
4. Amazon Cognito - Identity Pool
5. DynamoDB
6. Serverless Backend app
7. Serverless Frontend app

# SETUP
1. Do Git Pull for following 2 Repositories : 
	- serverless-notes-backend (URL: https://github.com/guptavinod/serverless-notes-backend) 
	- serverless-notes-webapp (URL:  https://github.com/guptavinod/serverless-notes-webapp)

2. Open both the projects folders in VS Code Editor

3. Create Amazon S3 public accessible bucket (e.g. vg.notesapp) with the property enabled as **"Static website hosting"** and provide index.html as index document. This will provide you an **Endpoint** similar to below:
```
 http://vg.notesapp.s3-website-us-east-1.amazonaws.com
```

4. Go to the Google Cloud Platform and create an Web application. From API & Services section, generate credentials for created web application, this will provide you **client ID** and **Client Secret Id** which will be required to configure Google Authentication providers in Amazon Cognito Identity Pool (see the next step)  

5. Go to Amazon Cognito and create Identity Pool. This will create Identity Pool ID and few default roles (e.g. Unauthenticated role and Authenticated role)
	- Go to the **Authentication providers** section and click on Google tab --> under google client Id enter **client ID** generated in step 4.
	
	- **Identity Pool ID :** This Id need to be placed in serverless.yml file of project "serverless-notes-backend"
	```
	provider:
	  name: aws
	  runtime: nodejs10.x
	  region: us-east-1
	  stage: dev
	  memorySize: 128
	  timeout: 5
	  endpointType: regional
	  environment:
		NOTES_TABLE: ${self:service}-${opt:stage, self:provider.stage}
		**COGNITO_IDENTITY_POOL_ID: us-east-1:555cb161-e348-44d0-b6d7-1daad84f76a6**
	```
 
	- **Authenticated role : ** Go to the IAM dashboard and search for serverless role created during identity pool creation (e.g. Cognito_ServerlessNotesAppAuth_Role ). Select this Role and provide In-line policy for executing APIs
	under Amazon API Gateway
	
6. Go the VS Studio for project "serverless-notes-backend" and run following commands:
	```
	> npm install -g serverless
	> npm install --save-dev serverless-offline
	> serverless -v
	> serverless create -t aws-nodejs -p serverless-notes-backend
	> npm init -y
	> npm install --save aws-sdk moment underscore uuid
	> serverless deploy
	```
	
	- Command "serverless deploy" will do following activities for you over the AWS
	```
	Serverless: Packaging service...
	Serverless: Excluding development dependencies...
	Serverless: Creating Stack...
	Serverless: Checking Stack create progress...
	........
	Serverless: Stack create finished...
	Serverless: Uploading CloudFormation file to S3...
	Serverless: Uploading artifacts...
	Serverless: Uploading service sls-notes-backend.zip file to S3 (13.04 MB)...
	Serverless: Validating template...
	Serverless: Updating Stack...
	Serverless: Checking Stack update progress...
	................................................................................................................................................
	Serverless: Stack update finished...
	Service Information
	service: sls-notes-backend  
	stage: dev
	region: us-east-1
	stack: sls-notes-backend-dev
	resources: 49
	api keys:
	  None
	endpoints:
	  GET - https://uc12g17vdg.execute-api.us-east-1.amazonaws.com/dev/auth
	  POST - https://uc12g17vdg.execute-api.us-east-1.amazonaws.com/dev/note
	  PATCH - https://uc12g17vdg.execute-api.us-east-1.amazonaws.com/dev/note
	  GET - https://uc12g17vdg.execute-api.us-east-1.amazonaws.com/dev/notes
	  GET - https://uc12g17vdg.execute-api.us-east-1.amazonaws.com/dev/note/n/{note_id}
	  DELETE - https://uc12g17vdg.execute-api.us-east-1.amazonaws.com/dev/note/t/{timestamp}
	functions:
	  authenticate-google: sls-notes-backend-dev-authenticate-google
	  add-note: sls-notes-backend-dev-add-note
	  update-note: sls-notes-backend-dev-update-note
	  get-notes: sls-notes-backend-dev-get-notes
	  get-note: sls-notes-backend-dev-get-note
	  delete-note: sls-notes-backend-dev-delete-note
	layers:
	  None
	Serverless: Run the "serverless" command to setup monitoring, troubleshooting and testing.
	```
	
7. Now go to the Amazon API Gateway, you can see all the functions deployed there and would be able to see their endpoints as well. Click on the left pane **Stages** --> click on specific stage (i.e dev) --> You will see the **Invoke URL** (i.e. https://uc12g17vdg.execute-api.us-east-1.amazonaws.com/dev)

8. We need to set Invoke URL path to variable **API_ROOT** within webpack.config.prod.js file of project serverless-notes-webapp. Now run following command in VS Code terminal:
	```
	> npm install express
	> npm run build:prod
	> npm start (To test application locally)
	```
	- This will build website code under folder **public**. Deploy this folder into S3 bucket i.e vg.notesapp (created in step 3)
	
9. Now, you can access the Notes application using ** S3 - Static website hosting"** URL:
	```
	http://vg.notesapp.s3-website-us-east-1.amazonaws.com
	```
	
	- You would be able to see UI(s)
	
	![Notes App - SignIn page](images/NotesApp-SignIn.png)	

10. User Authentication Workflow:

From Notes Frontend app --> within **login.component.html** --> onSignin() --> login.component.ts --> this.authService.login() --> this.setCredentials(id_token); --> This makes a call to Notes Backend app **auth Function** 
e.g. API_ROOT + STAGE + '/auth' 
.
	![Amazon-Cognito-(Federated Identities)-Authentication Flow](images/awscognito-authentication-flow.png)

## Sample AWS Credentials Response from notes backend app (auth lambda function) 
```
	**id_token:**
	eyJhbGciOiJSUzI1NiIsImtpZCI6ImJhZDM5NzU0ZGYzYjI0M2YwNDI4YmU5YzUzNjFkYmE1YjEwZmZjYzAiLCJ0eXAiOiJKV1QifQ.eyJpc3MiOiJhY2NvdW50cy5nb29nbGUuY29tIiwiYXpwIjoiMjkyMjM0NjAyNzQ5LTVyY2ZnaWxwa2ZmdjI5MDk0ZW81N2JpdmVnbGFwMWpzLmFwcHMuZ29vZ2xldXNlcmNvbnRlbnQuY29tIiwiYXVkIjoiMjkyMjM0NjAyNzQ5LTVyY2ZnaWxwa2ZmdjI5MDk0ZW81N2JpdmVnbGFwMWpzLmFwcHMuZ29vZ2xldXNlcmNvbnRlbnQuY29tIiwic3ViIjoiMTEzOTc4MzU2Mzg1NTYwNzM4MDM2IiwiZW1haWwiOiJtb29uZXNoa0BnbWFpbC5jb20iLCJlbWFpbF92ZXJpZmllZCI6dHJ1ZSwiYXRfaGFzaCI6IlV3SmRCM01fckhWdnlCR2VBOXhvREEiLCJuYW1lIjoiTW9vbmVzaCBLYWNocm9vIiwicGljdHVyZSI6Imh0dHBzOi8vbGgzLmdvb2dsZXVzZXJjb250ZW50LmNvbS9hLS9BQXVFN21EWWdYb2VrMHRtR09SOFpWSUVzYVZpcXM2QXR1dHRaUFhfRnFkMWNRPXM5Ni1jIiwiZ2l2ZW5fbmFtZSI6Ik1vb25lc2giLCJmYW1pbHlfbmFtZSI6IkthY2hyb28iLCJsb2NhbGUiOiJlbiIsImlhdCI6MTU3OTYwNDE5NCwiZXhwIjoxNTc5NjA3Nzk0LCJqdGkiOiJmYTEzYzY4ZTBjZjQ1OTE5MWJjZTgyMjhmNDAyNjE4OTM2MmYzMzcxIn0.cwUz_hUbJWFL6ULSma1uqHzXPmh4BeeyrUQhUofhAEwo31yFLDqDuU6Zz9uaL7vYf35o9kvsyLIiILXnltVdmts5Y7tCOrhk6XnHJybzSpDED0F2JEKoFPf2Lz8FM8bdi26aLgxXIzmuwp-DuM06CRtmnhe4mdblwHmViHrFUZYvCCxlVU0Z891-Z5jY_xocORk6tejQQ5pZXq2CY2toM3UUPDklwm1vaHUBtWheWMxR1_AswJdugS5i0YMNqdwBfj0afltBr4BEaPqz-GGbn4htXQI1AkRlLTKYfqP7saRSTC9CT0nA-TuMPqYOSENEzr9dUDfrL4l8HVf_4otJEg


	**AWS CREDENTIALS:**
	```
	IdentityId: "us-east-1:20sasac60178-fe95-321`-a490-133c2c0909a2b"

	Credentials: {AccessKeyId: "ASIATDK2FWGZO22CBE2M", SecretKey: "CebTnjFUDAmgjOgvmdqIDULVLsn6pBFHLDXbDszw",…}

	AccessKeyId: "ASIATDK2FWGZO22CBE2M"

	SecretKey: "CebTnjFUDAmgjOgvmdqIDULVLsn6pBFHLDXbDszw"

	SessionToken: "IQoJb3JpZ2luX2VjEHsaCXVzLWVhc3QtMSJHMEUCIFNGpcIhc6Q/fCujd+peMODCKFjas1XaZSpaAcZSXYjUAiEA+NIHIgcjRjZRrlgsPxZ4OSXaHEMhA6dffb0cxTODPmUq4gMIFBAAGgwyMTMzMjY4Njg5MTQiDHicIUNhrpZry+uNcyq/A+ak7ti95yQ+atgYJNE6SfOiln7luoJovv52mgwTfahLnMLCii7KkqQpIbCyoFR6rSsnAiLJ9OiM0yoWHG8cZcK1vXhFlq7JSAZe66gR/96pcBoBSxlaU+7ClayNVQpP2614xTSrmr64ZybhfC70J0PQ0/C6xAltawI7pSKgGCmXfsEEChV5DFE8hSHp6AT/LF5T6dmzAo8S/MBmFNOiMTzvrLit/L5tpLE9hIgWviP4zzWdMlOqDp7S67zc7yCnJsUC+m3rW92iklYn8D5ptyaXtLqIp//fWNvndv0SbQcY3wdOPPy2WsC2JhMdiNxqWj42l8NgS3QV80CbEw1PEgzDRZUZhQmkII4PawvVkbglUlXb70lPbJuJPaT7VY2cPm66rbGuhWS+3zz3JILi3SntfrB5RkpEj6WNnFgRqQjQu107mfElY4TZVWnUUdbheegH2hd+06Xtv5JtN7/8mUcqZHLDaLVsdh5T6N8BmeVhHQsiHoGckSanvq64B686smVa7LTDiQ0RQbSHSTAngjHRIpzGCgcB+uR1aHT24/iW5tOLou+Qerc71+ZLWev/tQ2wEyvAEhCeihTdXJtUszCWsJvxBTrLAkmHpsI6TnaRxJOspVI/dWehb4hhb+soR432ilG+DvlI7lNC82JF04AvRPhkQnT8XJGcPRLN0cWO59OzcmZexty66VY2NoYsmlCGji/71LrzqPCpiEF9VCZv+Ahlh/Crn9lmUWnSdanpjc1JLNKpHa3XqnxHQwekl0ho/jHzDuKZ0qDD0vZy0pMp47hqQ8jE/oUBGa2Ku5CZPNqwfxwKPc41YRdU+frrbJHcRUk6ZhPUmSlD13fgJAXO8bTRyRsaKXO/1eLXQsF2MBor6M2XzHJ6BNZNQVICYfwLN1rd4PtIcrLh2DsMPg2KAUW7df05MUUwQUjhMQzfpVwNZfPgLEuzzTAtHGvA0Kxfq827iC+lnK+cexEf0K9U8Ya3RFwAWFi0xe5bZ6xcoGsCyX/oz+EEBzvhLAIXd76eNnmtSexN+fkMFHOV+cQ07+E="

	Expiration: "2020-01-21T11:53:10.000Z"

	user_name: "Vinod Gupta"
```
![Notes App - Add page](images/NotesApp-add.png)
![Notes App - Listing page](images/NotesApp-listing.png)
	
10. **Clean UP Resources** - Go the VS Studio for project "serverless-notes-backend" and run following commands:
	```	
	> serverless remove
	```
   
	```
	Serverless: Getting all objects in S3 bucket...
	Serverless: Removing objects in S3 bucket...
	Serverless: Removing Stack...
	Serverless: Checking Stack removal progress...
	...........................................................................................
	Serverless: Stack removal finished...
	```

11. **STEPS TO RUN THIS Notes backedn APP in OFF-LINE MODE
- Within folder "sls-yml-files", there are 3 files:
	- serverless-offline-db.yml
	- serverless-offline-main.yml
	- serverless-online-main.yml
- First create dynamodb table using file "serverless-offline-db.yml"
Remove serverless.yml file (if already exists) and then copy this file to root and rename it to serverless.yml file and run following command:

```
> serverless deploy
```
This will create table in aws dynamo db

- Now copy following file "serverless-offline-main.yml" on root and rename it to serverless.yml. Then Run following command:
```
> serverless offline
``` 
**NOTE.** Now you will get API(s) to test within POSTMAN locally on localhost:3000

- After local testing done, you can delete this file and copy 3rd file "serverless-online-main.yml" and rename it to serverless.yml
and run following command:
```
> serverless deploy
```
**NOTE** This will create all the resources in AWS environment for accessing notes app over the cloud.. 

