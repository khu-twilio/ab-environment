# A/B environment 
A/B environment is a concept to run different version of code in production which links to the same Twilio Flex account. This example uses self-hosting technology on AWS S3 in order to implement this feature. 


## S3 Setup

1. Set up an AWS account 
2. Create an S3 bucket 
3. Create two folders under this S3 bucket, one folder for environment A; and the other folder for environment B. 
4. Upload index.html, pluginConfigDialpad.json, plugin-sample.js to folder A. 

## Plugin build file setup 
In order to load custom plugins, we need to store the plugin build file to an external URL. To generate this build file, go to the plugin directory and run 
twilio flex:plugins:build, and take the *.js file (for example, plugin-sample), upload to the S3 folder, copy/paste the S3 url. 

## Service function deployment 
Flex deployment often times include Twilio function deployment. There are multiple ways to maintain two versions of the function files with two different URLs. Here in this example we use two different service. 

Go to the Twilio function CLI folder directory and run "twilio serverless:deploy --service-name environmentalpha" and "twilio serverless:deploy --service-name environmentbeta" if you don't have existing services named "environmentalpha" and "environmentbeta". If these two environments already exist, go to "package.json" file under this directory and make sure the "name" here equals the name of the service you are going to deploy to, and then run "twilio serverless:deploy"

Copy paste the full URLs of these two deployed functions. 

## How it works

On the index.html, appConfig is sitting under the window global variables. Under pluginService, the url should point to a plugin configuration file which defines version and location of the plugin. 

<img width="1219" alt="pluginConfg" src="https://user-images.githubusercontent.com/82540340/164814272-9dd8c551-4df8-4d4e-a8ff-1b920ab13d07.png">

Specify the name and version (version could be defined by your own internal version number) and link to the place where it stores the plugin build file. On this json file you can configure multiple different plugins. Don't configure the same plugin multiple times under the same file, even though they might be on different version, to avoid conflict of loading plugins on UI. 

<img width="1007" alt="Screen Shot 2022-04-21 at 3 13 30 PM" src="https://user-images.githubusercontent.com/82540340/164560350-a7d7b34a-961e-49bb-98a4-2068f8abf43d.png">

ServiceConfiguration: {attributes: "V2"} under index.html appConfig is a self defined value, using this way you can store this configuration under window.appConfig and retrieve it from the plugin code. For instance, 

<img width="1002" alt="Screen Shot 2022-04-21 at 3 28 27 PM" src="https://user-images.githubusercontent.com/82540340/164561885-c95f2660-4a4c-4714-a0ed-de498c3f1ffa.png">

<img width="964" alt="Screen Shot 2022-04-21 at 3 28 38 PM" src="https://user-images.githubusercontent.com/82540340/164561888-7dee89da-74d4-40a4-b4d1-31cf5bcb2fcb.png">

This two base URL can be set using .env file, and they are the base URL of the two Twilio services after you deploy the functions. 

<img width="884" alt="Screen Shot 2022-04-21 at 3 31 33 PM" src="https://user-images.githubusercontent.com/82540340/164562172-3cec989b-045c-45ae-b8a3-e9c4781d149a.png">

## Swap A/B environment in production 
When swapping A/B environment in production, you can simply replace the URL with the URL from the other environment, and let the serviceConfiguration attribute set to the corresponding version under the other environment. 

## Gotchas 
####
a. "delete appConfig" before assigning to new value, since appConfig is set in window.appConfig and is a global variable, refreshing the page will not update the current config on the same browser, may cause issues when agents load the Flex page. 
b. putting the environment a and environment b folder into same S3 bucket. During testing the feature, setting two environment in two different buckets caused CORS issue even though CORS configuration was set correctly to the S3 bucket. 
c. make sure to set the service base URL in .env file, and handle the case when appConfig is undefined. 



