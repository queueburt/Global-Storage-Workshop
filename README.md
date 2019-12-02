---
layout: default
title: Architecting global storage with AWS Lambda and Storage Gateway
---
## Lab Guide

Follow along with your instructor to get logged into the AWS Console from [https://dashboard.eventengine.run](https://dashboard.eventengine.run/).  Once you’re all set, follow along with the lab guide below.

For the sake of minimizing confusion, this lab assumes we’re deploying all of our resources into the ‘**us-west-2**’ region.  Your console should be set for the correct region automatically, but the very first thing we should do is navigate to the region scope in the upper-right between your account info and the “Support” tab and verify it’s set to “**US West (Oregon)**".

![Image](media/Picture1.png) ![Image](media/Picture2.png)

Networking has already been set up in your lab environment, so the first thing we’ll do is create the appropriate storage buckets.

Select “**Services**” and either navigate to “**Storage – S3**” or type “**s3**” into the search bar and select “**S3**”.

![Image](media/Picture3.png)

Click the “**Create Bucket**” button.

![Image](media/Picture4.png)

For this demo we’ll be creating (2) different buckets, one of which will be a “*master*” bucket that contains an authoritative copy of our assets.  The other will be a “*studio*” bucket which will contain assets assigned to a specific studio.

Under “**Bucket name**” enter any name you want.  Note that bucket names are globally unique, so your initial choices might not be available.  This initial bucket will be the “*master*” bucket, so a good name choice might be something like “**yourinitials-mymedia-master**”.

Ensure the region is “**US West (Oregon)**” and press “**Next**”.

![Image](media/Picture5.png)

Select the box underneath “**Versioning**”, and also the one under “**Default encryption**” (security is paramount!).

![Image](media/Picture6.png)

Leave all other options at their defaults, click “**Next**” twice more, and finally click “**Create bucket**”.

Click the “**Create Bucket**” button again and repeat the same process, only this time naming the bucket for the “*studio*” that will be using it.  An example might be “**yourinitials-mymedia-studio**” to mirror our previous naming convention.  Make note of your bucket names, as we’ll be using them later!

Select “**Services**” and either navigate to “**Storage – Storage Gateway**” or type “**storage gateway**” into the search bar and select “**Storage Gateway**”.

![Image](media/Picture7.png)

Click “**Get started**”, choose “**File gateway**”, and click “**Next**”.

Under “**Select host platform**” choose “**Amazon EC2**”, expand the arrow next to “**Set up instructions for Amazon EC2**”, and follow the directions. Specifically (screenshots located below instructions):

* Click “**Launch instance**”
* Choose “**m5.xlarge**” as your instance type
* Click “**Next: Configure Instance Details**”
* Ensure the “**Network**” is set to your Media VPC **(it won’t be by default, so make sure to set it properly)**.
* Ensure “**Auto-assign Public IP**” is set to “Enable”
* Click “**Next: Add Storage**”
* Click “**Add New Volume**” and set the “**Size (****GiB****)**” to at least **_150 _****_GiB_** and select “**Delete on Termination**”. **(_WARNING!  Make certain you’ve created a SECOND volume and your screenshot looks like the one below.  You won’t be able to proceed if you just set the existing Root volume to 150 GB!_)**
* Click “**Next**” twice until you reach “**Configure Security Group**”
* Change “**Security group name**” to “**SGW**”
* Leave the SSH rule in place and click “**Add Rule**” and add the following rules:
    * “**SMB**” to “**Anywhere**”
    * “**HTTP**” to “**Anywhere**”
    * “**HTTPS**” to “**Anywhere**”
* Click “**Review and Launch**” and then “**Launch**”
* _**Did you download your key pair from Event Engine before logging into your AWS Console?  Feel free to skip the next step and launch the instances with the existing key pair.  If you didn’t, follow the step below and create a new one.**_
* From the drop-down, choose “**Create a new key pair**”
* Name it whatever you choose and click “**Download Key Pair**”
* Remember the download location of the Key Pair file and click “**Launch Instances**”
* Click “**View Instances**” and wait for your new instance to transition to “**running**”

![Image](media/Picture8.png)![Image](media/Picture9.png)![Image](media/Picture10.png)![Image](media/Picture11.png)![Image](media/Picture12.png)![Image](media/Picture13.png)

Make note of the “**IPv4 Public IP**” on the new instance (Handy tip: Select the instance and use the copy button next to “**IPv4 Public IP**” on the lower panel).

![Image](media/Picture14.png)

Return to the “**AWS Storage Gateway**” tab and click “**Next**” twice (ensuring “**Public**” is selected as the endpoint type).

Enter the IP Address of the instance you just deployed and click “**Connect to gateway**”.  Note that it may take a few minutes even after the instance status is “Running” before the connection process will work.  Just click “back” in your browser and re-enter the public IP.

Set the appropriate time zone and name your gateway whatever you’d like.  To match our hypothetical naming conventions, you might choose something like “*mymedia-sgw*”.

![Image](media/Picture15.png)

Click “**Activate gateway**”.

We’ll be setting the extra 150 GiB disk we created as a “cache” that will be used to store the most commonly requested assets dynamically on the gateway.  In an on-prem scenario, Storage Gateway can be installed in your existing environments and used to provide a local low-latency storage mechanism to on-prem clients.  The Gateway will take approximately 2 minutes to prepare the disks, but once it’s complete ensure that the 150 GiB disk is set to “**Cache**”, and click “**Configure logging**” and then “**Save and continue**”.

****Don’t see the 150 GB volume?  The likely reason is that you didn’t add a second volume during the setup of the file gateway instance.  This is correctable, so reach out to a support person to help.****

![Image](media/Picture16.png)

Select your new Gateway and choose “**Actions – Edit SMB settings**”.

![Image](media/Picture17.png)

Click the pencil next to “**Set guest password**”, create a guest password of your choice, and click “**Save**” and then “**Close**”.

![Image](media/Picture18.png)

Select “**File shares**” from the menu on the left side of the screen.

![Image](media/Picture19.png)

Click “**Create file share**”.

![Image](media/Picture20.png)

Next to “**Amazon S3 bucket name**” type the name of your studio bucket.  In our example it would be “*yourinitials-mymedia**-studio*”.  Then select “**Server Message Block (SMB)**” as your access type.  Click “**Next**”.

![Image](media/Picture21.png)

Leave the defaults and click “**Next**”.

Select “**Edit**” above the red box next to “**SMB share settings**”.

From the “**Select authentication method**” drop-down, select “**Guest access**” and choose “**Close**” on the right side.

![Image](media/Picture22.png)

Click “**Create file share**”.

Highlight your file share and draw your attention to the information pane.  Make note of both your “**File share ID**” as well as the command at the bottom next to “**On Microsoft Windows**”, as we’ll be using them both later.

Next we’ll be creating a Windows instance to act as our editing workstation.  In this lab we’ll be using an EC2 instance, but this could be an on-prem workstation or other virtualized studio asset.

From the AWS Console Dashboard, select “**Services**” and either navigate to “**Compute – EC2**” or type “**EC2**” into the search bar and select “**EC2**”.

![Image](media/Picture23.png)

Click the “**Launch Instance**” button and follow the directions below:

![Image](media/Picture24.png)

* Click “**Select**” next to “**Microsoft Windows Server 2019 Base**”
* Choose “**m****5.large**” as your instance type
* Click “**Next: Configure Instance Details**”
* Ensure the “**Network**” is set to your Media VPC **(again, it won’t be by default so ensure you select it)**.
* Ensure “**Auto-assign Public IP**” is set to “**Enable**”
* Click “**Next**” twice until you reach “**Configure Security Group**”
* Change “**Security group name**” to “**windows**” and leave the defaults
* Click “**Review and Launch**” and then “**Launch**”
* Ensure your key pair is set to whichever one you’re using from the earlier step, check the “**Acknowledge**” box, and finally “**Launch instances**”
* Click “**View Instances**” and wait for your new instance to transition to “**running**”

Windows instances take slightly longer to configure, so we'll have to wait approximately 2-4 minutes to proceed.  Our eventual objective will be to highlight our new instance and choose “**Actions — Get Windows Password**”.  If you do so before 2-4 minutes passes, however, you're likely to see an error that the password isn't available yet.  Fear not!  It'll be there soon!

![Image](media/Picture25.png)

Once you're able to see the box requesting your key pair, click “**Choose File**” and navigate to the key pair file you downloaded earlier.  Once you select and open it, you should see your key pair populate into the text box starting with “**-----BEGIN RSA PRIVATE KEY-----**”.  Click “**Decrypt Password**”.

![Image](media/Picture26.png)

**LEAVE THIS WINDOW OPEN UNTIL YOU FINISH THE NEXT STEP!**  Copy the “**Username**”, **Password**”, and “**Public DNS**” fields shown, and place them into a text file or other note document so you can readily access them at will.  Once you're confident you won't lose them, feel free to click “**Close**”.

Depending on your platform and preferences, you have a few different choices to access your new Windows workstation.  If you're on a Windows system, you should either have native support for Remote Desktop via the Windows RDP Client accessible by clicking “Start” and searching for “Remote Desktop”.  If you're on OS X, you can navigate to the Mac App Store and install the Microsoft Remote Desktop 10 Client located at “[https://itunes.apple.com/us/app/microsoft-remote-desktop-10/id1295203466”.](https://itunes.apple.com/us/app/microsoft-remote-desktop-10/id1295203466)

![Image](media/Picture27.png)

Open your software client of choice and enter either the Public DNS or Public IP address of the Windows instance you just created and connect.  The clients will likely prompt you to alert you the certificate is untrusted, but you may proceed anyway.  You'll be prompted for a username and password.  The username is “**Administrator**” and the password is the one you copied from the previous window.  Enter those and connect.

There's a very high likelihood you'll want to change that default administrator password, so you can click the “**Start**” menu, click the icon showing a person, and choose “**Change account settings**”.  Select “**Sign-in options**” and you'll see an option to change your password.  You'll need to enter the AWS default password one more time, but you can then change it to anything you choose (Password Hint is required.  )-: ).

![Image](media/Picture28.png)![Image](media/Picture29.png)

Click the Start Menu and type “**cmd**”.  Click on the “**Command Prompt**” application that appears.

![Image](media/Picture30.png)

If you have the command available from Storage Gateway File Share screen readily available, copy it and paste it into the command prompt, but don't press Enter quite yet.  If you don't have it handy, navigate back to the AWS Console and select from the “**Services**” menu “**Storage – Storage Gateway**” or type “**storage gateway**” into the search bar and select “**Storage Gateway**”.  From there click “**File shares**”, highlight your share, and you should see the command we're looking for.

Navigate back to your Windows workstation, return to the command prompt, and change the “**[WindowsDriveLetter]**” entry to a drive letter of your choice.  I personally like “Q:”.  Your command should look something like “**net use Q: \\10.138.x.x\mymedia-studio /user:sgw-5555555\smbguest**”.

Hit Enter and you should be prompted for a password.  Enter the password you created on the Storage Gateway earlier and hit Enter.  The terminal should tell you your command completed successfully.

![Image](media/Picture31.png)

In your Windows workstation select the “**Windows Explorer**” icon next to the Start Menu.

![Image](media/Picture32.png)

On the left side, click “**This PC**”.  You should see your newly mapped drive and it should be accessible, albeit empty.

Return to the AWS Console, select “**Services**” and either navigate to “**Compute – Lambda**” or type “**lambda**” into the search bar and select “**Lambda**”.

![Image](media/Picture33.png)

Click “**Create a function**”.

Leave “**Author from scratch**” selected, and under “**Function name**” type the name “**copy-assets**”.

Open the “**Runtime**” drop-down and select “**Python 3.8**”.

Extend the “**Choose or create role**” arrow, select “**Create a new role with basic Lambda permissions**” from “**Execution role**”, and click “**Create function**”.

![Image](media/Picture34.png)

In the “**Function code**” panel, select “**Upload a file from Amazon S3**” from the “**Code entry type**” drop-down.

Change the text in the “**Handler**” box to “**main.copy**”.

Enter “**s3://mediademo-public/copy-assets-function.zip**” in the “**Amazon S3 link URL**” box.

![Image](media/Picture35.png)

Further down the page in the “**Basic settings**” panel, change the “**Timeout**” to **5 minutes** and adjust the “**Memory**” slider to **256 MB**.

![Image](media/Picture36.png)

Click “**Save**” in the upper-right.  You should see the code populate into the inline editor allowing you to explore the basic Python that's going to execute when our function is called.

![Image](media/Picture37.png)

Select “**Services**” in the upper-left and either navigate to “**Security, Identity, and Compliance – IAM**” or type “**iam**” into the search bar and select “**IAM**”.

![Image](media/Picture38.png)

Select “**Roles**” from the menu on the left side.

![Image](media/Picture39.png)

Find the role that begins with “**copy-assets-role**” in the list and click it.

Click the “**Attach policies**” button.

![Image](media/Picture40.png)

In the “**Filter**” box, type “**s3full**” and select the checkbox next to “**AmazonS3FullAccess**” (but don't click “**Attach policy**” yet).

![Image](media/Picture41.png)

In the “**Filter**” box, change “**s3full**” to “**gatewayfull**” and select the checkbox next to “**AWSStorageGatewayFullAccess**”.

![Image](media/Picture42.png)

Click “**Attach policy**”.  Back on the summary screen you should see both new policies attached to your role.

![Image](media/Picture43.png)

Select “**Services**” in the upper-left and either navigate to “**Networking & Content Delivery – API Gateway**” or type “**api**” into the search bar and select “**API Gateway**”.

![Image](media/Picture44.png)

Click “**Get Started**”.

Ensure “**REST**” is selected and choose “**New API**”.

Name the API “**Media API**” and ensure it's set to Endpoint Type “**Regional**”.

Click “**Create API**”.

![Image](media/Picture45.png)

Select “**Actions**” and click “**Create Resource**”.

![Image](media/Picture46.png)

Name the resource “**copy-assets**”, select “**Enable API Gateway CORS**", leave everything else as default, and click “**Create Resource**”.

![Image](media/Picture47.png)

Ensure “**copy-assets**” is highlighted, select “**Actions**”, and click “**Create Method**”.

From the drop-down select “**POST**” and click the check-mark next to it.

![Image](media/Picture48.png)

In the subsequent “**POST - Setup**” screen, click the check-box next to “**Use Lambda Proxy Integration**”.

In the “**Lambda Function**” box enter the name of the Lambda function you created.  It should be “**copy-assets**” unless you happened to change the name.

![Image](media/Picture49.png)

Click “**Save**” and acknowledge the message about permissions.

By default, API Gateway will accept any format of POST, response, and Query string formats, but we may want to enforce a certain structure for the way our API communicates.  Let's go ahead and set up a model and validation that lets us restrict the types of data our API will accept.

From the menu on the left, select “**Models**”.

![Image](media/Picture50.png)

Click “**Create**”.

![Image](media/Picture51.png)

For “**Model name**” enter “**CopyAssetsInput**”.

For “**Content type**” enter “**application/json**”.

“Model schema” accepts anything written in the “JSON-schema” templating language.  For our example, the attributes we need for our API to function are a “Copy From” bucket ID, a “Copy To” bucket ID, a list of prefixes (folders) for assets we want to copy, and an option Storage Gateway “Share ID” to refresh once the copy is complete.  Copy the following schema to “Model schema” in order to enforce validation for those data structures, and then click “**Create model**”.


```
{
  "$schema": "http://json-schema.org/draft-07/schema#",
  "title" : "Copy Schema",
  "description": "A reference for object copies",
  "type": "object",
  "properties": {
    "copyfrom": {
      "type": "string"
    },
    "copyto": {
      "type": "string"
    },
    "shareid" : {
        "type": "string"
    },
    "prefixes": {
      "type": "array",
      "items": {
        "type": "string"
      },
      "minItems": 1,
      "uniqueItems": true
    }
  },
  "required": ["copyfrom", "copyto", "prefixes" ]
}
```

Select “**Resources**” and highlight “**POST**” under the “**copy-assets**” resource you created.  In the subsequent panel click “**Method Request**” in the box on the upper-left.

![Image](media/Picture52.png)

Click the pencil next to “**Request Validator**” and select “**Validate body, query string parameters, and headers**”.  Click the check-mark to save the choice.

Click the pencil next to “**API Key Required**” and select “**true**”.  Click the check-mark to save the choice.

Expand the “**Request Body**” section, click “**Add model**”, type “**application/json**” under “**Content type**”, and select “**CopyAssetsInput**” from the drop-down under “**Model name**”.  Click the check-mark to save the choice.

![Image](media/Picture53.png)

Just the right of the “**Actions**” button, click the “**← Method Execution**” link.

![Image](media/Picture54.png)

Click the “**Test**” button in the “**Client**” box.

![Image](media/Picture55.png)

In order to test our API and Lambda function, we'll be pulling some baseline assets from a public bucket and placing them in your “Master” bucket.  Copy the following JSON into the “Request Body” box and change the “copyto” value to the name of your master S3 bucket.  In our example nomenclature it would by “*yourinitials-mymedia-master*.”  Once it's correct, click “**Test**”.


```
{
  "copyfrom": "mediademo-public",
  "copyto": "<something>-master",
  "prefixes": ["Good Omens/"]
}
```

![Image](media/Picture56.png)

Once complete, a response body should appear on the right side that says “*Success*”.  To validate, let's navigate back to our S3 buckets, open our “master” bucket, and verify that “Good Omens” is there.

![Image](media/Picture57.png)![Image](media/Picture58.png)

Our test worked, so now we need to deploy our API.  We could theoretically deploy it into a stage as-is, but we should secure it too.

Click the “**Actions**” button and choose “**Deploy API**”.

![Image](media/Picture59.png)

From the “**Deployment stage**” drop-down select “**[New Stage]**”.  Name the stage “prod” and optionally fill out the descriptions.  Click “**Deploy**”.

![Image](media/Picture60.png)

On the subsequent “**prod Stage Editor**” screen, make note of the “**Invoke URL**” link at the very top (yours will differ from the one in the screenshot below).  We'll be making use of that shortly.

![Image](media/Picture61.png)

On the left side, navigate to “**Usage Plans**” and click “**Create**”.

![Image](media/Picture62.png)

Name the Usage Plan “**Unlimited**” and uncheck “**Enable throttling**” and “**Enable quota**”.  Click “**Next**”.

Click “**Add API Stage**”, select “**Media API**” from the “**API**” drop-down, choose “**prod**” from the “**Stage**” drop-down, and be sure to click the check-mark to save your choices.  Then click “**Next**”.

![Image](media/Picture63.png)

Click “**Create API Key and add to Usage Plan**”.

![Image](media/Picture64.png)

Name it “**Prod-Unlimited**” and make sure “**Auto Generate**” remains selected.  Click “**Save**”.

![Image](media/Picture65.png)

Click “**Done**”.

Click the “**API Keys**” tab and click the link to the “**Prod-Unlimited**” key.

![Image](media/Picture66.png)

Click the “**Show**” link next to “**API Key**” and copy the key to a handy location.  We'll be using it in conjunction with the “**Invoke URL**” we made note of earlier.

![Image](media/Picture67.png)

Last but certainly not least, let's deploy a web UI that can interface with our API.

Select “**Services**” in the upper-left and either navigate to “**Compute** **– ECS**” or type “ecs” into the search bar and select “**ECS**”.

![Image](media/Picture68.png)

Click “**Get started**”.

Click the “**Configure**” button in the “**custom**” box.

![Image](media/Picture69.png)

Name the container “**sgw-workshop-ui**”.

Next to “**Image**”, paste “**queueburt/sgw-workshop-ui:latest**”.

Type “**80**” into the “**Port mappings**” box.

![Image](media/Picture70.png)

Expand “**Advanced container configuration**”.

Under the “**Environment**” heading you'll find a section for “**Environment Variables**”.  Enter the following into the boxes replacing the URL and the Key with the items specific to you from the previous steps.  Use the screenshot below as a reference.

* “**SGW_DEMO_API_URL**”: Value: “**<YOUR_INVOKE_URL>/copy-assets**”
    * **(WARNING: The trailing text in the URL MUST match that of the resource in API Gateway.  If you’ve changed the name from ‘copy-assets’, ensure it’s set correctly here.  Mismatched resources generate errors in the UI that are notoriously difficult to trace.)**
* “**SGW_DEMO_API_KEY**”: Value: “**<YOUR_API_KEY>**”


![Image](media/Picture71.png)

Click “**Update**”.

Click “**Next**”.

Next to “**Load balancer type**”, select “**Application Load Balancer**” and click “Next”.

![Image](media/Picture72.png)

Next to “**Cluster name**”, type “**sgw-workshop**” and click “**Next**”.

![Image](media/Picture73.png)

At the bottom of the confirmation screen, click “**Create**”.

Once all of the tasks complete (it might take ~5 minutes or so), click “**View service**”.

![Image](media/Picture74.png)

Under “**Load Balancing**”, click the link underneath “**Target Group Name**”.

![Image](media/Picture75.png)

In the “**Basic Configuration**” panel, click the link to the right of “**Load balancer**”.

![Image](media/Picture76.png)

Copy the URL next to “**DNS name**”, open a new browser tab, and navigate to that address.  Note that despite the tasks being complete, it may still take a minute or so for the web interface to become visible.  If you see a 503 error, just refresh intermittently.

![Image](media/Picture77.png)

Let’s try our new UI in action!  Let’s do a test copy of a prefix from our master bucket to our studio.  Fill out the form with the following info and press “Copy Assets”.

* **“Copy From Bucket”: **<your-bucket>-master (“*yourinitials-mymedia-master*” in our example)
* **“Copy To Bucket”:** <your-bucket>-studio (“*yourinitials-mymedia-studio*“ in our example)
* **“Share ID”: **<your-sgw-share-ID>
* **“Enter S3 Prefix(s)”: **“Good Omens/S01E01/sq035/s13/“


![Image](media/Picture78.png)

In your Windows workstation you should now be able to enter the shared drive (or refresh from the root folder) and see your assets ready to work with!

Want to copy another?  Try running the same UI entries with “*Good Omens/S01E01/sq035/s17/*” as the prefix!

At this point feel free to experiment!  There are a wide array of free software utilities available for video editing that can showcase the functionality of our app.  The one we used in the earlier demo was Shotcut, which can be downloaded at [https://shotcut.org](https://shotcut.org/).  Editing the video and reversing the “Copy From” and “Copy To” buckets will create a similar versioning experience to the one you saw.  While this is a fairly simple example, addition of a persistent database to store studio to SGW share mappings can create a much more stateful experience for operators.
