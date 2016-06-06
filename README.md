## IG Web API B2B Vendor Sample Application

### Overview
This sample application shows a basic OAuth 2.0 authorization flow with the IG authorization server in order to enable
IG B2B vendor clients to access the IG Web API on behalf of their clients.

### Getting started
Configure the application:

    application.properties
    common.js

You will need to set your B2B API key, your OAuth client id and secret and your server hostname and port if required.

Use maven 3 or later to build the project:

    mvn clean install
    
This will create a Spring Boot executable jar file. To start the web application:

    java -jar target\ig-webapi-vendor-sample-1.0.0-SNAPSHOT.jar

or alternatively deploy it on an existing web server. Open a web browser and enter the following url:

    http://<hostname>:<port>/api-vendor-sample/index.html
    
where **<hostname>** is the name of your host (**NOTE**: do not use "localhost").

### Description
The welcome page has a link

    GET /oauth2/authorize
to the IG authorization server which will prompt the user for their IG credentials and ask them to consent IG to provide
the vendor with an authorization code. The IG authorization server will then return an authorization code on a 403 redirect 
response (specified by the vendor on the "redirect_url" query parameter). The redirect url needs to match the url configured
on the authorization server for security reasons which in this sample is configured for 

    /api-vendor-sample/authorization-handler
    
The authorization handler will retrieve the authorization code from the redirect request's query parameters and then

1) Request access and refresh tokens    

    POST /oauth2/access_token        (grant_type = authorization_code)    
2) Obtain the user's client id         

    GET  /oauth2/userinfo    
3) Store the refresh token for future use
4) Redirect the user to the result page with the access token

    /api-vendor-sample/result.html

The result page presented to the user now has an access token which it will use to load client account details via the
IG Web API. It prompts the user for one of his account ids to as loading client account details currently requires an 
account id. This will fail after the access token expires.

The main application thread (see Application class) does some periodic background processing (every **80 second**) for all users for 
which it is holding a refresh token. It uses these refresh tokens in each iteration to obtain new access token and refresh token by calling:

        POST /oauth2/access_token     (grant_type = refresh_token)
        
The response holds a new access token with the expiry in seconds (currently **59 seconds**) and a new refresh token. Refresh tokens
currently expire after **10 minutes**. The background task then uses the access token to retrieve client account details via the IG Web API.